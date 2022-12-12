# 如何记录 AWS Lambda Go 函数的 API 调用

> 原文：<https://www.moesif.com/blog/aws-lambda/go/How-to-Log-API-Calls-for-AWS-Lambda-Go-functions/>

AWS Lambda 是一种无服务器计算服务，它运行您的代码来响应事件，并自动为您管理底层计算资源。Amazon API Gateway 是一个托管服务，它允许开发人员定义 REST API 的 HTTP 端点，并将这些端点与相应的后端业务逻辑连接起来。API Gateway 位于 API 的后端服务和 API 用户之间，处理对 API 端点的 HTTP 请求、任何认证需求，并将它们路由到正确的后端。在无服务器环境中，我们对底层基础设施的控制较少，日志记录是构建后端的重要部分，对于无服务器 API 也是如此。通过跟踪 API 请求，不仅可以了解 API 的执行情况以及哪里需要优化，还可以了解 API 的使用情况以及哪些功能使用得最多。

在本指南中，我们将向您展示如何在几分钟内设置 Moesif API analytics，以使用 API Gateway 作为触发器来跟踪托管在 AWS Lambda 上的 API。

#### 创建 AWS Lambda 函数处理程序

在我们用 go 编写的 lambda 函数中，我们需要包含 [aws-lambda-go](github.com/aws/aws-lambda-go/lambda) 包，它实现了 Go 的 lambda 编程模型。此外，我们需要实现 handler 函数和 main()函数。

下面是一个示例类和 Lambda 函数，它接收 Amazon API Gateway 事件记录数据作为输入，并以 200 状态、标题和与请求相同的正文作为响应。

```py
package main

import (
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/aws/aws-lambda-go/events"
	"context"
)

func handleRequest(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {

	return events.APIGatewayProxyResponse{
		Body:       request.Body,
		StatusCode: 200,
		Headers: map[string] string {
			"RspHeader1":     "RspHeaderValue1",
			"Content-Type":   "application/json",
			"Content-Length": "1000",
		},
	   }, nil
}

func main() {
	lambda.Start(handleRequest)
} 
```

我们需要按照请求发生的顺序维护请求的历史。每个日志条目包含关于请求的信息，包括客户端 IP 地址、请求日期/时间、请求路径、HTTP 代码、提供的字节、用户代理等。使用 Moesif AWS Lambda 中间件 for Go ，我们将记录所有 API 调用并发送给 Moesif。

我们将为我们的函数定义配置选项。更多详情，请参考 [Moesif 配置选项](https://github.com/Moesif/moesif-aws-lambda-go#configuration-options)。

```py
func MoesifOptions() map[string]interface{} {
	var moesifOptions = map[string]interface{} {
		"Log_Body": true,
	}
	return moesifOptions
} 
```

现在，我们将把`Moesif Middleware`添加到我们的 lambda 函数中。

```py
func main() {
	lambda.Start(moesifawslambda.MoesifLogger(HandleLambdaEvent, MoesifOptions()))
} 
```

集成之后，我们将看到所有的 API 调用都被捕获并发送给 Moesif。

#### 安全记录数据

认证请求包含凭证，响应包含安全令牌，在日志中记录凭证和令牌是不安全的。有了 Moesif，我们可以安全地提取我们需要的信息，同时保持安全性，不需要对我们的代码库进行大量的修改。使用`Mask_Event_Model`配置选项，我们可以屏蔽请求/响应头字段或请求/响应体，以保持认证信息的机密性。此外，如果我们想删除 Moesif 的日志请求和响应主体，我们可以将配置选项`Log_Body`设置为`false`。我们还可以捕获所有从我们的应用程序到第三方(如 Stripe、Github 或我们自己的内部 API)的传出 API 调用。

要了解如何使用您的 API，请捕获 API 请求和响应，并记录到 Moesif，以便通过 AWS Lambda 对您的 API 流量进行分析和实时调试。关于如何集成 Moesif 的更多细节，您可以从 [GitHub](https://github.com/Moesif/moesif-aws-lambda-go-example) 中克隆并运行这个示例应用程序。

同时，如果您有任何问题，请联系 [Moesif 团队](mailto:team@moesif.com)