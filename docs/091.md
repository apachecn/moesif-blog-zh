# HIPAA 兼容 API 分析的安全代理

> 原文：<https://www.moesif.com/blog/technical/compliance/Secure-Proxy-for-HIPAA-Compliant-API-Analytics/>

在 HeathTech 应用程序中，通常情况下，你会处理私人或健康相关的数据。这需要遵守法规，例如美国的 HIPAA。这些法规迫使您以一种明确定义的方式处理敏感数据，因此只有特定的人可以读取这些数据，如果他们读取了这些数据，应该将其记录下来以供以后审计。

为了符合 HIPAA，必须在您的公司和应用中实施技术和管理保护措施。技术安全措施通常会导致更复杂的软件架构。因此，在开始之前，确保 HIPAA 合规性的额外工程开发工作是必要的，这是一个好主意。或者，你可能属于不遵守 HIPAA 安全的四种情况之一。

在[的配套文章](https://www.moesif.com/blog/business/compliance/Building-HIPAA-Compliant-APIs/)中，我们解释了如何构建符合 HIPAA 的 API。在本文中，我们将深入探讨如何使用安全代理来保护您的健康数据和您的应用 HIPAA 投诉。在我们的[文档部分](https://www.moesif.com/docs/platform/secure-proxy/)给出了关于如何使用 Moesif 专利安全代理的完整说明。

## 不使用 HIPAA 存储健康数据？

根据定义，如果您的客户在将数据发送给您之前对其进行加密，您就不知道该数据的内容。这导致了可信的否认。如果没有加密密钥，您在自己端存储和处理的数据对潜在的攻击者来说毫无价值。

这可以通过安全代理来完成。

安全代理是您和客户之间的 API 服务器，但位于客户的数据中心。离开客户数据中心的数据会被加密，然后发送到您自己数据中心的 API。

对于加密，代理使用只有您的客户知道的密钥。如果有人窃取了您的加密数据，没有您客户的加密密钥，他们就无法读取这些数据。

## 如何构建一个安全的代理？

第一步是构建一个服务器，作为您自己的 API，但可以由您的客户在内部部署。然后，API 的客户端将与该代理通信，而不是直接向 API 发送请求。

接下来，在将所有客户端数据发送到实际的 API 之前，将它们加密到正确的级别。因此，您的代理需要使用客户提供的加密密钥，而您永远无法访问这个密钥。

正确的加密级别取决于您的 API:

*   获取一个简单的键值存储 API。它只是将所有内容存储在请求正文中，在发送之前加密整个正文就足够了
*   对于任何更复杂的事情，您的代理需要以更复杂的方式处理加密。

以 Moesif API 分析为例。Moesif 的 API 接收事件并计数。如果这个 API 可以为每个请求接收多个事件，那么您必须独立地加密请求中的每个事件名，而不是一次加密整个请求体——否则，API 将无法理解该请求。但是由于事件名称被加密，Moesif 不知道事件是什么。与健康有关吗？是网络游戏里的吗？

最后，当您的客户想要读取数据时，您需要解密数据。因此，代理需要一种向客户显示明文数据的方式。

## 安全代理示例

我已经创建了一个 GitHub 库来说明安全代理的实现。它包含三个部分:一个分析 API 服务器、一个代理 API 服务器和一个客户端。

免责声明:这只是一个基本的例子，在这个例子中，我使用了一个我实际上没有审计的加密库。如果你建立你自己的系统，你将不得不独立地做那部分研究！

说完这些，让我们看看系统的三个部分。

### 分析应用编程接口

我从存储库中提取了分析 API [的基本部分。API 是用 Node.js 和 Express 框架构建的。](https://github.com/kay-is/secure-hipaa-proxy/blob/main/server/analytics.js)

```py
const eventStore = [];

api.post("/event", ({ body }, response) => {
  eventStore.push(body.event);
  response.status(201).json("201 - Created");
});

api.get("/results", (request, response) => {
  const results = eventStore.reduce(
    (results, event) => {
      if (!results[event]) results[event] = 0;
      results[event]++;
      return results;
    },
    {}
  );
  response.status(200).json({ results });
});

api.listen(9000); 
```

有两个端点，一个将接收事件并将其存储在数组中，另一个端点将计算每个事件的接收频率，并将这些总和发送回客户端。这里只是一些基本的东西，数据被接收，一些处理被完成，结果被发送到客户端。

### 分析 API 客户端

接下来，让我们看看客户端实际上是如何使用分析 API 的。同样，我从库中提取了重要的部分。

```py
async function logEvent(event) {
  return axios.post(
    "http://localhost:9000/event",
    { event }
  );
}

async function getResults() {
  const response = await axios.get(
    "http://localhost:9000/results"
  );
  return response.data.results;
}

await logEvent("plaintext-event-a");
await logEvent("plaintext-event-a");
await logEvent("plaintext-event-a");
await logEvent("plaintext-event-b");
await logEvent("plaintext-event-b");

const results = await getResults(); 
```

客户端将带有事件字符串的 JSON 对象发送到 analytics API，该 API 监听端口 9000。它最后还会获取结果。

### 运行示例

如果我们在添加了日志记录的情况下运行这个示例，我们会看到以下内容。

```py
[Client   ] Sending event: plaintext-event-a
[Analytics] Received: plaintext-event-a
[Client   ] Sending event: plaintext-event-a
[Analytics] Received: plaintext-event-a
[Client   ] Sending event: plaintext-event-a
[Analytics] Received: plaintext-event-a
[Client   ] Sending event: plaintext-event-b
[Analytics] Received: plaintext-event-b
[Client   ] Sending event: plaintext-event-b
[Analytics] Received: plaintext-event-b
[Analytics] Sending results:  {
  'plaintext-event-a': 3,
  'plaintext-event-b': 2
}
[Client   ] Results:  {
  'plaintext-event-a': 3,
  'plaintext-event-b': 2
} 
```

客户端将其事件发送到分析 API，分析 API 将其保存在事件存储中。最后，客户端获取结果，这是一个包含每个事件总和的 JSON。

我们看到分析 API 获得了事件的明文名称。如果事件包含健康相关数据，分析 API 必须符合 HIPAA。例如，这些事件可以跟踪一个人服用特定药物的频率。

### 安全代理 API

让我们看看这个系统的安全代理是什么样子的。

*注意:在示例项目中，所有系统都在 localhost 上运行，但是为了使安全代理有效，它必须在您客户的数据中心内部运行，虽然您自己的公司不需要符合 HIPAA，但是您的客户必须符合。但是如果他们一开始就在处理与健康相关的数据，他们可能已经在处理了。*

```py
api.post("/event", async ({ body }, response) => {
  const event = cryptoStore.encrypt(body.event);
  const res = await axios.post(
    "http://localhost:9000/event",
    { event }
  );
  response.status(res.status).end(res.data);
});

api.get("/results", async (request, response) => {
  const {
    data: { results },
    status,
  } = await axios.get(
    "http://localhost:9000/results"
  );

  const results = Object.keys(results).reduce(
    (results, encryptedEvent) => {
      const event = cryptoStore.decrypt(encryptedEvent);
      results[event] = results[encryptedEvent];
      return results;
    },
    {}
  );

  response
    .status(status)
    .json({ results });
});

api.listen(9999); 
```

像在分析 API 中一样，在安全代理中有相同的两个端点。对于客户端，它们的行为就像分析 API 的端点一样。这里唯一的区别是，安全代理 API 端点将在将事件名称转发给分析 API 之前对其进行加密和解密。

这样，安全代理 API 就像分析 API 的替代物一样。客户端不知道它的实现，可以像以前一样简单地向代理发送事件并获取结果。

我在这里选择的加密细节是事件的名称。当安全代理从分析 API 或客户端接收到 JSON 结构时，它会传递 HTTP 状态代码、求和并保存这些结构。

### 运行示例

如果我们运行添加了日志记录的示例，并且代理位于客户端和分析 API 之间，我们会看到以下输出:

```py
[Client   ] Sending event: secret-event-a
[Proxy    ] Received event: secret-event-a
[Analytics] Received event: 23c06d46723cd...
[Client   ] Sending event: secret-event-a
[Proxy    ] Received event: secret-event-a
[Analytics] Received event: 23c06d46723cd...
[Client   ] Sending event: secret-event-a
[Proxy    ] Received event: secret-event-a
[Analytics] Received event: 23c06d46723cd...
[Client   ] Sending event: secret-event-b
[Proxy    ] Received event: secret-event-b
[Analytics] Received event: 604c04290177e...
[Client   ] Sending event: secret-event-b
[Proxy    ] Received event: secret-event-b
[Analytics] Received event: 604c04290177e...
[Analytics] Sending results:  {
  '23c06d46723cd...': 3,
  '604c04290177e...': 2
}
[Client   ] Results:  {
  'secret-event-a': 3,
  'secret-event-b': 2
} 
```

在日志中，我们看到安全代理充当中间人。它接收带有客户端名称的事件，但是分析 API 接收加密的字符串。它不知道事件名称是否包含游戏分数或药物摄入信息。

在 API 分析的例子中，与 Moesif 一样，API 不知道跟踪哪种 API。唯一的要求是相同类型的所有事件都有相同的名称。这足以将它们关联起来并计算出结果。

## 结论

如果您的 API 不需要识别它处理的数据类型，安全代理是规避 HIPAA 遵从性的一个方便的解决方案。

您的客户自带加密密钥，并在自己的数据中心部署安全代理。HIPAA 相关的要求不再与你有关，因为你现在有可信的拒绝权；您永远不知道您正在处理哪种数据，因为所有数据在离开客户网络之前都是加密的。你也从未接触过加密密钥。