# 我们如何从一个带有 Chrome 扩展的网站标签中捕获 AJAX 请求

> 原文：<https://www.moesif.com/blog/technical/apirequest/How-We-Captured-AJAX-Requests-with-a-Chrome-Extension/>

## 背景

我以为会有很多 Chrome 扩展来监控来自 AJAX 调用的 HTTP 请求，但是我们发现的少数几个(比如 Postman Intercept Chrome 扩展)似乎只捕捉请求而不捕捉响应。尽管 Chrome DevTools 有网络选项卡，但很难与队友分享这些捕获的 HTTP 跟踪。

我们的工程师在前端和后端之间划分了职责，因此在调试 API 问题时，我们必须不断地将 HTTP 头和 JSON 有效负载从 DevTools 复制/粘贴到电子邮件或 Slack 中。发送多兆字节 [*。har 文件](http://www.softwareishard.com/blog/har-12-spec/)通过 Slack 也一样麻烦。除了不容易共享之外，在 Chrome DevTools 中检查 HTTP 流量也不理想，因为没有简单的方法来过滤 API 调用或检查 JSON 有效负载。网络标签是为传统的 HTML 网站设计的，而不是现代的返回 JSON 的单页应用和 API。

由于这个时间下沉，我们决定构建一个 [Chrome 扩展](https://chrome.google.com/webstore/detail/apirequestio-capture/aeojbjinmmhjenohjehcidmappiodhjm),使得从任何网站捕获和调试这些 AJAX 请求(和响应)变得非常容易。该扩展是为 REST APIs 设计的，比如那些支持单页面应用程序的 API。然而，由于 Chrome API 的局限性，需要一些猴子业务来捕获 Chrome 扩展 API。

## WebRequest 方法的问题

Chrome WebRequest API 是 Chrome API 扩展集的一部分。

最初，我查看了 Chrome 的 WebRequest APIs，以便从打开的浏览器标签中捕获 API 调用。我们之前为我们的 [CORS 和起源改变者](https://chrome.google.com/webstore/detail/moesif-origin-cors-change/digfbfaphojjndkpccljibejjbppifbc?hl=en)利用了这些 API。这似乎是获取数据最自然的地方。

然而，我们遇到了一个 bug，WebRequest API 没有公开一个接口来读取响应体。见铬的问题。

虽然我们对缺少响应主体 getters 感到惊讶，但我们现在理解了为什么大多数其他扩展不能捕获响应的可能原因。我们的要求是捕捉响应，以便与团队成员共享。如果没有响应，调试会很痛苦，解决问题也会很慢。

## XmlHttpRequest 事件侦听器方法

由于所有浏览器都使用 XmlHttpRequest 进行 AJAX 调用，我利用这一点通过 monkey 补丁来监控 API 调用。

有几个 XmlHttpRequest 侦听器可以挂钩到:

*   负载启动
*   进步
*   流产
*   错误
*   负荷
*   超时
*   装载的；子弹上膛的
*   readystatechange

从侦听器回调中，我可以获得事件目标，也就是 XmlHttpRequest 对象本身。查看 XmlHttpRequest api 文档，我意识到它有与 WebRequest API 完全相反的问题。该对象具有获取 HTTP 响应而不是原始请求的接口。有一个`setRequestHeaders()`，但是没有 getter 方法与之配对来获取请求 HTTP 头。事实上，它甚至没有一个公共方法或属性来获取原始的 URL 或路径。

## XmlHttpRequest 猴子补丁。

为了真正监控所有数据(即请求头/正文和响应头/正文)，我必须对其进行 monkey 修补。猴子补丁允许我们记录数据。

### 修补的主要方法有

*   每当发起一个新的 AJAX 调用时，这个方法就会被调用，然后我可以捕获这个方法和 url。
*   当请求头被设置时，这个方法被调用。我可以给它打补丁，这样每次调用这个方法时，我都会把头保存为 hash。
*   此方法用于将数据作为请求的一部分发送。我可以从这个补丁中捕获请求体。

现在，我可以简单地添加`addEventListener('load', event)`来捕获响应数据。

网上有一些资源，比如本文解释了如何打猴子补丁。

## 执行环境

代码准备好进行猴子修补后，我们必须执行它。 [Chrome 扩展标签 API](https://developer.chrome.com/extensions/tabs) 有一个执行代码的方法:`executeScript()`。然而，`executeScript()`执行的代码与网页上的代码在不同的上下文中执行，网页上的代码是 AJAX 调用的地方。这意味着猴子补丁不会对开放网站产生任何影响。

要了解关于执行环境如何工作的更多信息，请参见关于内容脚本的视频。

## 解决方案:注入脚本

由于孤立的上下文，我不得不采取不同的方法来成功地猴子补丁网站。我不得不使用`chrome.tabs.executeScript()`来创建一个`<script>`标签，注入到网站中，它将在网站的上下文中加载猴子补丁代码。

```py
 const actualCode = `
  var s = document.createElement('script');
  s.src = ${resourceUrl};
  s.onload = function() {
    this.remove();
  };
  (document.head || document.documentElement).appendChild(s);
`;

  chrome.tabs.executeScript(tabId, {code: actualCode, runAt: 'document_end'}, cb); 
```

有几种方法可以将代码添加到脚本标记中。因为我们还想添加一些不错的 UI 元素，所以我们决定将 monkey 补丁代码和 UI 相关代码放入一个脚本文件中，这个脚本文件与`resourceUrl`链接。如果这样做，请确保将资源 url 添加到清单中的`web_accessible_resource`。关于[网络无障碍资源的更多信息](https://developer.chrome.com/extensions/manifest/web_accessible_resources)

## 我们忽略的已知问题

### CSS 泄漏

由于代码是作为网页的一部分运行的，所以我们生成的 UI 会受到页面上已经存在的 CSS 的影响。我们使用 React 和 [Material-UI 库](http://www.material-ui.com/)，它使用内联样式来最小化 CSS 泄漏。然而，CSS 泄露仍然会在一些网站上发生。

### 内容安全策略:content-src

内容安全政策是另一个问题。如果`content-src`设置得非常严格，比如在[GitHub.com](https://github.com/blog/1477-content-security-policy)上，那么它可以阻止对不在白名单中的服务器的 AJAX 调用。由于我们想要记录和存储从 [ApiRequest.io](https://www.apirequest.io) 上的可共享工作区链接中检索的 API 调用，所以我们需要发布到我们自己的 API。这使得 HTTP 跟踪可以保存 30 天，作为以后调试时的参考。

Chrome 扩展执行环境并不完全受此限制。有一个修改内容安全策略的解决方案，将作为单独的文章发布。

目前，除了 GitHub 和 Twitter 等几个大网站之外，许多网站都没有使用如此严格的内容安全政策。我们假设，如果你正在使用我们的 Chrome 扩展，那么它很可能在你自己的网站上使用。如果是这样，那么您可以暂时关闭该限制。如果您对我们的 Chrome 扩展有任何问题或功能要求，请发送电子邮件给我们。

#### 更新:内容安全策略问题已解决

现在 Apirequest.io Chome 扩展可以在具有严格内容安全策略的网站上工作。我们创建了另一篇关于如何围绕内容安全策略工作的文章，它也对 Chrome 扩展的不同执行环境进行了更深入的解释。

## 结束语

我很喜欢创建这个 Chrome 扩展，所以我希望你喜欢使用它。和往常一样，对于使用扩展时的任何功能请求或问题，只需通过[给我们发电子邮件](mailto:hi@moesif.com?subject=On%20ApiRequest%20Chrome%20Extension)让我们知道。

您是否花费大量时间调试客户问题？
Moesif 使 RESTful APIs 和集成应用的调试更加容易

[了解更多信息](https://www.moesif.com?utm_source=blog)