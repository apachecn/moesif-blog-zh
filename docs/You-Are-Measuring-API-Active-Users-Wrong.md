# 你对 API 活跃用户的衡量是错误的

> 原文：<https://www.moesif.com/blog/api-management/customer-success/You-Are-Measuring-API-Active-Users-Wrong/>

API 提供者需要了解他们的消费者如何使用他们的 API。使用指标是必不可少的，因为它们告诉你 API 的采用情况，你的 API 如何随时间增长，以及哪些端点看到更多(或更少)的使用。当您查看 API 使用指标时，您应该从与您的服务最密切相关的角度来衡量 API 上的活跃用户。这将告知您的组织在哪里分配计算资源，您决定开发哪些 API 端点，以及您记录哪些 API 端点。

什么是活跃用户？乍一看，这似乎只是简单地跟踪使用一个端点的用户数量。但是 API 用户不能像网站用户一样被追踪。相反，你需要看看你的用户在你的产品环境中是如何访问你的 API 的。

API 上的活跃用户是什么样子的？一个活跃的 API 用户是任何一个经常调用你的 API 来获得真实结果的用户。这并不意味着他们必须不断地进行 API 调用才能被认为是活跃的。API 消费者调用 API 的频率取决于 API 的上下文。如果一个活跃的用户需要从您的 API 中获得价值，他可能每天都会进行多次 API 调用，或者如果他需要从您的 API 中获得的只是每周的数据处理，他可能只需要每周进行一次 API 调用。一种常见的做法是将 API 使用指标分为每日活跃用户和每月活跃用户，以便更清楚地了解用户在不同时间段的活跃程度。

虽然采取一刀切的方法来衡量活跃的 API 用户可能很有诱惑力，但是在决定衡量什么时，您需要考虑 API 的上下文。您的度量应该反映您的 API 如何在现实世界中为用户创造价值。例如，允许您发送 SMS 消息的 API 应该跟踪用户发送的 SMS 消息的数量。压缩视频的 API 会跟踪为每个用户压缩和发送的数据文件的大小。这些指标应该与所提供服务的价值直接相关。

### 坏度量，好度量

虽然 API 提供商通常通过跟踪独立用户的数量来衡量活跃用户，但这是一种虚荣的衡量标准，它让事情看起来很好，但并没有真正传达多少信息。由于独立用户的数量通常会随着时间的推移而增加，这很容易让人想到这样的指标，因为它可以向内部利益相关者讲述一个积极的故事。不幸的是，对活跃的独立用户进行简单的计数是一个糟糕的指标，因为它没有给出用户使用 API 做什么的详细信息。如果你是一家百货商店，这与计算橱窗购物者的数量没有太大区别。有趣的是，它没有告诉你有多少人来买东西，他们买了什么，或者他们花了多少钱。相应地，您想知道用户是否继续以有意义的方式使用您的 API，为您的 API 消费者创造了多少价值，以及哪些 API 端点和功能对您的用户最有价值。没有关于 API 消费者的有意义的信息，您的组织就不能决定在哪里应用新的开发或者如何分配资源。

随着用户群的增长，扩展 API 指标非常重要，但这一点经常被忽视。当您有少量的 API 消费者和少量的使用模式时，衡量活跃用户可能很简单。但是，随着 API 变得越来越复杂，准确测量有意义的用户活动变得越来越困难。您需要一个前瞻性的度量策略，随着您的 API 的发展而发展。这有一个额外的好处，那就是确保您的组织不会随着 API 的增长而抛弃老用户。像您的组织已经创建的 API 键的唯一数量，或者对您的 API 端点的调用总数这样的指标，并不总是指示它们最初测量的内容。例如，一个开发者可以创建多个 API 键。

让我们看看什么是好的度量标准。好的度量应该给你关于 API 消费者如何与 API 交互的有意义的信息。您的指标应该能够回答以下问题:

*   在您的 API 响应中传递了多少数据？都有用吗？
*   每个用户与您的 API 保持连接的时间有多长？
*   哪些端点最活跃？
*   用户在进行 API 调用时有多成功？

好的指标不仅仅告诉你有多少开发人员使用了你的 API。上面的问题是考察你的 API 消费者对你的 API 的参与程度的不同方式。如果你的 API 专注于在不同设备之间传输数据，那么传输的数据量是你的用户参与度的一个很好的指标。另一方面，如果您的 API 提供实时服务，比如视频内容的实时字幕，那么连接的长度将是一个更好的参与度指标。 [*了解更多*](https://www.moesif.com/solutions/api-product-management?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=measuring-apis-wrong) *关于 Moesif 如何帮助您跟踪参与度指标的信息。*

### API 活跃用户，真的

好的度量标准的一个好的起点是将您的 API 调用聚合成有意义的事件，而不是查看 API 调用的原始数量，这可能是一个嘈杂的信号。事件可以是对 API 调用的更详细的观察，结合了 API 调用及其结果。

测量事件而不是 API 调用还允许您查看不同类型的事件之间的比较，包括它们的共同点。观察失败的事件可以揭示失败的共同根源。例如，如果您的用户因为太多的流量流向您的 API 端点而遇到失败的事件，您的组织可以通过向您的 API 分配更多的资源来处理这个问题。或者，与您的开发人员入职相关的失败事件的模式可能指向您的用户在开始使用您的 API 时遇到的麻烦。这可能意味着需要更全面的文档，或者您的用户可能需要额外的支持来集成您的 API。

让我们看一些 API 事件度量的真实例子。执行双因素验证的 API 至少有四个基本的 API 事件。成功发送验证短信将是第一个 API 事件，并伴随着一个发送短信失败事件。另外两个事件是当验证代码被成功验证时，以及当验证代码未被验证时。这些事件一起显示了 API 消费者成功使用 API 实现其目标(在本例中，验证电话号码)的频率。通过将 API 交互分成事件来区分成功和失败的 API 交互，可以很容易地理解 API 的实际活动。

### 最佳实践

虽然我们已经讨论了如何通过事件来衡量活跃的 API 用户，从而更好地告知您的 API 消费者如何使用您的 API，但是还有其他好处。例如，您可以使用您的活动 API 使用指标，直接根据您的服务为用户创造的价值来为用户计费。您的组织可以对成功的活动进行计费，从而将计费与实际结果直接联系起来。例如，打电话的 API 可以根据通话时间计费，发送短信的 API 可以根据发送的短信数量计费。

依赖活跃用户指标时，实时报告至关重要。当你的用户使用你的 API 时，通过测量他们可以获得很多有价值的见解。例如，您可以在用户遇到失败事件时立即发现它们，使您的团队能够实时解决失败问题。如果用户的问题得到及时的支持，他们就不太可能离开你的服务。实时分析还允许您的组织查看在给定时刻(例如在重负载期间)成功事件的数量与失败事件的数量的对比情况。借助实时事件指标，您的团队可以快速响应用户需求，监控失败事件，并了解哪些终端为用户提供了最大价值。 [*打造更好的 API 产品*](https://www.moesif.com/solutions/api-product-management?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=measuring-apis-wrong)*moes if 提供的深刻见解。*

### 实时成功

衡量 API 用户的总数可能会产生误导，因为它没有提供关于 API 消费者如何从 API 中获得价值的上下文信息。将您的 API 调用批处理到事件中，将为您的组织提供大量 API 信息。您将能够评估失败的事件，以便您的组织能够了解哪个 API 端点需要更多的关注和资源。您还将能够比较成功事件的模式，以便您的组织能够了解哪些端点为您的 API 消费者创造了最大的价值。

要真正了解你的用户，把你的衡量标准集中在他们用你的 API 做什么，以及他们是否真正达到了他们想要的结果。您的度量标准应该关注为您的用户创造最大价值的 API 事件，以便您了解应该将资源集中在哪里。通过实时测量用户的活动来了解他们，给自己最好的机会来满足他们的需求。如果你一直以来对 API 活跃用户的衡量都是错误的，现在是时候重新考虑你的策略了。你的内部利益相关者和你的客户都会为此感谢你。