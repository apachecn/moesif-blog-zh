# REST APIs(跨来源资源共享)权威指南

> 原文：<https://www.moesif.com/blog/technical/cors/Authoritative-Guide-to-CORS-Cross-Origin-Resource-Sharing-for-REST-APIs/>

REST APIs 跨源资源共享(CORS)的深度指南，介绍了 CORS 的工作方式，以及常见的陷阱，尤其是在安全性方面。

### 什么是 CORS？

CORS 是一种安全机制，它允许来自一个域或来自[](#how-is-origin-defined)*的网页访问不同域的资源(跨域请求)。CORS 是对现代浏览器中执行的[同源策略](https://en.wikipedia.org/wiki/Same-origin_policy)的放松。如果没有像 CORS 这样的功能，网站只能通过同源策略访问同源资源。*

 *#### 同源为什么存在？

像许多网站一样，您可能会使用 cookies 来跟踪身份验证或会话信息。这些 cookies 在创建时被绑定到某个域。每次对该域进行 HTTP 调用时，浏览器都会附加为该域创建的 cookies。这是在每个 HTTP 调用上的，可能是静态图像、HTML 页面，甚至是 AJAX 调用。

这意味着当你登录*https://examplebank.com*时，会为*https://examplebank.com*存储一个 cookie。如果银行是一个单页面的 React 应用程序，他们可能已经在 https://examplebank.com/api 的 T4 为 SPA 创建了一个 REST API 来通过 AJAX 进行通信。

##### 跨域漏洞

假设你在登录*https://examplebank.com*时浏览到一个恶意网站*https://evilunicorns.com*。如果没有同源策略，黑客网站可以对*https://examplebank.com/api*到`POST /withdraw`进行经过验证的恶意 AJAX 调用，即使黑客网站不能直接访问银行的 cookies。

这是由于浏览器行为自动附加任何绑定到*https://examplebank.com*的 cookies，用于对该域的任何 HTTP 调用，包括从【https://evilunicorns.com】T2 到【https://examplebank.com】T4 的 AJAX 调用。通过将 HTTP 调用仅限于相同来源(即浏览器标签的域)，同源策略关闭了一些黑客后门，例如大约[跨站点请求伪造(CSRF)](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (尽管不是全部。像 CSRF 令牌这样的机制仍然是必要的)。

#### 原产地是怎么定义的？

*来源*包括**协议、域、**和**端口的组合。**这意味着*https://****API****. my domain . com*和*https://mydomain.com*实际上是不同的起源，因此受到同源政策的影响。同理，*http://localhost:****9000***和*http://localhost:****8080***也是不同的来历。当考虑原点时，路径或查询参数被忽略。

*Origin* 指的是发起请求的内容，通常是打开的浏览器标签，但也可能是 iFrame 窗口的来源。

#### 为什么创造了 CORS？

网站发出跨来源的 HTTP 请求是有正当理由的。也许在 https://mydomain.com 的一个单页应用需要对 https://api.mydomain.com 的 T2 进行 AJAX 调用；或者，https://mydomain.com*整合了一些第三方字体或分析提供商，如谷歌分析或 MixPanel。跨源资源共享* (CORS)支持这些跨域请求。

### CORS 是如何运作的？

* * *

CORS 请求有两种类型， [*简单请求*](#simple-requests) 和 [*预先请求。*](#preflighted-requests) 关于请求是否被预先发送的规则将在后面讨论。

#### 一.简单的请求

简单请求是一种 CORS 请求，在启动前不需要飞行前请求(初步检查)。

1.  一个打开到`https://www.mydomain.com` 的浏览器标签发起 AJAX 请求`GET https://api.mydomain.com/widgets`

2.  除了添加像`Host`这样的头之外，浏览器还自动为跨源请求添加`Origin`请求头:

```py
GET /widgets/ HTTP/1.1
Host: api.mydomain.com
Origin: https://www.mydomain.com
[Rest of request...] 
```

3.  服务器检查`Origin`请求头。如果允许原始值，它将把`Access-Control-Allow-Origin`设置为请求头`Origin`中的值。

```py
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: https://www.mydomain.com 
Content-Type: application/json
[Rest of response...] 
```

4.  当浏览器收到响应时，浏览器会检查`Access-Control-Allow-Origin`标题，看它是否与选项卡的原点匹配。否则，响应将被阻止。如果`Access-Control-Allow-Origin`与单个原点完全匹配，或者包含通配符 ***** ，则检查通过，如本例所示。

**响应`Access-Control-Allow-Origin: *` 的服务器允许所有可能成为[大安全风险的来源](#common-pitfalls)。

只有在你的应用绝对需要的情况下才使用*，比如创建一个开放/公共 API。**

##### 目的

如您所见，服务器可以根据请求的来源控制是否允许请求。浏览器保证`Origin`请求头的设置可靠而准确。

#### 二、议案预先发出的请求

预先请求是另一种类型的 CORS 请求。预检请求是一种 CORS 请求，要求浏览器在发送被预检的请求之前发送一个*预检请求*(即初步探测)，以询问服务器是否允许原始 CORS 请求继续进行。该预检请求本身是对同一 URL 的`OPTIONS`请求。

由于最初的 CORS 请求之前有一个飞行前请求，我们称最初的 CORS 请求*为飞行前请求*。

#### 在以下情况下，任何 CORS 申请都必须预先审核:

*   它使用 GET、HEAD 或 POST 之外的方法。此外，如果 POST 用于发送具有不同于 application/x-www-form-urlencoded、multipart/form-data 或 text/plain 的内容类型的请求数据，例如，如果 POST 请求使用 application/xml 或 text/xml 向服务器发送 XML 有效负载，则该请求将被预触发。
*   它在请求中设置自定义头(例如，请求使用 X-PINGOTHER 之类的头)

*来源:* [*Mozilla*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Preflighted_requests)

##### 需要预检请求的典型案例:

1.  一个网站调用 AJAX 将 JSON 数据发送到 REST API，这意味着`Content-Type`头是`application/json`

2.  一个网站对一个 API 进行 AJAX 调用，该 API 使用一个令牌来验证请求头中的 API，如`Authorization`

这意味着支持单页面应用程序的 REST API 让大多数 AJAX 请求被预触发是很常见的。

##### 示例流程

1.  一个打开到`https://www.mydomain.com` 的浏览器标签启动一个带有 JSON 有效负载的认证 AJAX 请求`POST https://api.mydomain.com/widgets`。浏览器首先发送`OPTIONS`请求(也称为飞行前请求),其中包含建议的请求方法和主请求的请求头:

```py
OPTIONS /widgets/ HTTP/1.1
Host: api.mydomain.com
Origin: https://www.mydomain.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
[Rest of request...] 
```

2.  服务器返回指定允许的 HTTP 方法和头。如果最初的 CORS 请求打算发送列表中没有的头或 HTTP 方法，浏览器将不尝试 CORS 请求而失败。

```py
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://www.mydomain.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
Content-Type: application/json
[Rest of response...] 
```

3.  由于头和方法通过了检查，浏览器发送原始的 CORS 请求。注意`Origin`头也在这个请求上。

```py
POST /widgets/ HTTP/1.1
Host: api.mydomain.com
Authorization: 1234567
Content-Type: application/json
Origin: https://www.mydomain.com
[Rest of request...] 
```

4.  响应在`Access-Control-Allow-Origin`标题中有正确的来源，因此检查通过，控制权交还给浏览器选项卡。

```py
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://www.mydomain.com
Content-Type: application/json
[Rest of response...] 
```

### 高级配置

* * *

您已经在前面的示例中看到了一些用于 CORS 的标题，如`Access-Control-Allow-Origin`和`Access-Control-Allow-Methods`，但是还有更多标题用于更好的控制。下面是控制 CORS 的标题的完整列表。

#### 请求标题

| 标题名称 | 示例值 | 描述 | 用于飞行前请求 | 用于 CORS 请求 |
| --- | --- | --- | --- | --- |
| 起源 | https://www.mydomain.com | 浏览器页签的**协议、**和**端口**的组合打开 | 是 | 是 |
| 访问控制请求方法 | 邮政 | 对于预检请求，指定原始 CORS 请求将使用的方法 | 是 | 不 |
| 访问控制请求报头 | 授权，X-PING | 对于预检请求，用逗号分隔的列表指定原始 CORS 请求将发送的标题 | 是 | 不 |

#### 响应标题

| 标题名称 | 示例值 | 描述 | 用于飞行前请求 | 用于 CORS 请求 |
| --- | --- | --- | --- | --- |
| 访问控制允许来源 | https://www.mydomain.com | 服务器指定的该请求的允许来源。如果它不匹配`Origin`头并且不是*，浏览器将拒绝请求。如果指定了域，则协议组件是必需的，并且只能是一个域 | 是 | 是 |
| 访问控制允许凭证 | 真实的 | CORS 请求通常不包括防止 CSRF 攻击的 cookies。当设置为`true`时，请求可以使用/将包括 Cookies 等凭证。标题应该省略，以暗示`false`，这意味着 CORS 请求将不会返回到打开的选项卡。[不能与通配符](#credentials-with-wildcard)一起使用 | 是 | 是 |
| 访问控制公开标题 | 日期，X 设备 Id | 除了[默认标题](#default-expose-headers)之外，向浏览器选项卡公开的附加响应标题的白名单 | 不 | 是 |
| 访问控制最大年龄 | Six hundred | 缓存飞行前请求结果的值(秒)(即`Access-Control-Allow-Headers`和`Access-Control-Allow-Methods`标题中的数据)。Firefox 最长 24 小时，Chromium 最长 10 分钟。再高就没有效果了。 | 是 | 不 |
| 访问控制允许方法 | 获取、发布、上传、删除 | 可以是*来允许所有方法。可用于 CORS 请求的允许方法的逗号分隔白名单。 | 是 | 不 |
| 访问控制允许标题 | 授权，X-PING | 可以是*以允许任何标题。可用于 CORS 请求的允许标头的逗号分隔白名单。 | 是 | 不 |

在当前的浏览器中，*Access-Control-Allowed-Headers、Access-Control-Allow-Methods 和 Access-Control-Expose-Headers*上的通配符(*)尚未完全实现。
最好列出头或者方法。[见铬虫](https://bugs.chromium.org/p/chromium/issues/detail?id=615313)

> *什么是 Moesif？ [Moesif](https://www.moesif.com/) 是最先进的 API 分析和调试平台，被数以千计的平台用于快速调试 CORS 问题，并了解您的客户如何访问您的 API。Moesif 为您喜爱的语言提供了开源中间件和 SDK。*

### 常见陷阱

* * *

##### 1.对 Access-Control-Allow-Origin 使用*运算符。

CORS 是在试图保持安全的同时放松同源政策。使用*会禁用 CORS 的大多数安全规则。有些情况下通配符是可以的，例如集成到许多第三方网站的开放 API。

您可以通过将 API 放在不同的域中来提高安全性。
T3 例如，你的开放 API【https://api.mydomain.com】T4 可以用`Access-Control-Allow-Origin: *` 来响应，但是你的主网站的 API*https://www.mydomain.com/api*仍然用`Access-Control-Allow-Origin: https://www.mydomain.com`来响应

##### 2.为访问控制允许来源返回多个域。

可惜规范不允许`Access-Control-Allow-Origin: https://mydomain.com, https://www.mydomain.com`。服务器只能用一个域或*来响应，但是您可以利用`Origin`请求头。

##### 3.使用通配符选择，如 **.mydomain.com* 。

这不是 CORS 规范的一部分，通配符只能用于暗示允许所有域。

##### 4.不包括协议或非标准端口。

`Access-Control-Allow-Origin: mydomain.com`无效，因为未包含协议。

同样，除非服务器实际上运行在像`:80`这样的标准 HTTP 端口上，否则`Access-Control-Allow-Origin: http://localhost`会有问题。

##### 5.在变化响应标头中不包括来源

大多数 CORS 框架都是自动完成的，您必须向客户端指定服务器响应会根据请求来源而有所不同。

##### 6.未指定访问控制公开标题

如果不包含必需的标头，CORS 请求仍会通过，但未列入白名单的响应标头将从浏览器选项卡中隐藏。对于 CORS 请求，始终公开的默认响应头是:

*   缓存控制
*   内容语言
*   内容类型
*   期满
*   最后修改的
*   杂注

##### 7.当访问控制允许凭证设置为*真*时使用通配符

这是一个棘手的案件，抓住了许多人。如果响应有`Access-Control-Allow-Credentials: true`，那么通配符不能用在任何响应头上，比如 Access-Control-Allow-Origin。

### 表演

* * *

一篇关于减轻 CORS 绩效处罚的文章将很快发布。*