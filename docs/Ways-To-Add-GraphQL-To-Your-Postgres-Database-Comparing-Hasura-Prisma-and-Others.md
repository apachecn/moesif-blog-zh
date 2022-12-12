# 将 GraphQL API 添加到 Postgres 数据库的六种简单方法，比较 Hasura、Prisma 和其他方法

> 原文：<https://www.moesif.com/blog/graphql/technical/Ways-To-Add-GraphQL-To-Your-Postgres-Database-Comparing-Hasura-Prisma-and-Others/>

[PostgreSQL](https://www.postgresql.org/) 是世界上最流行的开源 SQL 数据库之一， [GraphQL](https://graphql.org/) 是一个日益流行的新兴 API 规范。

将久经考验且广为人知的 PostgreSQL 与 GraphQL 带来的新的 API 创建方式相集成不是很好吗？

在本文中，我们将讨论六个不同的项目，它们试图将 SQL 与 GraphQL 世界融合在一起。其中一些甚至允许基于数据库结构自动创建模式。

## 以数据库为中心的方法

以数据库为中心的解决方案试图消除大多数配置和模式设置。他们将检查我们的数据库的外观，并为我们提供类型和端点。

由于他们知道数据库是如何构造的，他们可以为我们生成高性能的解析器，所以我们不会遇到 N+1 查询问题。

### 1.哈苏拉

> Postgres 上的即时实时图表

哈苏拉可能是目前球场上最令人兴奋的球员。是类固醇 PHPMyAdmin。

它运行在 Docker 容器中，作为数据库前面的服务器，并为我们提供了一个 DB 和 API 的管理 UI，有点像 PHPMyAdmin 所做的。

它自带认证和授权特性，甚至可以与其他认证提供者集成。

如果没有托管服务，它会变得越来越糟糕，所以如果你不喜欢像 AWS AppSync 这样的东西，但你喜欢 T2 的一些东西，那么就选择 Hasura 吧。

它是在 Apache 2.0 许可下开源的，大部分是用 Haskell 编写的。

此外，创作者提供有偿支持计划。

### 2.[海报](https://www.graphile.org/postgraphile/)

> 通过将 PostgreSQL 指向您现有的 PostgreSQL 数据库，立即启动 GraphQL API 服务器

PostgreSQL 类似于 Hasura，它允许从 PostgreSQL 模式生成 GraphQL API，并作为服务器在我们的数据库前面运行。它只是进入了一个不同的方向来实现这个目标。

它不使用 Docker 容器，并试图尽可能多地重用 Postgres 功能。例如用户管理、通过 RLS 的授权和可自动更新的视图。

所以它非常适合在设置和配置这种数据库方面有多年经验的 Postgres 专业人员。他们可以使用他们所有的技能，让 Postgraphile 为他们做 API 工作。

Postgraphiles 也主要关注 CLI 来完成所有的交互，这可能是数据库管理员更喜欢的。

这是一个在麻省理工学院许可下发布的开源产品，主要用 TypeSCript 编写。

创作者还提供了一个付费的专业版，提供额外的功能和付费支持。

### 3.[Prisma](https://www.prisma.io/)&[graph QL Nexus](https://nexus.js.org/)

*【2021-05-02 更新】*

> Prisma 取代传统 ORM

> [Nexus 是]声明性的、代码优先的 GraphQL 模式，用于 JavaScript/TypeScript

Prisma 是一套开源的数据库工具，用于数据访问(类似于传统的 ORM)、迁移和数据管理。

开发人员可以使用 SDL 的一个子集来定义一个数据模型，Prisma 将该模型映射到他们的数据库，从而简化了数据库迁移的过程。

Prisma 然后生成一个类型安全的数据库客户端，可以在您的 API 服务器内部使用。当与 GraphQL Nexus(一个代码优先的 GraphQL 模式构造库)和`nexus-prisma`集成一起使用时，开发人员可以利用自动生成的 CRUD 操作来处理数据库模型。这使得只需几行代码就可以构建完整的 GraphQL CRUD API！

然后可以根据应用程序的用例定制和扩展生成的 API。

它是在 Apache 2.0 下授权的开源软件，用 Scala 编写。

Prisma 还提供付费企业版。

> *什么是 Moesif？ [Moesif](https://www.moesif.com/features/graphql-analytics) 是最先进的 REST 和 GraphQL 分析平台，被数以千计的平台用来衡量您的查询执行情况，并了解您最忠实的客户正在使用您的 API 做什么。*

## 以模式为中心的方法

接下来的三个解决方案在方法上更经典一些，它们需要手动创建模式，没有太多额外的东西，但是它们试图帮助解决常见的问题。

它们还需要使用 Node.js，因为它们是常规的 Node.js 库。

### 4. [Node.js API 初学者工具包](https://github.com/kriasoft/nodejs-api-starter)

> 使用 Node.js 和 GraphQL 创作数据 API 后端的样板文件和工具

Node.js API 初学者工具包可能是启动和运行 GraphQL API 的最基本的方法。

这是一个样板项目，附带了连接到 Postgres DB 所需的所有 Node.js 库，运行 HTTP 服务器并创建 GraphQL 模式和解析器。

对于需要完全控制 API 服务器每个部分的绿地项目来说，这是一个良好的开端。

没有付费支持，只有免费的社区支持。

它是开源的，有麻省理工学院的许可，用 JavaScript 编写。

### 5. [graphql-sequelize](https://github.com/mickhansen/graphql-sequelize)

> 通过 Sequelize 为 MySQL 和 Postgres 提供 GraphQL 和 Relay

这是一个从序列模型中生成 GraphQL 解析函数的库。我们仍然需要创建我们的模式，但是不必再担心解析器了。

对于那些已经掌握了大量 Sequelize 知识并且不想放弃它的人来说，这是一个正确的解决方案。

这是一个用 JavaScript 编写的开源库，在麻省理工学院许可下发布。

### 6.[加入怪物](https://join-monster.readthedocs.io/en/latest/)

> 它是一个函数，接受 GraphQL 查询，并动态地将 GraphQL 转换为 SQL，以便在解析之前进行高效、批量的数据检索。

Join Monster 通过提供一种使用 Postgres 的全部 SQL 功能的方法来帮助 GraphQL 模式建模。它允许告诉每个 GraphQL 类型它属于哪个表，因此它可以从每个 GraphQL 查询生成最佳 SQL 查询。

对于那些想自己构建大部分 API 服务器但不想直接使用 SQL 的人来说，这是一个极好的解决方案。

Join Monster 是在麻省理工学院许可下开源发布的。是用 JavaScript 写的。

## 结论

有许多不同的解决方案可以通过 GraphQL API 访问 Postgres 数据库。各有利弊。

如果我们不能全押在云解决方案上，这里列出的系统允许我们选择我们希望在 API 中“手握”多少东西，以及我们希望自己做多少事情。

有了 Hasura 和 Postgraphile，我们终于有了与语言无关的方式来完成事情，这将让许多非 Node.js 开发人员感到高兴。