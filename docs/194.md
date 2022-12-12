# 我们如何构建 Node.js 中间件来记录 HTTP API 请求和响应

> 原文：<https://www.moesif.com/blog/technical/logging/How-we-built-a-Nodejs-Middleware-to-Log-HTTP-API-Requests-and-Responses/>

有许多不同的运行时和生态系统用于构建 API，在 Moesif，我们试图尽可能简单地与它们集成。我们构建了许多有助于这种集成的库，其中之一是 [Moesif Express 中间件库](https://github.com/Moesif/moesif-express)，或简称 Moesif-Express。

尽管名为 Moesif-Express，但它可以与使用内置`http`模块的 Node.js 应用程序一起使用。

Node.js 异步处理请求，这有时会导致问题，尤其是当我们想要调试系统或记录它们正在做什么的时候。

在本文中，我们将介绍构建 Moesif-Express 库的步骤、相关日志数据的位置以及如何挂钩 Node.js' `http`模块，以便在管道中的正确时间处理数据收集。

## Node.js' HTTP 模块

Node.js 附带了一个现成的 HTTP-server 实现，虽然它并不直接用于大多数应用程序，但它是理解请求和响应的基础的一个良好开端。

## GET 请求的基本日志记录

日志背后的想法是，我们将某种类型的数据写入某个持久数据存储中，以便我们可以在以后查看它。为了简单起见，我们先用`console.log()`写`stdout`。

> 有时记录到`stdout`不是一个选项，我们必须将记录数据发送到其他地方。就像在无服务器环境中运行时一样，其中的函数没有持久存储。

让我们尝试一个简单的服务器，它除了为每个请求发送空响应之外什么也不做，以说明如何获取可以记录的请求数据。

```py
const http = require("http");

const server = http.createServer((request, response) => {
  console.log(request);
  response.end();
});

server.listen(8888); 
```

如果我们向 [http://localhost:8888](http://localhost:8888) 发送一个请求，我们会看到一个巨大的对象被记录到`stdout`，它充满了实现细节，找到重要的部分并不容易。

让我们看看 Node.js 文档中的 [IncomingMessage](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage) ，我们的请求是这个类的一个对象。

在这里我们能找到什么信息？

*   `headers`和`rawHeaders`(无效/重复标题)
*   `httpVersion`
*   `method`
*   `url`
*   `socket.remoteAddress`(针对客户端 IP)

这对于 GET 请求应该足够了，因为它们通常没有主体。让我们更新我们的实现。

```py
const server = http.createServer((request, response) => {
  const { rawHeaders, httpVersion, method, socket, url } = request;
  const { remoteAddress, remoteFamily } = socket;

  console.log(
    JSON.stringify({
      timestamp: Date.now(),
      rawHeaders,
      httpVersion,
      method,
      remoteAddress,
      remoteFamily,
      url
    })
  );

  response.end();
}); 
```

输出应该如下所示:

```py
{  "timestamp":  1562331336922,  "rawHeaders":  [  "cache-control",  "no-cache",  "Postman-Token",  "dcd81e98-4f98-42a3-9e13-10c8401892b3",  "User-Agent",  "PostmanRuntime/7.6.0",  "Accept",  "*/*",  "Host",  "localhost:8888",  "accept-encoding",  "gzip, deflate",  "Connection",  "keep alive"  ],  "httpVersion":  "1.1",  "method":  "GET",  "remoteAddress":  "::1",  "remoteFamily":  "IPv6",  "url":  "/"  } 
```

对于我们的日志记录，我们只使用请求的特定部分。它使用 JSON 作为格式，所以这是一种结构化的日志记录方法，并且有时间戳，所以我们不仅知道谁请求了什么，还知道请求是何时开始的。

## 记录处理时间

如果我们想添加关于请求处理时间的数据，我们需要一种方法来检查请求何时完成。

当我们发送完响应时，请求就完成了，所以我们必须检查何时调用了`response.end()`。在我们的例子中，这相当简单，但有时这些结束调用是由其他模块完成的。

为此，我们可以看看 [ServerResponse](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse) 类的文档。它提到了一个`finish`事件，当所有的服务器完成发送它的响应时，该事件被触发。这并不意味着客户收到了所有东西，但这表明我们的工作已经完成。

让我们更新我们的代码！

```py
const server = http.createServer((request, response) => {
  const requestStart = Date.now();

  response.on("finish", () => {
    const { rawHeaders, httpVersion, method, socket, url } = request;
    const { remoteAddress, remoteFamily } = socket;

    console.log(
      JSON.stringify({
        timestamp: Date.now(),
        processingTime: Date.now() - requestStart,
        rawHeaders,
        httpVersion,
        method,
        remoteAddress,
        remoteFamily,
        url
      })
    );
  });

  process(request, response);
});

const process = (request, response) => {
  setTimeout(() => {
    response.end();
  }, 100);
}; 
```

我们将请求的处理传递给一个单独的函数来模拟负责处理它的另一个模块。由于`setTimeout`的原因，处理是异步发生的，所以同步日志记录不会得到想要的结果，但是`finish`事件通过在调用 `response.end()`之后触发*来处理这个问题。*

## 记录尸体

请求主体仍然没有被记录，这意味着 POST、PUT 和 PATCH 请求没有被 100%覆盖。

为了将主体也放入日志中，我们需要一种从请求对象中提取它的方法。

[IncomingMessage](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage) 类实现了 [ReadableStream](https://nodejs.org/dist/latest-v10.x/docs/api/stream.html#stream_class_stream_readable) 接口。当来自客户端的主体数据到达时，它使用该接口的事件发出信号。

*   当服务器从客户端接收到新的数据块时，触发`data`事件
*   发送完所有数据后，调用`end`事件
*   出问题时会调用`error`事件

让我们更新我们的代码:

```py
const server = http.createServer((request, response) => {
  const requestStart = Date.now();

  let errorMessage = null;
  let body = [];
  request.on("data", chunk => {
    body.push(chunk);
  });
  request.on("end", () => {
    body = Buffer.concat(body);
    body = body.toString();
  });
  request.on("error", error => {
    errorMessage = error.message;
  });

  response.on("finish", () => {
    const { rawHeaders, httpVersion, method, socket, url } = request;
    const { remoteAddress, remoteFamily } = socket;

    console.log(
      JSON.stringify({
        timestamp: Date.now(),
        processingTime: Date.now() - requestStart,
        rawHeaders,
        body,
        errorMessage,
        httpVersion,
        method,
        remoteAddress,
        remoteFamily,
        url
      })
    );
  });

  process(request, response);
}); 
```

这样，当出现问题时，我们会记录一条额外的错误消息，并将正文内容添加到日志中。

> 注意:主体可能非常大和/或二进制，所以需要验证检查，否则数据量或编码可能会搞乱我们的日志。

## 记录响应数据

既然我们已经记录了请求，下一步就是记录我们的响应。

我们已经监听了响应的`finish`事件，所以我们有一个非常安全的方法来获取所有数据。我们只需要提取响应对象包含的内容。

让我们看看 [ServerResponse](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_serverresponse) 类的文档，看看它为我们提供了什么。

*   `statusCode`
*   `statusMessage`
*   `getHeaders()`

让我们将它添加到代码中。

```py
const server = http.createServer((request, response) => {
  const requestStart = Date.now();

  let errorMessage = null;
  let body = [];
  request.on("data", chunk => {
    body.push(chunk);
  });
  request.on("end", () => {
    body = Buffer.concat(body).toString();
  });
  request.on("error", error => {
    errorMessage = error.message;
  });

  response.on("finish", () => {
    const { rawHeaders, httpVersion, method, socket, url } = request;
    const { remoteAddress, remoteFamily } = socket;

    const { statusCode, statusMessage } = response;
    const headers = response.getHeaders();

    console.log(
      JSON.stringify({
        timestamp: Date.now(),
        processingTime: Date.now() - requestStart,
        rawHeaders,
        body,
        errorMessage,
        httpVersion,
        method,
        remoteAddress,
        remoteFamily,
        url,
        response: {
          statusCode,
          statusMessage,
          headers
        }
      })
    );
  });

  process(request, response);
}); 
```

## 处理响应错误和客户端中止

目前，我们只在响应`finish`事件被触发时记录日志，如果响应出错或者客户端中止请求，情况就不是这样了。

对于这两种情况，我们需要创建额外的处理程序。

```py
const server = http.createServer((request, response) => {
  const requestStart = Date.now();

  let body = [];
  let requestErrorMessage = null;

  const getChunk = chunk => body.push(chunk);
  const assembleBody = () => {
    body = Buffer.concat(body).toString();
  };
  const getError = error => {
    requestErrorMessage = error.message;
  };
  request.on("data", getChunk);
  request.on("end", assembleBody);
  request.on("error", getError);

  const logClose = () => {
    removeHandlers();
    log(request, response, "Client aborted.");
  };
  const logError = error => {
    removeHandlers();
    log(request, response, error.message);
  };
  const logFinish = () => {
    removeHandlers();
    log(request, response, requestErrorMessage);
  };
  response.on("close", logClose);
  response.on("error", logError);
  response.on("finish", logFinish);

  const removeHandlers = () => {
    request.off("data", getChunk);
    request.off("end", assembleBody);
    request.off("error", getError);
    response.off("close", logClose);
    response.off("error", logError);
    response.off("finish", logFinish);
  };

  process(request, response);
});

const log = (request, response, errorMessage) => {
  const { rawHeaders, httpVersion, method, socket, url } = request;
  const { remoteAddress, remoteFamily } = socket;

  const { statusCode, statusMessage } = response;
  const headers = response.getHeaders();

  console.log(
    JSON.stringify({
      timestamp: Date.now(),
      processingTime: Date.now() - requestStart,
      rawHeaders,
      body,
      errorMessage,
      httpVersion,
      method,
      remoteAddress,
      remoteFamily,
      url,
      response: {
        statusCode,
        statusMessage,
        headers
      }
    })
  );
}; 
```

现在，我们也记录错误和中止。

当响应完成时，日志处理程序也被删除，所有的日志都被转移到一个额外的函数中。

## 记录到外部 API

目前，脚本只将日志写入控制台，在许多情况下，这就足够了，因为操作系统允许其他程序捕获`stdout`并使用它做自己的事情，比如写入文件或发送给第三方 API，比如 Moesif。

在某些环境中，这是不可能的，但是由于我们将所有信息收集到一个地方，我们可以用第三方函数替换对`console.log`的调用。

让我们重构代码，使其类似于一个库，并记录到一些外部服务。

```py
const log = loggingLibrary({ apiKey: "XYZ" });
const server = http.createServer((request, response) => {
  log(request, response);
  process(request, response);
});

const loggingLibray = config => {
  const loggingApiHeaders = {
    Authorization: "Bearer " + config.apiKey
  };

  const log = (request, response, errorMessage, requestStart) => {
    const { rawHeaders, httpVersion, method, socket, url } = request;
    const { remoteAddress, remoteFamily } = socket;

    const { statusCode, statusMessage } = response;
    const responseHeaders = response.getHeaders();

    http.request("https://example.org/logging-endpoint", {
      headers: loggingApiHeaders,
      body: JSON.stringify({
        timestamp: requestStart,
        processingTime: Date.now() - requestStart,
        rawHeaders,
        body,
        errorMessage,
        httpVersion,
        method,
        remoteAddress,
        remoteFamily,
        url,
        response: {
          statusCode,
          statusMessage,
          headers: responseHeaders
        }
      })
    });
  };

  return (request, response) => {
    const requestStart = Date.now();

    // ========== REQUEST HANLDING ==========
    let body = [];
    let requestErrorMessage = null;
    const getChunk = chunk => body.push(chunk);
    const assembleBody = () => {
      body = Buffer.concat(body).toString();
    };
    const getError = error => {
      requestErrorMessage = error.message;
    };
    request.on("data", getChunk);
    request.on("end", assembleBody);
    request.on("error", getError);

    // ========== RESPONSE HANLDING ==========
    const logClose = () => {
      removeHandlers();
      log(request, response, "Client aborted.", requestStart);
    };
    const logError = error => {
      removeHandlers();
      log(request, response, error.message, requestStart);
    };
    const logFinish = () => {
      removeHandlers();
      log(request, response, requestErrorMessage, requestStart);
    };
    response.on("close", logClose);
    response.on("error", logError);
    response.on("finish", logFinish);

    // ========== CLEANUP ==========
    const removeHandlers = () => {
      request.off("data", getChunk);
      request.off("end", assembleBody);
      request.off("error", getError);

      response.off("close", logClose);
      response.off("error", logError);
      response.off("finish", logFinish);
    };
  };
}; 
```

有了这些改变，我们现在可以像使用 [Moesif-Express](https://github.com/Moesif/moesif-express) 一样使用我们的日志实现了。

`loggingLibrary`函数将 API-key 作为配置，并返回实际的日志记录函数，该函数将通过 HTTP 将日志数据发送到日志记录服务。日志记录函数本身接受一个`request`和`response`对象。

## 结论

登录 Node.js 并不像人们想象的那样简单，尤其是在 HTTP 环境中。JavaScript 异步处理许多事情，所以我们需要挂钩正确的事件，否则，我们不知道发生了什么。

幸运的是，有许多日志库，所以我们不必自己编写一个。

以下是一些例子:

*   班扬-https://github.com/trentm/node-bunyan
*   调试-https://github.com/visionmedia/debug
*   log4js-https://github.com/log4js-node/log4js-node
*   摩根 https://github.com/expressjs/morgan 公司
*   NPM log-https://github.com/npm/npmlog
*   温斯顿-https://github.com/winstonjs/winston