# 从 GraphQL 模式生成 REST APIs，教程

> 原文：<https://www.moesif.com/blog/technical/graphql/Generating-Rest-APIs-from-GraphQl-Schemas/>

关于技术选择的争论是一件大事，RESTFul 和 GraphQL 的争论就是一个例子。但有时也有方法把不同的方式结合起来。 [Sofa](https://sofa-api.com/) 就是试图将 REST 和 GraphQL 结合起来的其中一家。

## 为什么

REST 和 T2【graph QL 是构建基于 HTTP 的 API 的不同方法，但是对于哪种方法最适合某个问题，不同的开发者有不同的看法。

我不知道其中一种是否优于另一种，但是如果我们只为客户提供一种类型的 API，有时它会成为一个障碍。

Sofa 允许我们从 GraphQL 模式自动生成 REST API，这样我们就不必自己实现两个 API。

## 什么

[Sofa](https://sofa-api.com/) 是一个 [Node.js](https://nodejs.org/en/) 包，它采用一个 [GraphQL 模式](https://graphql.org/learn/schema/)定义，并从中创建一个 [Express 中间件](https://expressjs.com/en/guide/using-middleware.html)。这个中间件提供了 REST-API 端点。

## 怎么

让我们在 Sofa、GraphQL 和 Express 的帮助下建立一个小的 API 服务器，然后尝试从它创建和读取消息。

### 履行

首先，我们使用 npm 初始化一个新的 Node.js 项目并安装软件包。

```py
$ mkdir api-server
$ cd api-server
$ npm init
$ npm i express express-graphql graphql-tools sofa-api 
```

接下来，我们在一个新的`schema.gql`文件中创建一个 GraphQl 模式。

```py
type  Message  {  id:  ID!  text:  String!  }  type  Query  {  message(id:  ID!):  Message  messages:  [Message!]  }  type  Mutation  {  writeMessage(title:  String!):  Message  }  schema  {  query:  Query  mutation:  Mutation  } 
```

我们只有一个`Message`类型和相应的查询和突变类型。

我们还需要为每个查询和变异定义解析器。让我们为此创建一个新的`resolver.js`文件。

```py
const messages = [];

module.exports = {
  Query: {
    message: (_, { id }) => messages[id],
    messages: () => messages
  },
  Mutation: {
    writeMessage: (_, { text }) => {
      const message = { id: messages.length, text };
      messages.push(message);
      return message;
    }
  }
}; 
```

我们使用`message`数组作为数据存储，使用它的`length`为我们的消息生成 id。这就足够了，因为我们没有可能搞乱 IDs 的删除突变。

现在，我们用 GraphQL 和 Sofa 连接模式。我们在一个新的`index.js`文件中这样做。

```py
const fs = require("fs");
const { makeExecutableSchema } = require("graphql-tools");
const express = require("express");
const graphql = require("express-graphql");
const sofa = require("sofa-api").default;

const typeDefs = fs.readFileSync("./typeDefs.gql", "utf8");
const resolvers = require("./resolvers");

const schema = makeExecutableSchema({ typeDefs, resolvers });

const server = express();

server.use("/graphql", graphql({ schema, graphiql: true }));
server.use("/rest", sofa({ schema }));

server.listen("9999"); 
```

让我们看一下这份文件的关键部分。

首先，我们从磁盘读取 GraphQL 类型定义，并需要相应的解析器。

然后我们使用`graphql-tools`从解析器和类型定义中创建一个模式。

之后，我们创建一个 Express 服务器，并为两个端点`/graphql`和`/rest`设置中间件。

我们还启用了默认的 GraphQL UI`graphiql`，这样我们就可以在浏览器中测试 graph QL 端点。

最后，我们在端口`9999`上启动一个服务器。

### 使用

让我们启动服务器。

```py
$ node . 
```

并在浏览器中打开 Graphiql:`http://localhost:9999/graphql`

在左边的文本区域，我们现在可以输入查询和突变。让我们从一个突变开始，因为我们的商店目前是空的。

```py
mutation  {  writeMessage(text:  "my messsage")  {  id  text  }  } 
```

我们可以用 *Ctrl+Enter* 将消息发送到服务器。

答案应该如下:

```py
{  "data":  {  "writeMessage":  {  "id":  "0",  "text":  "my messsage"  }  }  } 
```

让我们用不同的文本再做一次，看看 id 是否像预期的那样增加了:

```py
mutation  {  writeMessage(text:  "another message")  {  id  text  }  } 
```

返回的 JSON 应该反映新的`text`和`id`。

让我们试试这些查询。

首先加载所有邮件:

```py
query  {  messages  {  id  text  }  } 
```

这将为 JSON 提供我们所有消息的列表。

只缺少一个查询:`message`

这一次我们需要提供想要加载的消息的 ID。

```py
query  {  message(id:  1)  {  id  text  }  } 
```

我们现在知道 GraphQL API 按预期工作，让我们试试 REST 版本。

变异通过 POST 请求工作，这可以用 cURL 来完成。

```py
$ curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"text":"REST message"}' \
  http://localhost:9999/rest/add-message 
```

这些查询处理 GET 请求，因此我们可以在浏览器中使用一个简单的链接。

列出所有邮件:

`http://localhost:9999/rest/messages`

收到一条消息:

`http://localhost:9999/rest/message/2`

## 结论

沙发当然是一个有趣的解决方案，可以解决休息的问题，休息的问题，休息的问题，休息的问题，T2 的问题，图表的问题。它利用了这样一个事实，即 GraphQL 需要一个标准化的模式和解析器来将许多概念映射回 REST APIs。

这允许 API 创建者为他们的客户提供不同的 API 类型，而只需要维护一个代码库。