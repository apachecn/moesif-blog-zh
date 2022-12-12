# 开发 API 的最佳免费 REST API 调试工具

> 原文：<https://www.moesif.com/blog/technical/api-tools/The-Best-Free-REST-API-Debugging-Tools-For-Developing-APIs/>

当我们开发 Moesif API 分析平台时，我们需要创建和使用许多 API。其中一些是支持我们各种服务的内部 API，而另一些是外部 API，如我们的支付提供商或认证服务。来自内部和第三方的信息。当然，我们在自己的 API 上使用自己的 API 分析来“吃你自己的狗粮”，但我们也使用许多其他工具，其中许多是免费的。我们认为我们应该分享一些我们真正喜欢并在开发和使用 API 时经常使用的最佳方法。虽然这篇文章关注的是开发，但请期待后续的文章，讨论在生产中交付可靠 API 的最佳工具。

这篇文章按照用例组织了各种工具。

## 发送 API 请求

大多数 web 开发人员都需要在某个时候发送 API 请求。无论你是一个移动应用开发者，测试后端请求还是开发和运行你自己的服务。当然，您可以通过将 URL 直接放在浏览器中来发出 GET 请求，但是浏览器可能会通过本地缓存进行阻止，所以您真的不知道是什么在攻击您的服务器。此外，浏览器有严格的安全策略，防止改变诸如原始标题或 URL 大小限制之类的东西。

### 邮递员

最受欢迎的 HTTP 客户端之一是 [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en) 。它有一个非常漂亮的 GUI 界面，非常容易使用，不管你是刚开始使用 RESTful APIs 还是一个专家。存储了过去通话的历史记录，以便您可以快速重新发出它。Postman 甚至包括一些不错的功能，比如自动完成标准的 HTTP 头，支持和呈现各种有效载荷，从 JSON 到 HTML，甚至是 multipart。

### 卷曲

如果你喜欢基于命令行的工具， [cURL](https://en.wikipedia.org/wiki/CURL) 非常快。cURL 包含在大多数*nix 发行版中，这使得它成为任何人登录远程主机的便利工具，而不必担心安装定制工具。鉴于 cURL 的广泛安装基础，您可以依赖 cURL 构建测试和部署脚本，而不必担心被局限于单一供应商。

## 在浏览器中捕获 AJAX 调用

鉴于 web 应用程序中的 API 是前端和后端之间的接口和契约，当出现问题时，知道哪一方违反了该契约是有用的。如果您的 web 应用程序不能正确地从后端加载数据，您可能要做的第一件事就是检查 API 请求和响应是否有任何异常。

### Chrome 开发者工具

大多数桌面浏览器都包含某种类型的*开发者工具*，比如 [Safari 的网络开发工具](https://developer.apple.com/safari/tools/)、 [Chrome 的 DevTools](https://developer.chrome.com/devtools) 和 [Firefox 的开发者工具](https://developer.mozilla.org/en-US/docs/Tools)。这些工具默认包含在浏览器中，使您能够快速检查 API 调用。

## 从非浏览器应用程序捕获 HTTP 请求

从非 web 应用程序中捕获 API 请求可能具有挑战性，因为您没有丰富的浏览器开发工具来检查 API 调用(不包括额外的编排)。许多解决方案需要使用反向或正向代理来拦截和记录 HTTP 流量。

### Moesif API 分析

Moesif 是我们自己的 API 日志分析和分析服务，也有免费计划。Moesif 使您能够实时跟踪来自您自己或第三方 API 的实时 API 流量，并让您深入了解 API 上发生的事情。Moesif 与 Postman 等其他工具连接，以重放任何 API 调用。除了 API 日志分析和调试之外，Moesif 还提供产品洞察，告诉您最佳客户与流失客户使用 API 的不同之处。

### 查尔斯代理

Charles Proxy 是一个非常受欢迎的免费工具，但它并不托管在云中。你必须安装这个软件，并在你自己的机器上安装和运行它。Charles Proxy 的工作原理是将所有本地机器的流量作为 HTTP 代理路由通过它。您还可以在电脑上打开一个端口，并将您的 iPhone 或 Android 设备配置为通过 Charles 发送流量。只要记得在你的电脑上停止查尔斯时，在你的智能手机的设置中禁用代理。否则，您的手机将无法连接到互联网。因为所有 web 流量都将通过 Charles 路由，所以您可以利用过滤器只记录对特定域或主机的请求。[查看配置 Charles](https://www.charlesproxy.com/documentation/configuration/browser-and-system-configuration/) 了解更多信息。SSL 证书会导致额外的复杂性，因为代理需要成为中间人并破坏 SSL 隧道。您可以在您的设备上安装 Charles SSL 证书。

### SandroProxy

[SandroProxy](https://play.google.com/store/apps/details?id=org.sandroproxy) 就像安卓的查尔斯。如果您正在进行调试，并且没有可用的桌面和局域网，这可能会很有用。你也可以把它连接到 Chrome 开发工具上。

### 游手好闲的人

Fiddler 类似于 Charles，它在本地机器上设置代理。你必须下载并安装软件。

## 重播捕获的 API 请求。

偿还捕获的请求使您能够重现之前创建的相同流量。这对于在不同的主机上重放(比如在您的本地开发机器上重放生产流量模式)或者验证修复是否已被纠正是很有用的。

### 邮递员拦截器

邮递员拦截器提供了一种捕获 API 调用并重放它们的方法。但是，Postman 拦截器只能捕获 HTTP 请求数据。它无法捕获 HTTP 响应。主要焦点是重放请求，而不是记录调试。

## 共享 API 请求数据

如果您正在与一个团队一起工作，您通常希望与团队的其他成员共享捕获的 HTTP 跟踪，或者使他们能够重放一个场景，而不必手动模拟 API 请求。

虽然你可以打开 Chrome Developer 的工具，复制并粘贴其中的数据，与你的团队分享，但这样做很麻烦。

### ApiRequest.IO

一个由我们构建的免费工具，我们无耻地推广它。Api 捕获 Chrome 扩展从任意网站捕获所有 AJAX 调用，然后存储在云服务 [apirequest.io](https://apirequest.io) 中。该服务创建了一个共享工作空间，可以通过一个模糊的 URL 与其他人共享 30 天。有点像分享谷歌文档链接。

> *什么是 Moesif？ [Moesif](https://www.moesif.com/) 是最先进的 API 分析平台，被成千上万的平台用来了解你最忠实的客户在用你的 API 做什么，他们如何访问它们，以及从哪里访问。Moesif 为您喜爱的语言提供了开源中间件和 SDK。*

## 模拟服务器

如果您正在编写针对 API 进行测试的客户端代码，通常需要模拟 API 服务器。然而，有时 API 服务甚至还没有开发出来，或者启动和运行起来很复杂。在这些情况下，您可以使用模拟服务器，它在核心将一些预定义的 JSON 转发回请求者。

### JSON 服务器

JSON 服务器是一个开源的 moch 服务器，你可以在你的机器上克隆和运行它。

### JSON 占位符

如果您不想在本地运行自己的模拟服务器，可以使用托管服务，比如 [JSON 占位符](https://JSONplaceholder.typicode.com)。

## 模仿不同的 HTTP 响应

JSON 服务器和 JSON 占位符 API 调用总是用 *200 OK* 响应。如果您想测试您的客户机如何处理不同的 HTTP 错误响应，您可以设置一个更复杂的模拟服务器来提供这种灵活性。

### httpbin

Runscope 的 httpbin 在他们的网站上提供托管服务，或者你可以克隆他们的 T2 开源回购协议并运行你自己的服务。httpbin 为从 gzip 编码的响应到像照片这样的二进制数据提供了模拟端点。

### Mocky.io

和 httpbin 一样， [mocky.io](https://www.mocky.io/) 是[开源](https://github.com/julien-lafont/Mocky)，也有托管版本。Mocky 比以前的工具更加灵活，因为它允许任意的响应体和状态代码。如果您需要将某些特定的 JSON 返回给您的客户机，这一点很重要。它还支持各种各样的内容类型，如文本/xml、多部分和 HTML。此外，您可以设置预定义的延迟，如`?mocky-delay=100ms`。这是伟大的测试超时条件或当添加加载指标到一个网站。

## 开挖隧道

如果您正在开发一个 webhook 来处理来自第三方 API 或服务的回调，那么如果您想要本地调试，您将需要启用这些云服务来调用您在本地机器上运行的 API，除非您想要通过打开您机器上的端口并获得静态 IP 来解决费用和潜在的安全问题。这些隧道服务将请求转发到您的本地主机，而不需要您打开端口。

### ngrok

ngrok 是最受欢迎的一款。主机名是临时的，随机生成的，但是如果你想要一个永久的主机名，他们会提供付费服务。对于大多数用例，免费版本已经足够了。要使用 ngrok，您需要做的就是用 ngrok 提供的 URL 替换 webhook URL，然后运行本地代理。

## JSON 工具

### 代码美化

[代码美化了](https://codebeautify.org/jsonviewer)的 JSON 查看器，可以漂亮地打印，将 JSON 转换成 XML 和 CSV，等等。

### JSON Lint

JSON Lint 是一个 JSON 验证器和重新格式化器。

### JSON 生成器

JSON 生成器创建随机生成的 JSON，该 JSON 是有效的或者基于您可以上传的模式。这有助于生成大量的测试数据。

### JWT.io

如果您正在使用一个使用 JSON Web 令牌进行身份验证的 API，您将需要快速解码 base64 并查看令牌内容。JWT.io 是 Auth0 提供的一个免费服务，它就是这样做的。

要了解更多关于 JWT 如何工作以及 JWT 如何与不透明令牌相比较的信息，请查看我们关于 RESTful APIs 认证和授权的指南。

## 性能试验

有相当少的付费性能测试服务，这是有意义的，因为设置性能测试需要做更多的工作。因此，免费服务往往涉及更多的工作，尤其是对于托管服务，它可能是资源密集型的。

### 阿帕奇 Jmeter

Apache Jmeter 是一个开源工具，用于对你的 API 进行负载测试。

### Loader.io

[Loader.io](https://loader.io/) 是一个对 API 进行负载测试的工具，可以快速产生大量流量。

## 克-奥二氏分级量表

许多 API，尤其是支持单页面应用的私有 API，利用[跨源资源共享](/blog/technical/restful-apis/Authorization-on-RESTful-APIs/)来安全地对不同的域进行 AJAX 调用。

### 莫夫·CORS

[CORS 的 Chrome 扩展](https://chrome.google.com/webstore/detail/moesif-origin-cors-change/digfbfaphojjndkpccljibejjbppifbc?hl=en)。我们自己的扩展由 Moesif 创建，每周有超过 17，000 名用户使用。它可用于规避或更改本地测试和调试的 CORS 限制。

## 我们读者的其他建议

我们欢迎读者的建议，包括您使用的工具或您构建的工具。如果我们错过了你最喜欢的工具，它可以帮助你更好地使用 API，那么请在下面留下你的建议或者给我们发消息。

以下是 ExtendsClass 团队的建议:

*   [模拟服务器](https://extendsclass.com/mock-rest-api.html)
*   [休息客户端](https://extendsclass.com/rest-client-online.html)

## 摘要

一旦 API 投入生产使用，寻找有用工具的后续文章。Moesif 有一个 API 分析平台，一旦你的 API 上线，这个平台就非常有用。请随意试一试。