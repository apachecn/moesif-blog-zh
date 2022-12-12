# 在 NodeJS 栈中为 REST APIs 选择库和框架

> 原文：<https://www.moesif.com/blog/nodejs/frameworks/Choosing-the-Libraries-and-Frameworks-for-REST-APIs-in-the-NodeJS-Stack/>

有很多在 NodeJs 上构建 RESTful APIs 的教程，但这些教程通常已经选择了库或框架。本指南旨在提供各种库和设计决策的比较。

## 介绍

如果将 RESTful APIs 归结为通过 HTTPs 的请求，并通过 JSON 进行通信(大多数情况下)，*在 NodeJS 中创建*API 会非常简单。

```py
var express = require('express');
var app = express();

app.get('/greeting', function (req, res) {
  res.json({ hello: 'world' });
}); 
```

我们需要了解帮助我们构建 API 的每一层的设计原则和技术，然后我们可以回过头来选择对我们有帮助的工具和库。

## REST 设计原则概述

让我们回顾一下什么是好的 RESTful API 设计。您应该遵循的一些核心原则:

*   语义上有意义:
    *   URI 端点应该是资源(即名词)和人类可读的，如`/items`或`/users`。功能或操作不是资源。
    *   HTTP 动词(`GET`、`POST`、`PUT`、`DELETE`)表示客户端可以对资源采取的动作。
    *   HTTP 响应代码(例如，`201`(已创建)、`404`(未找到)和`401`(未授权))表示发生了什么。
    *   关系可以表示为子资源。同样，它使东西可读。例如，`/authors/{id}/posts` endpoint 将代表特定作者的帖子。
*   无状态:服务器不需要代表客户端保存状态。这使得扩展 REST APIs 变得容易，因为一个新的请求可以访问负载均衡器后面的任何虚拟机。在请求之间维护临时游标或存储临时文件不是无状态的。
*   优雅地处理重复来电:
    *   可缓存性:GET 和 HEAD 方法通常被缓存。当考虑可变性时，您的 API 应该考虑这一点。
    *   幂等性:对于改变一个资源的状态的动作，“PUT”和“DELETE”，对于相同数据的重复调用会产生相同的结果。
    *   Safe: GET、HEAD、OPTIONS 和 TRACE，对于 readonly，不改变状态。

当然，在设计上有许多自以为是的建议，比如命名资源的最佳方式(camel case vs . snake _ case vs . spinal-case，[复数 vs .单数](https://stackoverflow.com/questions/6845772/rest-uri-convention-singular-or-plural-name-of-resource-while-creating-it))，设置 JSON 模式名称的最佳方式([信封](http://www.thedevpiece.com/the-basic-you-should-know-before-building-your-rest-api/#envelopyourresponse) vs [无信封](https://danbovey.uk/blog/modern-http-practices))，符合 HATEOAS，如何最好地[处理过滤器&分页](/blog/technical/api-design/REST-API-Design-Filtering-Sorting-and-Pagination/)等等。在做出选择之前，请务必阅读并理解它们，这些设计决策应该在您做出任何技术决策之前做出。

## 设置 Restful API 的主要技术层。

*   HTTP 服务器和路由器。
*   数据
*   安全性
*   代理人

## HTTP 服务器和路由器

NodeJS 自带一个 [Http 服务器](https://nodejs.org/api/http.html)。

```py
var http = require('http');
http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write(req.url);
    res.end();
}).listen(8080); 
```

这个默认服务器不处理我们用来定义端点的路由。我们希望能够将`GET /users`路由到一个函数，将`GET /items`路由到另一个函数。由于 HTTP 动词、路径和参数的许多组合，路由可能会变得复杂，但幸运的是，除了用于构建 REST APIs 的其他关键中间件之外，我们还有许多可以处理路由的框架。

*   express 是目前最流行的构建 REST APIs 的框架。这也是 Moesif 发布的第一个框架，也是我们最受欢迎的集成。Express 相信组合和代码胜于配置。您的路线直接编码在业务逻辑的旁边。没有集中的“routes.conf”或类似的文件。尽管这个框架很老，但它依靠可选的中间件来保持精简。因此，如果您正在构建一个 REST API，您不会像启用 HTML 模板引擎和 cookie 解析器那样额外膨胀。下面是一个快速路线的例子。

```py
router.get('/:id', function (req, res) {
  // ... where id is parameterized.
}); 
```

*   [Koa](https://github.com/koajs/koa) Koa 被列出，尽管它不支持路由。但是，在某些情况下，它是 Express 的替代选项。，但人们总是把它列为 Express 的替代，你可以单独添加 [Koa 路由器](https://github.com/alexmingoia/koa-router)。最初创建 Koa 是为了避开*回调地狱*，这在 express 中很容易发生。Koa 从 [co](https://github.com/tj/co) 开始处理 ES2016 之前的异步调用，支持`async`和`await`。

*   hapi 是由 WalmartLabs 创造的。它遵循配置胜于代码的理念。它从节点的 HTTP 模块中提供了比其他模块更高的抽象层次。

代码如下所示:

```py
server.route({
    method: 'GET',
    path: '/{name}',
    handler: function (request, reply) {
          // ... where name is parameterized
    }
}); 
```

*   restify 是专为 RESTful API 设计的，所以它删除了 express 的一些功能，如 HTML 模板和视图，但增加了 API 所需的其他内置功能，如速率限制和 [SPDY 支持](https://en.wikipedia.org/wiki/SPDY)。Restify 的语法和 express 非常相似。

我们总是可以添加中间件来为每个框架添加功能和特性。点击此处查看关于中间件[的深入文章。](/blog/engineering/middleware/What-Is-HTTP-Middleware/)

## JSON 反序列化

Javascript 原生支持`JSON.parse(my_json_string)`或`JSON.stringify(my_javascript_object)`。然而，如果这是自动的和幕后的，生活会更容易。

*   如果使用 Express，可以使用默认的[主体解析器](https://github.com/expressjs/body-parser)中间件。它支持多种类型的文本和二进制数据，当然还有 JSON，4 种最常用的格式 RESTful APIs。

```py
var express = require('express')
var bodyParser = require('body-parser')
var app = express()
// parse application/json
app.use(bodyParser.json()) 
```

## 数据库

一旦您选择了一个数据库，您选择的库将主要由与该数据库兼容的内容驱动。节点。JS 生态系统包括许多不同数据库的驱动程序，从 [mongojs](https://github.com/mafintosh/mongojs) ，到 [mysql](https://github.com/mysqljs/mysql) ，以及 [PostgreSQL](https://github.com/brianc/node-postgres) 。

虽然每个数据库都有 NodeJS 中的驱动程序，但是您可能希望考虑使用 ORM(对象关系映射),而不管是 SQL 还是非 SQL 技术。ORM 在 Enterprise Java 和 C#世界中已经使用了很长时间，Node.js 也没有什么不同，即使在 Node.js 和 MongoDb 中有原生的 JSON 支持。ORM 允许您在代码中将数据库模式建模为对象，然后 ORM 管理从实际数据库中检索/更新数据，并将它们映射到代码中的域对象。对于需要模式迁移的数据库，ORM 可以简化这个过程。

Node.js 生态系统中的一些常见 ORM:

*   [mongose](http://mongoosejs.com/):本质上是 MongoDB 的 ORM。鉴于平均堆栈的流行，这是非常受欢迎的。
*   Sequelizejs :基于承诺，适用于 PostgreSQL、MySQL、SQLite 和 MSSQL。
*   [orm](https://github.com/dresende/node-orm2) :创意命名。
*   [书架](https://github.com/bookshelf/bookshelf):构建在 [Knex.js](http://knexjs.org/) 之上，一个查询构建器。
*   [waterline](http://waterlinejs.org/) : Waterline 使用适配器的概念将一组预定义的方法翻译成查询。它还支持各种 SQL 和非 SQL 数据库。

## 安全性

我们建议回顾一下为 RESTful API 构建认证和授权的[步骤，权衡您的认证架构中的各种选项，例如比较 JWT(JSON Web 令牌)与不透明令牌，比较 cookies 与 HTTP 头。](/blog/technical/restful-apis/Authorization-on-RESTful-APIs/)

### JWT 代币资源

[JWT 令牌](https://en.wikipedia.org/wiki/JSON_Web_Token)实际上是一个完整的 JSON 对象，已经过 base64 编码，然后使用对称共享密钥或使用[公钥/私钥](https://en.wikipedia.org/wiki/Public-key_cryptography)对进行签名。如果您决定将 JWT 作为您的身份验证令牌，有几个库可以帮助您。

[jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 是一个用于签署 jwt 的通用实用程序库。

要为用户生成令牌，请执行以下操作:

```py
var jwt = require('jsonwebtoken');
jwt.sign({
  exp: Math.floor(Date.now() / 1000) + (60 * 60),
  data: 'foobar'
}, 'secret'); 
```

令牌可以包含任何 JSON，比如 user_id 和允许的范围或角色。

```py
jwt.sign({
  exp: Math.floor(Date.now() / 1000) + (60 * 60),
  admin: true
}, 'secret'); 
```

因为令牌是使用您的秘密签名的，所以您可以保证令牌没有被恶意方篡改或修改。

尽管您也可以使用 *jsonwebtoken* 库来解码和验证您接收到的 JWT，但是有另一个库使它更容易与 HTTP 服务器和路由器集成。

express-jwt 是 Auth0 提供的一个开源库，它可以与任何遵循 express like 中间件约定的标准路由器/服务器一起工作。这确保您的令牌已经过检查，并且 base64 解码供您的业务逻辑使用。

使用它非常简单:

```py
var jwtMiddleware = require('express-jwt');

app.get('/protected',
  jwtMiddleware({secret: 'your secret'}),
  function(req, res) {
    if (!req.user.admin) return res.sendStatus(401);
    res.sendStatus(200);
  }); 
```

您用您的验证密钥初始化中间件，这使中间件能够检查令牌是否由您的秘密签名。base64 解码字段填充在`req.user`中。

请注意，在将任何代码投入生产使用之前，请审核您的代码。这些例子非常简单，需要做更多的工作才能锁定并投入生产。

### 不透明代币

如果您选择使用不透明令牌策略，则授权信息(即允许用户访问的内容没有编码在令牌中)，因此需要在 Redis 等数据库中进行查找。

所以这里你需要两项技术。一个处理逻辑的中间件，一个基于 O(1)哈希的数据库，用于查找权限和其他数据。O(1)非常重要，因为每次 API 调用都要调用它。例如，redis 将是一个不错的选择。

至于要使用的中间件，最流行的是 [passport.js](http://www.passportjs.org/) ，因为它支持许多策略(包括 JWT)。然而，您最有可能对 REST APIs 使用*载体*策略。这里的授权策略是使用`passport.js`来确定用户是谁(例如 *userId* )，然后在 Redis 中查找您授予该 userId 的权限，然后决定是否可以调用 API。

### 限速

速率限制对于防止 DDoS 攻击或雄心勃勃的自由层用户非常重要。一种与语言无关的方法是使用 API 网关，如 [Tyk](https://tyk.io/) 或 [Apigee](https://apigee.com/api-management/) 来处理您的 API 管理需求。也有中间件为你处理这些，比如[快速限速](https://github.com/nfriedly/express-rate-limit)

## 反向代理

我们创建的许多 API 将被放在反向代理后面。反向代理可以处理到许多服务和这些服务的版本的高级路由。反向代理还可以处理安全性、日志记录和缓存原因。

Nginx 和 [HaProxy](https://www.haproxy.org/) 是两个非常流行的高性能 HTTP 代理，但是需要大量的配置工作。Node.js 生态系统有一个非常简单但性能不错的代理，叫做 [node-http-proxy](https://github.com/nodejitsu/node-http-proxy) ，可以直接在 Node.js 应用中运行。

## 附加选项

### 自动生成 API

即使使用路由框架，编写所有的路由回调仍然需要大量的手工工作。如果您的应用程序主要需要 CRUD(创建、读取、更新、删除)操作，而没有大量的自定义逻辑，那么您可以研究更高级别的样板框架，它们可以位于您的数据库前面，并直接基于数据模型生成端点。

*   [loopback](https://loopback.io/) 由 IBM 子公司 StrongLoop 提供支持。它被定位为一个成熟的框架，允许您快速创建主要由数据库驱动的 API。有许多工具可以不费吹灰之力地插入 loopback，例如:Swagger、ORM/ODM(杂耍)和访问级别控制。它采用了*约定优于配置*的理念，并基于您的模式生成路由。请记住，如果你开始编写不同于常规的东西，框架可能会受到限制。

*   Nodal 是一个固执己见的框架，它能为你做很多决定，让你快速开始。生成的路线基于您定义的控制器类别。

*   假设您构建的每个 API 都有需要支持 CRUD 操作的数据对象集合。它还提供了用于创建 API 的 web UI 界面。

*   生成器和样板文件:基于 Yeoman 的生成器很少，它们可以自动设置 API。

### Web 套接字

有一些框架允许你使用 [WebSockets](https://en.wikipedia.org/wiki/WebSocket) 而不是 HTTP 来服务你的 API。这对于某些实时应用程序(如聊天应用程序和游戏)非常有用。请记住，web sockets 仍然比典型的 HTTP REST API 更难扩展，并且工具更少。在某些方面，web sockets 是反 REST 的。

*   [风帆](https://sailsjs.com/)
*   [特性 jss](https://feathersjs.com/)

## 结论

NodeJS 生态系统可能是更灵活的生态系统之一，并成为构建由流行框架(如 Express 和 React)驱动的 API 的最大生态系统。从而比大多数其他生态系统(如 Ruby 或 Python)允许更多选择。根据我们的使用数据，Node.js 是构建 REST API[最流行的](/blog/engineering/api-analytics/Popular-Languages-For-Building-RESTful-APIs-Based-On-Our-Usage-Data/)

Moesif 是最先进的 API 分析平台。成千上万的平台公司利用 Moesif 进行调试、监控和发现见解。

[了解更多](https://www.moesif.com?utm_source=blog)