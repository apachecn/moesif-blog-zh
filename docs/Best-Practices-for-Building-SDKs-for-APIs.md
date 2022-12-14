# 为 API 构建 SDK 的最佳实践

> 原文：<https://www.moesif.com/blog/technical/sdks/Best-Practices-for-Building-SDKs-for-APIs/>

2019 年，没有 API 程序，你是做不了 B2B 公司的。无论你是 API *还是 API*，产品或 API 都被用来为你的 web 应用提供额外的集成和功能。

尽管 SDK 在代码行方面看起来很简单，但 SDK 需要可靠并易于扩展。设计不佳的 SDK 可能会削弱客户的基础设施，降低客户对您服务的信任度。在 Moesif，我们投入了大量精力来创建 SDK，这些 SDK 既具有高性能，又添加了故障保险，以防发生不好的事情。本文将介绍其中的一些实践。鉴于 Moesif 是一个 API 分析服务，其中一些实践是特定于大容量数据收集的。但是，不管您的 SDK 目的是什么，其他功能都是适用的。

## 安全异步地初始化 SDK

对于大多数带有 SDK 的 SaaS 解决方案，您的用户需要初始化 SDK 并传入 API 密钥。您可能需要从您的服务器获取某些帐户信息或获取某些设备信息，如操作系统版本。这项工作应该在后台线程上执行，或者利用异步调用。通过利用后台线程，主线程可以继续响应而无需处理。根据您的 SDK 是用于前端还是后端，主线程可能是主 UI 线程，也可能是响应传入 HTTP 请求的线程。

除了卸载初始化工作之外，您的 API 密钥不应该假设 SDK 运行在一个安全可信的环境中。许多 SDK 包含在移动应用程序包中或通过浏览器 javascript 实现，所有这些都可以很容易地被分解，从而向任何可以下载或访问该应用程序的人透露 API 密钥。降低这种风险的一种方法是将 API 键的范围缩小到特定的 API 资源和动作。例如，如果您正在构建一个 analytics SDK 来收集和记录信息，那么 API 键可以是只写的，并且仅用于与您的数据接收 API 进行通信，但是对数据导出 API 的访问被阻止。

## 批量繁重的网络流量

对于与另一个服务器创建的任何 HTTP 连接，在传输任何数据之前都会有一些开销。这包括客户机和服务器之间创建 TCP 连接的 SYN 和 ACK 数据包的相互交换、建立 SSL/TLS 的握手等。您可以通过 [Keep-Alive](https://blog.insightdatascience.com/learning-about-the-http-connection-keep-alive-header-7ebe0efa209d) 重用连接，而不是为每个 HTTP 请求重复这些操作。

除了保持活动状态，您还可以将多个事件或命令批处理到同一个出站 HTTP 请求中。回到我们的 analytics SDK 示例，为每个需要记录的客户端事件发出 HTTP 请求是低效的。TCP 本身有开销，而且你还为每个请求发送许多 HTTP 头来处理认证、缓存等。批处理减少了这种开销，减少了系统调用的次数，同时将所有数据保存在同一个内存缓冲区中。

批处理是通过发送一组事件或命令来完成的，而不是每个 HTTP 请求发送一个事件或命令。它通常在与本地排队结合使用时效果最好。

## 利用本地排队

为了正确地实现上面的一些特性，比如批处理，您可以利用本地队列。本地队列将捕获或处理数据的逻辑与批处理数据并将其发送到服务器所需的逻辑分离开来。如果有断电的风险，事件可以存储在内存中的内存队列中，或者刷新到磁盘以保持持久性。更精细的排队架构可以利用分布式数据存储，如 Redis，尽管这增加了设置 SDK 的复杂性，所以除非绝对需要，否则不推荐使用。

当 API 关闭时，排队还可以增加 SDK 的可靠性。当 API 关闭时，本地事件可以继续被推入队列。一旦启动，就可以大批量清空队列。如果 API 长时间关闭，建议实现防止队列消耗过多内存或磁盘空间的逻辑。做到这一点的一种方法是通过实现一个固定大小的队列，该队列在新值出现时自动丢弃旧值。虽然一些事件可能会被丢弃(如果事件仅用于分析目的，这可能是可以的)，但这可能是一个很好的权衡，可以保证您的 SDK 不会崩溃或使您客户的基础架构过载。

### 刷新队列

正确的排队将需要某些触发器来将事件或命令刷新到您的服务器。推荐方法是同时利用基于时间和基于计数的触发器。例如，从上次刷新起 10 秒后，一旦缓冲区达到 50 个事件 OP，您就可以刷新事件。这确保了您的 SDK 可以在高峰流量期间批处理许多事件，同时仍然保持较低的端到端延迟。

如果没有基于时间的刷新，如果没有其他事件被推入队列，单个事件可能会无限期地停留在队列中。

## 压缩

压缩非常容易利用，但也很容易被遗忘。默认情况下，并非所有 HTTP 客户端库都压缩有效负载，您的后端需要支持您首选的压缩编码。通过使用 zlib 或类似工具将有效负载压缩为 gzip，您可以将有效负载的大小从纯文本减少 10 倍以上。你也可以研究像 Brotli 这样的新格式，它可以比 gzip 进一步减少 10%到 20%的开销。

## 设置`User-Agent`

您应该在`User-Agent` HTTP 头中包含 SDK 名称和版本。这使您能够了解 SDK 的采用情况，并将问题与特定版本关联起来。例如，我们 Moesif 在我们的 SDK 中采用了标准格式`libraryname/semvar`,因此:

```py
 const request = require('request');
  const  pjson = require('../package.json')

  request({
  headers: {
    'User-Agent': 'nodejs/' + pjson.version
  },
  uri: 'https://api.example.com',
  method: 'POST'
}, function (err, res, body) {
  //it works!
}); 
```

## 利用 API 分析

一旦您构建并发布了这些 SDK，关键是您要有正确的 [API 分析](https://www.moesif.com/features/api-analytics)来衡量您的 API 的性能和利用率，并查看您可以在 SDK 或 API 方面做出哪些改进，例如调整批量大小或添加更高效的端点。正确实现的分析可以让您深入了解在分页或查找不正确使用的端点方面可以做出哪些改进。

## 文档更改

利用 GitHub 的发布流程或创建一个 CHANGELOG.md 来彻底记录变更，即使是很小的变更。当 SDK 的用户遇到错误或问题时，他或她可以检查的第一件事是 changelog 和任何类似问题的标签。有时小的变化会在你不知道的情况下在旧的或特定的环境中发生。

重大变更应该有更全面的文档来描述什么是重大变更以及如何迁移。您可能有一些 SDK 用户试图从版本 1 迁移。X.X 到 3。而其他人可能会从 2。X.X 到 3.X.X。记录完成迁移所需的内容会非常有帮助。