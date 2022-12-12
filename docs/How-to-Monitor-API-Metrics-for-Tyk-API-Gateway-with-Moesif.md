# 如何使用 Moesif 监控 Tyk API 网关的 API 指标

> 原文：<https://www.moesif.com/blog/technical/tyk/How-to-Monitor-API-Metrics-for-Tyk-API-Gateway-with-Moesif/>

在过去的几年里，API 网关变得越来越流行。通过将标准 API 相关功能捆绑到单个软件组件中，开发资源得以释放，系统架构得以简化。

Tyk 就是这样一个 API 网关。它是[开源的](https://github.com/TykTechnologies/tyk)，用 Go 编程语言编写。它具有一系列特性，比如身份验证、访问控制、配额和速率限制、API 版本控制等等。

Tyk 遵循一种轻量级、模块化的 API 网关软件方法，专注于访问控制。这使得为 Tyk API 网关构建 Moesif 插件变得很容易。

在本文中，我们将讨论 Moesif 给你的见解，以及如何将它集成到你的 Tyk 设置中。

## 使用 Tyk 设置 Moesif API 分析

Tyk 配备了一个[分析泵](https://tyk.io/docs/getting-started/tyk-components/pump/)，用于 *Moesif API 分析*。你所要做的就是在`pump.conf`中配置正确的分析泵，重新加载 Tyk，然后你就可以开始了。

Moesif 建议对所有 Tyk 网关实例和数据中心区域使用一个应用 ID。然而，*生产*、*试运行*和*开发*环境应该得到不同的 id。

`pump.conf`文件应该是这样的:

```py
"pumps":  {  "moesif":  {  "name":  "moesif",  "meta":  {  "application_id":  "Your Moesif Application Id"  }  },  } 
```

Tyk 的 Moesif 分析泵是开源的。在 [GitHub](https://github.com/TykTechnologies/tyk-pump/blob/master/pumps/moesif.go) 上回顾一下。

## Moesif API 分析如何增强 Tyk？

Tyk analytics pump 也提供了 Moesif 的大多数 SDK 特性。

### 服务和路由检测

Moesif 可以通过 REST 模板按路由过滤，这样 id 就可以作为通配符使用。

例如，`GET /media/videos/:id`会将所有`GET`请求匹配到一个视频，而与它们的 ID 无关。

### 地理分段

除了时间指标，Tyk 分析泵还将捕获 IP 数据，实现过滤和地理热图。下图显示了延迟超过 1 秒的位置。

![Geo heatmap](img/09c9dc8d1439e9f9e59237ab3569db9a.png)

### 用户会话

通过为 Tyk 配置身份验证，Moesif 分析泵将自动捕获相关凭据。根据会话信息，Moesif 能够生成 API 会话跟踪和用户配置文件，其中包含附加的详细信息，例如用户第一次看到的时间、用户最后一次看到的时间等。

> 有了 Moesif 浏览器软件开发工具包，甚至可以记录用户从第一次点击登陆页面到最后一次请求 API 的过程

您可能会发现添加用户人口统计信息(如姓名、电子邮件、电话等)非常有用。只要您拥有用户的 API 密钥/会话令牌，就可以使用任何 Moesif API 库来添加/更新用户配置文件元数据。

![User Dashboard](img/c145f43ec6192e420990fcffad4cae9a.png)

使用人口统计信息，Moesif 可以执行更高级的用户群组分析，例如允许您跟踪 30 天的活动保留。

> 保持率衡量的是一个群体中再次使用并保持活跃的用户的百分比。

![Cohort Analysis](img/dd9a656274d3461c78ec70a18cbf994c.png)

### 传入和传出 API 调用

Moesif analytics pump 可用于跟踪您的 API 调用，以及向您的合作伙伴发出的 API 请求。这可用于了解和监控您合作伙伴的基础架构的运行情况，并让您尽早发现任何问题。只要有流量通过你的 Tyk 网关，Moesif 就能分析它。

![API Calls](img/7f0a7c2f06b22a4065b91df7bcdc52c9.png)

## 摘要

*moes if API Analytics*plus Tyk 是一个强大的软件包，有助于在整个 API 生命周期中最大限度地减少开发时间并降低维护成本。

Tyk 允许将所有常见的 API 特性捆绑到一个系统中，称为*Tyk*T2】API 网关。

随着 Moesif 的加入，可以分析以用户为中心的数据。通过结合 Moesif 的[分析泵](https://www.moesif.com/implementation/log-http-calls-from-tyk-api-gateway?useCase=incoming)和[浏览器 SDK](https://www.moesif.com/implementation?useCase=outgoing) ，所有用户交互都可以被保存(从第一次登录页面访问到最后一次发送请求)，记录并供进一步分析使用。