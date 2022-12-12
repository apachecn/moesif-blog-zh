# 如何用 Moesif 为 Postman 自动创建 API 测试

> 原文：<https://www.moesif.com/blog/technical/api-testing/How-to-Automate-Creating-API-Tests-for-Postman-with-Moesif/>

在创建 API 时，我们都有自己喜欢的工具。这里的主要玩家之一是[邮差](https://www.getpostman.com/)。一个图形化的 HTTP API 客户端，允许我们构造、发送和保存对我们选择的 API 的请求。

Moesif API Analytics 甚至允许我们[从用户收集的真实请求中生成邮递员集合](https://www.moesif.com/docs/api-analytics/run-in-postman/)。

## 保存和管理请求

保存请求并在集合和文件夹中管理它们的能力是测试自动化的基础，因为对于每个请求、文件夹和集合，我们还可以保存两个脚本。Postman 在请求发生前后执行这些脚本。

这些集合可以导出为 JSON 文件，并通过一个名为 Newman 的 CLI 工具运行。这个导出允许我们将所有的请求版本化，用一个友好的 UI 工具为它们编写测试，然后将它们集成到我们的 CI/CD 管道中。

为了将测试脚本与 Postman 请求集成，我们必须首先创建一个新的集合。对于一个简单的 API，我们可以对整个 API 使用一个集合。

要创建集合，我们必须执行以下步骤:

1.  点击左侧工具条中的**收藏**标签
2.  点击**收藏**选项卡内的**新收藏**
3.  输入新集合的名称，如*“我的 API”*
4.  点击**创建**

有了保存请求的集合，我们现在可以定义这些请求。

要创建请求，我们必须遵循以下步骤:

1.  右键单击新集合
2.  选择**添加请求**
3.  输入请求名称
4.  点击**保存到我的 API**

新的请求显示在我们创建的新集合下的左侧栏中。它不会有任何目标网址，所以目前无法发送。

如果我们点击请求，我们可以**输入请求 URL** 。让我们输入`example.com`，点击**发送**！。

Postman 向`example.com`服务器发送一个`GET` HTTP 请求，然后服务器用一个 HTML 页面进行响应。

如果我们现在单击 **Save** ，我们的请求将在我们之前创建的集合中更新。

## 集成测试和请求

现在我们已经保存了我们的请求，我们可以在每次启动 Postman 时加载它，然后再做一次。

我们甚至可以通过点击左侧工具条中收藏右下角的 **…** 并选择**导出**来导出它所在的收藏。该导出创建一个 JSON 文件；我们可以与其他开发人员共享，并将它放入版本控制中。

我们要做的下一件事是向我们的请求添加脚本。

有两种类型的脚本:

1.  **预请求脚本**，在发送请求之前执行
2.  **测试**，在收到响应后执行

如果我们选择我们的请求，我们可以在输入目标 URL 的文本字段下方看到**预请求脚本**和**测试**选项卡。

让我们通过选择 **Test** 选项卡并将以下代码粘贴到文本区域来添加一个测试脚本:

```py
pm.test("response is ok", () => {
  pm.response.to.have.status(200);
}); 
```

当我们点击右上角的**发送**时，代码在 Postman 收到响应后执行。在这个测试中，我们简单地检查响应是否有一个`200`状态。

我们可以在点击**发送**后看到这个测试的结果；我们只需要向下滚动；在文本区域下，我们用来输入测试。在**测试结果(1/1)** 标签中。

当我们点击 **Save** 时，除了我们的请求之外，我们的测试也被保存在我们的集合中。如果我们稍后导出它，它可以作为一个字符串在我们的集合 JSON 中找到。

## 写作测试

在文本区的右边，我们可以找到代码片段和 Postman 脚本文档的链接。这些代码片段也适用于**预请求脚本**；它们可以用来动态地生成我们请求的一部分，比如头或参数。

让我们在一个代码片段的帮助下编写一个稍微复杂一点的测试！

为此，我们首先删除我们的测试代码，然后单击文本区域右侧的代码片段。

选择**响应体:包含字符串**

这会生成以下代码:

JavaScript ` ` pm . test(" Body matches string "，function(){ pm . expect(pm . response . text())). to . include(" string _ you _ want _ to _ search ")；});

```py
This snippet extracts the text of our response and tries to find a specific string in it.

To find an actual string in our response from `example.com`, we need to scroll down and select the **Body** tab.

Here we see the HTML that got delivered. We can select a part of it and use it in our test instead of the `"string_you_want_to_search"` placeholder.

Let's select **This domain is for use in illustrative examples in documents.** and update our test, so it looks like the following:

```javascript
const expectedText = "This domain is for use in illustrative examples in documents.";
pm.test("Body matches string", function () {
  pm.expect(pm.response.text()).to.include(expectedText);
}); 
```py

如果我们再次点击**发送**，我们可以在**测试结果**下看到我们的测试工作正常。

## 导出测试

下一步是导出我们的测试，这样我们就可以在 CI/CD 管道中使用它。为此，我们需要导出我们的集合，其中包括我们的测试。

通过点击左侧栏中我们收藏的省略号( **…** )并从上下文菜单中选择 **Export** 来导出收藏。

在对话框中，我们选择**收藏 v2.1** ，因为邮递员建议我们这样做，然后点击**导出**。

导出格式有点粗糙，但易于阅读。这些测试是带有 JavaScript 的 JSON 字符串，因此它们可以用编辑器来操作，但是使用 Postman 至少给了我们一些 IDE 特性。

## 通过 Newman CLI 运行测试

现在我们已经以 JSON 格式导出了我们的集合，我们可以使用这个导出的文件在命令行中运行我们的测试。

名为 [newman](https://github.com/postmanlabs/newman) 的 CLI 工具用于在命令行中运行 Postman 集合。

当我们用`run`命令执行它并提供我们导出的集合文件的路径时，它试图发送我们定义的所有请求并执行测试脚本。

对于我们的 **My API** 集合，可能是这样的:

```
$ npx newman run My\ API.postman_collection.json 
```

如果 NPM 不可用，也可以在全球范围内安装纽曼。

## 使用 Moesif 创建邮递员集合

Moesif 允许我们从真实的用户请求中[生成邮递员集合](https://www.moesif.com/docs/api-analytics/run-in-postman/)。这允许我们用真实数据在本地调试我们的 API！我们也可以稍后在我们的测试套件中集成请求，在我们的 CI/CD 管道中与 newman 一起运行它，因此我们可以确保过去导致问题的请求不是任何回归错误的一部分。

要使用 Moesif 创建集合，我们必须遵循以下步骤:

1.  登录 Moesif
2.  点击顶部的 **API 分析**选项卡
3.  选择我们希望包含在集合中的请求
4.  点击**运行邮递员**
5.  点击**下载邮差合集**

然后可以将生成的集合导入到 Postman UI 应用程序中，然后添加测试或修改请求，以便它们可以与 Newman CLI 工具一起使用。

## 结论

Postman 是一个非常通用的工具，可以帮助我们构建和测试我们的 API。如果我们导出我们的集合并为它们编写测试，Postman 与 Newman 一起会变得更加强大。

我们可以用友好的用户界面编写测试，将它们导出到 JSON，这样它就可以和我们的 API 代码一起存储。导出允许轻松的版本控制和与 CI/CD 管道的集成。

Moesif 通过允许我们为 Postman 导出从真实用户请求生成的请求集合，降低了创建这些测试的工作量，因此我们可以通过用真实数据替换假设来强化我们的测试套件。

当新开发人员检查我们的代码时，他们可以简单地启动 Postman 并运行一些带有相应测试的请求来完成他们的工作，因此它甚至有助于入职。