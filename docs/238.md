# 解决 Chrome 扩展中的内容安全策略问题

> 原文：<https://www.moesif.com/blog/engineering/chrome-extensions/Working-Around-Content-Security-Policy-Issues-in-Chrome-Extensions/>

之前，我们讨论了一个用例，一个 [Chrome 扩展通过*脚本*标签将一个脚本注入到网页](/blog/technical/apirequest/How-We-Captured-AJAX-Requests-with-a-Chrome-Extension/)中。该脚本将在该网页的上下文中运行，以便扩展可以访问资源并与网页共享 javascript 对象。然而，一些网页有一个内容安全策略，阻止 AJAX 调用不属于白名单的域。这篇文章解释了我们解决这个问题的方法。

## 执行上下文

本质上有三个执行上下文，其中每一个都是几乎完全隔离的环境。

*   **网页**的执行环境，这些包括网站最初加载的任何脚本，或者添加到文档的 DOM 中的*脚本*标签所包含的任何内容。在此上下文中运行的任何脚本都受网页的原始内容安全策略的约束。此外，你不能直接访问任何 Chrome 扩展资源。([查看如何将脚本加载到这个环境中](/blog/technical/apirequest/How-We-Captured-AJAX-Requests-with-a-Chrome-Extension/))

*   **内容脚本**的执行环境。它们是由`chrome.tabs.executeScript()`启动的脚本。内容脚本可以操作主机网页的 DOM。在这个执行上下文中，您不能访问网页上的任何 javascript 对象或函数。但它仍然可以访问 chrome 扩展资源，如`chrome.runtime`或`chrome.tabs`。

*   **Chrome 扩展**本身的执行环境。

## APIRequest.io Chrome 扩展的背景

创建了 APIRequest.io Ajax Capture Chrome 扩展，以捕获单页面应用程序的请求和响应(以使这些应用程序的协作和调试更加容易)。在这个扩展存在之前，据我们所知，由于 [Chrome WebRequest API](https://developer.chrome.com/extensions/webRequest) 的限制，没有可以捕捉响应的扩展。我们找到的解决方案包括使用一个**脚本**标签将一个脚本注入到网页的上下文中，如本文[中所讨论的](/blog/technical/apirequest/How-We-Captured-AJAX-Requests-with-a-Chrome-Extension/)

但是，由于时间限制，最初的版本没有添加与内容安全策略的兼容性。因此，在网页的执行上下文中，如果原始网页的内容安全策略不允许我们与不属于原始白名单的域进行通信，我们就不能进行 AJAX 调用(需要将捕获的数据存储在持久化和可共享的链接中)。

## 解决办法

为了与任意内容安全策略兼容，解决方案是将数据传递到另一个不受内容安全策略约束的执行上下文，执行 AJAX 调用，并处理结果。

### 网页上下文和内容脚本之间的消息传递。

这涉及到使用`window.postMessage()`

#### 发送消息

```py
 const domain = window.location.protocol + '//' + window.location.hostname + ':' + window.location.port;
  // console.log(domain);
  window.postMessage({ type: 'API_AJAX_CALL', payload: payload}, domain); 
```

`domain`变量是消息将被发送到的网页。因为网页和内容脚本实际上是在同一页面上执行的，所以通过使用主机网页的同一域，消息将被传递给内容脚本，反之亦然。

虽然可以做`window.postMessage(data, '*')`，但是“*”暗示任何页面都可以读取消息，这可能是危险的。如果消息是敏感的，您不希望恶意网页(在其他选项卡中)看到这些消息。

#### 1.接收消息并进行 AJAX 调用

在内容脚本上下文中，我们不受内容安全策略的约束，我们可以接收消息并进行 API 调用。

```py
window.addEventListener("message", function(event) {
  // We only accept messages from ourselves
  if (event.source != window)
    return;

  // console.log("Content script received event: " + JSON.stringify(event.data));

  if (event.data.type && (event.data.type == "API_AJAX_CALL")) {
    //make my ajax call here with the payload.
    const request = superagent.post(myAPIEndPointUrl)

    request.send(event.data.payload)
      .end(function (err, res) {
          returnAjaxResult(err, res.body)
      });
  }
}, false); 
```

由于发送消息的`window`是同一个窗口，我们应该在接受消息之前检查以确保它是相同的。这确保了我们知道消息来自哪里。

#### 2.将 Ajax 的结果返回到原始上下文

`windows.postMessage`没有回调方法。因此，要将 AJAX 结果传递回原始网页，我们必须再次使用`windows.postMessage`。

```py
function returnAjaxResult(err, resbody) {
  const domain = window.location.protocol + '//' + window.location.hostname + ':' + window.location.port;
  // console.log(domain);
  window.postMessage({ type: 'API_AJAX_RESULT', payload: {error: err, responsebody: resbody}}, domain);
} 
```

通过这种方式，内容脚本就像是对不在内容安全策略中的域的 AJAX 调用的代理。

### ContentScript 和扩展之间的消息传递

因为两者都可以访问 Chrome 扩展相关的对象，所以你可以使用这些资源

从内容脚本到扩展:

```py
chrome.runtime.sendMessage({payload: playload}, function(response) {
  // callback

}); 
```

将消息从扩展传递到内容脚本更有趣，因为这取决于您执行内容脚本的选项卡。(chrome.tabs.executeScript 也需要一个`tabId`，所以你只要记住就可以了。)

```py
 chrome.tabs.sendMessage(tabId, {playload: playload}, function(response) {
    // callback

  }); 
```

消息传递也有回叫，这使得处理起来容易了许多。

## 结束语

我们的重点不是构建 chrome 扩展，但作为我们自己使用的一个附带项目工具，这绝对是一个有趣的项目。对于这个内容安全策略问题，我曾搁置了一段时间，以代替时间限制，但是后来一个用户发消息给我说，他能够使用消息传递来解决这个问题。我们很高兴其他人也发现我们的附带项目很有用，因为我们经常为我们自己的单页应用程序使用 [APIRequest.io Capture Chrome 扩展](https://chrome.google.com/webstore/detail/apirequestio-ajax-capture/aeojbjinmmhjenohjehcidmappiodhjm)和我们非常受欢迎的 [Moesif CORS 扩展](https://chrome.google.com/webstore/detail/moesif-origin-cors-change/digfbfaphojjndkpccljibejjbppifbc?hl=en)。