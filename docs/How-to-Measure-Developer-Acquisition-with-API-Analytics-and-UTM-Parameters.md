# 如何用 API 分析和 UTM 参数衡量开发者获取

> 原文：<https://www.moesif.com/blog/business/acquisition/How-to-Measure-Developer-Acquisition-with-API-Analytics-and-UTM-Parameters/>

在建立企业时，衡量什么可行，什么不可行是至关重要的。对于 API 至上的公司来说尤其如此。他们需要知道:

*   他们的 API 是否稳定——他们的客户满意吗
*   有多少客户在使用他们的 APIs 这是一项可行的业务吗
*   使用了哪些 API 特性，如何使用它们——需要进行哪些产品增强
*   不一定直接映射到 API 的技术方面的标准，例如谁是他们的用户，他们是如何获得的

[Moesif API Analytics](https://www.moesif.com/features/api-analytics) 允许合并业务和技术视角，以便呈现操作环境的完整画面。我们的解决方案利用了一个漏斗，从客户与我们的登录页面的最早交互开始，通过集成和 TTFHH 过渡，以客户每天通过数十亿次 API 调用的稳定状态结束。

## 开发者归属

开发者归因是将使用我们服务的开发者归因于他们最初发现我们服务的渠道的行为。比如脸书广告、谷歌搜索结果、博客文章或推文。

通常，使用我们的 API 的开发人员正在旅途中。

1.  他们发现了我们的服务
2.  他们开始使用我们的服务
3.  他们停止使用我们的服务

这个旅程的第一步是最重要的一步，因为如果没有人知道我们的 API，它可能是人类有史以来创造的最好的东西，但没有人使用它。

这使得找出哪些营销努力导致了转化变得至关重要，在这种情况下，是注册，哪些没有。

在旅程的后续步骤中，了解开发人员通过哪种渠道找到我们也是至关重要的。

可能来自某个特定渠道的开发者很快就停止使用我们的 API 了。跟踪这些信息可以让我们检查这些渠道是否会给开发者带来错误的期望。

有了关于用户如何找到我们的知识，我们可以改变我们通过该渠道提供的信息，或者我们可以改进我们的 API 来满足这些期望。

许多因素决定了 API 业务的成败，而且并非所有因素都是技术性的。

## Moesif 浏览器 SDK

大多数 Moesif SDKs 运行在后台。它们捕获我们的 API 接收的所有请求，以及我们从那里发送给第三方 API 的请求。

但是，要想全面了解用户的旅程，我们还需要某种方式来实现开发者归属，也就是搞清楚一个用户是怎么找到我们的。

针对这个问题， [Moesif 提供了一个基于 JavaScript 的 SDK](https://github.com/Moesif/moesif-browser-js) ，运行在浏览器中。

这有点像谷歌分析，但完全集成了我们的 API 分析的其余部分。

它自动捕获发送到我们的 API 和客户端第三方 API 的请求。所有后端不会注意到的事情。

## 活动数据

Moesif Browser SDK 的一个优点是，它还可以捕获发送这些请求的用户的信息。

这些识别信息包括用户或会话 id，也包括自定义数据，如电子邮件、姓名、年龄、性别、公司、位置等。

然而，它也捕捉活动数据，如推荐网站或 UTM 参数。

### UTM 参数

UTM 代表*顽童追踪模块*，这是谷歌分析前身的名字。它是存储在查询字符串或 cookie 中的一组参数，告诉网站当前用户来自哪里，就像 referrer 一样，但也是一组经常用于营销活动的额外信息。

*   `utm_source`喜欢推荐人
*   如果链接在应用程序、电子邮件或其他媒介中
*   `utm_campaign`此链接属于哪个营销活动
*   使用了哪些搜索词来获得链接
*   `utm_content`链接位于网站、电子邮件或应用程序的哪个位置

### 自动捕获

Moesif Browser SDK 会自动捕获这些值(如果有的话),并将用户或会话 ID 与之相关联。这允许我们找出哪个 API 请求属于哪个营销活动。

然后我们可以问这样的问题:

*   脸书广告公司的用户如何使用我们的 API？
*   是否有比其他营销渠道更能吸引长期用户的营销渠道？
*   来自 Google 的用户和来自 Twitter 的用户在 API 使用上有显著差异吗？
*   等等。

## 结论

使用一个统一的 API 监控和业务分析解决方案，有助于提出与 API 使用相关的高级营销问题。

在商业中，了解全局是至关重要的，而我们可能从来没有亲自见过我们的客户，这使得我们更难看到正在发生的事情。

针对这些问题使用不同的分析解决方案需要我们以某种方式将它们联系起来，但 Moesif API 分析套件及其服务和针对不同前端和后端的所有 SDK 有助于实现这一切。