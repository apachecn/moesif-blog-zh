# 如何监控第三方 API 集成

> 原文：<https://www.moesif.com/blog/monitoring/third-party/How-To-Monitor-Third-Partry-API-Integrations/>

许多企业和 SaaS 公司依赖各种外部 API 集成来构建出色的客户体验。一些整合可能会将某些业务功能(如处理支付或搜索)外包给 Stripe 和 Algolia 等公司。您可能已经整合了其他合作伙伴，从而扩展了您的产品功能。例如，如果您想要向分析工具添加实时警报，您可能想要将 PagerDuty 和 Slack APIs 集成到您的应用程序中。

如果你像大多数公司一样，你会很快意识到你正在将数百个不同的供应商和合作伙伴集成到你的应用中。其中任何一个都可能存在影响客户体验的性能或功能问题。更糟糕的是，集成的可靠性可能不如您自己的 API 和后端那么明显。如果登录功能被破坏，你会有很多客户抱怨他们不能登录到你的网站。但是，如果您的 Slack 集成被破坏，那么只有那些将 Slack 添加到其帐户的客户会受到影响。最重要的是，由于集成是异步的，您的客户可能直到几天后才意识到集成被破坏了，因为他们已经有一段时间没有收到任何警报了。

你如何确保你的 API 集成是可靠的和高性能的？毕竟，如果你卖的是*实时警报*功能，你的警报最好是*实时*的，至少保证送达。从客户体验的角度来看，因为您的 Slack 或 PagerDuty 集成是不可接受的，所以丢弃警报。

## 监控什么

### 潜伏

具有极高延迟的特定 API 集成可能是您的集成即将失败的信号。也许您的分页方案不正确，或者供应商没有以最有效的方式为您的数据建立索引。

#### 延迟最佳实践

平均延迟只能告诉你故事的一半。一个持续花费一秒钟完成的 API 通常比一个高方差的 API 要好。例如，如果一个 API 平均只需要 30 毫秒，但是 10 个 API 调用中有 1 个需要 5 秒，那么您的客户体验就有很大的差异。这使得追踪 bug 更加困难，在客户体验中更难处理。这就是为什么第 90 个和第 95 个百分点更值得关注。

### 可靠性

可靠性是一个需要监控的关键指标，尤其是因为您正在集成您无法控制的 API。API 调用失败的百分比是多少？为了跟踪可靠性，您应该对什么构成失败有一个严格的定义。

#### 可靠性最佳实践

虽然任何具有 4xx 或 5xx 系列响应状态代码的 API 调用都可能被视为错误，但您可能会遇到这样的特定业务案例:API *似乎*成功完成，但 API 调用仍应被视为失败。例如，如果一个数据 API 集成始终没有返回匹配项或内容，那么这个集成就会被认为是失败的，即使状态代码总是 *200 OK* 。另一个 API 可能会返回虚假或不完整的数据。*数据验证*对于衡量返回的数据是否正确和最新至关重要。

并非每个 API 提供商和集成合作伙伴都遵循建议的[状态代码映射](https://www.restapitutorial.com/httpstatuscodes.html)

![Monitoring API Errors](img/0ba9d5ac02f8559a8dc54a56574a9747.png)

## 有效性

虽然可靠性特定于错误和功能正确性，但可用性和正常运行时间是纯粹的基础设施指标——它们衡量服务中断的频率，即使是暂时的。可用性通常以每年正常运行时间的百分比或[个 9 的数量来衡量。](https://en.wikipedia.org/wiki/High_availability)

| 可用性% | 每年停机时间 | 每月停机时间 | 每周停机时间 | 每天停机时间 |
| --- | --- | --- | --- | --- |
| 90%(“一九”) | 36.53 天 | 73.05 小时 | 16.80 小时 | 2.40 小时 |
| 99%(“两个九”) | 3.65 天 | 7.31 小时 | 1.68 小时 | 14.40 分钟 |
| 99.9%(“三个九”) | 8.77 小时 | 43.83 分钟 | 10.08 分钟 | 1.44 分钟 |
| 99.99%(“四个九”) | 52.60 分钟 | 4.38 分钟 | 1.01 分钟 | 8.64 秒 |
| 99.999%(“五个九”) | 5.26 分钟 | 26.30 秒 | 6.05 秒 | 864.00 毫秒 |
| 99.9999%(“六个九”) | 31.56 秒 | 2.63 秒 | 604.80 毫秒 | 86.40 毫秒 |
| 99.99999%(“七个九”) | 3.16 秒 | 262.98 毫秒 | 60.48 毫秒 | 8.64 毫秒 |
| 99.999999%(“八个九”) | 315.58 毫秒 | 26.30 毫秒 | 6.05 毫秒 | 864.00 微秒 |
| 99.9999999%(“九个九”) | 31.56 毫秒 | 2.63 毫秒 | 604.80 微秒 | 86.40 微秒 |

### 使用

许多 API 提供商对 API 的使用进行定价。即使 API 是免费的，他们也很可能在 API 上实现了某种速率限制，以确保坏的参与者不会饿死好的客户端。这意味着跟踪您的每个集成合作伙伴的 API 使用情况是至关重要的，这样您就可以确定您当前的使用情况何时接近计划或速率限制。

#### 使用最佳实践

建议将使用与您的最终用户联系起来，即使 API 集成在您的客户体验的下游。这有助于衡量特定集成的直接投资回报率并确定趋势。例如，假设你的产品是 CRM，你每月向 Clearbit 支付 199 美元来充实多达 2500 家公司。这是与客户的使用相关的直接成本。如果您有一个免费层，并且某个特定客户正在使用您的大部分 Clearbit 配额，那么您可能需要重新考虑您的定价策略。也许在这种情况下，Clearbit enrichment 应该只在付费层提供。

## 如何监控 API 集成

监控 API 集成似乎是控制这些问题的正确方法。然而，像 New Relic 和 AppDynamics 这样的传统应用程序性能监控(APM)工具更侧重于监控您自己的网站和基础设施的健康状况。这包括基础设施指标，如内存使用和每分钟请求数，以及应用程序级别的健康状况，如 appdex 分数和延迟。当然，如果您正在使用运行在其他人的基础设施中的 API，您不能仅仅要求您的第三方提供商安装您有权访问的 APM 代理。您将需要一种间接监控第三方 API 的方法，或者通过一些其他的工具方法。

## 监控传出 API 调用的挑战

为了监控第三方 API 调用，您需要一种机制来记录来自应用程序的传出 API 调用(在它们到达第三方之前)和响应。许多 APM 代理在进程级别捕获信息，这对于捕获 JVM 线程、配置文件或内存百分比等基础设施指标非常有用，但对于捕获特定于应用程序的上下文来说却很糟糕。第二个挑战是，您的应用程序中的传出调用通常分散在您自己的代码中和您在应用程序中使用的所有供应商的 SDK 中。每个第三方 SDK 都有自己的回调和逻辑来处理 API 通信。这使得很难通过横切关注点或集中式代理来捕获传出的 API 调用。因此，您可能决定不利用特定于供应商的 SDK，或者修改它们以拦截 API 流量。后者代表了大量繁忙的工作，很少有工程团队有时间去做。

[真实用户 API 监控](https://www.moesif.com/features/api-monitoring)是现代科技公司推荐的做法。真实用户 API 监控查看真实的客户流量，而不是预定的探测。这使您能够捕捉到更深层次的集成问题，比如特定的访问模式，而健康证明不会捕捉到这些问题。

### 我们在 Moesif 是如何做到的

在 Moesif，我们建立了一个跟踪第三方 API 的 SaaS 解决方案。我们通过利用 monkey 修补和反射在通用编程语言的核心 HTTP 客户端库中注入跟踪代码来做到这一点。因为工具在应用程序本身中，所以有机会添加特定于业务的上下文，例如发起 API 调用的原始最终用户，以记录特定的响应头，从而提供对速率限制或其他软错误的洞察。我们可以做到这一点，因为大多数语言依赖于单个或少量的核心 HTTP 库。对于 Node.js，所有的 HTTP 通信都基于 [http](https://nodejs.org/api/http.html) 或 [https](https://nodejs.org/api/https.html) 模块。对于 Python，大多数高级客户端将利用 [Python 请求](https://requests.readthedocs.io/en/master/)库。

下面是 Node.js 用于检测传出 API 调用的中间件示例:

```py
var express = require('express');
var app = express();
var moesifExpress = require('moesif-express');

// 2\. Set the options, the only required field is applicationId.
var moesifMiddleware = moesifExpress({
  applicationId: 'Your Moesif Application Id',
  logBody: true
});

// 3\. Start capturing outgoing API Calls to 3rd party services like Stripe
moesifMiddleware.startCaptureOutgoing(); 
```

`startCaptureOutgoing()`将通过用我们修改过的方法覆盖`http`和`https`对象上的每个方法来为 Node.js `http`和`https`模块打补丁，修改后的方法拦截传出的 API 调用。下面是我们如何为 Node.js 执行 monkey 补丁的例子，完整的 SDK 也可以在 [GitHub 上获得。](https://github.com/Moesif/moesif-express/blob/master/lib/outgoing.js)

```py
function _patch(recorder, logger) {
  var originalGet = http.get;
  var originalHttpsGet = https.get;

  var originalRequest = http.request;
  var originalHttpsRequest = https.request;

  http.request = function(options, ...requestArgs) {
    var request = originalRequest.call(http, options, ...requestArgs);
    if (!request._mo_tracked) {
      request._mo_tracked = true;
      track(options, request, recorder, logger);
    }
    return request;
  };

  https.request = function(options, ...requestArgs) {
    var request = originalHttpsRequest.call(https, options, ...requestArgs);
    if (!request._mo_tracked) {
      request._mo_tracked = true;
      track(options, request, recorder, logger);
    }
    return request;
  };

  http.get = function(options, ...requestArgs) {
    var request = http.request.call(http, options, ...requestArgs);
    request.end();
    return request;
  };

  https.get = function(options, ...requestArgs) {
    var request = https.request.call(https, options, ...requestArgs);
    request.end();
    return request;
  };

  function _unpatch() {
    http.request = originalRequest;
    https.request = originalHttpsRequest;
    http.get = originalGet;
    https.get = originalHttpsGet;
  }

  return _unpatch;
} 
```

## 如何利用监控

仅仅设置仪表板和报告您的 API 集成只是故事的一半。您可以通过多种方式利用这些数据来改善客户体验:

1.  当指标超出界限或有异常行为时，设置警报。一个简单的方法是通过传呼机、Slack 或其他渠道。
2.  让合作伙伴对其 SLA 负责。当您一直遇到延迟问题，或单一供应商的失败时，让他们知道。如果您已经有一个完整的审计日志显示发生了什么，合作伙伴可能能够调整他们的基础设施，以更好地适应您的访问模式，甚至在他们未能履行合同义务时向您退款。
3.  避免停机，并确保您自己的团队能够响应合作伙伴的问题。这可能包括通过*功能标志*关闭功能，直到合作伙伴解决问题。