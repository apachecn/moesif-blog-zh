# API 代理 vs API 网关:有什么区别，应该用哪个？

> 原文：<https://www.moesif.com/blog/technical/api-gateways/API-Proxy-Vs-API-Gateway-What-Are-The-Differences-And-Which-Should-You-Use/>

在本文中，我们将从较高的层面来看 API 代理和 API 网关之间的区别。当开发人员发布一个公共 API 时，该 API 有必要拥有安全策略和对 API 消费者隐藏后端逻辑的方法。

将 API 从后端服务中分离出来，可以让你的应用免受后端代码更改的影响，并允许用户调用你的 API 而不用担心可用性。如果对某个端点进行了更改，或者发布了新版本，用户可以继续工作而不会中断。此外，API 代理或 API 网关可以帮助您轻松、统一地保护 API 端点。这可以增加另一层防御，防止攻击者渗透到您的系统中。

尽管 API 代理和网关有一些相似之处，但它们的不同之处正是它们的不同之处。让我们来看看这两种解决方案之间的区别，并确定哪一种最适合您的用例。

## API 代理

API 代理是位于前端和后端服务之间的接口。这种抽象有助于进一步将前端与后端的实现细节分离开来。代理将公开一个 URL，您的前端将使用它来访问 API。当请求到达代理时，代理将这个请求路由到配置好的 API 端点，响应返回给代理，然后代理将响应返回给调用者。

API 代理的工作方式类似于 API 网关，它可以处理数据转换、安全性和路由。当您定义代理时，您可以向代理下的不同 API 添加传输安全性、监控(SLA、性能)、配额和访问级别。

许多企业拥有由多个应用程序公开的现有服务，并且这些应用程序可能是全球分布的，这使得 API 管理和可靠性变得困难。对于构建一个基本 API 并希望对其应用一些基本安全策略的人来说，API 代理可能是一个很好的解决方案。然而，在规模上，为了满足真正的企业 API 需求，您将需要一个 API 网关。

## API 网关

API 网关可以为开发人员提供比 API 代理更多的定制，比如为您的有效负载增加端到端的安全性。API 网关处理认证、授权、拒绝服务攻击、SQL 注入检查、负载平衡、缓存、请求整形、编排、中介和传输安全。一些 API 网关甚至允许用户从现有服务创建全新的端点，或者创建仅在网关上运行的虚拟端点，而没有任何后端。如您所见，API 网关提供了 API 代理的所有特性，甚至更多。

作为开发人员，API 网关可以节省您的开发时间，因为您可以专注于构建应用程序逻辑。这为开发人员提供了一个优势，使他们不必担心通过 API 网关直接在 API 代码中构建特性，如安全性和缓存。能够利用 API gateway 的核心基础设施允许您快速创新和快速原型化。

API 网关提供了比 API 代理丰富得多的功能。您可以使用 API 网关通过将多个现有服务组合在一起来构建 API，这是 API 代理无法做到的。一个设计良好的网关将根据其使用方式自动优化其配置；它应该是轻量级的，并提供卓越的性能。要了解更多关于可靠的 API 网关的信息，以及 API 网关的好处，[查看我们的文章](https://www.moesif.com/blog/technical/api-gateways/How-to-Choose-The-Right-API-Gateway-For-Your-Platform-Comparison-Of-Kong-Tyk-Apigee-And-Alternatives/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences)。

## API 监控

无论您决定使用 API 网关还是 API 代理，API 监控都是一个重要的工具，开发人员需要使用它来确保他们的 API 的可靠性。API 监控允许开发人员查看实时数据，包括错误、异常和其他关键事件。此外，开发人员可以配置 API 事务性数据的日志记录，并分析 API 使用情况以获得见解和趋势。此外，API 监控可以自动生成报告，并帮助您的企业为产品经理和销售人员创建整体的 [KPI 和指标数据](https://www.moesif.com/blog/technical/api-metrics/API-Metrics-That-Every-Platform-Team-Should-be-Tracking/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences)。

使用 [Moesif](https://www.moesif.com/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences) ，您可以监控端到端的用户体验，包括您的所有 API 和前端事件。您可以自动获得关于 API 性能问题的实时警报，或者根据用户的行为自动发送电子邮件。Moesif 已经作为一个插件与开发人员可用的大多数 API 网关集成在一起，这就是为什么您应该考虑将 Moesif 用于定制 API 监控解决方案。如果你正在使用 Kong，Moesif 已经是一个顶级的社区插件，你可以通过[访问此链接](https://docs.konghq.com/hub/moesif/kong-plugin-moesif/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences)了解更多信息并与之集成。你也可以通过使用[我们的 SDK](https://www.moesif.com/docs/server-integration/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences)之一将 Moesif 直接插入你的 API 代码。

## 应该选择 API 代理还是 API 网关？

一个设计良好的 API 网关将充当一个代理，允许您打开或关闭某些与应用程序需求无关的功能。每个企业都是不同的，因此需要一个可定制的 API 网关来定制您的 API 体验，以适应您想要的用例。一旦你选择了你的 API 平台，你将希望确保你的 API 按照预期的方式运行。实施一个轻量级的分析解决方案，如 [Moesif](https://www.moesif.com/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences) 通过为您提供可见性和报告来补充您的 API 网关。我们建议开发人员使用 API 网关，这样他们可以从解耦、减少往返、安全性和横切关注点中受益。此外，从 API 网关开始您的开发之旅将减少技术债务，因为当他们的应用程序或客户群扩展时，您的开发人员将不再需要实现 API 网关。

## 提升分析和监控水平

无论您选择使用 API 代理还是 API 网关，都必须将您的解决方案与分析解决方案相结合。Moesif 为您的 API 提供端到端的可见性，以帮助您构建令人惊叹的 API 产品。在 Moesif 中可以很容易地发现对使用、趋势、错误和采用的深刻见解。使用 API 网关或代理向您的用户交付可靠和安全的 API，同时使用 Moesif 确保您交付无与伦比的价值。由于我们支持许多最流行的框架和 API 管理平台，与 Moesif 的集成变得简单而容易。[立即注册](https://www.moesif.com/signup/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-gateway-differences)moe SIF，升级您的 API，并使用分析和监控来增强您的网关或代理解决方案。