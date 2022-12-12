# 为 API 产品公司构建与购买行为电子邮件平台

> 原文：<https://www.moesif.com/blog/technical/behavioral-emails/Build-vs-Buy-Behavioral-Email-Platform-for-API-Product-Companies/>

行为电子邮件是根据客户的行动或行为自动发送给客户的有针对性的消息。通过触发你的客户如何与你的网站或产品互动，你能够发送内容与他们正在做的事情实际一致的电子邮件，因此更有可能引起共鸣。

API 公司在利用行为电子邮件方面处于独特的位置；如果监控和分析做得好，那么对平台内部正在发生的事情的洞察就可以浮出水面；有洞察力，甚至有先见之明的公报是可以发表的。

API 平台公司中行为电子邮件的常见功能包括帮助开发人员加速 API 集成和尝试新产品功能，以及让客户了解订阅和平台问题。

在之前的一篇[文章](https://www.moesif.com/blog/technical/behavioral-emails/How-To-Accelerate-API-Integration-with-Behavioral-Emails-and-Developer-Segmentation/)中，我们介绍了 Moesif 的[行为电子邮件解决方案](https://www.moesif.com/features/user-behavioral-emails)的功能，在另一篇[配套文章](https://www.moesif.com/blog/developer-marketing/behavioral-emails/Top-Five-Behavioral-Emails-Every-Developer-Tool-Should-Have/)中，我们详细介绍了每个 API 产品公司应该使用的 5 大行为电子邮件。

在这篇文章中，我们将从技术上分析如何构建你自己的行为邮件解决方案。

## 构建自己的行为电子邮件解决方案需要什么？

一个行为电子邮件产品至少包括三个部分:

1.  从 API 获取监控数据的方法
2.  一种定义规则和验证监控数据的方法
3.  符合规则时向用户发送特定电子邮件的过程

## 从 API 获取监控数据

根据您的自信程度，您可以从头开始构建一个完整的监控解决方案，也可以采用现成的开源系统，如 [Prometheus](https://en.wikipedia.org/wiki/Prometheus_(software)) 。

Prometheus 是微服务的监控解决方案。通过在您的应用程序中包含一个 Prometheus 客户机，当您想要记录一个指标时，可以调用它。Prometheus 服务器将定期轮询数据源并保存结果。Prometheus docs 有一个设置指南[来帮助安装。](https://prometheus.io/docs/prometheus/latest/getting_started/)

Prometheus 客户端自带配置的默认指标，因此您不必手动设置 CPU 或内存监控等基本功能。

对于基于 Node.js/Express.js 的 API，您可以通过 NPM 为 Prometheus 安装 [Node.js 客户端](https://www.npmjs.com/package/prom-client),配置只需要几行代码:

```py
const prometheus = require('prom-client');
const register = new prometheus.Registry();
prometheus.collectDefaultMetrics({ register }); 
```

默认指标是基本的，与 API 无关，因此您必须自己设置更复杂的指标。

一个非常简单的 API 指标的代码可能如下所示:

```py
const activeRequests = new client.Gauge({
  name: 'activeRequests'
});
registry.registerMetric(activeRequests);
api.use((req, res, next) => {
  activeRequests.inc();
  res.on('close', () => activeRequests.dec());
  next();
}); 
```

根据我们的[文章](https://www.moesif.com/blog/ebooks/15-api-metrics-every-platform-team-should-be-tracking-infographic/)您应该跟踪的 15 个最重要的 API 指标，Prometheus 只涵盖了我们确定的前三个指标，其余的必须用自定义代码定义。

## 定义规则和验证数据

一旦您在 Prometheus 中存储了监控数据，您需要测试客户是否做了会触发行为电子邮件的事情。顾名思义，行为邮件要求我们观察用户行为。

为此，您必须设置一项服务，让您定义电子邮件规则，轮询监控数据并根据您的规则验证数据。

要轮询数据，您可以使用 Prometheus 的查询语言 [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) ，它允许通过 HTTP API 收集数据。从 Prometheus 收集了所需的数据后，您可以使用逻辑操作、通配符和正则表达式来定义应该查询监控数据的规则。

状态为 400 的响应的自定义普罗米修斯度量可以定义如下。使用标签功能，可以将自定义数据(如 userID)添加到跟踪的每个值中:

```py
const status400 = new promClient.Counter({
  name: "status_400",
  labelNames: ["userId"],
});

expressApi.use((req, res, next) => {
  res.on("finish", () => {
	if (res.statusCode == 400)
  	status400.labels(req.session.userId).inc();
  });
  next();
}); 
```

如果您想获得过去 24 小时内的错误，您必须使用 PromQL 查询来查询它们:

```py
status_400{userId="abc-123"}[24h] 
```

快速编写这些查询变得很复杂，如果您不是工程师，会特别困难。您会让普通商务人士编写这样的查询吗:

```py
rate(http_requests_total{status_code=~"5.*"}[5m]) / rate(http_requests_total[5m]) 
```

大概不会。所以你也要建立一个编辑器。要么是可视化的，要么带有一些简化的规则语言和自动完成功能，因此没有工程学位的人也可以设置这些东西。

根据这个规则编辑器的复杂程度，它可以根据规则中指定的数据自动生成所需的 PromQL。您还需要另一个可视化编辑器来允许人们将他们的数据需求放在一起。

## 向用户发送电子邮件

系统的最后一部分实际上是发送电子邮件。您需要一种方法来定义用用户数据填充的电子邮件模板，然后您需要确保它们符合您在上一步中描述的规则。

同样，取决于你的电子邮件创建者的技术技能，这可能需要一个更完善的编辑器。今天，许多人有信心编写 HTML 和 CSS，这是一件好事，因为富文本电子邮件已被证明比纯文本电子邮件更成功。

我们已经为每个普罗米修斯指标关联了一个唯一的用户 ID，所以我们现在需要找到属于该 ID 的电子邮件。电子邮件系统需要访问用户的帐户数据，这在大多数情况下意味着连接到 SQL 数据库并查询其中的用户配置文件。比如 SQL 查询可能是这样的:

```py
SELECT firstName, lastName, email FROM Accounts
WHERE userId=abc-123; 
```

将电子邮件发送到客户的收件箱是另一个问题。如果你建立自己的电子邮件服务器，它们可能会被放在垃圾邮件文件夹中。解决这个问题的典型方法是在主要电子邮件提供商的允许列表上集成电子邮件传递服务。

其中一个提供者是 SendGrid。他们提供了一个 API，你可以通过他们的服务直接从你的后台发送电子邮件。他们的 Node.js 包使用起来很简单。它甚至带有批处理功能，所以你可以一次发送多封电子邮件，这对我们的用例是一个很好的补充。

与 SendGrid 集成的示例如下所示:

```py
const sendgrid = require("@sendgrid/mail");
sendgrid.setApiKey(SENDGRID_API_KEY);

sendEmails([
  {
	to: "recipient1@example.org",
	from: "sender@example.org",
	subject: "Hello recipient 1",
	text: "Hello plain world!",
	html: "<p>Hello HTML world!</p>",
  },
  {
	to: "recipient2@example.org",
	from: "other-sender@example.org",
	subject: "Hello recipient 2",
	text: "Hello other plain world!",
	html: "<p>Hello other HTML world!</p>",
  },
]);

async function sendEmails(emails) {
  try {
	await sendgrid.send(emails);
  } catch (error) {
	console.error(error);
	if (error.response) console.error(error.response.body);
  }
} 
```

## 维护您自己开发的系统

在你建立了你的系统之后，你还需要维护它。它需要更新、修复错误、改进，而且随着您的成长，可能还需要扩展更多的硬件。

最好能够分享你的电子邮件的表现，让同事、经理和合作伙伴都知道它是怎么做的:

*   实际触发了哪些规则？
*   电子邮件会被投递吗？
*   邮件打开了吗？
*   有多少客户打开了哪封邮件？
*   他们打开邮件后对你的 API 做了什么？

还应该构建和维护一个可视化电子邮件数据的用户界面，以便可以在整个公司范围内有效地共享。

## 摘要

构建一个行为邮件平台并不神奇，可以由精通 API 监控的程序员来完成。可以利用开源工具更快地启动和运行，而不需要从头开始构建一切。

但是，仍然需要大量的定制代码来集成这些工具。行为电子邮件平台需要由非技术团队成员操作，这需要构建图形用户界面，如用于创建查询、规则和电子邮件模板的编辑器。一个允许你查看发生了什么的仪表板也是更好的选择。

构建这些 GUI 是非常代码密集的，并且是一项耗时的任务。如果您经营一家小型 API 公司，或者不想投资构建和维护监控和分析工具，那么您最好使用像 Moesif 这样的工具。

Moesif 专注于 API 分析和监控，并且已经走上了从零开始构建行为电子邮件平台的道路。我们的可扩展解决方案可以处理数十亿个 API 调用和数千封电子邮件，并且只需几行代码即可集成到您的平台中。