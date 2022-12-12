# REST API 设计:过滤、排序和分页

> 原文：<https://www.moesif.com/blog/technical/api-design/REST-API-Design-Filtering-Sorting-and-Pagination/>

无论 API 是公共的还是内部使用的，API 设计正在成为 API 产品策略的核心支柱。良好的 API 设计可以改善任何 API 程序的整体开发体验(DX ),并且可以提高性能和长期可维护性。

但是，没有标准或官方的 API 设计指南。RESTful 只是一种架构风格。有许多关于 api 设计的初学者 API 指南随时可用，如[本指南](https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9)和[本指南](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)。然而，我们没有找到很多关于更高级的过滤和分页的 api 指南，这激发了我们发表这篇文章。

## 过滤

URL 参数是向 REST APIs 添加基本过滤的最简单方法。如果您有一个作为待售商品的`/items`端点，您可以通过属性名进行过滤，例如`GET /items?state=active`或`GET /items?state=active&seller_id=1234`。然而，这只适用于精确匹配。如果你想做一个范围，如价格或日期范围？

问题是 URL 参数只有一个键和值，但是过滤器由三个组件组成:

*   属性或字段名称
*   操作员如 *eq* 、 *lte* 、 *gte*
*   过滤值

有多种方法可以将三个组件编码到 URL 参数键/值中。

### LHS 括号

对操作符进行编码的一种方式是在键名上使用方括号`[]`。例如，`GET /items?price[gte]=10&price[lte]=100`将查找价格大于或等于 10 但小于或等于 100 的所有商品。

我们可以根据需要拥有任意数量的操作符，例如*【LTE】**【GTE】**【存在】**【regex】**【之前】**【之后】*。

LHS 括号在服务器端解析起来有点困难，但是为客户端提供了更大的过滤值灵活性。不需要不同地处理特殊字符。

#### 利益

*   方便客户使用。有许多查询字符串解析库可以轻松地将嵌套的 JSON 对象编码到方括号中。qs 就是这样一个自动编码/解码方括号的库:

    ```py
    var qs = require('qs');
    var assert = require('assert');

    assert.deepEqual(qs.parse('price[gte]=10&price[lte]=100'), {
        price: {
            gte: 10,
            lte: 100
        }
    }); 
    ```

*   易于在服务器端解析。URL 参数键包含字段名称和运算符。易于`GROUP BY`(属性名，操作符)而无需查看 URL 参数值。

*   当 operator 作为文本筛选条件时，不需要对筛选值中的特殊字符进行转义。当过滤器包含用户可能设置的附加自定义元数据字段时，尤其如此。

#### 下降趋势

*   可能需要在服务器端做更多的工作来解析和分组过滤器。您可能需要编写一个定制的 URL 参数绑定器或解析器来将查询字符串键分成两个部分:字段名称和操作符。然后您需要`GROUP BY`(属性名，操作符)。

*   变量名中的特殊字符可能很难使用。您可能需要编写一个定制的绑定器来将查询字符串键分成两个部分:字段名称和操作符。

*   难以管理定制的组合过滤器。具有相同属性名和运算符的多个筛选器会导致隐式 and。如果 API 用户想用过滤器来代替。即查找价格低于 10 或高于 100 的所有商品？

### RHS 冒号

与括号方法类似，您可以设计一个 API，让操作员使用 RHS 而不是 LHS。例如，`GET /items?price=gte:10&price=lte:100`将查找价格大于或等于 10 但小于或等于 100 的所有商品。

#### 利益

*   最容易在服务器端解析，尤其是在不支持重复过滤器的情况下。不需要自定义绑定。许多 API 框架已经处理了 URL 参数数组。多个价格过滤器将位于同一个变量“价格”下，该变量可以是序列或映射。

#### 下降趋势

*   文字值需要特殊处理。例如，`GET /items?user_id=gt:100`将被翻译成查找*用户 id* 大于 100 的所有项目。然而，如果我们想找到所有用户 id 等于 *gt:100* 的项目，因为这可能是一个有效的 id，该怎么办呢？

### 搜索查询参数

如果需要在端点上进行搜索，可以直接使用 search 参数添加对过滤器和范围的支持。如果您已经在使用 ElasticSearch 或其他基于 Lucene 的技术，您可以直接支持 Lucene 语法或 ElasticSearch 简单查询字符串。

例如，我们可以搜索包含术语*红椅子*且价格大于或等于 10 且小于或等于 100: `GET /items?q=title:red chair AND price:[10 TO 100]`的商品

这样的 API 可以允许模糊匹配，提升某些术语，等等。

#### 利益

*   API 用户最灵活的查询

*   后端几乎不需要解析，可以直接传递给搜索引擎或数据库(只是要注意净化输入的安全性)

#### 下降趋势

*   对于初学者来说，开始使用 API 更难。需要熟悉 Lucene 语法。

*   全文搜索并不是对所有资源都有意义。例如，模糊性和术语提升对于时间序列度量数据没有意义。

*   需要 URL 百分比编码，这使得使用 cURL 或 Postman 更加复杂。

> *什么是 Moesif？ [Moesif](https://www.moesif.com/) 是最先进的 REST API 分析平台，被成千上万的平台用来了解您的客户如何使用您的 API 以及他们最常用的过滤器。Moesif 拥有流行 API 网关的 SDK 和插件，如[孔](https://docs.konghq.com/hub/moesif/kong-plugin-moesif/)、 [Tyk](https://tyk.io/docs/configure/tyk-pump-configuration/#moesif-config) 和 [more](https://www.moesif.com/docs/server-integration/) 。*

## 页码

大多数返回实体列表的端点都需要某种分页。

如果没有分页，一个简单的搜索可能会返回数百万甚至数十亿次点击，从而导致额外的网络流量。

分页需要隐含的排序。默认情况下，这可能是项目的唯一标识符，但也可以是其他有序字段，如创建日期。

### 偏移分页

这是最简单的分页形式。Limit/Offset 在使用 SQL 数据库的应用程序中变得很流行，SQL 数据库已经将 Limit 和 Offset 作为 SQL SELECT 语法的一部分。实现限制/偏移分页只需要很少的业务逻辑。

极限/偏移分页类似于`GET /items?limit=20&offset=100`。该查询将返回从第 100 行开始的 20 行。

#### 例子

(假设查询按创建日期降序排序)

1.  客户请求最近的项目:`GET /items?limit=20`
2.  在滚动/下一页，客户发出第二个请求`GET /items?limit=20&offset=20`
3.  在滚动/下一页，客户发出第三个请求`GET /items?limit=20&offset=40`

作为一条 SQL 语句，第三个请求如下所示:

```py
SELECT
    *
FROM
    Items
ORDER BY Id
LIMIT 20
OFFSET 40; 
```

#### 利益

*   最容易实现，除了将参数直接传递给 SQL 查询之外，几乎不需要编码。

*   服务器上的无状态。

*   不管自定义的 *sort_by* 参数如何，都有效。

#### 下降趋势

*   对大偏移值无效。假设您执行一个偏移量为 1000000 的查询。数据库需要从 0 开始扫描和计数行，并跳过前 1000000 行(即丢弃数据)。

*   向表格中插入新项目时不一致(即页面漂移),当我们首先按最新项目排序时，这一点尤其明显。考虑以下按 Id 降序排序的情况:

    1.  查询`GET /items?offset=0&limit=15`
    2.  表中增加了 10 个新项目
    3.  查询`GET /items?offset=15&limit=15`第二个查询将只返回 5 个新项目，因为添加 10 个新项目会将偏移量向后移动 10 个项目。要解决这个问题，客户机确实需要为第二个查询`GET /items?offset=25&limit=15`偏移 25，但是客户机不可能知道其他对象被插入到表中。

即使有限制，偏移分页也易于实现和理解，并且可以用于数据集上限较小的应用程序中。

### 键集分页

键集分页使用上一页的过滤器值来获取下一组项目。这些列将被索引。

#### 例子

(假设查询按创建日期降序排序)

1.  客户请求最近的项目:`GET /items?limit=20`
2.  在滚动/下一页，客户端从先前返回的结果中找到 *2021-01-20T00:00:00* 的最小创建日期。然后使用日期作为过滤器进行第二次查询:`GET /items?limit=20&created:lte:2021-01-20T00:00:00`
3.  在滚动/下一页，客户端从先前返回的结果中找到 *2021-01-19T00:00:00* 的最小创建日期。然后使用日期作为过滤器进行第三次查询:`GET /items?limit=20&created:lte:2021-01-19T00:00:00`

```py
SELECT
    *
FROM
    Items
WHERE
  created <= '2021-01-20T00:00:00'
ORDER BY Id
LIMIT 20 
```

#### 利益

*   与现有的过滤器一起工作，无需额外的后端逻辑。只需要一个额外的*限制* URL 参数。

*   即使在表格中插入新的项目，也能保持一致的排序。按最近的优先排序时效果很好。

*   即使失调较大，也能保持一致的性能。

#### 下降趋势

*   分页机制与过滤器和排序的紧密耦合。强制 API 用户添加过滤器，即使没有过滤器。

*   不适用于低基数字段，如枚举字符串。

*   当使用定制的 *sort_by* 字段时，对于 API 用户来说很复杂，因为客户端需要根据用于排序的字段来调整过滤器。

对于具有单个自然高基数键的数据，例如可以使用时间戳的时间序列或日志数据，键集分页非常适用。

### 寻找页码

寻道分页是键集分页的扩展。通过添加一个 *after_id* 或 *start_id* URL 参数，我们可以消除分页与过滤器和排序之间的紧密耦合。由于唯一标识符天生具有高基数，所以我们不会遇到像按低基数字段(如状态枚举或类别名称)排序那样的问题。

基于 seek 的分页的问题是，当需要自定义排序顺序时，很难实现。

#### 例子

(假设查询按创建日期升序排序)

1.  客户请求最近的项目:`GET /items?limit=20`
2.  在滚动/下一页，客户端从先前返回的结果中找到最后一个 id“20”。然后使用它作为起始 id 进行第二次查询:`GET /items?limit=20&after_id=20`
3.  在滚动/下一页，客户端从先前返回的结果中找到最后一个 id“40”。然后使用它作为起始 id 进行第三次查询:`GET /items?limit=20&after_id=40`

Seek pagination 可以提炼为一个`where`子句

```py
SELECT
    *
FROM
    Items
WHERE
  Id > 20
LIMIT 20 
```

如果通过 *id* 进行排序，上面的例子工作得很好，但是如果我们想要通过*电子邮件*字段进行排序呢？对于每个请求，后端需要首先获取标识符与 after_id 匹配的项目的电子邮件值。然后，使用该值作为`where`过滤器执行第二次查询。

让我们考虑一下查询`GET /items?limit=20&after_id=20&sort_by=email`，后端需要两个查询。第一个查询可以是 O(1)查找哈希表，以获得电子邮件枢纽值。这被输入到第二个查询中，只检索邮件在我们的 *after_email* 之后的项目。我们按照电子邮件和 id 这两个列进行排序，以确保在两个电子邮件相同的情况下排序稳定。这对于基数较低的字段至关重要。

1.

```py
SELECT
    email AS AFTER_EMAIL
FROM
    Items
WHERE
  Id = 20 
```

2.

```py
SELECT
    *
FROM
    Items
WHERE
  Email >= [AFTER_EMAIL]
ORDER BY Email, Id
LIMIT 20 
```

#### 利益

*   分页逻辑与过滤逻辑之间没有耦合。

*   即使在表格中插入新的项目，也能保持一致的排序。按最近的优先排序时效果很好。

*   即使失调较大，也能保持一致的性能。

#### 下降趋势

*   相对于基于偏移量或基于键集的分页，后端实现起来更复杂

*   如果从数据库中删除项目， *start_id* 可能不是有效的 id。

Seek 分页是一个很好的整体分页策略，也是我们在 [Moesif 公共 API](https://www.moesif.com/docs/api) 上实现的。它需要在后端多做一点工作，但确保不会给 API 的客户端/用户增加额外的复杂性，同时即使在较大的寻道中也能保持高性能。

## 整理

像过滤一样，排序对于任何返回大量数据的 API 端点来说都是一个重要的特性。如果您要返回一个用户列表，您的 API 用户可能希望按照最后修改日期或电子邮件进行排序。

为了支持排序，许多 API 都添加了一个 *sort* 或 *sort_by* URL 参数，该参数可以将字段名作为值。

然而，好的 API 设计能够灵活地指定*升序*或*降序*顺序。像过滤器一样，指定顺序需要将三个组件编码成一个键/值对。

### 示例格式

*   `GET /users?sort_by=asc(email)`和`GET /users?sort_by=desc(email)`

*   `GET /users?sort_by=+email`和`GET /users?sort_by=-email`

*   `GET /users?sort_by=email.asc`和`GET /users?sort_by=email.desc`

*   `GET /users?sort_by=email&order_by=asc`和`GET /users?sort_by=email&order_by=desc`

### 多列排序

不建议使用排序和顺序不配对的最后一种设计。您最终可能允许按两列或更多列排序:

```py
SELECT
    email
FROM
    Items
ORDER BY Last_Modified DESC, Email ASC
LIMIT 20 
```

要对这种多列排序进行编码，您可以允许多个字段名称，例如

`GET /users?sort_by=desc(last_modified),asc(email)`或者

`GET /users?sort_by=-last_modified,+email`

如果排序字段和排序没有配对，则需要保留 URL 参数排序；否则，什么样的排序应该与什么样的字段名配对就不明确了。然而，许多服务器端框架一旦被反序列化为映射，就可能无法保持顺序。

您还必须确保为任何缓存键考虑 URL 参数排序，但这将对缓存大小造成压力。

## 结论

良好的 API 设计是开发人员体验(DX)的重要组成部分。API 规范可以比许多底层服务器实现更持久，这需要考虑 API 的未来用例。