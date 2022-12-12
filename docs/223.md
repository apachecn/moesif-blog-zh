# 无服务器计算入门:AWS Lambda vs Google Cloud Functions vs Azure Functions

> 原文：<https://www.moesif.com/blog/engineering/serverless/A-Primer-On-Serverless-Computing-AWS-Lambda-vs-Google-Cloud-Functions-vs-Azure-Functions/>

## 什么是无服务器计算？

无服务器计算是一种新形式的基于云的计算，类似于在云提供商上运行的虚拟机和容器。虽然这并不意味着没有服务器，但服务器的管理、扩展和容量规划由底层云提供商负责。应用程序开发人员只需要关注功能和业务逻辑。

越来越多的开发人员正在为下一代应用程序和 API 采用无服务器技术。无服务器已经比 Docker 和 containers 快 10 倍。本文将提供一些关于无服务器生态系统的信息，以及不同提供商和框架之间的差异。

## 微服务底漆

在无服务器计算之前，许多企业采用微服务，这是一种面向服务的架构(SOA)形式。微服务使应用程序能够被组织为通过 API 连接在一起的松散耦合服务的集合。每个服务在它自己的进程/容器/VM 中都是一个完全独立的迷你应用程序。主要的好处是我们在计算机科学 101:模块化和关注点分离中学到的。微服务*可以*让复杂的应用在大公司更容易开发和扩展。应用程序可以很容易地在职能团队之间划分，每个团队负责一个或一组微服务。代码库可以保持很小，并且为了快速迭代开发而构建得很短。应用程序的各个部分可以独立于其他微服务进行扩展。

然而，随着微服务的出现，基础设施和运营工作大大增加。突然出现了许多需要跟踪的持续集成/持续交付管道。需要考虑服务的每个版本或变体的互操作性，以及它所依赖的其他服务。有复杂的编排来管理更多的移动部分。日志记录上下文现在分散在许多单独的进程中。集成测试的负担更重了。事实上，像 Basecamp [这样的公司认为为什么 monolothic](https://m.signalvnoise.com/the-majestic-monolith-29166d022228) 架构对某些像初创公司这样的小公司有意义。

## 走向无服务器

无服务器计算将微服务推向了极致。基础设施、编排层和部署都被取消了。仍然有服务器和虚拟机，但它们完全由云提供商管理。作为应用开发者，你只需要写业务逻辑和功能，剩下的交给 AWS 或者 Azure。

无服务器计算还可以降低您的计算成本。虽然大多数云提供商将按小时收取保留虚拟机的费用，但无服务器计算可以使用基于消耗的定价模式。如果应用程序没有主动使用计算或内存资源，则无需付费。

## 无服务器计算提供商

云计算供应商正在推动这一领域的大量创新。下面的比较集中在三个方面:构建(支持的语言)、部署(易于设置和管理)和触发器(触发功能)。

### 自动气象站λ

AWS lambda 是 2014 年推出的首批无服务器计算产品之一。

AWS Lambda 原生支持多种语言，包括 Node.js、Python、Java 和 C#(。网芯)。通过从 AWS Lambda 沙箱中允许的受支持语言之一生成子进程，可以支持其他语言。AWS 沙箱隔离不依赖于任何允许这种灵活性的语言结构。

开发可以使用 AWS 的嵌入式编辑器在线完成。然而，一旦您需要添加依赖项以使您的代码运行，您需要在您的本地机器上进行开发，并上传一个由您的代码以及任何依赖项组成的包。如果你开发一个节点函数，AWS Lambda 将不会运行`npm install`。部署需要创建并上传一个部署包，将代码和依赖项捆绑在一起。

AWS 将 AWS Lambda 定位在 AWS 的前端和中心，而不仅仅是一个额外的产品，它提供大量的 AWS 服务，可以作为一键[触发器](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html)，在使用 Lambda 方面提供很大的灵活性。如果您正在开发一个 API，可以使用非常流行的 AWS API 网关触发器。API Gateway 使您能够将针对 RESTful API 端点(如`GET /items/{id}`)的 HTTP 请求映射到 AWS lambda 函数`getItem(event, context, callback) { }`。在 REST APIs 之外，AWS 提供了各种触发器，从 Amazon Kinesis(类似于 Kafka)的事件流到 DynamoDB 或 S3 的更新，甚至可以从 Alexa 技能应用程序触发。事实上，AWS 正在推动将 AWS Lambda 作为开发 Alexa 新技能的*主要*方式。

### Azure 函数

随着 Azure 功能在 2016 年年中推出，Azure 在 AWS 之后不久进入了无服务器领域。与 AWS 相比，Azure 支持更多种类的语言。除了 Node.js、C#和 Python，Azure 还支持 F#、PHP、Bash 和 PowerShell。2017 年 10 月 4 日，Azure 宣布他们将在旧金山 JavaOne 大会期间支持 Java。此外，他们的[逻辑应用](https://azure.microsoft.com/en-us/services/logic-apps/)和[流程](https://flow.microsoft.com/en-us/)使非开发者能够为业务流程建立自己的逻辑，就像 Zapier 如何连接多个业务工具一样。

Azure 提供了类似 AWS 的在线编辑器。然而，Azure 的编辑器是基于 Visual Studio Online 构建的。与 AWS 和 Google 不同，Azure 围绕部署提供了更多的基础设施。您可以从 Visual Studio Team Services、Bitbucket 和 Github 中的源设置连续构建和部署部署。

在架构上，Azure Functions 与 AWS Lambda 截然不同，因为 Azure Functions 的大部分基础设施都来自 Azure 应用服务和应用服务计划。Azure 功能被逻辑地分组到一个名为*应用服务的应用容器或环境中。*应用服务中的所有 Azure 功能共享相同的资源，如计算或内存。这也使得能够部署应用程序而不是单个功能。你可以把 Azure 函数看作是 AWS Lambda 和更传统的 Azure Web Apps/AWS Elastic Beanstalk 类环境的混合体。

与 AWS 不同，更多的 HTTP 触发器功能是在 Azure 函数中本地构建的，不需要设置单独的 API 网关。这很大程度上与 Azure 在应用服务容器下对功能的逻辑分组有关。Azure 支持各种其他的[触发器](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-cosmos-db-triggered-function)。这样的触发器可以包括 Azure Blob 存储、Azure 事件中心和队列。

### 谷歌云功能

相对于 Azure 和 AWS，Google Cloud functions 在语言方面是最受限制的，因为它们只支持 Node.js。不幸的是，Google Cloud Functions 相对于 AWS 和 Azure 似乎是最不发达的。这部分不是因为谷歌的云功能，而是谷歌在谷歌云中提供的可以作为触发器的服务最少。AWS 有 Kinesis，Azure 有 EventHubs，但谷歌没有类似的服务。与 AWS 不同，谷歌没有将云功能推到前台和中心。谷歌的内部开发更多地依赖于像 Kubernetes 这样的工具，而不是云功能。

从积极的一面来看，Google 允许在进程被终止之前执行长达 9 分钟的时间。谷歌还为 Firebase 提供了一个独立的产品*云功能，如果你是一个已经依赖 Firebase 的移动应用初创公司，这可能会很有用。Firebase 使您能够从 Firebase Db 的更新中产生一个新的云函数。与 AWS 不同，Google 将 HTTP 功能直接集成到云功能中，不需要设置单独的 API 网关。*

对于部署，您可以上传一个 zip 文件或从 Google 仓库进行部署。

## 独立的无服务器框架

因为无服务器计算是由云提供商高度管理的服务，所以有很大的局限性。触发器的实现是特定于每个提供商的，因此你不会在 Google 或 Azure 上找到相同的 AWS Kinesis 触发器。除了不同的触发器之外，根据使用 AWS Lambda vs Google Cloud Functions vs Azure Functions，传入的上下文和顶级函数签名也是不同的。

虽然锁定的可能性很高，但是您可以通过利用能够翻译各种服务的供应商中立垫片来减轻这种情况。这种供应商中立的垫片允许您以类似于 Azure EventHubs 的方式使用 AWS Kinesis。此外，HTTP APIs(无服务器的一种非常常见的用法)可以完全开放和透明。

有独立的开源框架可以做到这一点。此外，它们使部署标准化，并使您可以访问的特定于供应商的上下文对象标准化。

有相当多的框架根据您的需求而变化。一些受欢迎的包括:

*   [无服务器](https://github.com/serverless/serverless)
*   [kub less](https://github.com/kubeless/kubeless)
*   [裂变](http://fission.io/)
*   [功能](https://funktion.fabric8.io/)

## 监测挑战

无服务器计算日益增长的挑战之一是监控和调试所有这些功能。与微服务架构相比，日志记录上下文现在分散在更多的组件中。许多日志都是特定于供应商的，不像标准的 NGINX 或 HaProxy 日志。

此外，在相同的环境中，很难在本地镜像和运行用于调试的函数。至关重要的是通过故障场景尽可能多地包含调试上下文，而不仅仅是在本地重现问题。有时，您无法在云供应商的环境中重现该问题。

在 Moesif，我们正在努力提供对这种无服务器功能调用的可见性，并且已经有了带有不同触发器的 AWS Lambda 的 SDK，如 [Moesif 的用于 API 网关的 Lambda 中间件](https://www.moesif.com/docs/server-integration/aws-lambda-nodejs/)和 [Moesif 的用于 Alexa 技能工具包的 Lambda 中间件。](https://www.moesif.com/docs/server-integration/alexa-skills-nodejs/)