# REST vs GraphQL APIs，好的，坏的，丑陋的

> 原文：<https://www.moesif.com/blog/technical/graphql/REST-vs-GraphQL-APIs-the-good-the-bad-the-ugly/>

自从被脸书引入以来，GraphQL 作为 REST APIs 的替代者，已经在 API 世界掀起了一场风暴。GraphQL 修复了 API 开发者和用户在 RESTful 架构中发现的许多问题。然而，它也带来了一系列需要评估的新挑战。因为 GraphQL 不仅仅是 REST 的进化替代品，这篇文章将深入探讨每种技术的优缺点以及 GraphQL 何时对您的应用程序有意义。

## 历史

在 RESTful APIs 之前，我们有 RPC、SOAP、CORBA 和其他不太开放的协议。许多 pre-REST API 需要复杂的客户端库来通过网络序列化/反序列化负载。与今天的 RESTful APIs 相比，一个根本的区别是 SOAP 是通过 WSDL (Web 服务描述语言)使用正式契约强类型化的。这可能会破坏互操作性，因为即使取消 32 位整数限制来接受 64 位整数也意味着破坏上游客户端。SOAP 不是一个体系结构，而是一个完整的协议实现，包括安全性、错误处理、ACID 事务等。有些复杂性是由于 SOAP 中包含了许多抽象层。例如，SOAP 能够在 HTTP、TCP、UDP 等上运行。虽然协议实现提供了很多抽象，但是客户机和服务器之间的应用程序和数据层的严格约定在两者之间产生了紧密的耦合。

RESTful 架构是在 2000 年作为一种更简单的方式引入的，它只使用普遍存在的 HTTP 协议来实现机器对机器的通信，而不需要以无状态和无类型的方式添加额外的层。这使得系统能够松散耦合，并且对系统之间(比如公司之间)的契约变更更加宽容。

## 今天休息

REST APIs 已经成为公司部署 API 和开发平台的事实标准。REST 的美妙之处在于，使用别人的 API 的开发人员不需要任何特殊的初始化或库。请求可以简单地通过通用软件如 cURL 和网络浏览器发送。

REST 使用标准的 CRUD HTTP 动词(GET、POST、PUT、DELETE ),利用 HTTP 约定，以数据资源(HTTP URIs)为中心，而不是试图对抗 HTTP。因此，拥有资源`api.acmestore.com/items`的电子商务可以表现得类似于拥有资源`api.examplebank.com/deposits`的银行。两者都可能需要在那个资源上进行 CRUD 操作，并且更喜欢缓存查询(也就是说，`GET api.acmestore.com/items`和`GET api.examplebank.com/transactions`都将被缓存)。新 API 的第三方开发者只需要推理数据模型，剩下的就交给 HTTP 约定，而不需要深入研究成千上万的操作。换句话说，与 SOAP 相比，REST 与 HTTP 和 CRUD 的耦合更紧密，但是提供了松散的数据契约。

## 休息的问题

随着越来越多的 API 投入生产使用并扩展到极致，RESTful 架构中出现了一些问题。你甚至可以说 GraphQL 介于 SOAP 和 REST 之间，各取所需。

## 服务器驱动的选择

在 RESTful APIs 中，服务器创建一个资源的表示来响应客户端。

然而，如果客户想要一些特定的东西，比如返回一个用户的朋友的朋友的名字，他们的工作是*工程师*。

休息时，您可能会看到这样的内容:

```py
GET api.example.com/users/123?include=friend.friend.name&friend.friend.ocupation=engineer 
```

GraphQL 允许您以更简洁的方式表示这个查询:

```py
{  user(id:  123)  {  friends  {  friends(job:  "engineer")  {  name  }  }  }  } 
```

## 获取多个资源

GraphQL 的主要好处之一是使 API 不那么繁琐。我们中的许多人都见过这样的 API，其中我们首先必须先`GET /user`然后通过`GET /user/:id/friend/:id`端点分别获取每个朋友，这可能导致 N+1 个查询，并且是 API 和数据库查询中众所周知的性能问题。换句话说，RESTful API 调用在最终的显示形式形成之前被链接在客户端。GraphQL 可以通过允许服务器在单个查询中聚合客户机的数据来减少这种情况。

## 更多深度分析

而 API 分析对 GraphQL apis 也是一个不利因素，因为那里几乎没有工具。支持 GraphQL APIs 的工具可以提供比 RESTful APIs 更多的查询洞察。

## GraphQL 的问题

## 贮藏

缓存内置在 HTTP 规范中，RESTful APIs 能够利用它。与缓存相关的 GET vs POST 语义被很好地定义，使得浏览器缓存、中间代理和服务器框架能够遵循。可以遵循以下准则:

*   GET 请求可以被缓存
*   GET 请求可以保留在浏览器历史记录中
*   GET 请求可以加入书签
*   GET 请求是幂等的

GraphQL 不遵循 HTTP 规范进行缓存，而是使用单个端点。因此，开发人员有责任确保为可以缓存的非可变查询正确实现缓存。必须为缓存使用正确的密钥，这可能包括检查主体内容。

虽然您可以使用像 Relay 或 Dataloader 这样理解 GraphQL 语义的工具，但这仍然不能涵盖像浏览器和移动缓存这样的东西。

## 减少无共享架构

RESTful APIs 的美妙之处在于它们很好地补充了无共享架构。例如，Moesif 有一个`api.moesif.com/v1/search`端点和一个`api.moesif.com/v1/alerting`端点。公开地说，这两个端点看起来就像两个不同的 REST 资源。但在内部，他们指出了隔离计算集群上的两种不同的微服务。搜索服务是用 Scala 写的，提醒服务是用 NodeJS 写的。通过主机或 URL 路由 HTTP 请求的复杂性比检查一个 GraphQL 查询和执行多个连接要低得多。

## 暴露给任意请求

虽然 GraphQL 的一个主要优势是使客户端能够查询他们需要的数据，但这也可能是有问题的，尤其是对于组织无法控制第三方客户端查询行为的开放 API。必须非常小心地确保 GraphQL 查询不会导致代价高昂的连接查询，这会降低服务器性能，甚至会造成服务器 DDoS。RESTful APIs 可以被约束以匹配所使用的数据模型和索引。

## 查询的严格性

GraphQL 消除了在 API 之上定制查询 DSL 或副作用操作的能力。例如，Elasticsearch API 是 RESTful 的，但也有一个非常强大的 Elasticsearch DSL 来执行高级聚合和度量计算。这种聚合查询可能更难在 GraphQL 语言中建模。

## 不存在的监控

RESTful APIs 的优势在于它像网站一样遵循 HTTP 规范。这使得许多工具能够探测一个 URL，例如`api.moesif.com/health`，如果不正确，它将返回 5xx。对于 GraphQL APIs，除非您支持将查询作为 URL 参数放置，否则您可能无法利用此类工具，因为大多数 ping 工具不支持 HTTP 和请求主体。

除了 ping 服务，很少有 SaaS 或开源工具支持 API 分析或对 API 调用进行更深入的分析。客户端错误在 GraphQL API 中显示为 200 OK。预计会出现 400 个错误的现有工具将无法工作，因此您可能会错过 API 上发生的错误。然而与此同时，给予客户更多的灵活性需要更多的工具来捕捉和理解 API 的问题。

> *什么是 Moesif？ [Moesif](https://www.moesif.com/features/graphql-analytics) 是最先进的 REST 和 GraphQL 分析平台，被数以千计的平台用来衡量您的查询执行情况，并了解您最忠实的客户正在使用您的 API 做什么。*

## 结论

GraphQL APIs 可能是令人兴奋的新技术，但是在做出这样的架构决策之前理解权衡是很重要的。一些 API，比如那些只有很少实体和跨实体关系的 API，比如分析 API，可能不适合 GraphQL。而具有许多不同域对象的应用程序，如电子商务，其中有商品、用户、订单、支付等等，可能能够更多地利用 GraphQL。

实际上，GraphQL vs REST 就像比较 SQL 技术 vs noSQL。在某些应用程序中，在 SQL Db 中建模复杂的实体是有意义的。而其他只有“消息”的应用程序，如高容量聊天应用程序或分析 API，其中唯一的实体是“事件”，可能更适合使用像 Cassandra 这样的东西。