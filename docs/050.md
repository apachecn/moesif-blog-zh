# GraphQL 和 RESTAPI 哪个对 API 的可观察性更好

> 原文：<https://www.moesif.com/blog/graphql/api-development/GraphQL-Versus-RESTAPI-Which-is-Better-for-API-Observability/>

## API 可观察性

API 提供者需要观察他们的 API，以获得关于它们在实践中是否被使用以及如何被使用的有意义的数据。API 可观察性是一种监视形式，它被动地将 API 流量记录到可观察性服务中。与传统的 API 监控不同，利用 API 可观察性，您可以:

监控交互以改善开发人员体验了解客户如何使用您的 API 对您的 API 进行故障排除

观察 REST API 得到了很好的理解和支持，但并不是每个 API 都是 REST API。特别是，当涉及到可观察性时，GraphQL APIs 的行为非常不同。让我们深入了解这意味着什么，以及如何利用它。

## GraphQL APIs 提供了优势，但是挑战了可观察性

许多组织通过监控用户访问的端点、记录错误消息和测量端点请求的延迟来实现 API 的可观察性。这种策略对于 REST APIs 来说很好，但是对于 GraphQL APIs 的可观察性来说还不够。

API 消费者喜欢 GraphQL，因为它足够灵活，可以在单个请求中查询来自多个来源的数据，并处理复杂的状态和缓存管理，这为开发人员带来了很好的体验。

GraphQL 的好处不仅仅在于 API 消费者。使用 GraphQL，API 提供者可以在不影响现有查询的情况下更新他们的 API。使用 GraphQL 提供数据也是平台无关的，因此 API 提供者可以选择最适合他们的数据提供者，包括将多个数据资源捆绑到一个查询中。支持 GraphQL 对于致力于构建优秀开发人员体验的 API 提供者来说非常重要，在这种体验中，查询是高效、强大和灵活的。

GraphQL 的灵活性对于 API 提供者和 API 消费者来说都是一个重要的好处，但是这种灵活性也意味着您需要一种不同的方法来观察 GraphQL APIs。让我们更深入地看看 API 可观察性在 GraphQL 上下文中提出的挑战，以及如何处理它们，以便您的组织可以观察您的 GraphQL API。

## 在实践中观察 GraphQL

为了从 REST API 访问数据，开发人员分别调用多个端点，将客户机需要的所有数据拼凑在一起。对于复杂的应用程序，GraphQL 方法更有吸引力，因为对不同数据源的请求都发生在一个查询中。这意味着开发人员只需要在使用 GraphQL 时请求他们需要的数据，而 REST 查询结构不允许这种级别的可定制查询。例如，一个显示产品信息的购物应用程序，谁在卖它，以及交付需要多长时间，将由如下三个独立的 REST API 调用组成:f GET/category/<product-id>/product info GET/seller/<seller-id>/GET/delivery/<location-id>/</location-id></seller-id></product-id>

需要多次 GET 调用来接收加载产品页面所需的所有数据。从产品端点调用数据，该端点返回包括销售者 id 在内的信息。有了卖家 id，应用程序向卖家端点发出数据调用，以获取卖家的信息，包括位置 id。最后，使用 location-id 调用交付端点，以便页面知道该产品的交付需要多长时间。每个响应都依赖于之前 API 调用的信息，这种方法经常获取过多或过少的数据。只从多个来源获取所需信息的 GraphQL 调用可以通过一个查询来完成:

```py
{
  category{
    name
    product(product-id){
      name
      productInfo{
        size
        price
        seller-id
      }
      seller(seller-id){
        name
        location-id
        delivery(location-id){
          name
        }
      }
    }
  }
} 
```

虽然 GraphQL 方法的单个查询的灵活性对于开发人员来说很大，但当使用 GraphQL 监控数据查询时，这就带来了挑战。如果在使用 REST API 的事务中出现部分失败，或者如果一个端点未能返回数据，HTTP 请求将返回一个失败状态响应。当调用不成功时，API 提供者和 API 使用者都会收到 HTTP 500 或 400 错误，而对于成功的端点调用，则会收到成功的 HTTP 200。我们购物示例中的 REST API 故障如下所示:

```py
HTTP/1.1 400 Unprocessable Entity
{
  "error": 
    {
      "code": "400",
      "message": "Product ID does not exist"
    }
} 
```

使用 GraphQL，即使 API 调用没有返回任何查询数据，部分故障也会成功解决并返回 HTTP 200。成功的 HTTP 响应是查询到达服务器的结果。如果查询调用了不存在的数据或者消费者无权访问的数据，HTTP 响应代码不会向您提供这些信息。这使得处理错误更加困难，因为诊断错误需要更多的努力。API 消费者可以检查 errors 对象的响应对象，这将显示哪个端点遇到了错误以及导致错误的原因。下面是我们的 GraphQL 购物应用程序查询示例中部分失败的情况:

```py
{
  "errors": [
    { "path": [ "product" ],
      "locations": [ { "line": 3, "column": 12 } ],
      "extensions": {
        "message": "Object not found",
        "type": 2
      }
    }
  ]
} 
```

API 消费者能够通过检查客户端的 GraphQL 错误对象来诊断错误，如下所示。不幸的是，在服务器端，如果您不检查每个调用，您可能不会发现这些错误。对于 API 提供者来说，要解决这个问题，他们需要监控他们的 API 返回的 GraphQL 有效负载，这样他们就可以像使用他们的 API 的开发人员一样检查错误对象。监视 API 有效负载还有一个额外的好处，那就是让您知道开发人员是如何使用您的 API 的，哪些查询是最常见的，哪些查询具有最低的延迟，以及 API 使用者是否成功地完成了查询。

## 通过观察学习

监控 GraphQL 有效负载的另一个好处是，您可以记录指标，这样您就可以根据开发人员访问 API 的方式做出长期决策。日志可以告诉你哪些 GraphQL 查询是成功的，谁是你最忠实的用户，哪些查询是首次用户做的，哪些查询经常被捆绑在一起，等等。这可以为您组织的客户成功战略提供信息，这将允许您编写更有效的文档，找到在 API 调用方面有困难的用户，并为使用您的 API 的开发人员提供更主动的支持。有了详细的日志，您可以详细了解您的 API。过滤查询是一种从 GraphQL 日志中聚集信息的强大技术。

将 GraphQL 查询聚合到您的 API 中，然后按类别(如字段名和参数值)对它们进行过滤，这将为您提供关于对您的 GraphQL API 进行的大多数查询的可操作信息。这将告诉您 API 的哪些部分为您的客户创造了最大的价值，因此您知道在编写文档时应该关注什么，应该向开发人员提出哪些问题，以及哪些附加功能可能会为您的用户创造价值。如果您的组织同时拥有 GraphQL 和 REST APIs，那么过滤您的查询将允许您直接比较开发人员如何使用您的不同 API。

我们已经介绍了您的组织应该如何实现 GraphQL API 可观察性，以及它对于一个专注于促进客户成功的组织有多么重要。虽然 REST API 可观察性实现起来更直观，并且有更多现有的 REST API 分析工具，但 GraphQL 可观察性比 REST API 可观察性有一些独特的优势。

## GraphQL API 可观察性的优势

GraphQL 查询准确地指出了需要什么信息。REST API 消费者经常欠挖或过挖数据，因为他们不能选择他们需要的具体数据。GraphQL 查询只调用消费者需要的数据，这些数据会准确地告诉您哪些数据正在为您的消费者创造价值。使用 REST API analytics，很难确定多个连续的 API 端点调用是来自消费者的单个事务的一部分，还是它们是不相关的事件。GraphQL analytics 将单个交易所需的所有调用捆绑到单个查询中，因此您可以更好地了解消费者如何使用您的 API。

GraphQL API analytics 可以在产品开发中为您提供优于 REST API analytics 的优势，并且它们可以使您的组织更容易关注客户成功。如果您的组织了解首次用户在遇到挑战时是如何试图查询您的 API 的，那么您的客户支持可以在解决您的客户遇到的问题方面发挥更积极的作用，从而提供更详细和更有建设性的支持。如果一个客户因为调用一个不再存在的 REST 端点而无法查询您的 API，您的组织可能不会注意到这个失败，您的客户支持也无法联系到这个客户。与 GraphQL API 调用形成对比，在 graph QL API 调用中，查询指向单个端点。如果客户试图查询不再存在的数据，您的监控将记录这一失败，并且您的客户支持可以直接联系用户，以帮助您的客户进行成功的 API 调用。

这些年来，GraphQL 变得非常流行，越来越多的团队选择使用 GraphQL APIs。作为一个 API 提供者，观察您的 GraphQL API 可能具有挑战性。绝大多数 API 分析工具都是为了支持 REST APIs 而构建的，而 GraphQL 通常是事后才想到的。Moesif 与众不同，旨在观察 GraphQL 和 REST APIs，以便您的组织能够促进客户成功并为您的客户创造价值。Moesif 提供了产品内置的 GraphQL 可观察性的所有优点。当您实现 GraphQL API 时，您不需要放弃强大的分析工具！