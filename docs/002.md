# 如何销售你的 API

> 原文：<https://www.moesif.com/blog/api-product-management/api-analytics/How-To-Sell-Your-APIs/>

众所周知，API 绝对无处不在。API 驱动着现代科技企业甚至非科技企业的几乎每一个方面。您可能有一个内部 API，开发人员使用它来支持内部系统和外部 API，后者更公开地公开功能。与任何功能一样，API 也可以通过出售给有需求的用户来增加收入。无论你是销售 REST API、GraphQL API，还是其他 API，学习如何销售你的 API 已经成为一个流行的问题。让我们来看看销售 API 的一些注意事项。

## 你如何构建一个 API 并销售它？

构建一个 API 并打算出售它需要一个重要的东西:价值。在查看 API 产品时，您应该能够确定是否存在对该 API 的外部需求、需要此类服务的企业数量，以及最重要的是，他们是否愿意为此付费。如果你已经确定一个人或企业愿意为你的服务付费，这是一个很好的起点。

一旦构建了一个 API，要销售它，你需要做一些事情。首先要做的是进行身份验证和授权。这是必要的，这样您就可以确保付费用户被授予访问权限，并且没有未经授权的或非付费用户可以访问该 API。

接下来，您需要确保设置了适当的速率限制和配额限制。这确保了用户只根据他们协议中的条款访问 API。它还确保端点不会被流量淹没，从而防止拒绝服务攻击。

在此之后，您将需要找到一种对 API 使用收费的方法。这包括决定定价策略、注册/签约流程以及统计使用和收费的方式。这一部分通常被称为 API 货币化，是实际收入的来源。

## 销售 API 有哪些挑战？

销售 API 的挑战包括正确实现货币化、服务可用性和安全性。这些挑战在上一节中有所介绍，通常可以通过使用合适的 API 管理工具、创建和部署安全的 API 以及使用易于使用的货币化平台来解决。

除了货币化组件之外，销售 API 的挑战与简单地提供 API 供一般消费非常相似。安全和保持严格的授权是一场持久战。通过高可用性设置使服务持续可用也是至关重要的，因为服务宕机可能是灾难性的，尤其是在 API 为使用它的企业带来收入的情况下。可靠性是让用户留在平台上或继续为你的 API 付费的一个重要因素。

要考虑的最后一个挑战是 API 在内部花费了您多少成本。例如，假设您已经创建了一个信用评级 API，它从各个信用机构检索用户的信用评分。您向用户收取每个 API 调用 3 美元的费用，后付费。信用局还对每次 API 调用收取 1 美元，这意味着每次调用的净收入是 2 美元。如果用户使用了你的服务，并决定在月底不支付他们的发票怎么办？即使你的客户拒绝付款，你仍然要对信用局的 API 费用负责。这个挑战并不是 API 特有的，而是在考虑潜在挑战时应该考虑的。

## 你如何将一个 API 货币化？

您已经构建了一个 API，并决定出售它，现在需要弄清楚如何收费和收取收入。为此，您需要有一个记录 API 流量的系统，并可以对其收费。总是有机会构建一个定制的解决方案来处理货币化，但它可能会有陷阱。对于任何自主开发的解决方案，必须准确预测工程和持续支持成本。一般来说，这些成本会膨胀，尤其是当您扩展时。

另一种方法是使用现成的解决方案，可根据您的使用情况进行定制。Moesif 通过平台上的计费表功能提供了这样一个解决方案。有了 Moesif [计费表](https://www.moesif.com/solutions/metered-api-billing?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_content=how-to-sell-your-apis)，API 流量被记录、过滤并发送给计费提供商，如 [Stripe](https://stripe.com/) 或 [Recurly](https://recurly.com/) ，这样就可以收取收入。这种方法非常灵活，因为 Moesif 可以与多个 API 集成和计费提供者一起工作。

货币化能力到位后，最后一步是为您的 API 决定一个定价模型。一旦定价决定并纳入货币化设置，你就开始创造收入了。

## API 市场与 API 门户

API 门户是开发者可以注册和管理对 API 的访问的地方。大多数情况下，API 网关将用于生成这个面向客户的 API 门户，以便用户可以访问已发布的 API。API 门户是一种非常流行的管理 API 访问的方式，它为用户提供了一种直接使用 API 的方式。典型的 API 门户允许用户浏览特定公司提供的可用 API，注册并生成 API 密钥以供访问。门户还可能包含 API 文档或其他 API 知识库文章，以帮助用户了解如何使用 API。

API 市场是一个开发者可以列出他们的 API 来吸引更多用户的地方。流行的 API 市场的例子包括 [Rapid API](https://rapidapi.com/hub) 、 [APILayer](https://apilayer.com/) 等等。API 市场允许开发者浏览市场上列出的任何公司提供的 API。市场通常也提供一个 API 门户，这样用户可以以自助的方式管理他们的 API 访问。

## 让开发者使用你的 API

销售 API 的一切都准备好了，最后一步是让开发者使用它。这可以通过在 API marketplace 上列出它，建立一个网站并围绕它进行营销，或者你通常推广产品的许多其他方式来实现。

一旦你吸引了一些开发者，让 API 上线对于让开发者使用你的服务是至关重要的。如果 API 很难使用或注册，开发者可能会粗制滥造而不使用该服务。跟踪你的入职流程是一个很好的方法，可以确保用户快速方便地从你的服务中获得价值。当试图吸引和留住使用 API 的开发人员时，入门和整体易用性是需要关注的两件事。

## 包扎

销售 API 来推动业务收入变得越来越普遍。希望上面的概述能给我们一些关于如何去做和需要克服的挑战的见解。在这篇文章中，我们讨论了如何销售 API，面临的挑战，API 门户和 API 市场之间的差异，以及如何实现货币化。最后，我们看了一些吸引和留住使用你的 API 的开发者的因素。

为了确保你跟踪和分析 API 流量，并有效地将你的 API 货币化，请查看 Moesif。[立即注册](https://www.moesif.com/signup?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_content=how-to-sell-your-apis)，试用[时间序列图表](https://www.moesif.com/docs/api-analytics/time-series-analysis/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_content=how-to-sell-your-apis)、[留存报告](https://www.moesif.com/docs/user-analytics/cohort-retention-analysis/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_content=how-to-sell-your-apis)、[计费表](https://www.moesif.com/docs/metered-billing/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_content=how-to-sell-your-apis)以及其他帮助您销售 API 的优秀工具。