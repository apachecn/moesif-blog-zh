# Node.js 中的 GraphQL 堆栈:工具、库和框架的解释和比较

> 原文：<https://www.moesif.com/blog/technical/graphql/GraphQL-Stack-Nodejs-Tools-Libraries-Frameworks-Explained-and-Compared/>

## 介绍

GraphQL 通常被视为 [RESTful API](https://www.moesif.com/blog/api-guide/getting-started-with-apis/#core-principles-of-restful-api) 的替代品。虽然创建 GraphQL APIs 有明显的优势，但负面影响和转换成本可能会阻止许多组织从 RESTful 迁移。有很多文章描述了 GraphQL 的优点和缺点[。关键的优点是 GraphQL 让客户端确定它想要的数据，同时避免了对服务器的多次请求。](https://www.moesif.com/blog/api-guide/comparisons-of-api-architectural-styles/#comparing-graphql-and-restful)

GraphQL 是脸书提出的标准。实现 GraphQL API 的方法有很多，但是工具、库和框架的选择数量多得令人应接不暇。有许多很好的教程以一种自以为是的方式开始使用 GraphQL。这篇文章并不打算成为一套预选工具的入门指南，而是探索在新 GraphQL API 的设计和规划阶段将会出现的不同选项。

## 堆叠的层

在我们深入研究不同的选项之前，让我们来看一下使用 graphQL 系统建立生产环境的要素。

*   第一层是 HTTP 服务器，用于处理 GraphQL 服务器的传入 HTTP 请求。
*   第二层通常是核心层，是查询处理，它需要几个子部分:
    *   *模式定义*，静态时完成。
    *   *解析*和*解析*查询，即确定对每个查询采取什么动作或方法。
    *   *产生*和*合计*输出。
*   第三，您最终需要将其连接到数据库，即如何将 GraphQL 模式绑定到您的数据库模式。
*   第四，您需要全面考虑安全模型，并设置正确的授权和身份验证方案。

在客户端，有几个主要元素:

*   帮助您构建请求和处理查询返回值的工具和库。
*   工具和库如何通过将查询绑定到 UI 的组件来将数据注入到 UI 中。

让我们探索每一层。

## 构建和定义模式的工具

GraphQL 模式本身是语言不可知的，它是一种 DSL(领域特定语言),在教程中有很好的记录。这个 DSL 有很多方面，包括继承、静态类型、参数、操作符等等。所以学习它并有效地使用它可能需要一些时间。

GraphQL 查询通常如下所示:

```py
type  Person  {  name:  String!  age:  Int!  posts:  [Post!]!  } 
```

`graphql.js`官方图书馆是来自[Graphql.org](https://graphql.org)

您可以自己编写 DSL，加载它并让它被`buildSchema`函数解释。

```py
var { buildSchema } = require('graphql');

var schema = buildSchema(
  `
  type Person {
    name: String!
    age: Int!
    posts: [Post!]!
  }
  `
); 
```

`graphql.js`的 buildSchema 不是唯一的解析器，还有几个，比如 Apollo 的 [graphql-tools](https://github.com/apollographql/graphql-tools) 。graphql-tools 的好处是它有助于使调制更容易。

GraphQL 工具使您能够用 javascript 创建 GraphQL 模式的字符串表示，[您可以在这里](http://graphql.org/learn/schema/)阅读和了解它，并解析它以便它可以被其他工具使用。

如果您喜欢以编程方式构建模式，有 Javascript 库可以帮助您完成。

```py
import {
  graphql,
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString
} from 'graphql';

var schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
      hello: {
        type: GraphQLString,
        resolve() {
          return 'world';
        }
      }
    }
  })
}); 
```

如果您已经有了一个现有的项目，通常您可能已经定义了一个模式，比如 Mongoose 模式。有人构建工具来从您现有的模式生成 GraphQL 模式。有些相对较新，如[mongose-schema-to-graph QL](https://github.com/sarkistlt/mongoose-schema-to-graphql)，而[graffitti-mongose](https://github.com/RisingStack/graffiti-mongoose)已经过时。挑战在于，通常情况下，GraphQL 模式实际上比典型的 mongoose 模式更具表达性，因此，如果您进行直接移植，有时您可能无法充分利用 GraphQL 的特性。尽管如此，尝试将任何现有产品移植到 GraphQL 中仍然是一项艰巨任务。

| 图书馆 | 方法 | 利弊 |
| --- | --- | --- |
| *graphql.js* 与 *graphql-tools* | 编写架构 | 语言不可知 |
| *graphql.js* | 通过编程编写架构 | 在创建模式时更容易模块化和防止错误 |
| *mongose-schema-to-graph QL . js* | 从现有架构生成架构 | 自动生成的模式不够灵活，因为 GraphQL DSL 比 Mongo 模式定义更具表现力。 |

**注意**我个人认为使用`GraphQLSchema`、`GraphQLString`函数“以编程方式”生成您的模式似乎是不必要的，因为 GraphQL DSL 本身是非常干净的、声明性的和独立于语言的。没有理由再增加一层复杂性。此外，甚至尝试根据另一个数据库模式自动生成模式也是不必要的。如果您决定采用 GraphQL 作为您的应用程序的主干，那么仔细考虑所有事情并仔细设计模式是值得的，这是您整个应用程序的核心。

### 下决心者

解析器是一组对应于模式的数据元素的函数。在查询被验证之后，解析器在查询被遍历时被触发。解析器按照模式的规定完成所需的数据或变化(即更新数据库中的数据)。

因为解析器只是函数，所以除了与数据库交互之外，它们还可以执行任何操作。解析器函数通常如下所示:

```py
Query:  {  human(obj,  args,  context)  {  return  context.db.loadHumanByID(args.id).then(  userData  =>  new  Human(userData)  )  }  } 
```

解析器是您需要编写的大部分代码，包括任何需要的业务逻辑。打个比方，这些是 RESTful APIs 的控制器。

没有任何框架可以取代您自己的业务逻辑代码，您必须自己编写这些代码，但是如果大部分数据字段都直接解析为数据库字段，那么可能会有许多可以编写脚本的样板代码。

**注意**旋变器可以是同步或异步的。Node.js 的伟大之处在于它已经为[非阻塞 IO](https://nodejs.org/de/docs/guides/blocking-vs-non-blocking/) 而设计，利用这一点很重要。任何网络调用(比如对另一个 API 的调用或单独的数据库获取)都应该放在异步解析器中。

### 连接到数据层

对于许多常见的数据库，如 PostgresSQL 和 MongoDB，有[可用的驱动程序和库](https://www.moesif.com/blog/api-guide/development/api-resources-for-nodejs-developers/#databases-query-builder-and-orms)使查询更容易，帮助您管理模式、迁移等。

您不一定需要使用为 GraphQL 设计的数据库驱动程序。但是，如前所述，有一些工具可以帮助您基于数据库模式生成 GraphQL 模式。您的应用程序需求可能需要比生成器所能创建的更多的自定义模式。而没有复杂关系的非常简单的 CRUD 应用程序可以受益于自动模式生成。

[Prisma](https://www.prisma.io/) 走反路线。它允许您在 GraphQL 中创建模式，然后在您打算使用的数据库中生成相应的模式。它提供了一套工具，用于生成到数据库的链接，连接到这些数据库，并提供标准的预装代码，如分页。

[Dataloader](https://github.com/facebook/dataloader) 实用程序可用作应用程序数据提取层的一部分，通过批处理和缓存，为各种远程数据源(如数据库或 web 服务)提供简化且一致的 API。同样，尽管 facebook 说它是通用的，但它主要用于 GraphQL 应用程序。

### 连接到 HTTP 服务器

通常，除了简单地连接到 HTTP 服务之外，引擎实际上解析查询，并决定调用哪个解析器。它几乎相当于一个路由器，但它做得更多，通常引擎也处理这些事情:

1.  正在验证查询。
2.  正在解析。
3.  路由和触发解析器。
4.  放回解析器的结果。

其中最简单的可能是`express-graphql`，尽管顾名思义它是用于“express.js”的，但它实际上支持任何基于节点的 https 服务器，这些服务器支持`next`风格的[中间件](/blog/engineering/middleware/What-Is-HTTP-Middleware/)。

使用它非常简单:

```py
app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true,
})); 
```

其中`rootValue`是解析器的入口点。

基本上，您可以添加已经在使用的任何类型的快速中间件，比如身份验证或授权。

然而，有几个其他的引擎或框架提供了更多的特性。

[阿波罗服务器](https://www.apollographql.com/docs/apollo-server/)由流星背后的公司提供。Apollo Server 有一个更简单的接口，并且只支持一种传递查询的方式。Apollo Server 支持更多的 https 服务器(Express、Connect、哈比神、Koa、Restify ),并为每个服务器创建了单独的库。它是阿波罗(即流星)提供的工具套件的重要组成部分。它还排除了[【GraphiQl】](https://github.com/graphql/graphiql)服务器，该服务器更多的是一个开发工具，而不是生产所需的。

[Graph Cool](https://www.graph.cool/) 也是 GraphQL 的开源后端框架，非常强调[无服务器技术/架构](/blog/engineering/serverless/A-Primer-On-Serverless-Computing-AWS-Lambda-vs-Google-Cloud-Functions-vs-Azure-Functions/)。因为它是一个框架，它不仅仅是设置 HTTP 服务器。在文章的最后，我将总结考虑到堆栈多层的主要框架的选择。

### GraphQL 中的身份验证和安全性

所以您创建了一个 GraphQL API，但是现在您需要考虑几个安全问题，尤其是如果它可以从互联网访问的话。

对于传统的 REST API，我们编写了[一篇深入的文章，这里涵盖了设置](/blog/technical/restful-apis/Authorization-on-RESTful-APIs/)的一些关键考虑事项，GraphQL 也需要一些相同的考虑事项。关键区别在于，对于 RESTful API，您可以在路由级别设置安全性要求，但是对于 GraphQL API，它是在`/graphql`的单个端点，因此为了安全性，您需要与 GraphQL 引擎更紧密地耦合。

安全性的另一个考虑是 GraphQL 在构建查询时更加灵活，这使得某些人更有可能构建如此复杂的查询，以至于他们可以意外或恶意地 DDoS 您的服务，或者导致占用服务器资源的无限循环。

### 在客户端进行查询

构建获取数据的查询与 JSON 非常相似。例如，要获取一个 id 为 1000 的人，并选择(项目)、姓名和身高字段，可以编写如下的 GrapQL 查询:

```py
{  human(id:  "1000")  {  name  height  }  } 
```

这里有大量关于查询的教程。

有生成和构建查询的工具，因此您不必依赖 javascript 字符串。[查询生成器](https://github.com/opentable/graphql-query-generator) [Graphql 查询生成器](https://github.com/codemeasandwich/graphql-query-builder)

由于向服务器发送查询只是简单的任何 HTTP 请求，您可以使用任何流行的 https 客户端，如 [SuperAgent](https://visionmedia.github.io/superagent/) 或 [Fetch](https://github.com/andris9/fetch) 或 [Axios](https://github.com/axios/axios) 或 [Request](https://github.com/request/request) 。

虽然您可以手动发出查询请求，但由于在大多数用例中，查询的结果将显示给最终用户，即呈现在 UI 中。由于有许多前端 UI 框架，有许多选择来帮助将查询绑定到 UI，这些库可以消除手动执行查询的需要，还提供关键功能，如缓存数据和订阅数据更改。

**注意**graph QL 的一个伟大之处是[订阅模式](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)，它可以让用户界面体验比让客户端不断获取数据要好得多。然而，这对于像聊天这样的应用来说是有意义的，但是并不是在所有的场景中都有意义。例如，如果用户想看到一个数据表，如果数据不断地重新呈现，这可能会很烦人。您可以让用户触发数据的重新加载。

### 客户端:将查询绑定到 UI。

[React](https://reactjs.org/) 、 [Angular](https://angular.io/) 、 [Vue](https://vuejs.org/) 、 [Ember](https://www.emberjs.com/) 大概是当今最白杨的前端框架了。如果您是从零开始一个项目，在选择 GraphQL 客户端工具之前，先决定使用哪个 UI 框架可能是值得的。虽然根据 Github stars 的说法，react 目前似乎主导着市场份额。

Apollo 客户端为每个框架提供绑定器，也为 Android 和 iOS 提供绑定器。

[Relay](https://facebook.github.io/relay/) 虽然设计的非常通用，理论上可以用于任何 UI 框架，但是基本上是由创建 React 的同一个团队支持的。因此，这些库和工具与 react 紧密相关。

这两种技术还有更深入的比较，我将快速比较这两种技术。

| 技术 | 服务器端要求 | UI 框架兼容性 | 查询模式 | 订阅支持 | 贮藏 |
| --- | --- | --- | --- | --- | --- |
| *继电器* | 需要额外配置。这些是可用的工具。架构的每个节点都需要一个唯一的 id。 | 理论上任何框架，但在实践中反应 | 更具声明性，例如，对于每个组件，您描述您需要的数据，库将为您构造查询 | 极好的支持。 | 内置的。保证本地存储与服务器处于一致状态 |
| *阿波罗* | 兼容任何 GraphQL 服务器。 | 支持主要的 UI 框架。 | 直接构建查询。 | 需要[额外的库。](https://github.com/apollographql/subscriptions-transport-ws) | 缓存在大多数情况下都工作得很好，但是您可能需要手动执行 [updateQueries](https://www.apollographql.com/docs/react/advanced/caching) |

总之，如果 Apollo 客户端看起来更容易学习和开始，但从长远来看，Relay 是一个更复杂的系统，如果您的项目可能会变得非常大和复杂，也许值得投资。

## 样板或框架

GraphQL 的设计并不固执己见，但具有讽刺意味的是，大多数框架和样板文件都有点固执己见。

鉴于构建基于 GraphQL 的应用程序的技术堆栈的每一层都有如此多的技术选择，尤其是对于全新的应用程序，您可能会考虑所有决策都是在哪里做出的，您可以快速启动并运行，只有在绝对必要时才替换掉 swap out 技术。这就是框架和样板文件的用武之地。

*   阿波罗已经被提到过几次了。它本质上是一个完整的堆栈，将服务器和客户机之间的代码分开，你可以使用任何一个，而不必绑定到另一端(当然，如果你使用它们的整个堆栈，会更容易)。
*   [GraphCool](https://github.com/graphcool/graphcool-framework) 专注于服务器端。它试图坚持开放标准，包括基于 JWT 的认证、订阅等功能，甚至包括 Docker 之类的东西。
*   [甘松](https://github.com/spikenail/spikenail)也专注于服务器端，它开箱即可兼容中继，并且还支持 ES7。
*   [火神](https://github.com/VulcanJS/Vulcan)是全栈的，但重点是以流星为基础。选择 Meteor 本身是一个需要仔细考虑的重大决定，因为有很多优点和缺点。

样板文件和框架之间的界限有时会变得更窄，但是通常样板文件会为您做出更多的决定。

*   [节点 GraphQL 服务器](https://github.com/graphql-boilerplates/node-graphql-server)是相当小的样板文件。
*   [nodejs api starter](https://github.com/kriasoft/nodejs-api-starter) 是厨房水槽自带的样板文件，包括数据库(PostgreSQL)和 Docker。所以这是最广泛的，但对初学者来说可能是好的开始。
*   graphql-yoga 是另一个样板文件，主要基于 Apollo 栈，比如 express-apollo、subscriptions-transport-ws。

**注意**虽然选择一个框架似乎可以让决策变得容易，但有时你会因为不需要的东西而变得臃肿。你总是可以从一个最小的栈开始，然后随着你了解更多的东西而增加。

## 摘要

选择 GraphQL 本身作为新应用程序的主干可能是一项艰巨的任务，但是在您决定使用 GraphQL 之后，它的缺点是这是一种非固执己见的技术，我们必须做出如此多的库和工具选择。有时候感觉像是决策瘫痪。即使你通过采用样板文件或框架来避免做出大量决策，了解所有的考虑因素也是值得的。

尽管任何技术都没有灵丹妙药，但 GraphQL 也存在一些问题，比如它变得更加难以调试，尤其是当你有一个开放的公共 API 时，你不知道你的 API 将使用哪种查询模式。对生产 API 调用的分析变得更加重要。

**Moesif 相信 GraphQL 会一直存在，因此我们很高兴在我们的高级 API 分析平台**

[上推出原生 GraphQL 支持了解更多信息](https://www.moesif.com/features)