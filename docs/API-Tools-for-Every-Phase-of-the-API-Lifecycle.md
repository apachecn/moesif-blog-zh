# API 生命周期每个阶段的 API 工具

> 原文：<https://www.moesif.com/blog/technical/api-tools/API-Tools-for-Every-Phase-of-the-API-Lifecycle/>

当你开始构建你的第一个 API 的时候，你很可能会不知所措或者忘记要点。API 工具的生态系统非常庞大，为项目的每个阶段选择合适的工具至关重要。

在本文中，我们将经历一个 API 项目通常会经历的不同阶段。对于每个阶段，我都会列出有帮助的要点和工具。

## 构建阶段

第一阶段是构建阶段。它是由我们在开始实现之前必须做出的重大决策以及随后 API 的实际实现来定义的。我们不得不问我们想要构建什么类型的 API。是 RPC 还是 REST？我们必须实时向用户推送更新吗，或者我们可以用基于拉取的方法来获得更新吗？

根据我们用例的需求，我们必须选择合适的技术。另一方面，我们还必须关注开发人员的技能组合；如果我们选择没人能掌握的技术，我们就造不出好东西。

### 设计

设计工具可以帮助我们规划我们的 API，甚至生成它的公共部分。它们迫使我们遵循最佳实践，因此即使技能较低的人员也可以构建安全且高性能的 API。

常用的 API 设计工具包括:

*   [OpenAPI](https://swagger.io/solutions/api-design/)
*   [养蜂场. io](https://apiary.io/)
*   [IBM API 连接](https://cloud.ibm.com/catalog/services/api-connect/overview/)
*   [云 API 设计者](https://www.talend.com/products/application-integration/cloud-api-designer/)
*   [红绿灯工作室](https://stoplight.io/design/)

另外，在使用这些工具之前，理解 API 设计模式是很重要的。[了解更多](https://www.moesif.com/blog/api-guide/api-design-guidelines/)。

### 实体模型

原型库是工具带的另一部分。它们可以用于测试，并帮助客户实现者在实际的 API 准备好之前开始构建。

常见的 API 模型库有:

*   [JsonStub](http://jsonstub.com/)
*   [诺克](https://github.com/nock/nock)
*   [MockServer](https://www.mock-server.com/)
*   [b 接受器](https://beeceptor.com/)
*   [摩卡](https://mockoon.com/)
*   [莫奇. io](https://designer.mocky.io/)
*   [邮递员模拟 API](https://learning.postman.com/docs/postman/mock-servers/mock-with-api/#create-a-mock-using-the-postman-api)

### 语言/运行时

虽然一些编程语言，如 Go，是为了构建 API 而创建的，而像 Elixir 这样的语言非常适合并行编程，但它们通常是根据员工的可用技能来选择的。

同样重要的是要注意，语言通常规定了框架。但是 API 框架的一些方法，比如 Sinatra-like，在大多数语言中都有对等的用法。

一些常用于构建 API 的语言有:

*   [出发](https://golang.org/)
*   [Python](https://www.python.org/)
*   [C#](https://docs.microsoft.com/en-us/dotnet/csharp/)
*   [Java](https://www.java.com)
*   [JavaScript](https://nodejs.or)
*   [红宝石](https://www.ruby-lang.org/en)
*   [仙丹](https://elixir-lang.org)
*   [生锈](https://www.rust-lang.org)

Moesif 在 [API 指南](https://www.moesif.com/blog/api-guide/)中也有一个按语言分类的 API 资源列表

### 结构

就像编程语言一样，框架是复杂的结构，所以选择一个框架通常还是取决于你的员工的技能。有些更小；有的功能更全。有些语言提供了更多现成的东西，根本不需要太多框架工具。

以下是一些在互联网上支持 API 的框架:

*   [快递](http://expressjs.com)
*   姜戈
*   [Ruby on Rails](https://edgeguides.rubyonrails.org/api_app.htm)
*   [游戏框架](https://www.playframework.co)
*   [春天](https://spring.io)
*   [火箭](https://rocket.rs)
*   [狂欢](http://revel.github.io)
*   [凤凰](https://www.phoenixframework.org)

## 测试和部署阶段

测试和部署阶段是让整个项目生产就绪，保持一致，并托管在您选择的数据中心。同样，一些工具会限制其他决策。例如，亚马逊 API 网关只能从云提供商 AWS 获得。

### 云提供商

云提供商的产品略有不同，他们有不同的优势。Heroku 使用起来很简单。AWS 最早拥有最新的技术，Azure 与微软生态系统整合得非常好，Google Cloud 处于机器学习技术的前沿。

最大的云提供商有:

*   [Heroku](https://www.heroku.com)
*   [亚马逊网络服务](https://aws.amazon.com)
*   [微软 Azure](https://azure.microsoft.com/en-gb)
*   [谷歌云](https://cloud.google.com)
*   [IBM 云](https://www.ibm.com/clou)

### 门

API 网关有助于管理系统的不同部分，并提供一个中心入口点。有些是开源的，有些不是。有些是托管的，有些是自托管的。有各种各样的网关可供选择。

最常用的网关有:

*   [孔](https://konghq.com/kong)
*   【T0 厚，io】t1
*   [顶点](https://cloud.google.com/apigee/api-management)
*   [亚马逊 API 网关](https://aws.amazon.com/api-gateway)
*   [Azure Gateway](https://azure.microsoft.com/services/api-management)
*   [快递网关](https://www.express-gateway.io)

### 测试

API 的自动化测试有助于防止开发速度变慢。在开始时把事情做好是一回事，但是在一个 API 项目的整个生命周期中把事情做好是另一回事。

可以帮助完成这项任务的测试服务和框架有:

*   [运行范围](https://www.runscope.co)
*   [邮递员](https://www.postman.com)
*   [放心](http://rest-assured.io)
*   [肥皂泡](https://www.soapui.org/downloads/soapui)
*   [JMeter](http://jmeter.apache.org/download_jmeter.cg)
*   [空手道](https://intuit.github.io/karate)
*   [柑橘框架](https://citrusframework.org)
*   [金牛座](https://gettaurus.org)

### 监视

当你在一个遥远的数据中心将一个 API 部署到互联网上时，知道它在做什么是至关重要的。您不能让错误被忽视，您也不想过度或不足配置您的系统。有两种不同类型的监控。*综合监控*如 Runscope，它按照设定的时间表 pings 您的 API；真实用户监控，如 Moesif，它监控来自客户的 API 流量。

优秀的托管监控服务包括:

*   [莫西](https://www.moesif.com/features/api-monitoring)
*   [运行范围](https://www.runscope.com)

## 发布和共享阶段

当你准备好你的 API，你想与世界分享，但你也想让世界使用它，看看它是如何做到的。文档和分析对此至关重要。

### 证明文件

正确的文档决定了 API 业务的成败。毕竟，如果没有人知道如何使用你的系统，那么它的超级创新也于事无补。

幸运的是，有许多文档工具可以提供帮助:

*   [OpenAPI](https://swagger.io/solutions/api-documentation)
*   [ReDoc](https://redoc.ly)
*   哲基尔
*   [石板](https://github.com/slatedocs/slate)

### 分析学

与监控一样，分析也是一种跟踪 API 健康指标的方法，但分析更进一步，将监控与业务价值和客户统计数据结合起来。这为基础设施之外的团队(如产品、销售和营销团队)提供了对客户 API 使用情况的可见性，以了解客户是如何采用和使用您的 API 的。

像 [Moesif](https://www.moesif.com/features/api-analytics) 这样的工具也可以跟踪用户行为和网络活动，这使得复杂的产品分析成为可能，比如跟踪从最初注册到第一次 API 调用的漏斗。

API analytics 可以回答商业问题，例如:*通过一个渠道进入的客户与通过其他渠道进入的客户行为是否不同？*或者更具体一点，*我通过 Twitter 获得的客户比我通过谷歌广告获得的客户流失更多吗？*