# 如何跟踪每个合作伙伴或客户的 API 键进行的 API 调用的数量

> 原文：<https://www.moesif.com/blog/api-analytics/usage-tracking/How-To-Track-Number-of-API-Calls-Made-By-Each-Partner-or-Customer-API-Key/>

## 为什么要跟踪 API 调用的数量？

如果你是一家 B2B 公司，你很有可能想要向组织外的团队公开你的一些内部 API。这些可能是为你的 API 付费的客户，或者你的合作伙伴。无论哪种方式，当 API 公开时，跟踪每个合作伙伴或客户进行了多少次 API 调用是至关重要的。通过这样做，您可以发现是一个客户消耗了您的大部分 API 资源，还是您的 API 没有被您的客户采用。

## 处理过期的 API 密钥

跟踪每个合作伙伴的 API 调用数量可以通过两种机制来完成，这取决于您的 API 密钥是如何发布的。如果您的 API 密钥是永不过期的永久令牌，并且每个客户最多可以拥有一个令牌，那么跟踪就变得相对简单了，每当您看到针对特定令牌的 API 调用时，您可以增加内存中的计数器。这些计数器可以通过快速键值存储(如 Redis)保存到磁盘中。计数器可以在每月的第一天重置。

但是，这种方案在以下情况下可能会有问题:

*   您的 API 令牌会在几小时或几分钟后过期
*   您的客户可以创建多个 API 密钥，例如，如果您允许更改令牌的范围

如果是这样，你的追踪就变得更复杂了。最好的处理方式是通过一个*别名表*

### 会话别名表

会话别名表将跟踪客户创建的所有未完成令牌，并将这些令牌映射到单个 *user_id* 。您可以在 Redis 中保留一个别名表，其中键是 API 键，值是 user_id。这样，当您的 API 服务器收到新的 API 请求时，它可以通过 API 键查找 user_id。

### 第一步。捕获 API API 数据

您将需要为您的 API 框架创建中间件，该中间件可以在 API 请求进入并调用您的服务时捕获和处理这些请求。如果你正在寻找一个托管服务，Moesif 已经为许多流行的框架提供了预构建的中间件，如 [Moesif Express SDK](https://github.com/Moesif/moesif-express) 否则，如果你正在尝试推出自己的分析服务，可以在 GitHub 上找到中间件源代码。中间件应该满足某些要求:

1.  事件处理是在核心业务逻辑的带外异步完成的
2.  敏感数据在发送到下游分析服务之前会被清除

#### Node.js 的一个例子

让我们看一个例子，创建一个构建在 Node.js 上的 API，并添加中间件来跟踪每个合作伙伴的请求数量。首先，我们创建一个 app.ks，导入 Express 框架。我们将设置一个返回一些基本 JSON 的端点`/api`。

因为，API 通常需要通过某种类型的 API 密钥或令牌进行认证，让我们添加`express-jwt`库。

```py
const express = require('express');
const jwt = require('express-jwt');
const app = express();

// Add JSON Web Token Library for authentication
const jwtMiddleware = jwt({
  secret: Buffer.from('12345'),
});

app.use(jwtMiddleware);
app.get('/api', (req, res) => {
    res.json({
      my_object: {
        my_message: 'Hello World!'
      }
    }));
});

app.listen(8080, () => console.log('API listening on port 8080!')); 
```

#### 添加日志中间件

接下来，让我们创建一个简单的中间件，它可以捕获关于谁在调用 API 的特定信息

```py
const moesifExpress = require('moesif-express');

const moesifOptions = {

  applicationId: process.env.MOESIF_APPLICATION_ID, // You should set your APPLICATION_ID as an environment variable

  identifyUser: function (req, res) {
    // The subject of the JWT (i.e. the user_id) is automatically saved to req.user via JWT middleware
    if (req.user) {
      return req.user.id;
    }
    return undefined;
  },

  identifyCompany: function (req, res) {
    // Our company id is in the form of the URL via GET /accounts/1234/items
    const extracted = req.path.match(/accounts\/\d+[\/-]/);
    const companyId = extracted && extracted[0];
    return companyId;
  },

  getSessionToken: function (req, res) {
    return req.headers['Authorization'];
  }
}; 
```

一旦我们设置了获取所需信息的选项，我们就可以添加 moesif 日志中间件:

```py
const express = require('express');
const jwt = require('express-jwt');
const app = express();

// Add JSON Web Token Library for authentication
const jwt = jwt({
  secret: Buffer.from('my_super_secret'),
});

app.use(jwtMiddleware);
app.use(moesifMiddleware);

app.get('/accounts/:id/message', (req, res) => {
    res.json({
      my_object: {
        my_message: 'Hello World!'
      }
    }));
});

app.listen(8080, () => console.log('API listening on port 8080!')); 
```

### 第二步。启动您的 API 并创建流量

一旦中间件连接起来。启动节点服务器。

```py
node app.js 
```

我们需要生成一个*假的*合作伙伴 API 令牌。由于我们使用 JWT，我们可以通过 JWT.io 快速生成一个。为了使事情更容易，请查看我预先填充的 [jwt.io](https://jwt.io/#debugger-io?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1MTMiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE1MTYyMzkwMjJ9.FPN2Bj5b6OUzeTJ7lrNQPWUkZnnBnhMFYeyMjJ_-tN0) 链接。

JWT 有几个关键要素:

*   JWT 秘密`my_super_secret`被设置为我们在 Node.js 应用中使用的相同秘密。当然，对于生产，您应该生成一个安全的 256 位密码。
*   有效负载包含一些我们的中间件将记录的关键信息:

```py
{  "sub":  "513",  "name":  "John Doe",  "iat":  1516239022  } 
```

`sub`字段是*主题*的缩写，是认证术语中 user_id 的技术名称。我们还看到了另外两个字段，用户名和字段`iat`，它是在发布的*的缩写。我们可以向 JWT 有效负载添加任意字段，比如向 JWT 添加 tenant_id，以满足任何要求。*

下面是我创建的 JWT.io(可以从`jwt.io`网站复制。)

```py
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1MTMiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE1MTYyMzkwMjJ9.FPN2Bj5b6OUzeTJ7lrNQPWUkZnnBnhMFYeyMjJ_-tN0 
```

接下来，使用新创建的合作伙伴 API 令牌向您的 API 发出一些 API 请求

```py
curl -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1MTMiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE1MTYyMzkwMjJ9.FPN2Bj5b6OUzeTJ7lrNQPWUkZnnBnhMFYeyMjJ_-tN0' http://localhost:8080/accounts/abcdef/message 
```

注意对于 JSON Web 令牌，它们是不记名令牌，这意味着任何持有令牌的人都可以访问 API。有关 API 认证和授权的更多信息，请查看这篇博文。

改变`sub`字段，如 *1234* 和 *2560* 时，用几个额外的令牌重复`curl`命令。

### 第二步。可视化 API 数据

现在，您已经进行了一些 API 调用，登录到您的[www.moesif.com](https://www.moesif.com)并点击 *API 分析- >事件流*。您应该可以在 Moesif 中看到通过 curl 命令创建的 API 调用。现在，转到 *API 分析- >分段*，选择按*用户 Id* 分组。您应该会看到如下图表:

![API Usage by Customer](img/152f48983ae821f7fc22c38a14908419.png)

## 结束语

使用中间件和 Moesif，您看到了如何跟踪进行最多 API 调用的客户。接下来，您可以跟踪他们使用了哪些 API 路线，以及他们在您的 API 平台上坚持了多长时间。