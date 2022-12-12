# 监控用 Apollo & Express 构建的 GraphQL APIs

> 原文：<https://www.moesif.com/blog/technical/graphql/Monitoring-GraphQL-APIs-Built-With-Apollo-and-Express/>

当我们想了解最新的功能和性能问题时，监控我们的 API 是必须的。

Keyur Doshi 已经写了一篇关于[监控用 Django 和 Graphene](/blog/technical/graphql/Monitoring-GraphQL-APIs-built-with-Django-and-Graphene/#) 构建的 GraphQL APIs 的文章，所以让我们深入 JavaScript 生态系统，看看需要什么来启动和运行它！

有许多 JavaScript GraphQL 实现，在本文中我们将讨论两个最突出的参与者:[Apollo-server](https://github.com/apollographql/apollo-server)&[express-graph QL](https://github.com/graphql/express-graphql)

本文的第一部分是关于如何为每个 GraphQL 实现设置 Moesif API 分析。

第二部分是关于如何使用 Moesif API 分析来保持我们的 API 游戏的领先地位！

## 设置 Apollo 服务器

Apollo server 提供了一个独立版本和多个框架的集成。我们将在这里讨论独立版本。

首先，我们和 NPM 一起建立了一个项目。

```py
$ mkdir apollo-example
$ cd apollo-example
$ npm init 
```

其次，我们安装所需的两个 NPM 软件包。

1.  `apollo-server`包括运行 GraphQL 服务器所需的一切
2.  `moesif-express`连接到 Moesif API 分析服务

```py
$ npm i apollo-server moesif-express 
```

接下来，我们创建一个保存所有设置代码的`index.js`。

```py
const { ApolloServer, gql } = require("apollo-server");
const moesif = require("moesif-express");

const applicationId = "<MOESIF_APPLICATION_ID>";
const typeDefs = ...;
const resolvers = ...;

new ApolloServer({ typeDefs, resolvers })
  .listen(8888)
  .then(({ server }) => {
    const moesifMiddleware = moesif({ applicationId });
    server.on("request", moesifMiddleware);
  }); 
```

代码的关键部分在最后。`ApolloServer`对象的`listen`方法返回一个承诺。

这个承诺让我们可以通过`server`对象访问底层的 HTTP-server。

这个服务器对象是`request`事件的事件发射器。

我们可以在`moesif`工厂函数的帮助下创建一个中间件函数，并将其用作`request`事件的事件监听器。

## 设置 Express GraphQL

Apollo server 也有一个 Express 集成，但是有一个替代的 GraphQL 服务器实现可以与 Express 一起工作，所以让我们看看如何设置它。

首先，我们创建一个项目。

```py
$ mkdir express-example
$ cd express-example
$ npm init 
```

其次，我们安装所需的软件包。

*   `express`是底层的 HTTP 服务器框架
*   `express-graphql`是 express 中间件，它使用 Node.js 的参考 GraphQL 实现
*   是连接我们和 Moesif API Analytics 的库

```py
const express = require('express');
const expressGraphql = require('express-graphql');
const moesif = require("moesif-express");

const applicationId = "<MOESIF_APPLICATION_ID>";
const schema = ...;

const moesifMiddleware = moesif({ applicationId });
const graphqlMiddleware = expressGraphql({ schema });

const app = express();
app.use(moesifMiddleware);
app.use("/graphql", graphqlMiddleware);

app.listen(8888); 
```

这里的过程相当简单。我们使用`moesif`工厂函数来获得一个可以与 Express `app`对象一起使用的中间件函数。

就代码而言，这是所有要做的工作。现在，每个到达服务器的请求，无论是 Express 还是 Apollo，都将被转发到 Moesif API 分析服务，并可以进一步检查。

## GraphQL 的 moesif api 分析

在我们用我们选择的框架建立了我们的 GraphQL 服务器之后，Moesif 中间件将把请求数据发送给 Moesif 服务。就 GraphQL 而言，这意味着 Moesif 服务现在知道我们的查询和变化。

> *如果我们使用 apollo-server 独立版本，GraphQL API 会在所有路由上应答。Moesif 期望 GraphQL 查询在`/graphql`路线上，所以我们必须使用这条路线。*

如果我们用浏览器登录到我们的 [Moesif 控制台](https://www.moesif.com)，我们会在 **API 分析**部分找到所有记录的事件。这包括但不限于:将 GraphQL 请求发送到`/graphql`路由。

为了只显示 GraphQL 查询请求，而不显示其他 HTTP 请求，比如加载 GraphiQL UI 的 HTML 或获取 GraphQL API 信息的自省查询，我们必须使用左侧的**过滤器**侧栏。

在 **API 过滤器**中，我们选择**请求- > URI 路由**，并将其设置为`/graphql`以去除非 GraphQL 请求。

现在我们只有 GraphQL 请求，我们可以应用 **GraphQL 查询过滤器**来关注我们想要监控的特定查询。

例如，我们可以通过选择`request.graphql.operation_name`作为 **GraphQL 查询过滤器**并将其设置为 **≠ IntrospectionQuery** 来消除`IntrospectionQuery`请求。

## 结论

用 Node.js 框架将 **Moesif API Analytics** 添加到我们的 GraphQL API 构建中只是几分钟的事情。我们必须通过 NPM 安装`moesif-express`,需要中间件，用我们的 Moesif 应用密钥配置它，我们就可以开始了。

设置完成后，我们所有的请求将被发送到 Moesif API 分析服务，并保存以备后用。

稍后，当我们的 API 启动并运行时，我们可以根据 GraphQL 查询的内容进行过滤、排序和分段，以深入了解我们的查询。

哪些是

*   最受欢迎？
*   最慢的？
*   最容易出错？