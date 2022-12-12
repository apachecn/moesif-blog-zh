# 如何用嵌入的指标展示你的 API 的商业价值

> 原文：<https://www.moesif.com/blog/business/dashboards/How-To-Show-The-Business-Value-Of-Your-APIs-With-Embedded-Metrics/>

当你向你的客户提供 API 的时候，你希望确保他们从中获得价值。同时，最好的 API 被设计成完全自动化，不需要人工干预。这可能会让您的客户不知道您的 API 是否被组织使用，以及您是否满足企业合同中的任何 SLA 义务。

## 要显示的度量类型

大多数 API first 公司都有某种类型的*开发者门户*供客户登录、管理 API 密钥和定制特性。这个区域也是向您的客户展示关键指标的好方法，展示他们从您的 API 中获得了多少价值。这可以是一个简单的计数器，显示一个计费周期内进行的 API 事务的数量，或者提供有关这些事务的其他指标。每个客户都有他们想要查看的不同指标。开发人员希望查看访问日志，因为产品和工程领导对使用和性能指标更感兴趣。最后，财务部门可能需要查看容量和财务规划的计费使用情况。

### 提供 API 日志

向开发人员展示你的同情心的一个快速简单的方法是提供 API 日志。工程师们现在正在集成无数的 API 来发布他们的应用程序。他们没有时间去理解你的 API 创建的每一个响应或错误代码。然而，拥有详细的结构化日志对于调试集成问题非常有帮助。您可以轻松地直接在开发人员门户中提供 API 日志，而不是依赖您的客户来检测您的集成。如果您采用这种方法，请通过包括以下功能来确保您的访问日志是有用的:

*   结构化数据，使开发人员能够快速向下钻取，而无需缓慢搜索
*   除了请求参数(如客户端位置或 API 版本)之外，还包括上下文
*   高级过滤器支持创建包含正文字段的复合查询
*   Curl、Swagger UI 和 Postman 等工具中的重放功能
*   可以与开发人员支持共享的事件 id

### 显示 API 用法

虽然业务和工程领导可能需要检查 API 日志本身，但他们渴望拥有正确的指标来做出明智的决策。如果他们赞助一个新项目，他们希望确保他们的钱花得值。提供平台使用指标有助于展示企业领导者获得的价值。

![Embedded API logs](img/d4bc532fe6472451d649cfd1c60fec42.png)

API 调用本身并不显示价值，但是它们代表了某种商业交易，无论是付款还是发送文本消息来提醒终端用户有货物。因此，请确保您的使用情况报告可以显示给定时间段内的历史事务数量。对使用情况报告的要求包括:

*   能够看到使用不同的时间间隔，如小时/天/月
*   显示业务交易的数量，即使您的 API 可以批量处理请求
*   一种按用户统计细分使用情况的方法

### 显示性能指标

虽然有些 API 仅由内部应用程序使用，但许多 API 集成在面向客户的应用程序中。如果您的 API 有性能或可靠性问题，这会影响他们的最终用户。这可能会损害他们的声誉或导致收入损失。如果您有义务满足某些 SLA(服务水平协议)，请向您的客户展示这一点。这可以让您放心，因为您知道您的 API 是可靠和高性能的。

![Embedded SLA metrics](img/6a6b35d2316186ea315f37dae22dde5d.png)

### 提供计费仪表板

大多数 API 利用基于消费的计费。如果你正在提供一个支付 API，这可能是每月的交易数量或美元交易量的百分比。然而，如果您提供消息传递 API，您可能会根据一个计费周期内发送的消息数量来计费。您的客户会希望优化他们的使用来最大化您的 API 的价值，这意味着您应该显示他们的钱花在了哪里。请记住，您应该有一个一致的真实账单来源，否则您的支持团队将会因为账单差异而收到大量请求。

除了计费之外，您还应该显示您的客户受到的任何费率限制或配额。事实上，这可以是一种简单的方法，通过显示它们离当前的限制有多近来追加销售更大的计划。因为许多业务团队不知道或者不登录您的开发人员门户，所以您也应该发出通知，让您的客户在配额被突破时知道。

![Subscription quota emails](img/e34ce37952c8d0051e65d45a2ba82ba6.png)

## 如何开始使用嵌入式图表

使用 Moesif，您可以在几分钟内嵌入 API 日志和使用报告。集成有三个部分

1.  使用正确的视图和过滤器在 Moesif UI 中创建一个嵌入模板。您还应该选择动态变量名，如`user_id`或`company_id`，它们将在第二步中设置
2.  后端的一个 API 调用 Moesif 管理 API 来获得一个有时间限制的工作区 URL。这将返回到您的前端
3.  使用带有生成的工作区 URL 的 iFrame 嵌入图表。

一个使用 Moesif 嵌入模板的 React 应用程序[在这里](https://github.com/Moesif/moesif-browser-embedded-api-dashboard)可用。

要嵌入图表或 API 日志:

### 第一步。在 Moesif 中创建新的嵌入模板

1.  从顶部菜单转到*事件*并选择您想要嵌入的视图。
2.  添加要应用于嵌入图表的任何筛选器和其他图表设置。
3.  点击右上角的*共享*按钮，然后选择*嵌入模板*选项卡。

![Creating embedded API logs](img/b91693c662d448ce1d618b22c18b592f.png)

1.  添加任何动态变量，如用户 Id 或公司 IdT2。
2.  点击*获取嵌入代码*，您将看到要添加到您的应用程序中的代码片段(如下所示)。

### 第二步。生成工作区 URL

创建一个新的端点，调用 Moesif API 来生成工作区 URL。当您调用 Moesif API 时，设置您在步骤 1 中添加到模板中的动态变量。在本例中， *user_id* 是一个需要填充正确值的动态字段。

```py
curl -X POST -H 'Authorization: Bearer YOUR_MANAGEMENT_API_KEY' -H 'Content-Type: application/json' -i 'https://api.moesif.com/v1/portal/~/workspaces/{WORKSPACE_ID}/access_token
  -d '{
  "template": {
    "values": {
      "user_id": "1234"
    }
  }
}' 
```

Moesif 返回一个工作区 URL。该 URL 只能访问步骤 1 定义的数据。在这个例子中，数据的范围是用户 id `1234`。

```py
{
  access_token: '{WORKSPACE_ACCESS_TOKEN}',
  url: 'https://www.moesif.com/public/{WORKSPACE_ACCESS_TOKEN}/ws/{WORKSPACE_ID}?embed=true
} 
```

### 第三步。添加 iFrame

使用步骤 2 中生成的工作区 URL 向前端添加 iFrame。具有工作区 URL 的用户只能访问基于您在步骤 2 中设置的动态变量的数据。

```py
<iframe
  id="preview-frame"
  src="https://www.moesif.com/public/{WORKSPACE_ACCESS_TOKEN}/ws/{WORKSPACE_ID}?embed=true"
  name="preview-frame"
  frameborder="0"
  noresize="noresize">
</iframe> 
```

## 结论

展示你的客户所获得的价值可以帮助你的客户走向成功。使用 Moesif，只需几行代码就可以轻松地将 API 指标和日志嵌入到您的应用程序中。