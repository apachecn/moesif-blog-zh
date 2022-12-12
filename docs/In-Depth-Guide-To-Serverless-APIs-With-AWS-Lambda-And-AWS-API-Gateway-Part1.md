# AWS Lambda 和 AWS API 网关无服务器 API 深度指南(第 1 部分)

> 原文：<https://www.moesif.com/blog/serverless/aws-lambda/In-Depth-Guide-To-Serverless-APIs-With-AWS-Lambda-And-AWS-API-Gateway-Part1/>

*TL；带有示例项目的 DR 库可以在[GitHub](https://github.com/kay-is/serverless-api-example)T3】上找到*

围绕 API 构建软件产品是多年来的事情，从一开始就使用无服务器技术似乎很有意思，原因有很多——按需定价、自动伸缩和更少的运营开销。

## 为什么

当我读到人们谈论无服务器技术时，我感觉还有许多问题没有解决。

如何

1.  [使用查询字符串还是路由参数？](/blog/serverless/aws-lambda/In-Depth-Guide-To-Serverless-APIs-With-AWS-Lambda-And-AWS-API-Gateway-Part2.md/#how-use-a-query-string-or-route-parameters)
2.  使用 POST 请求的参数或正文？
3.  [设置 HTTP 状态码？](#how-to-set-http-status-codes)
4.  [设置回复标题？](#how-to-set-an-http-response-header)
5.  [从一个 Lambda 调用一个 Lambda？](/blog/serverless/aws-lambda/In-Depth-Guide-To-Serverless-APIs-With-AWS-Lambda-And-AWS-API-Gateway-Part2.md/#how-use-a-query-string-or-route-parameters#how-to-call-a-lambda-from-a-lambda)
6.  [为第三方 API 存储密钥？](/blog/serverless/aws-lambda/In-Depth-Guide-To-Serverless-APIs-With-AWS-Lambda-And-AWS-API-Gateway-Part2.md/#how-use-a-query-string-or-route-parameters#how-store-credentials-for-third-party-apis)

所以我决定写一篇关于如何用无服务器技术构建 API 的文章，特别是 AWS Lambda 和 API-Gateway。

这篇文章分为两部分。

第一个是关于架构、设置和认证的。

第二个是关于*实际的*工作，上传图片，用第三方 API 给它们加标签，然后检索标签和图片。

## 什么

我们将构建一个简单的 RESTful 图像标记 API。它让我们上传图片，用名为 [Clarifai](https://www.clarifai.com/) 的第三方 API 自动标记它们，并将标记和图片名称存储到 Elasticsearch 中。

我心目中的工作流程是:

*   注册并登录
*   上传图像
*   获取所有图像的所有标签
*   获取所有图像及其标签
*   通过标签获取图像

## 怎么

对于 API，我们使用 API-Gateway，这是 Amazons 的全方位无服务器 HTTP 解决方案。它针对 RESTful APIs 进行了优化，是我们系统的入口点。

**Amazon Cognito** 处理认证。Cognito 是一个托管的无服务器认证、授权和数据同步解决方案。我们用它来注册我们的用户，因此我们不必在这里重新发明轮子。

我们 API 的实际计算工作由 **AWS Lambda** 完成，这是一个**功能即服务**解决方案。Lambda 是一个无服务器的基于事件的系统，允许在发生事情时触发功能，例如，一个 HTTP 请求击中了我们的 API，或者有人直接上传了一个文件到 S3。

这些图像被储存在亚马逊 S3 的一个桶里。S3 是一种无服务器的基于对象的存储解决方案。它允许通过 HTTP 直接访问和上传文件，并且可以作为 **API 网关**，成为**λ**的事件源。

标签数据和相应的图像名称存储在**亚马逊弹性搜索服务**中；一个 AWS 管理的 Elasticsearch 版本。遗憾的是，这不是无服务器的，而是构建在 EC2 实例上的，但是前 12 个月有一个免费层。Elasticsearch 是一个非常灵活的文档存储，并带有一个强大的查询语言。

一项名为 Clarifai 的非 AWS 服务提供了图像识别魔法。AWS 对此有其服务，称为 Rekognition，但通过使用 Clarifai，我们可以了解如何存储第三方 API 密钥。

我们构建的整个基础设施由 **AWS SAM** 、**无服务器应用模型**管理。SAM 是 AWS CloudFormation 的扩展，它减少了设置 AWS Lambda 和 API 网关资源所需的一些样板代码。

我们使用 **AWS Cloud9** 作为 IDE，因为它预装了使用 AWS 资源的所有工具和权限。

## 先决条件

*   浏览器
*   Cloud9 环境下的 AWS 帐户。*(设置可以在[这里找到](https://dev.to/kayis/10-easy-steps-to-create-aws-lambda-functions-with-the-serverless-framework--reason-in-aws-cloud9-8d1)中的前 5 步。)*
*   基本的 JavaScript 知识
*   基本的 Lambda、API 网关和 SAM 知识

## 设置

让我们从一个基本的无服务器 API 开始，它只是在 AWS SAM、API-Gateway 和 Cognito 的帮助下实现一个登录。

```py
mkdir serverless-api
cd serverless-api
mkdir functions
touch template.yaml 
```

首先，我们创建文件夹结构和一个`template.yaml`文件，该文件包含我们用 SAM 创建的基础设施的定义。

## 实现 SAM 模板

我们的 SAM 模板的内容如下:

```py
AWSTemplateFormatVersion: "2010-09-09"
Transform: "AWS::Serverless-2016-10-31"
Description: "An  example  REST  API  build  with  serverless  technology"

Globals:
  Function:
    Runtime: nodejs8.10
    Handler: index.handler
    Timeout: 30
    Tags:
      Application: Serverless API

Resources:
  ServerlessApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod
      Cors: "'*'"
      Auth:
        DefaultAuthorizer: CognitoAuthorizer
        Authorizers:
          CognitoAuthorizer:
            UserPoolArn: !GetAtt UserPool.Arn
      GatewayResponses:
        UNAUTHORIZED:
          StatusCode: 401
          ResponseParameters:
            Headers:
              Access-Control-Expose-Headers: "'WWW-Authenticate'"
              Access-Control-Allow-Origin: "'*'"
              WWW-Authenticate: >-
                'Bearer realm="admin"'

  # ============================== Auth =============================
  UserPool:
    Type: AWS::Cognito::UserPool
    Properties:
      UserPoolName: ApiUserPool
      LambdaConfig:
        PreSignUp: !GetAtt PreSignupFunction.Arn
      Policies:
        PasswordPolicy:
          MinimumLength: 6

  UserPoolClient:
    Type: AWS::Cognito::UserPoolClient
    Properties:
      UserPoolId: !Ref UserPool
      ClientName: ApiUserPoolClient
      GenerateSecret: no

  PreSignupFunction:
    Type: AWS::Serverless::Function
    Properties:
      InlineCode: |
        exports.handler = async event => {
          event.response = { autoConfirmUser: true };
          return event;
        };

  LambdaCognitoUserPoolExecutionPermission:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt PreSignupFunction.Arn
      Principal: cognito-idp.amazonaws.com
      SourceArn: !Sub "arn:${AWS::Partition}:cognito-idp:${AWS::Region}:${AWS::AccountId}:userpool/${UserPool}"

  AuthFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functions/auth/
      Environment:
        Variables:
          USER_POOL_ID: !Ref UserPool
          USER_POOL_CLIENT_ID: !Ref UserPoolClient
      Events:
        Signup:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Path: /signup
            Method: POST
            Auth:
              Authorizer: NONE
        Signin:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Path: /signin
            Method: POST
            Auth:
              Authorizer: NONE

Outputs:
  ApiUrl:
    Description: The target URL of the created API
    Value: !Sub "https://${ServerlessApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
    Export:
      Name: ApiUrl 
```

首先，我们定义一个 ServerlessApi 资源。在 SAM 模板中，该资源通常是为我们隐式创建的，但是当我们想要启用 CORS 或使用授权者时，我们需要显式定义它。

我们还将所有路由的默认授权者设置为 CognitoAuthorizer，然后立即使用我们稍后在模板中定义的 Cognito user pool 对其进行配置。

然后我们定义一个`UNAUTHORIZED`网关响应，因为 API-Gateway 不会自己给我们的响应添加 CORS 头。在我们的 Lambda 支持的路由中，我们可以通过 JavaScript 代码来实现，但是当我们没有权限调用它们时，这没有用。网关响应是用 API-Gateway 直接添加头的一种方式。

接下来，我们定义与认知相关的资源:

*   `UserPool`是 Cognito 的一部分，保存着我们用户的账户。
*   `UserPoolClient`是 Cognito 的一部分，允许与用户池进行编程交互。
*   `PreSignupFunction`是一个 Lambda 函数，在实际注册用户池之前调用。它允许我们激活用户，而无需向他们发送电子邮件。它只有几行代码，所以我们内联它。
*   `LambdaCognitoUserPoolExecutionPermission`授予被管理的认知服务执行我们的`PreSignupFunction`的许可。

最后，我们定义一个 Lambda 函数，由 API-Gateway routes 调用。具体来说就是`POST /signup`和`POST /signin`。这个`AuthFunction`还获取用户池 ID 和用户池客户机 ID 作为环境变量。这些值是在部署时动态生成的，但是环境变量是将这些值从 SAM/Cloudformation 传递到函数中的一种方式。

用我们设置在顶部的`Global/Function/Handler`和我们的`AuthFunction`属性中的`CodeUri`，我们可以决定我们的 JavaScript 文件必须在哪里，以及它必须如何为 Lambda 导出一个处理函数。

```py
...
Handler: index.handler
...
CodeUri: functions/auth/ 
```

该文件必须被称为`index.js`，它必须导出一个名为`handler`的函数，它必须在`functions/auth/`目录中。

端点定义的`Auth`属性被设置为`Authorizer: NONE`，因此 API-Gateway 允许我们请求端点而不需要令牌。

在文件的末尾，我们有一个名为`ApiUrl`的输出；我们在部署之后使用它从 CloudFormation 获取实际的 API URL。

## 实现 Auth Lambda 函数

我们一开始就创建了`functions`目录，所以我们只需要添加一个带有`index.js`的`auth`目录

`index.js`包含以下代码:

```py
const users = require("./user-management");

exports.handler = async event => {
  const body = JSON.parse(event.body);
  if (event.path === "/signup") return signUp(body);
  return signIn(body);
};

const signUp = async ({ username, password }) => {
  try {
    await users.signUp(username, password);
    return createResponse({ message: "Created" }, 201);
  } catch (e) {
    console.log(e);
    return createResponse({ message: e.message }, 400);
  }
};

const signIn = async ({ username, password }) => {
  try {
    const token = await users.signIn(username, password);
    return createResponse({ token }, 201);
  } catch (e) {
    console.log(e);
    return createResponse({ message: e.message }, 400);
  }
};

const createResponse = (
  data = { message: "OK" },
  statusCode = 200
) => ({
  statusCode,
  body: JSON.stringify(data),
  headers: { "Access-Control-Allow-Origin": "*" }
}); 
```

它需要一个`user-management.js`来完成它的工作，但是我们稍后再讨论这个。首先，让我们看看这个文件做什么。

正如我们在`template.yaml`中所说的，它导出一个`handler`函数，当有人向`/signup`或`/signin`端点发送 POST 请求时，该函数从 API-Gateway 接收 HTTP 请求事件。

这里我们可以看到一个最常见问题的答案。

### 如何访问请求正文？

```py
exports.handler = async event => {
  const body = JSON.parse(event.body);
  ...
}; 
```

JavaScript Lambda 函数的第一个参数包含一个事件对象。当使用 API-Gateway 事件调用该函数时，该对象有一个`body`属性，如果客户端发送了一个请求，该属性保存请求体的字符串。

在我们的例子中，我们期望一个带有`username`和`password`的 JSON，这样我们就可以创建新的用户帐户或者让用户登录他们的帐户，所以我们需要首先`JSON.parse()`主体。

接下来，我们有两个函数，`signUp`和`signIn`，它们使用所需的`user-management`模块，这里称为`users`来完成它们的工作。它们都是作为参数传递的`username`和`password`。

`createResponse`是一个构建响应对象的实用函数。这个对象是 Lambda 函数必须返回的。

在这里，我们从一开始就有了其他问题的答案。

### 如何设置 HTTP 状态码？

```py
exports.handler = async event => {
  ...
  return { statusCode: 404 };
}; 
```

每个 Lambda 函数都需要返回一个响应对象。该对象必须至少有一个`statusCode`属性。否则，API-Gateway 认为请求失败。

### 如何设置 HTTP 响应头？

```py
exports.handler = async event => {
  ...
  return {
    statusCode: 200,
    headers: {
      "Access-Control-Allow-Origin": "*"
    }
  };
}; 
```

我们的 Lambda 函数必须返回的响应对象也可以有一个`headers`属性，它必须是一个以头名称作为对象键，以头值作为对象值的对象。

在这里，我们可以看到，我们需要手动设置 CORS 标头。否则，浏览器不会接受响应。

现在，让我们看看我们需要的`user-management.js`文件。

```py
global.fetch = require("node-fetch");
const Cognito = require("amazon-cognito-identity-js");

const userPool = new Cognito.CognitoUserPool({
  UserPoolId: process.env.USER_POOL_ID,
  ClientId: process.env.USER_POOL_CLIENT_ID
});

exports.signUp = (username, password) =>
  new Promise((resolve, reject) =>
    userPool.signUp(username, password, null, null, (error, result) =>
      error ? reject(error) : resolve(result)
    )
  );

exports.signIn = (username, password) =>
  new Promise((resolve, reject) => {
    const authenticationDetails = new Cognito.AuthenticationDetails({
      Username: username,
      Password: password
    });

    const cognitoUser = new Cognito.CognitoUser({
      Username: username,
      Pool: userPool
    });

    cognitoUser.authenticateUser(authenticationDetails, {
      onSuccess: result => resolve(result.getIdToken().getJwtToken()),
      onFailure: reject
    });
  }); 
```

首先，我们使用`node-fetch`包来填充`fetch`浏览器 API。这个 polyfill 是我们接下来需要的`amazon-cognito-identity-js`包所需要的，它为我们做了与 Cognito 服务对话的繁重工作。

我们在从 SAM 模板获得的环境变量的帮助下创建一个`CognitoUserPool`对象，并在我们导出的`signIn`和`signUp`函数中使用这个对象。

最激动人心的部分是在`signIn`函数中，它从 Cognito 用户池中获取一个 ID 令牌并返回它。

这个令牌需要在对我们的 API 的每个请求上，在带有前缀`Bearer`的`Authorization`请求头中传递，并且当它过期时需要重新获取。我们的 SAM 模板的 API 配置中的 CognitoAuthorizer 告诉 API-Gateway 如何用 Cognito 处理其他所有事情。

现在我们需要安装我们使用过的包。`node-fetch` polyfill 包和`amazon-cognito-identity-js`包。

为此，我们需要进入`functions/auth`目录并初始化一个 NPM 项目，然后安装软件包。

```py
npm init -y
npm i node-fetch amazon-cognito-identity-js 
```

当我们用剩下的 API 将函数部署到 Lambda 时，`functions/auth`目录中的所有文件都被上传到 S3。

## 部署 API

为了将我们的项目部署到 AWS，我们使用了`sam` CLI 工具。

首先，我们需要打包我们的 Lambda 函数源代码，并将其上传到 S3 部署桶。我们可以用`aws` CLI 创建这个 bucket。

```py
aws s3 mb s3://<DEPLOYMENT_BUCKET_NAME> 
```

桶名必须是全局唯一的，所以我们需要发明一个。

当创建成功时，我们需要`DEPLOYMENT_BUCKET_NAME`到`package`我们的 Lambda 源。

```py
sam package --template-file template.yaml \
--s3-bucket <DEPLOYMENT_BUCKET_NAME> \
--output-template-file packaged.yaml 
```

这个命令创建了一个`packaged.yaml`文件，其中保存了我们在 S3 上打包的 Lambda 源代码的 URL。

接下来，我们需要使用 CloudFormation 进行实际部署。

```py
sam deploy --template-file packaged.yaml \
--stack-name serverless-api \
--capabilities CAPABILITY_IAM 
```

如果一切顺利，我们使用下面的命令来获取 API 的基本 URL。

```py
aws cloudformation describe-stacks \
--stack-name serverless-api \
--query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' \
--output text 
```

该 URL 应该如下所示:

```py
https://<API_ID>.execute-api.<REGION>.amazonaws.com/Prod/ 
```

这个 URL 现在可以用来向我们创建的`/signup`和`/signin`端点发出 POST 请求。

## 结论

本文是关于在 AWS 上创建无服务器 API 的两篇文章中的第一篇。我们讨论了这样做的动机、我们完成工作所需的 AWS 服务，并在 AWS Cognito 的帮助下实现了基于令牌的身份验证。

我们还回答了一些用 API-Gateway 和 Lambda 构建 API 时出现的最紧迫的问题。

在下一部分中，我们实现了两个 Lambda 函数，它们允许我们使用 API。

*   `ImagesFunction`负责创建 S3 存储桶的上传链接。
*   `TagsFunction`处理 S3 上传、第三方 API 集成和已创建标签的列表。