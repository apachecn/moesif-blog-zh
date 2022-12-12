# 如何在开发人员门户中嵌入 API 调试仪表板

> 原文：<https://www.moesif.com/blog/technical/dashboards/How-To-Embed-an-API-Debug-Dashboard-in-your-Developer-Portal/>

您已经创建了仪表板和报告。也许你也和你的同事合作过。现在，您希望您的客户能够像您的内部团队成员一样了解您的 API。通过在您的开发人员门户中嵌入 API 日志，您可以为您的客户提供简单的入门体验和简单的调试工具，使他们能够尽快集成您的 API，而不会出现问题。

### 什么是 Moesif 工作区？

Moesif 平台有*公共工作区*的概念。这些是在 Moesif 平台中创建的仪表板，就像您为内部团队成员创建的仪表板一样。最大的区别是公共工作空间有自动约束来保护您的数据。数据可以通过像客户的*用户 id* 或*账户 id* 这样的字段来沙箱化。此外，访问公共工作区的用户不需要登录 Moesif。相反，他们可以使用工作区访问令牌或*共享链接*来访问数据。

Moesif 支持多种工作空间类型，包括 API 事件日志、分段、时间序列、热图等。

### 创建模板

Moesif 提供模板来构建强大的图表，并将这些图表嵌入到面向客户的门户中。可以使用动态字段创建模板，以约束要与客户共享的数据。您可以通过登录并点击`Share`按钮并选择`Embed Template`来从您的 API 数据创建一个模板。

![create-template.png](img/04ee1fcdefe2acd957be7f6d0b1f9687.png)

嵌入工作区模板需要两个步骤。首先，一个端点生成具有正确限制的 URL。然后，使用 URL 将图表嵌入面向客户的门户。

## 创建工作区 URL

使用管理 API 键和动态字段生成工作区 URL 的请求。

```py
curl -X POST -H 'Authorization: <Authorization Token>' -H 'Content-Type: application/json' -i 'https://api.moesif.com/v1/portal/~/workspaces/<workspace_id>/access_token
  -d '{
  "template": {
    "values": {
      "user_id": "{YOUR_DYNAMIC_VALUE}"
    }
  }
}' 
```

带有工作区 URL 和工作区访问令牌的响应。

```py
{
  access_token: '{WORKSPACE_ACCESS_TOKEN}',
  url: 'https://www.moesif.com/public/{WORKSPACE_ACCESS_TOKEN}/ws/<workspace_id>?embed=true'
} 
```

*注意:您可以根据您需要的过滤器修改请求体，以限制共享的数据。*

## 嵌入工作区

您可以将工作区嵌入到用户期望看到它们的门户或网站中。Embed 适用于任何支持使用 iFrame 嵌入内容的门户或网站。您可以通过使用上一步中生成的 iframe 来显示工作区。数据访问将限于先前设置的动态值。

```py
<iframe
  id="preview-frame"
  src="https://www.moesif.com/public/{WORKSPACE_ACCESS_TOKEN}/ws/<workspace_id>?embed=true"
  name="preview-frame"
  frameborder="0"
  noresize="noresize">
</iframe> 
```

![Embedded API Dashboard showing latency](img/d684ed75431393ed8f27c93d44945373.png)

对贴有白色标签的工作区感兴趣，请联系 [Moesif 团队](mailto:team@moesif.com)了解更多信息。