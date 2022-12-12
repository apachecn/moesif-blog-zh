# 来自 Moesif 在 GraphQL 和 Serverless 上的开发者会议的视频

> 原文：<https://www.moesif.com/blog/meetups/graphql/Moesif-Developer-Meetup-on-GraphQL-and-Serverless/>

Moesif 在 GraphQL、Serverless 和 API 上举办了第一次会议。我们与 Box 开发者平台团队的创始人以及 Back4App 和 StdLibs 的首席执行官进行了富有洞察力的谈话。

有大量的免费啤酒和比萨饼，这是与更大的 API 和开发人员社区的成员交流的一段美好时光。以下是会谈中的一些亮点。

## Jeremy Glassenberg 谈开发者平台的最佳实践

首先发言的是 Jeremy Glassenberg，他是 Box 开发平台团队的创始人，现在是 [API 策略师。](https://www.apistrategist.com/)开发者平台是那些可以半途而废，但很难正确吸引开发者的东西之一。Jeremy 讨论了避免激怒开发人员的不同方法，比如 API 版本控制和与开发人员社区交流的力量。

主要采取的方式是把你的 API 当作一个产品，而不仅仅是试图发布内部 API。然而，API 不是网站，因此 API 产品管理不同于一般的 API UI 设计。像 A/B 测试和分析这样的常规 UI 实践更难实现，但同样重要。如果你推出一个突破性的改变，你可能会严重影响一小群非常高收入的顾客。与可以不断更新的网站不同，您的开放 API 可能会被数百家不同的公司用于数千种不同的产品和集成。

[https://www.youtube.com/embed/VI-gXpnVCnc](https://www.youtube.com/embed/VI-gXpnVCnc)

要获得更多资源，我们建议查看我们的 [API 开发者体验指南。](/blog/api-guide/api-developer-experience/)

## Back4App 的 CEO 给出了一个使用 GraphQL 像 BaaS 一样增强解析的演示。

[Back4App](https://www.back4app.com) 是一个后端即服务，让开发者无需担心后端就能快速创建应用。

达维·马塞多(Back4App 的首席执行官)一直在 AWS Lambda 上试验 GraphQL，供内部使用，并将其作为 Back4App 产品的一部分。许多为要在 AWS Lambda 上托管的 GraphQL APIs 生成无服务器函数的搭建工具可能会造成某些复杂性，尤其是当您的应用程序很大时。具体来说。达维注意到，支架工具不是在真正的无服务器/FaaS 架构中创建许多小函数，而是生成一个单独的 AWS Lambda 函数来保存整个 GraphQL API 的所有业务逻辑。这可能会导致可伸缩性问题，例如冷启动缓慢，在函数能够响应传入的 GraphQL 请求之前，必须下载许多 javascript 库。

不幸的是，我们没有记录整个会议，下面只是一个简短的片段，但查看[Back4App.com](https://www.back4app.com/)了解更多信息。

[https://www.youtube.com/embed/FKcVtsCXy1o](https://www.youtube.com/embed/FKcVtsCXy1o)

## StdLib 首席执行官演示如何在 30 秒内创建一个 API

Stdlib 帮助开发人员和非开发人员快速创建 API，而无需考虑底层基础设施。

他们的 CEO Keith Horwood 在 StdLib 上给我们演示了如何创建一个快速的 Hello World API。看到您可以如此轻松地启动和运行 API 是非常令人着迷的。如果你所做的只是将 Salesforce 连接到另一个服务，你可能不需要微服务或甚至 AWS Lambda 的全部重量。很高兴看到 Zapier 的易用性和更大的灵活性。下面看。

[https://www.youtube.com/embed/dPYZ3JgY9F4](https://www.youtube.com/embed/dPYZ3JgY9F4)

## 结论

这是 Moesif 主办的第一次开发者聚会，但很高兴将来能主办更多。如果您未能出席，请加入[meetup 群组](https://www.meetup.com/Serverless-GraphQL/)或我们的邮件列表(在左侧栏),以便获得未来活动的通知。如果你真的参加了，我们很乐意通过[给我们发邮件来听听你对我们能做的任何改进。](mailto:team@moesif.com)

Moesif 是最先进的 API 分析平台。成千上万的平台公司利用 Moesif 进行调试、监控和发现见解。

[了解更多](https://www.moesif.com?utm_source=blog)