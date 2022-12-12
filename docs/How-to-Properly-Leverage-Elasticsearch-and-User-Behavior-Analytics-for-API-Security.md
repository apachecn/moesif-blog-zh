# 如何恰当地利用弹性搜索和用户行为分析实现 API 安全性

> 原文：<https://www.moesif.com/blog/technical/api-security/How-to-Properly-Leverage-Elasticsearch-and-User-Behavior-Analytics-for-API-Security/>

Kibana 和 ELK 栈的其他部分(Elasticsearch、Kibana、Logstash)非常适合解析和可视化各种用例的 API 日志。作为一个开源项目，它可以免费开始(你仍然需要考虑任何计算和存储成本，这对分析来说并不便宜)。Kibana 最近发展起来的一个用例是为 API 安全提供分析和取证，随着公司向客户、合作伙伴公开越来越多的 API，并被单页面应用和移动应用利用，这是工程领导者和 CISO 越来越关注的问题。这可以通过检测应用程序将所有 API 流量记录到 Elasticsearch 来实现。然而，一个简单的实现只会存储原始的 API 日志和调用，这对于 API 安全用例来说是不够的。

## 为什么 API 日志是 API 安全的一种幼稚的方法

原始 API 日志仅包含与执行单个操作相关的信息。通常会记录 HTTP 标头、IP 地址、请求正文和其他信息，以供以后分析。可以通过购买 Elasticsearch X-Pack 的许可证来添加监控功能。问题是，孤立地查看 API 调用并不总能检测到安全事件。相反，黑客能够执行精心设计的行为流程，以一种意想不到的方式运行您的 API。

让我们以一个[简单分页攻击](/blog/technical/api-security/API-Security-Threats-Every-API-Team-Should-Know/)为例。分页攻击是指黑客能够通过像`/items`或`/users`这样的资源分页，在不被发现的情况下抓取您的数据。也许这些信息已经是公开的、低风险的，例如在电子商务平台中列出的项目。然而，资源也可能有 PII 或其他敏感信息，如`/users`，但没有得到正确的保护。在这种情况下，黑客可以编写一个简单的脚本来转储存储在数据库中的所有用户，如下所示:

```py
skip = 0
while True:
    response = requests.post('https://api.acmeinc.com/users?take=10&skip=' + skip),
                      headers={'Authorization': 'Bearer' + ' ' + sys.argv[1]})
    print("Fetched 10 users")
    sleep(randint(100,1000))
    skip += 10 
```

有几点需要注意:

1.  黑客在每次调用之间随机等待一段时间，以免遇到速率限制
2.  由于前端应用程序一次只获取 10 个用户，黑客一次只获取 10 个用户以避免引起任何怀疑

单个 API 调用中绝对没有任何东西可以区分这些坏请求和真正的请求。相反，您的 API 安全和监控解决方案需要全面地检查用户行为。这意味着检查由单个用户或 API 键发出的所有 API 调用，这被称为*用户行为分析*或 UBA。

## 如何在 Kibana 和 Elasticsearch 中实现用户行为分析

为了在 Kibana 和 Elasticsearch 中实现用户行为分析，我们需要将我们以时间为中心的数据模型转变为以用户为中心的数据模型[通常，API 日志存储为时间序列，使用事件时间或请求时间作为组织数据的日期。通过这样做，旧的日志可以很容易地标记为只读，移动到较小的基础架构，或根据保留策略停用。此外，当您只查询有限的时间范围时，它可以加快搜索速度。](/blog/technical/metrics/User-Centric-Metrics-vs-Infrastructure-Metrics-and-How-to-Choose-The-Right-Analytics-Architecture-and-Data-Store/)

### 用用户 id 标记 API 日志

为了将它转换成以用户为中心的模型，我们需要用用户标识信息标记每个事件，比如租户 id、用户 id 或类似信息。因为大多数 API 都是由某种 OAuth 或 API 键保护的，所以将 API 键直接映射到永久标识符(如用户 id)或者在 Redis 这样的键/值存储中维护这种映射是相当容易的。这样，您的日志可能看起来像这样:

| 请求时间 | 动词 | 途径 | 用户标识 |
| 2021-08-02t 02:14:48 | 得到 | `/items` | One thousand two hundred and thirty-four |
| 2021 年 8 月 02 日 15 时 49 分 | 得到 | `/items` | One thousand two hundred and thirty-four |
| 2021-08-03T02:16:19Z | 得到 | `/users` | Six thousand seven hundred and eighty-nine |
| 2021-08-03T02:24:49Z | 得到 | `/users` | One thousand two hundred and thirty-four |

### 将相关的 API 日志分组在一起

现在，您已经用用户 id 标记了所有 API 日志，您将需要运行一个 map reduce 作业来将用户的所有事件分组在一起，并为每个用户计算任何指标。不幸的是，Logstash 和 Fluentd 等日志聚合管道一次只能丰富单个事件，因此您需要一个定制的应用程序，该应用程序可以在 Spark 或 Hadoop 等分布式计算框架上运行分布式 map/reduce 作业。

按用户 id 分组后，您会希望在“用户配置文件”中存储一些项目，例如:

1.  用户的 Id 和人口统计数据
2.  该用户制作的原始事件
3.  汇总指标，如 API 键的数量或用户下载的数据量

### 存储用户配置文件

即使您按用户 id 分组，将所有事件存储到单个数据库实体也是不可行的，因为这样会降低时间序列数据存储的灵活性，包括:

1.  包含太多数据的胖实体
2.  无法废弃旧数据
3.  由于接触了大量数据，查询变得很慢

为了解决这个问题，我们可以用这种以用户为中心的方法覆盖我们原来的时间序列架构，创建一个两级数据模型。

| 用户标识 | 开始时间 | 结束时间 | 登录次数 | 接触的用户数量 | API 键的数量 | 事件 |
| One thousand two hundred and thirty-four | 2021-08-02T00:00:00Z | 2021-08-02T23:59:59Z | Two | Two hundred and fifty thousand two hundred and twenty-three | one | [] |
| Six thousand seven hundred and eighty-nine | 2021-08-0300:00:00Z | 2021-08-03T23:59:59Z | Thirteen | Two hundred and thirty-two | Twelve | [] |
| One thousand two hundred and thirty-four | 2021-08-0300:00:00Z | 2021-08-03T23:59:59Z | Zero | Three hundred and twenty-three thousand nine hundred and ninety-seven | Zero | [] |

在这种情况下，我们每天都会创建一个新的“用户配置文件”，其中包含相关的安全指标和原始事件。

### 检测 API 安全漏洞

现在，我们已经将 API 数据重新组织为以用户为中心，无论是通过视觉检查、静态警报规则还是高级异常检测，识别坏用户和好用户都变得容易得多。

在这种情况下，我们看到典型的用户(6789)只接触或访问了 232 个用户和 12 个项目。显然，这看起来像标准的交互式流量。另一方面，我们有一个坏演员(1234)，他在过去两天里每天触摸或下载超过 250，000 个项目。此外，他在第二天访问 API 时没有任何相应的登录。现在，您可以创建一个基础结构来以编程方式检测这种情况，并在任何用户“一天内接触超过 10，000 个项目”时向您发出警报像 [Moesif](https://www.moesif.com/solutions/api-security) 这样的 API 安全和监控解决方案已经内置了这种功能。

### 为了 API 安全，将 API 日志保留多长时间

与用于调试目的的 API 日志不同，这些实体应至少存储一年，因为大多数违规研究表明检测数据违规的时间超过 200 天。如果为了降低成本，您只保留几天或几周的 API 数据，那么您将无法访问审计和事后审查所需的有价值的取证数据。像对待数据库备份一样对待 API 数据，因为您永远不知道何时可能需要它们，并且应该定期测试您的系统以确保捕获正确的数据。大多数安全专家建议保留 API 日志至少一年。天真的决策过于强调降低存储和计算成本，而没有考虑他或她给公司带来的风险有多大。

由于 API 日志可能包含 PII 和个人数据，将数据存储一年会使您的 GDPR 和 CCPA 合规性复杂化。幸运的是，GDPR 和 CCPA 已经对出于检测和防止欺诈和未经授权的系统访问以及确保 API 安全的合法目的而收集和存储日志的行为进行了豁免。此外，由于您已经将所有 API 日志绑定到个人用户，处理 GDPR 请求(如*擦除权*或*访问权*)变得轻而易举。