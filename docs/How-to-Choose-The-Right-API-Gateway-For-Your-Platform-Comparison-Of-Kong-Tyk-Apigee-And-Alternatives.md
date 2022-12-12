# 如何为您的平台选择正确的 API 网关:Kong、Tyk、Apigee 和备选方案的比较

> 原文：<https://www.moesif.com/blog/technical/api-gateways/How-to-Choose-The-Right-API-Gateway-For-Your-Platform-Comparison-Of-Kong-Tyk-Apigee-And-Alternatives/>

## 为什么选择 API 网关？

API 是许多大大小小应用的驱动力。无论你是发布一个公共 API 还是建立一个新的集成市场，API 正在成为做生意的方式。就像 web 时代有 HTTP 服务器为生产中的网站提供服务一样，API 也有 API 网关来为生产中的 API 提供服务。人们可以利用 API 网关来帮助向您的客户和合作伙伴交付考虑到高可用性的 API。它们是一种代理服务器，位于您的 API 之前，执行身份验证、速率限制、将公共可访问的端点路由到适当的微服务、跨多个内部服务的负载平衡等功能。

## 历史背景

### 企业集成中间件

从历史上看，对 API 网关的需求源于集成挑战。在 REST 和 GraphQL APIs 之前，公司正在构建由结构化或非结构化数据组成的基于 SOAP 和 XML 的 API。API 网关可以提供统一的接口，并将多个遗留应用程序链接在一起。在这种用例中，API 网关可以采用遗留的 SOAP 服务，将数据转换应用到 API，例如从 SOAP 转换到 REST 以及从 [JSON](https://www.json.org/) 转换到 [XML](https://www.w3.org/XML/) 。这些类型转换通常不是自动的。例如，[RESTful API 的核心原则](/blog/api-guide/getting-started-with-apis/#core-principles-of-restful-api)与 SOAP 非常不同，因此它不像将 XML 转换成 JSON 那么简单。

### 打破巨石

[微服务架构](/blog/api-guide/the-next-api-platform-serverless-and-blockchain/#microservices)是构建和部署独立服务以组成更大应用的策略。微服务与整体架构的利弊超出了本文的范围。从高层次来看，微服务架构正在成为构建 API 的方式。它使多个独立的团队能够在一个大的应用程序上工作，而不用互相监督或处理长时间的部署。

除了微服务，还有更小的计算单元，如纳米服务和无服务器计算。由于管理成百上千个服务的复杂性以及向客户提供统一接口或合同的要求，API 网关正在成为使用微服务和无服务器计算的架构中的常见位置。

## API 网关的优势

无论您使用的是微服务还是无服务器计算，或者您的 API 是内部使用还是公开访问，使用 API 网关都有很多好处:

*   **解耦**:如果您无法控制的客户端与许多独立的服务直接通信，重命名或移动这些服务可能会很困难，因为客户端与底层架构和组织相耦合。API 网关使您能够基于路径、主机名、报头和其他关键信息进行路由，从而使您能够将面向公众的 API 端点从底层微服务架构中分离出来。
*   **减少往返行程**:某些 API 端点可能需要跨多个服务连接数据。API 网关可以执行这种聚合，这样客户端就不需要复杂的调用链，并减少往返次数。
*   **安全性** : API 网关提供了一个集中的代理服务器来管理速率限制、僵尸检测、认证、CORS 等。许多 API 网关允许建立一个数据存储库(如 Redis)来存储会话信息。
*   **横切关注点**:日志记录、缓存和其他横切关注点可以在一个集中的设备中处理，而不是部署到每个微服务上。事实上，Moesif 为许多 API 网关(如 Kong 和 Tyk)提供了[插件](https://www.moesif.com/extensions/data-ingestion)，因此您无需安装任何 SDK 就可以获得现代客户和 API 分析。

![API-Gateway-with-Microservices](img/af6d8b11715148c9ea8cd451cdc8f9b7.png)

### API 平台的其他优势

除了上面列出的好处，为客户和合作伙伴构建可公开访问的 API 的公司还有其他好处。这种 API 平台由 Stripe 或 Twilio 等 API first 公司以及 Github 或 Twitter 等开发平台公司构建。如今，随着客户和合作伙伴要求更多的定制和集成，B2B 公司向平台转型变得越来越重要。

使用 API 网关的其他好处包括:

*   为开发人员管理 API 密钥，包括提供一致的授权和认证方式
*   基于配额或使用量的速率限制和计费。
*   为客户和合作伙伴提供开发人员门户，以创建 API 令牌、弃用令牌等。

> *什么是 Moesif？ [Moesif](https://www.moesif.com/) 是最先进的 API 分析平台，被成千上万的平台用来了解你最忠实的客户在用你的 API 做什么，他们如何访问它们，以及从哪里访问。Moesif 有流行 API 网关的插件，比如[孔](https://docs.konghq.com/hub/moesif/kong-plugin-moesif/)、 [Tyk](https://tyk.io/docs/configure/tyk-pump-configuration/#moesif-config) 和 [more](https://www.moesif.com/docs/server-integration/) 。*

## 要比较的变量

### 部署复杂性

它是单节点设备还是网关需要运行多种类型的节点来启动和设置数据库？一些网关需要多种类型的数据库。

### 开源与专有

当您想用附加功能扩展网关时会发生什么？有插件吗？如果是，插件是开源的吗？

### 内部部署与云托管

内部部署会增加额外的时间来规划部署和维护。然而，云托管解决方案可能会由于额外的跳跃而增加一点延迟，并且如果供应商倒闭，可能会降低您的服务的可用性。

### 特征

一些网关更像是为服务 API 而修改的基本 HTTP 服务器。而另一些包含了整个包，包括开发人员门户、安全性等等。如果网关包含这样的特性，那么像开发者门户这样的特性是否具有良好的用户体验和设计，或者使您能够调整设计以满足您的需求。

### 社区

开发人员是否在网关之上构建了额外的功能？就像 Apache Tomcat 和 NGINX 拥有庞大的开源追随者一样。一些 API 网关有大型开发人员社区来构建脚本，回答堆栈溢出等问题。

### 价格

如果你是一家小公司，他们有好的免费版本或开源版本吗？然而，如果你是一家老牌企业，这家公司有你需要的支持吗？

## API 网关领域的主要参与者

### 孔（姓）

Kong 是一个基于 NGINX 的开源 API 网关。)这是一个非常流行的开源 HTTP 代理服务器。尽管 Kong 是开源的， [KongHQ](https://www.konghq.com) 为大型企业提供维护和支持许可。虽然开源版本拥有基本功能，但某些功能，如管理 UI、安全性和开发人员门户，只能通过企业许可证获得。

部署:Kong 的最大优势之一是它广泛的安装选项，有预先制作的容器，如 Docker 和 range，这样您可以快速运行部署。NGINX 是仅次于 Apache 和 IIS 的最流行的 HTTP 服务器，即使在高请求速率下也有很高的性能。NGINX 有一个巨大的 Lua 脚本和扩展社区，所以在寻找一些定制时，你不会被落在后面。就部署而言，孔的复杂性适中。它确实需要运行 Cassandra 或 Postgres。一些插件，比如限速插件，需要额外的数据存储，比如 Redis。然而，生产部署并不像 Apigee 那样复杂。

功能完整性:Kong 开箱即用提供了 API 管理的许多预期功能，如创建 API 密钥、路由到多个微服务等。它没有很多转换层(主要是基于 HTTP 的转换，没有 SOAP 或 to XML)。然而，如果您没有很多遗留应用程序，您可能不需要额外的数据转换层。尽管它有速率限制，但它没有计费集成。可以通过 CLI 或 curl 命令对 REST API 执行管理任务，这使得管理更容易集成到您现有的 devops 行动手册中。

Kong 提出了 [*服务*](https://docs.konghq.com/1.0.x/admin-api/#service-object) 、 [*路由*](https://docs.konghq.com/1.0.x/admin-api/#route-object) 和 [*消费者*](https://docs.konghq.com/1.0.x/auth/#consumers) 的概念，在处理组成您的 API 的数百个微服务和调用您的 API 的不同类型的消费者时提供了很大的灵活性。这使得插件和转换能够被附加到一个特定的路径，甚至是一个单独的消费者。

Kong 有一个大型的社区社区开发插件，他们在 2018 年[推出了](https://konghq.com/blog/welcome-to-the-new-kong-hub/) [Kong Hub](https://docs.konghq.com/hub/) ，并且它已经有几十个插件。 [Moesif 是那里的一个插件](https://docs.konghq.com/hub/moesif/kong-plugin-moesif/)。

Kong 是我们极力推荐的 API 网关之一。如果你不需要遗留的包袱，想要一个流行的开源 API 网关，那么 Kong 是绝对不会错的。它是现代的，旨在管理现代微服务，而不仅仅是给传统的 monoliths 增加一个转换门面，并且有一个快速增长的插件社区，从 Moesif 这样的 API 分析到缓存层和验证 JWT (JSON Web Tokens)。

### Tyk.io

和 Kong 一样，Tyk 也是[开源](https://github.com/TykTechnologies/tyk)，但是是在 MPL 许可下，比 Kong 的 Apache 2.0 许可宽松一些。同时，Tyk 的企业用户使用与社区用户完全相同的网关。您不必为某些企业功能支付额外费用。Tyk 不依赖于额外的插件和 Lua 脚本，它更像是一个包含了 API 网关的*电池。像 OIDC、OAuth2、无记名令牌、基本认证、相互 TLS、HMAC 这样的认证方案都是开箱即用的，不需要插件。它还支持 XML- > JSON、JSON- > XML、JSON 模式验证。*

Tyk 建立在 GoLang 之上，作为一种系统语言，它是为高吞吐量和并行性而设计的。背后的公司， [Tyk.io](https://tyk.io/) 提供云托管版本和专业支持许可。与 Kong/NGINX 的 Lua 不同，有些人可能会发现 Golang 更现代，也更容易编程。除了 Golang，Tyk 还有解释器来运行其他语言的插件，比如 Javascript 和 Lua。请记住，与可以部署在与您的内部服务相同的 VNET 上的内部版本不同，云版本需要将您的一些服务直接公开到互联网上。

部署:Tyk 提供云托管 SaaS 解决方案或内部部署。您可以在 Heroku 或 AWS 上部署实例。他们的网站提供了关于如何使用 T1 的教程。开源版本的部署相对简单，只需要 Redis，而 Kong 需要运行 Cassandra 或 Postgres 集群。

Tyk 具有密钥管理、配额、速率限制、API 版本、访问控制等功能，但没有集成的计费功能。Tyk 有一个 REST API 和一个 web 仪表板来完成管理任务。虽然他们有一个扩展列表，但是 Tyk 没有像 Kong 那样大的社区或者插件中心。然而，他们确实保持他们的网关设计良好，并试图保持精简。

### 养蜂场

[Apigee](https://apigee.com/) 是本文中列出的最古老的 API 网关。它成立于 2004 年，2016 年被谷歌收购。它不是开源的，构建在 enterprise Java 之上。它们最初是作为 XML/SOA 应用程序开始的，但后来转向了 API 管理领域。Apigee 旨在将遗留的 monoliths 转换成可供第三方使用的 API。他们不太关注微服务和内部 API。

因为 Apigee 具有复杂的多节点架构，所以相对于开源 API 网关，部署的复杂性要高得多。Apigee Edge 要求在本地运行至少 9 个节点，包括运行 Cassandra、Zookeeper 和 Postgres，这迫使部署由一个集中的基础架构团队进行规划，该团队花费数月时间来规划部署。

虽然大多数 Apigee 客户使用内部版本，但在加入谷歌后，他们引入了云托管解决方案。然而，它更接近 IaaS，必须部署到特定的谷歌云数据中心，而不是纯粹的 SaaS。与其他托管版本一样，托管代理版本会增加延迟，并需要保护您自己的服务。

> 当使用托管 API 网关时，除非它与您的上游服务在同一个数据中心，否则会增加一点延迟。

与其他 API 不同的是，Apigee 支持[端到端集成计费](https://apigee.com/api-management/#/product/monetize-apis)来直接货币化你的 API。管理门户构建在 Drupal 之上。取决于你的观点，Apigee 要么看起来功能臃肿，要么就是完整的解决方案。同时，由于是专有的，它没有一个大的开发者社区贡献插件或扩展。

### 亚马逊 AWS API 网关

[亚马逊 AWS](https://aws.amazon.com/) ，作为最大的云厂商，也有 [AWS API 网关](https://aws.amazon.com/api-gateway/)。这是唯一的云选项。如果您已经在使用 AWS Lambda 或 EC2，您可以在与上游服务相同的数据中心区域部署 AWS API gateway，这样增加的延迟就不是问题。AWS API Gateway 是完全受管的，只需在 AWS 门户中点击几下就可以部署。

当与 AWS Lambda 结合使用时，AWS API Gateway 为[无服务器](/blog/engineering/serverless/A-Primer-On-Serverless-Computing-AWS-Lambda-vs-Google-Cloud-Functions-vs-Azure-Functions/)API 提供了一个很好的解决方案。无服务器就像类固醇上的微服务，需要对 API 端点进行完美的管理，以将传入的 API 调用路由到适当的无服务器函数。

除了 AWS Lambda，AWS API Gateway 还有最好的一键式解决方案，可以将传入的 API 调用路由到其他 AWS 服务，如 Amazon Kinesis 和 Amazon DynamoDB。此外，您可以使用现有的 IAM 基础设施为 API 提供身份验证，而不会产生太多开销。

论相貌，它比得上孔。然而，AWS API Gateway 并没有一个大型的开发人员社区来编写扩展或插件。使用 AWS API Gateway 的最大问题之一是供应商锁定。

### 其他人

上面的列表并不详尽，下面是一些其他列表的快速总结:

*   Azure API Gateway 与 AWS 的产品非常相似。当然，如果你使用的是微软 Azure，并且对 Azure 功能有很好的支持，那就更适合了。

*   [Express API Gateway](https://www.express-gateway.io/) 是由 [LunchBadger](https://www.lunchbadger.com/about-us/) 打造的新入口，它是完全开源的，基于非常流行的 [Node.js](https://nodejs.org/en/) [Express 框架](https://expressjs.com/)。他们的设计理念是保持它的最小化和声明性。如果您正在 Node.js 上构建大量核心基础设施，并且熟悉 express 中间件，那么值得一看。

*   [KrakenD](https://www.krakend.io/) 也是 GoLang 内置的开源产品。我们还没用过这个。如果你有任何反馈，请告诉我。

## 摘要

下面以表格形式简要总结了调查结果:

| 制品 | 孔（姓） | Tyk.io | 养蜂场 | AWS 网关 | Azure 网关 | 快速网关 |
| --- | --- | --- | --- | --- | --- | --- |
| 部署复杂性 | 单一节点 | 单一节点 | 许多具有不同角色的节点 | 云供应商平台即服务 | 云供应商平台即服务 | 灵活的 |
| 需要数据存储 | 卡珊德拉还是波斯特格里斯 | 雷迪斯 | 卡珊德拉、动物园管理员和波斯特格里斯 | 云供应商平台即服务 | 云供应商平台即服务 | 雷迪斯 |
| 开放源码 | 是的，Apache 2.0 | 是的，MPL | 不 | 不 | 不 | 是的，Apache 2.0 |
| 核心技术 | ngix/月球 | GoLang | Java 语言(一种计算机语言，尤用于创建网站) | 不公开 | 不公开 | Node.js 快递 |
| 在内部 | 是 | 是 | 是 | 不 | 钼 | 是 |
| 社区/扩展 | 大的 | 中等 | 不 | 不 | 不 | 小的 |
| 授权/API 密钥 | 是 | 是 | 是 | 是 | 是 | 是 |
| 限速 | 是 | 是 | 是 | 是 | 是 | 是 |
| 数据转换 | 超文本传送协议 | 超文本传送协议 | 是 | 不 | 不 | 不 |
| 集成计费 | 不 | 不 | 是 | 不 | 不 | 不 |

随着更多的公司部署更精细的 mciroservice 和无服务器架构，API 网关的使用只会增加。此外，在看到 Twilio、Salesforce 和 Stripe 等公司的早期成功后，越来越多的公司开始推出自己的开发人员计划。我们非常期待看到 API 经济和开发者平台是如何发展的，并且很乐意为此做出贡献。