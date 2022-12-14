# 开发者体验与用户体验

> 原文：<https://www.moesif.com/blog/developer-relations/developer-marketing/Developer-Experience-Vs-User-Experience/>

虽然乍看之下它们似乎很相似，但是开发者体验(DX)不仅仅是“开发者的用户体验(UX)”。更确切地说，DX 是 UX 的一个扩展，专注于使用技术语言和工具进行构建的用户。DX 遵循与 UX 相同的核心原则，但通过认识到技术细节和机械过程可以被开发者有效地理解和利用，对其进行了扩展。

> 当开发人员感到有人在和他们说话，并且他们的需求得到了直接满足时，伟大的 DX 就发生了。这意味着展示代码，提供大量细节，并对多种用例给出清晰的说明。

用户体验是一套最常用于最终用户的通用原则，这些最终用户可能不是技术人员，他们使用软件来满足业务需求和个人消费。开发人员体验的重点是让开发人员能够使用软件构建解决方案。UX 和 DX 之间的区别来自于最终用户和开发者的不同需求。在这篇文章中，我们将探索什么是好的 UX 和 DX，它们之间的主要区别，以及当你为开发者开发产品时的实际区别。

## 伟大的 DX 始于 UX 的原则

好的 UX 有两个组成部分:让开始使用你的产品变得容易，并确保继续使用它是令人愉快的。开发人员经验扩展了这些相同的核心思想，应用于面向目标的技术受众。

首先，尽可能容易地开始使用新产品。在 UX，这通常意味着尽可能避免注册流程，或者在必要时保持简单。产品应该是直观的——最终用户或开发者应该能够开始使用你的产品，并在最少的指导下实现他们的目标。记住你的产品提供的价值也很重要。您可以使用一些相同的工具来评估 UX 和 DX，如 A/B 测试、用户调查和分析软件。

UX 和 DX 的第一个关键区别是用户体验。好的 UX 让你的产品的用户之旅尽可能简单明了。太多的选择会让最终用户不知所措，这使得完成任务和从产品中获取价值变得更加困难。好的 DX 需要更多的灵活性来支持许多不同的可能的开发者目标。类似地，虽然最终用户不需要技术细节就能成功地使用你的产品，但是开发者需要获得详细的信息，这样他们的需求才能得到评估。好的 DX 让开发人员很容易理解产品或服务是如何工作的，这样他们就可以成功地使用它。

虽然良好的 UX 使具有不同技术能力的用户同样可以使用产品，但 DX 应该考虑不同用户的技术能力，并提供不同级别的支持。经验丰富的开发人员希望使用更强大的工具，而经验不足的开发人员可能需要从更简单的工作流开始，这可能更容易配置。为了让你的开发者体验更好地适应你的产品，了解你的主要用户的技术能力水平是很重要的。

既然我们已经比较了开发人员体验和用户体验之间的高层次差异，我们就来看一个应用程序示例，它展示了实践中的差异。

## UX Vs. DX -一个实际的例子

让我们看看 UX 和 DX 在现实世界中的样子。

我们大多数人可能都是消费者购物应用的最终用户。想象一个用户第一次使用一个新的购物应用程序，寻找一件新毛衣。他们可能会在搜索框中输入搜索词“毛衣”，或者点击分类导航。然后，他们会选择他们喜欢的毛衣，并将其添加到购物车中。为了结账，他们可能需要创建一个包含账单和运输信息的账户，或者他们可以简单地点击“用 PayPal 支付”,让网站处理剩下的事情。我们的用户现在需要做的就是等待他们的毛衣到达。

现在，想象一个开发人员在一个新的应用程序中构建相同的功能。抛开任何用于搜索功能的 API 不谈，他们至少会有一个 API 来处理认证和计费，甚至更多。首先，他们必须安装任何必需的库或 SDK，并解决依赖性问题。接下来，开发人员将生成一个 API 密钥，并将其添加到应用程序的环境变量中，以实现可重用性。然后，开发人员将为用户编写对身份验证端点的 API 调用，并编写一个侦听器来确认用户已经过身份验证。下一步是编写一个对应用程序后端的调用，以确认仍然有毛衣可用，并编写一个 API 调用来处理付款。最后，如果支付成功，开发人员应该编写代码来侦听来自 API 的响应，更新后端以从库存中删除一件毛衣，向履行中心发送信息，并安排包裹的交付。

开发人员要成功构建这款应用，需要了解要使用哪些 API 端点、不同端点允许哪些方法、期望从 API 获得哪些响应，以及如何使用 API 密钥安全地验证 API 调用。没有好的 DX，如果开发人员不能快速解决问题，他们可能会在任何时候悄悄地放弃您的产品，或者如果他们没有信心能够成功，他们可能永远不会尝试实现。

正如消费者在购买毛衣时有很多选择一样，开发人员也有很多工具选择。让您的选择成为一个令人信服的选择始于良好的 DX。如果开发者觉得有信心可以使用你的产品，他们会选择你的产品，如果他们觉得成功，他们会继续使用你的产品。

出色的开发者体验始于了解开发者作为用户的独特需求。在下一节中，我们将看看一般的 UX 标准如何应用于面向开发人员的产品，以及如何针对开发人员的受众对它们进行定制。

## 通过良好的开发人员体验释放 API 产品价值

开发人员体验包括开发人员在构建过程中与您的服务或产品进行交互时发生的事情。开发人员希望感受到创造和创新的力量，好的 DX 帮助他们看到实现想法的可能性和步骤。您需要建立一种激发信心的开发人员体验，然后进行监控以确保您的用户成功并扩展他们的集成。

让我们更详细地看看好的 DX 的特性，以及它们在实践中通常是如何实现的。

开发人员需要能够配置他们使用的服务，以适应各种工作流。线性的用户旅程并不理想，因为它不能反映开发人员的工作方式或应用程序的构建方式。Great DX 提供了支持灵活性的工具和信息，因此您的产品应该尽可能支持不同的配置。实现这一点的好方法包括:

*   提供一系列范围明确的 API 端点
*   用多种语言开发 SDK
*   将代码分割成更小的可重用方法
*   创建可读的无术语代码
*   全面记录你的产品

你必须给开发者足够的信息来理解你产品的技术细节。

您的文档是开发人员进入您的产品的切入点，并告知他们实现产品的决定。如果开发人员能够理解您的产品是如何工作的，他们就可以放心地在他们的工作流程中实现它。您需要清晰、最新和全面的文档，以便开发人员可以学习如何使用您的产品、解决问题以及将其集成到他们现有的工作流中。如果您为具有不同技能和专业水平的开发人员构建产品，这一点尤其重要。

最终，开发人员需要感觉到你理解他们的痛点，并且你的产品、工具和文档解决了他们的问题。有了好的 DX，任何开发人员都应该能够用最少的指令建立产品的基本实现。更有经验的开发人员应该有信心扩展产品的功能以适应独特的用例。您可以通过展示示例代码、在您的文档中提供详细的解释、使用 API 分析来了解开发人员如何使用您的 API，以及演示多个用例的清晰示例来实现这一点。

## 开发者体验:超越第一印象

留住开发者需要的不仅仅是第一印象。正如良好的 UX 需要评估，完善，并随着时间的推移测试，良好的 DX 是一项长期投资。如果不使用分析来评估您的 DX 和测试更改，您将不会知道自己取得了多大的成功。监控您的 API 有助于您识别未能成功进行 API 调用的用户，为开发人员找到成功和失败的模式，并了解不同用户如何随着时间的推移使用您的产品。虽然对于关注最终用户的产品来说，跟踪 UX 指标相对简单，但是 DX 指标在很多重要方面有所不同。你需要为 API 分析制定一个好的策略，这样你就可以[跟踪相关的商业价值指标，同时避免虚荣指标](https://www.moesif.com/blog/api-product-management/api-metrics/Vanity-Metrics-for-APIs-vs-Tracking-Business-Value-From-API-Transactions/?utm_campaign=Int-site&utm_source=blog&utm_medium=body&utm_term=devex-vs-userex)。

虽然开发者体验分享了用户体验的许多关键方面，但重要的是要记住，开发者有超出标准 UX 原则的特定需求。当您为开发人员构建产品时，您需要理解 DX，这样您就可以吸引开发人员用户，激发他们的信心和创造力，并随着时间的推移支持他们日益复杂的集成。构建良好的 UX 和 DX 可能具有挑战性，但是使用 [Moesif API Analytics](https://www.moesif.com/solutions/api-product-management/?utm_campaign=Int-site&utm_source=blog&utm_medium=body&utm_term=devex-vs-userex) 您可以监控您的 API，并使用指标来打造完美的 API 开发人员体验。