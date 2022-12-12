# GraphQL API 的 5 个安全提示

> 原文：<https://www.moesif.com/blog/technical/security/5-Security-Tips-for-Your-GraphQL-API/>

2015 年，GraphQL 由脸书创建，作为 REST APIs 的替代方案，通过使 API 调用更加灵活，为前端开发人员提供更多功能。GraphQL 通过为其 API 消费者提供一种查询语言来实现这一目标，这种语言允许他们只查询自己需要的数据。

虽然 GraphQL 可以改善前端开发人员的体验，但它的规范在安全性方面没有意见。GraphQL API 需要将查询转换为数据获取，这给 API 架构增加了另一层复杂性。

我们已经写了[一篇关于十大 API 安全线程](https://www.moesif.com/blog/technical/api-security/API-Security-Threats-Every-API-Team-Should-Know/)的文章，它们都适用于 GraphQL APIs。在本文中，您将了解提高 GraphQL API 安全性的其他增强功能。

美国和欧盟的新隐私法使得每个安全漏洞对 API 公司来说都是巨大的财务威胁。

# 1.综合查询授权

尽管小型 API 通常提供 HTTP 可访问的方式来访问数据库，但 GraphQL 通常是数据源不可知的。这意味着 GraphQL 不关心数据来自哪里。数据可以来自数据库，也可以从另一个 API 获取。这也意味着，GraphQL 不是一个数据存储库，所以它必须对实际的数据源进行认证，并检查用户是否被授权访问来自这些数据源的特定数据。

假设一个服务于许多不同客户端的 GraphQL 服务器被认证为上游服务或数据库中的一个用户。在这种情况下，**graph QL 服务器必须自己在解析器内部进行授权检查**。否则，这可能会导致 CCPA 法律规定的私人或更糟的医疗数据的泄露，并使您承担法律责任。

如果需要，您可以在每个解析程序中检查权限:

```py
const resolvers = {
  Query: {
    adminResolver: async (parent, args, context) => {
      if(!context.user || !context.roles.includes("admin")) throw new Error("Permission denied!");

      ...

      return data;
    },
  },
}; 
```

# 2.输入验证和规范化

过去，开发人员会在前端生成 SQL 查询，并将它们发送给 API，以便从 SQL 数据库中获取数据；于是，SQL 注入诞生了。攻击者可以编写 SQL 并将其发送给 API，API 会执行它而不会询问更多问题。

虽然没有人阻止 GraphQL API 开发人员创建一个接受 SQL 字符串的类型，但这种情况很少发生。

但是接受来自客户的数据总是有风险的。这就是为什么在任何数据获取发生之前，所有的输入都应该被验证和规范化。特别是[自定义标量](https://uptoskill.com/graphql-custom-scalars/)容易受到这种威胁，因为它们不做默认验证。

# 3.自省限制

GraphQL 为其 API 消费者提供了一个方便的自省特性，它允许 GraphQL 客户端询问 API 它提供了什么类型的数据。这很好，因为现在客户端开发人员不必查看文档，而是可以直接询问 API 服务器有哪些数据可用。

但是如果没有严格控制，内省也会被滥用。例如，当提供管理功能的 GraphQL 类型可以被普通用户发现和使用时。

GraphQL API 创建者不得不**使用严格的授权方案进行自省**以使攻击者更难发现 API 漏洞。如果外部开发者不使用你的 API，那么**在生产环境中禁用自省特性**也是一个好主意。

例如，Apollo server 允许使用一个简单的配置标志来禁用自检:

```py
const IS_PRODUCTION = process.env.ENVIRONMENT === "production";

const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: !IS_PRODUCTION,
});

server.listen(); 
```

# 4.查询限制

GraphQL 查询为 API 消费者提供了很大的灵活性，只需一个请求就可以准确地获取他们想要的数据，但是这个特性也可能被以多种方式滥用。

出于各种原因，恶意客户端可以创建非常深入、复杂或通常需要长时间运行的查询。如果这些攻击者发现了导致大量计算的边缘情况，或者利用了 N+1 查询问题，他们可以使 API 过载，并大幅降低其他客户端的性能。

这就是为什么 GraphQL API 应该**将查询执行时间限制在一个合理的最大值**。更好的是，实现这一目标的另一种方式是**限制查询深度或复杂性**，这样就更难执行常见的 GraphQL DDoS 攻击，这些攻击往往使用递归和深度查询，只是为了中断服务。

# 5.上游错误隐藏

如前所述，GraphQL API 不是一种数据存储机制；这意味着它使用上游服务，如数据库或其他 API 来获取实际数据。这些服务也可能有错误。如果您将上游服务的错误传递给客户端，攻击者就可以利用它们来了解您的架构。

为了减轻这种威胁，您应该总是在将上游错误提交给客户端之前处理它们。这样，你可以**隐藏你获取数据的服务**，并阻止攻击者利用这些服务可能存在的漏洞或安全漏洞。

让我们看看下面的代码示例:

```py
const resolvers = {
  Query: {
    myResolver: async (parent, args, context) => {
      try {
        const data = await fetchFromRemoteDataSource();
        return process(data);
      } catch (upstreamError) {
        const cleanError = analyzeUpstreamError(upstreamError);
        throw cleanError;
      }
    },
  },
}; 
```

解析程序尝试从远程数据源获取一些数据，但这可能会失败。我们捕获的错误来自远程数据源，因此它可能包含关于数据源的信息。

我们需要分析错误，并在将它交付给客户端之前清除任何上游信息。

# 保证 GraphQL APIs 的安全

GraphQL APIs 是为前端团队改善开发人员体验的一个很好的方式。它为客户提供了一种指定他们需要什么的方法，从而有助于优化数据获取。但这是以 API 架构的更高复杂性为代价的，这增加了 API 的攻击面。

随着规定处以数百万美元罚款的 [GDPR](https://gdpr-info.eu/) 和 [CCPA](https://www.oag.ca.gov/privacy/ccpa) 法律的兴起，对于 API 开发者来说，在开发 API 时牢记[常见的 API 威胁](https://www.moesif.com/blog/technical/api-security/API-Security-Threats-Every-API-Team-Should-Know/)以及寻找可能出现的 GraphQL 特定弱点比以往任何时候都更加重要。