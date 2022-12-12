# AWS Lambda 和 AWS API 网关无服务器 API 深度指南(第 2 部分)

> 原文：<https://www.moesif.com/blog/serverless/aws-lambda/In-Depth-Guide-To-Serverless-APIs-With-AWS-Lambda-And-AWS-API-Gateway-Part2/>

*TL；带有示例项目的 DR 库可以在[GitHub](https://github.com/kay-is/serverless-api-example)T3】上找到*

这是关于用 AWS 技术构建无服务器 API 的两部分系列的最后一篇文章。

在第一部分中，我们学习了认证、请求体、状态码、CORS 和响应头。我们建立了一个连接 API-Gateway、Lambda 和 Cognito 的 AWS SAM 项目，这样用户就可以注册并登录。

在本文中，我们将讨论第三方集成、数据存储和检索以及*“从 Lambda 函数调用 Lambda 函数”*(我们实际上不会这样做，但会了解一种实现类似功能的方法。

## 添加图像上传

我们将从图像上传开始。它的工作原理如下:

1.  通过我们的 API 请求一个预先签署的 S3 网址
2.  通过预先签名的网址直接上传图片到 S3

所以我们需要两个新的 SAM/CloudFormation 资源。一个 Lambda 函数，为我们的图像生成预签名的 URL 和 S3 桶。

让我们更新 SAM 模板:

```py
AWSTemplateFormatVersion: "2010-09-09"
Transform: "AWS::Serverless-2016-10-31"
Description: "A  example  REST  API  build  with  serverless  technology"

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

  # ============================== Auth ==============================
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

  PreSignupFunction:
    Type: AWS::Serverless::Function
    Properties:
      InlineCode: |
        exports.handler = async event => {
          event.response = { autoConfirmUser: true };
          return event;
        };

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

  LambdaCognitoUserPoolExecutionPermission:
    Type: AWS::Lambda::Permission
    Properties: 
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt PreSignupFunction.Arn
      Principal: cognito-idp.amazonaws.com
      SourceArn: !Sub 'arn:${AWS::Partition}:cognito-idp:${AWS::Region}:${AWS::AccountId}:userpool/${UserPool}'

  # ============================== Images ==============================
  ImageBucket:
    Type: AWS::S3::Bucket
    Properties: 
      AccessControl: PublicRead
      CorsConfiguration:
        CorsRules:
          - AllowedHeaders:
              - "*"
            AllowedMethods:
              - HEAD
              - GET
              - PUT
              - POST
            AllowedOrigins:
              - "*"

  ImageBucketPublicReadPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref ImageBucket
      PolicyDocument:
        Statement:
          - Action: s3:GetObject
            Effect: Allow
            Principal: "*"
            Resource: !Join ["", ["arn:aws:s3:::", !Ref "ImageBucket", "/*" ]]

  ImageFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functioimg/
      Policies:
        - AmazonS3FullAccess
      Environment:
        Variables:
          IMAGE_BUCKET_NAME: !Ref ImageBucket
      Events:
        CreateImage:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Path: /images
            Method: POST

Outputs:

  ApiUrl:
    Description: The target URL of the created API
    Value: !Sub "https://${ServerlessApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
    Export:
      Name: ApiUrl 
```

好吧，我们有了三个新资源。我们还需要一个`BucketPolicy`来允许公众读取我们的新图像桶。

`ImagesFunction`有一个 API 事件，所以我们可以用它来处理 POST 请求。该函数获得一个 S3 访问策略和一个环境变量，因此它知道`ImageBucket`。

我们需要为功能代码`functioimg/index.js`创建一个新文件。

```py
const AWS = require("aws-sdk");

exports.handler = async event => {
  const userName = event.requestContext.authorizer.claims["cognito:username"];
  const fileName = "" + Math.random() + Date.now() + "+" + userName;
  const { url, fields } = await createPresignedUploadCredentials(fileName);
  return {
    statusCode: 201,
    body: JSON.stringify({
      formConfig: {
        uploadUrl: url,
        formFields: fields
      }
    }),
    headers: { "Access-Control-Allow-Origin": "*" }
  };
};

const s3Client = new AWS.S3();
const createPresignedUploadCredentials = fileName => {
  const params = {
    Bucket: process.env.IMAGE_BUCKET_NAME,
    Fields: { Key: fileName }
  };
  return new Promise((resolve, reject) =>
    s3Client.createPresignedPost(params, (error, result) =>
      error ? reject(error) : resolve(result)
    )
  );
}; 
```

那么，这个函数会发生什么呢？

首先，我们从`event`对象中提取`userName`。API-Gateway 和 Cognito 控制对我们函数的访问，所以我们可以确定当函数被调用时用户存在于`event`对象中。

接下来，我们基于随机数、当前时间戳和用户名创建一个唯一的`fileName`。

`createPresignedUploadCredentials`助手函数将创建预签名的 S3 URL。它将返回一个具有`url`和`fields`属性的对象。

我们的 API 客户端必须向`url`发送一个 POST 请求，并在其主体中包含所有字段和文件。

## 集成图像识别

现在，我们需要集成第三方图像识别服务。

当图片上传时，S3 将触发一个由 lambda 函数处理的上传事件。

这就把我们带到了第一篇文章开头的一个问题。

### 如何从一个 Lambda 调用一个 Lambda？

简而言之:你没有。

为什么？虽然您可以通过 AWS-SDK 直接从 Lambda 调用 Lambda，但这带来了一个问题。如果出现问题，我们必须实现所有需要发生的事情，比如重试等等。此外，在某些情况下，调用 Lambda 必须等待被调用的 Lambda 完成，我们也必须为等待时间付费。

那么有哪些选择呢？

Lambda 是一个基于事件的系统，我们的功能可以由不同的事件源触发。主要的方法是使用另一个服务来实现。

在我们的例子中，我们希望在文件上传完成时调用一个 Lambda，所以我们必须使用一个 S3 事件作为源。但是也有其他事件源。

*   [阶跃函数](https://aws.amazon.com/en/step-functions/)让我们用状态机来协调λ函数
*   SQS 是一个队列，我们可以将结果放入其中，以便其他 Lambdas 获取这些结果
*   SNS 是一项服务，它允许我们将一个 Lambda 结果以扇出方式并行发送给许多其他 Lambda。

我们在 SAM 模板中添加了一个新的 Lambda 函数，它将被 S3 调用。

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

  # ============================== Auth ==============================
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

  PreSignupFunction:
    Type: AWS::Serverless::Function
    Properties:
      InlineCode: |
        exports.handler = async event => {
          event.response = { autoConfirmUser: true };
          return event;
        };

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

  LambdaCognitoUserPoolExecutionPermission:
    Type: AWS::Lambda::Permission
    Properties: 
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt PreSignupFunction.Arn
      Principal: cognito-idp.amazonaws.com
      SourceArn: !Sub 'arn:${AWS::Partition}:cognito-idp:${AWS::Region}:${AWS::AccountId}:userpool/${UserPool}'

  # ============================== Images ==============================
  ImageBucket:
    Type: AWS::S3::Bucket
    Properties: 
      AccessControl: PublicRead
      CorsConfiguration:
        CorsRules:
          - AllowedHeaders:
              - "*"
            AllowedMethods:
              - HEAD
              - GET
              - PUT
              - POST
            AllowedOrigins:
              - "*"

  ImageBucketPublicReadPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref ImageBucket
      PolicyDocument:
        Statement:
          - Action: s3:GetObject
            Effect: Allow
            Principal: "*"
            Resource: !Join ["", ["arn:aws:s3:::", !Ref "ImageBucket", "/*" ]]

  ImageFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functioimg/
      Policies:
        - AmazonS3FullAccess
      Environment:
        Variables:
          IMAGE_BUCKET_NAME: !Ref ImageBucket
      Events:
        CreateImage:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Path: /images
            Method: POST

  TagsFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functions/tags/
      Environment:
        Variables:
          PARAMETER_STORE_CLARIFAI_API_KEY: /serverless-api/CLARIFAI_API_KEY_ENC
      Policies:
        - AmazonS3ReadOnlyAccess # Managed policy
        - Statement: # Inline policy document
          - Action: [ 'ssm:GetParameter' ]
            Effect: Allow
            Resource: '*'
      Events:
        ExtractTags:
          Type: S3
          Properties:
            Bucket: !Ref ImageBucket
            Events: s3:ObjectCreated:*

Outputs:

  ApiUrl:
    Description: The target URL of the created API
    Value: !Sub "https://${ServerlessApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
    Export:
      Name: ApiUrl 
```

我们添加了一个新的无服务器函数资源，它有一个 S3 事件，当在我们的`ImageBucket`中创建了一个新的 S3 对象时，就会调用这个事件。

在我们的 Lambda 函数中需要调用第三方 API，即 Clarifai API，这给我们带来了下一个问题。

### 如何存储第三方 API 的凭证？

有许多方法可以做到这一点。

一种方法是用 KMS 加密凭证。我们通过 CLI 对其进行加密，将加密的密钥作为环境变量添加到 template.yaml 中，然后在 Lambda 中，我们使用 AWS-SDK 对凭证进行解密，然后再使用它们。

另一种方法是使用 **AWS 系统管理器**的**参数库**。该服务允许存储加密的字符串，这些字符串可以通过 Lambda 中的 AWS-SDK 进行检索。我们只需要为我们的 Lambda 提供定义凭证存储位置的名称。

在本例中，我们将使用参数存储。

如果你还没有创建一个 Clarifai 账户，现在是时候了。他们将为您提供一个 API 密钥，我们接下来需要将它存储到参数存储中。

```py
aws ssm put-parameter \
--name "/serverless-api/CLARIFAI_API_KEY" \
--type "SecureString" \ 
--value "<CLARIFAI_API_KEY>" 
```

这个命令将把密钥放在参数存储中并加密它。

接下来，我们需要通过一个环境变量告诉 Lambda 函数这个名字，并允许它通过 AWS-SDK 调用`getParameter`。

```py
Environment:
        Variables:
          PARAMETER_STORE_CLARIFAI_API_KEY: /serverless-api/CLARIFAI_API_KEY_ENC
      Policies:
        - Statement:
          - Action: [ 'ssm:GetParameter' ]
            Effect: Allow
            Resource: '*' 
```

让我们看看 JavaScript 方面的东西，为此，我们创建了一个新文件`functions/tags/index.js`。

```py
const AWS = require("aws-sdk");
const Clarifai = require("clarifai");

exports.handler =  async event => {
  const record = event.Records[0];
  const bucketName = record.s3.bucket.name;
  const fileName = record.s3.object.key;
  const tags = await predict(`https://${bucketName}.s3.amazonaws.com/${fileName}`);

  await storeTagsSomewhere({ fileName, tags });
};

const ssm = new AWS.SSM();
const predict = async imageUrl => {
  const result = await ssm.getParameter({
    Name: process.env.PARAMETER_STORE_CLARIFAI_API_KEY, 
    WithDecryption: true
  }).promise();

  const clarifaiApp = new Clarifai.App({
    apiKey: result.Parameter.Value
  });

  const model = await clarifaiApp.models.initModel({
    version: "aa7f35c01e0642fda5cf400f543e7c40",
    id: Clarifai.GENERAL_MODEL
  });

  const clarifaiResult = await model.predict(imageUrl);

  const tags = clarifaiResult.outputs[0].data.concepts
    .filter(concept => concept.value > 0.9)
    .map(concept => concept.name);
  return tags;
}; 
```

通过 S3 对象创建事件调用该处理程序。将只有一个记录，但我们也可以告诉 S3 批量记录在一起。

然后，我们为新创建的图像创建一个 URL，并将其交给`predict`函数。

`predict`函数使用我们的`PARAMETER_STORE_CLARIFAI_API_KEY`环境变量来获取参数存储中的参数名。这允许我们在不改变 Lambda 代码的情况下改变目标参数。

我们得到解密的 API 密钥，并可以调用第三方 API。然后我们把标签放在某个地方。

## 列出和删除标记图像

现在我们可以上传图像，它们会被自动标记，下一步是列出所有的图像，通过标记过滤它们，如果不再需要就删除它们。

让我们更新 SAM 模板！

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

  # ============================== Auth ==============================
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

  PreSignupFunction:
    Type: AWS::Serverless::Function
    Properties:
      InlineCode: |
        exports.handler = async event => {
          event.response = { autoConfirmUser: true };
          return event;
        };

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

  LambdaCognitoUserPoolExecutionPermission:
    Type: AWS::Lambda::Permission
    Properties: 
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt PreSignupFunction.Arn
      Principal: cognito-idp.amazonaws.com
      SourceArn: !Sub 'arn:${AWS::Partition}:cognito-idp:${AWS::Region}:${AWS::AccountId}:userpool/${UserPool}'

  # ============================== Images ==============================
  ImageBucket:
    Type: AWS::S3::Bucket
    Properties: 
      AccessControl: PublicRead
      CorsConfiguration:
        CorsRules:
          - AllowedHeaders:
              - "*"
            AllowedMethods:
              - HEAD
              - GET
              - PUT
              - POST
            AllowedOrigins:
              - "*"

  ImageBucketPublicReadPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref ImageBucket
      PolicyDocument:
        Statement:
          - Action: s3:GetObject
            Effect: Allow
            Principal: "*"
            Resource: !Join ["", ["arn:aws:s3:::", !Ref "ImageBucket", "/*" ]]

  ImageFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functioimg/
      Policies:
        - AmazonS3FullAccess
      Environment:
        Variables:
          IMAGE_BUCKET_NAME: !Ref ImageBucket
      Events:
        ListImages:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Path: /images
            Method: GET
        DeleteImage:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Pathimg/{imageId}
            Method: DELETE
        CreateImage:
          Type: Api
          Properties:
            RestApiId: !Ref ServerlessApi
            Path: /images
            Method: POST

  TagsFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functions/tags/
      Environment:
        Variables:
          PARAMETER_STORE_CLARIFAI_API_KEY: /serverless-api/CLARIFAI_API_KEY_ENC
      Policies:
        - AmazonS3ReadOnlyAccess # Managed policy
        - Statement: # Inline policy document
          - Action: [ 'ssm:GetParameter' ]
            Effect: Allow
            Resource: '*'
      Events:
        ExtractTags:
          Type: S3
          Properties:
            Bucket: !Ref ImageBucket
            Events: s3:ObjectCreated:*

Outputs:

  ApiUrl:
    Description: The target URL of the created API
    Value: !Sub "https://${ServerlessApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
    Export:
      Name: ApiUrl 
```

我们向我们的`ImagesFunction`添加了一些新的 API 事件，现在也需要为它更新 JavaScript。

```py
const AWS = require("aws-sdk");

exports.handler = async event => {
  switch (event.httpMethod.toLowerCase()) {
    case "post":
      return createImage(event);
    case "delete":
      return deleteImage(event);
    default: 
      return listImages(event);
  }
};

const createImage = async event => {
  const userName = extractUserName(event);
  const fileName = "" + Math.random() + Date.now() + "+" + userName;
  const { url, fields } = await createPresignedUploadCredentials(fileName);
  return response({
    formConfig: {
      uploadUrl: url,
      formFields: fields
    }
  }, 201);
};

const deleteImage = async event => {
  const { imageId } = event.pathParameters;
  await deleteImageSomewhere(imageId);
  return response({ message: "Deleted image: " + imageId });
};

// Called with API-GW event
const listImages = async event => {
  const { tags } = event.queryStringParameters;
  const userName = extractUserName(event);
  const images = await loadImagesFromSomewhere(tags.split(","), userName);
  return response({ images });
};

// ============================== HELPERS ==============================

const extractUserName = event => event.requestContext.authorizer.claims["cognito:username"];

const response = (data, statusCode = 200) => ({
  statusCode,
  body: JSON.stringify(data),
  headers: { "Access-Control-Allow-Origin": "*" }
});

const s3Client = new AWS.S3();
const createPresignedUploadCredentials = fileName => {
  const params = {
    Bucket: process.env.IMAGE_BUCKET_NAME,
    Fields: { Key: fileName }
  };

  return new Promise((resolve, reject) =>
    s3Client.createPresignedPost(params, (error, result) =>
      error ? reject(error) : resolve(result)
    )
  );
}; 
```

删除功能很简单，但也回答了我们的一个问题。

### 如何使用查询字符串或路由参数？

```py
const deleteImage = async event => {
  const { imageId } = event.pathParameters;
  ...
}; 
```

API-Gateway 事件对象有一个`pathParameters`属性，保存我们在 SAM 模板中为事件定义的所有参数。在我们的例子中`imageId`，因为我们定义了`Pathimg/{imageId}`。

以类似方式使用的查询字符串。

```py
const listImages = async event => {
  const { tags } = event.queryStringParameters;
  ...
}; 
```

剩下的代码并不太复杂。我们将使用偶数数据来加载或删除图像。

查询字符串将作为对象存储在我们的`event`对象的`queryStringParameters`属性中。

## 结论

有时新技术会带来范式的转变。无服务器就是这些技术中的一种。

it 的全部力量通常归结为尽可能使用托管服务。不要使用自己的加密、身份验证、存储或计算。使用您的云提供商已经实施的技术。

这里的大问题往往是，做事情有很多方法。正确的方法不只有一种。我们应该直接使用 KMS 吗？我们应该让系统经理为我们处理事情吗？我们应该通过 Lambda 实现 auth 吗？我们是否应该使用 Cognito 并结束它？

我希望我能在某种程度上回答使用 Lambda 时出现的问题。