# 在您的 API 平台中实现 HIPAA 技术保护

> 原文：<https://www.moesif.com/blog/technical/compliance/Implementing-HIPAA-Technical-Safeguards-in-your-API/>

健康保险可携带性和责任法案(简称 HIPAA)是一套关于在信息系统中处理健康相关数据的法律。它定义了安全措施，即您在为客户处理健康数据时必须遵循的规则。

有三种保护类别:

*   管理保护措施，涵盖您公司的人员和流程
*   物理保护，涉及运行处理健康数据的软件的硬件
*   技术保护，软件必须实现的要求

如果您希望您的 API 符合 HIPAA，那么这三个类别都必须正确处理。在另一篇文章中，我们介绍了这些关键需求以及如何构建符合 HIPAA 标准的 API 平台。在本文中，我们将深入探讨如何实现技术安全措施的具体细节。

有九种技术安全措施，其中四种是必需的，五种是可解决的。在任何情况下都必须实施必要的保障措施；如果合理的话，应该实施可寻址的安全措施。如果您不能实现可寻址的安全措施，您需要记录原因并解释为什么没有实现它们——这在数据泄露的情况下非常重要。

虽然物理安全不是本文的主题，但是请记住，您需要将您的基础设施托管在符合 HIPAA 的提供商处。否则，有可能通过物理访问服务器来绕过某些技术保护措施。如果您计划内部部署，您需要自己遵循物理安全措施。

## 所需的技术保障

让我们从四个必需的安全措施开始。从技术角度来看，这些是符合 HIPAA 的最低要求。

### 唯一用户标识

系统的每个用户都需要有一个唯一的标识符。以便系统上的每个操作都可以追溯到执行该操作的用户。

一种简单的方法是使用通用唯一标识符(UUID)。每种主要的编程语言都有一个用于创建 UUID 的库。UUID 生成器遵循标准化的算法来创建全球唯一的字符串，而无需中央权威机构。

符合 HIPAA 的托管认证服务，如 [AWS Cognito](https://aws.amazon.com/about-aws/whats-new/2017/07/amazon-cognito-achieves-hipaa-eligibility-and-pci-compliance-adds-a-simple-way-to-sign-in-users-by-email-address-or-phone-number-and-localizes-the-management-console/) ，将为您完成这项工作。Cognito 用户池将在注册时给每个新用户一个唯一的 ID。

如果您必须自己实现唯一用户标识，您可以在这里找到 UUID 的实现:

*   [Node.js](https://www.npmjs.com/package/uuid)
*   [Java](https://docs.oracle.com/javase/7/docs/api/java/util/UUID.html)
*   [Python](https://docs.python.org/3/library/uuid.html)
*   [。网络](https://docs.microsoft.com/en-us/dotnet/api/system.guid.newguid)

### 紧急进入程序

您将需要一种在紧急情况下访问受保护的医疗保健数据的方法。在这种情况下，紧急情况意味着你的系统或应用程序发生了不好的事情。例如，你的电力中断了，或者你受到了网络攻击。在这种情况下，需要通过一种替代方式来访问数据。

要实现这种紧急访问，您需要解决两个问题。第一种是某种类型的异地备份，即使您的数据中心被夷为平地，它也能让您访问数据。第二个是一个警报系统，通知你紧急情况正在发生。

对于托管数据库，比如 Amazon DynamoDB，这意味着您需要[为您的表激活时间点恢复](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.html)。一旦打开，最近 35 天的表格数据每五分钟备份一次，并且可以恢复。

如果您用 CloudFormation 定义您的表，您只需要为此添加两个属性:

```py
Type: 'AWS::DynamoDB::Table'
Properties:
  TableName: phiTable
  PointInTimeRecoverySpecification:
    PointInTimeRecoveryEnabled: true 
```

对于其他著名的数据库，您可以在下面找到如何设置备份的指南:

*   [PostgreSQL](https://www.postgresql.org/docs/9.1/backup.html)
*   [马里亚布](https://mariadb.com/kb/en/backup-and-restore-overview/)
*   [SQL 服务器](https://docs.microsoft.com/en-us/sql/relational-databases/backup-restore/create-a-full-database-backup-sql-server?view=sql-server-ver15)
*   [MongoDB](https://docs.mongodb.com/manual/core/backups/)

### 审计控制

审计控制就是记录谁访问了医疗保健数据。它与唯一用户标识协同工作。如果您被审计，您将需要让审计员查看审计日志数据，以向他们显示一切都被正确跟踪。

一种简单的方法是为医疗保健数据建立专用的数据存储，并记录所有访问。这样，您就可以确保在您的应用程序中添加新功能时，没有开发人员会忘记实现日志记录。此外，当您将医疗保健数据与非医疗保健数据一起存储时，您不会有太多的审计日志。

另一种方法是[实现事件源](https://microservices.io/patterns/data/event-sourcing.html)。在这种数据架构模式中，您只将更改保存在一个不可变的列表中，并动态地生成表。这样，三次更新实际上是事件源数据库中的三条记录，而不仅仅是最终结果。您可以看到数据在每个时间点发生了什么变化，以及是谁导致了这些变化。

您可以在以下位置找到有关如何为重要数据库启用访问日志的信息:

*   [PostgreSQL](https://www.postgresql.org/docs/9.1/runtime-config-logging.html)
*   [马里亚布](https://mariadb.com/kb/en/overview-of-mariadb-logs/)
*   [SQL 服务器](https://blog.netwrix.com/2019/05/23/how-to-enable-sql-server-audit-and-review-the-audit-log/)
*   [MongoDB](https://docs.mongodb.com/manual/core/auditing/)

## 证明

对所有用户进行身份验证，这样就不会发生匿名访问医疗保健数据的情况。最佳做法是在任何可能访问医疗保健数据的地方使用密码和多因素身份认证。如果用户的身份认证因素之一被盗，例如解锁的智能手机，他们需要能够证明他们后来没有访问过受保护的数据。

使用像 [AWS Cognito](https://aws.amazon.com/about-aws/whats-new/2017/07/amazon-cognito-achieves-hipaa-eligibility-and-pci-compliance-adds-a-simple-way-to-sign-in-users-by-email-address-or-phone-number-and-localizes-the-management-console/) 这样的托管服务可以节省您的时间，因为它开箱即符合 HIPAA 标准。 [Auth0](https://auth0.com/docs/compliance) 和 [Okta](https://www.okta.com/hipaa/) 也是非常好的托管认证服务，它们也符合 HIPAA。

对于您自己的实现，您可以在下面找到主要 API 平台的身份验证库:

*   [快递](http://www.passportjs.org/)
*   姜戈
*   [ASP.NET](Core https://docs.microsoft.com/en-gb/aspnet/core/security/)
*   [Spring Boot](https://spring.io/guides/gs/securing-web/)

## 可解决的技术安全措施

可寻址的技术安全措施代表了 HIPAA 要求的大部分，尽管只有 52%的微弱多数。严格地说，它们是可选的，但是您应该将它们的“可寻址”本质视为安全措施，允许实现上的一些灵活性。

事实上，你可以实现，找到一个替代方案来完成同样的目标，或者不实现。如果您选择不实施，您需要记录下原因，并可能在进一步实施之前咨询审计专家/律师。

### 自动注销

保持您的用户会话尽可能短，并自动注销。基本的想法是，如果有人在登录后离开他们的设备，它可能会被未经授权的人使用。如果您在用户在线一两分钟不活动后将其注销，发生未授权访问的几率会降低。

要做到这一点，您需要降低 API 连接的会话超时。客户端可以自动请求新的会话令牌，所以与 GUI 应用程序相比，这通常不是什么大问题。使用 GUI，用户必须手动重新登录。

### 加密和解密

所有医疗保健数据都应加密，仅在需要时解密。这意味着静态加密，比如在您的服务器或用户设备上。这也意味着当数据通过网络传输时要加密。原因很明显——如果数据是加密的，窃取数据不会帮助攻击者。

虽然 HIPAA 规定的加密要求是技术中立的，但是您应该尽可能合理地加密数据。这意味着 ROT13 是不够的，但一次性垫是多余的。检查您的技术提供了哪些最先进的加密机制，并坚持使用。

同样，AWS DynamoDB 等托管服务会自动加密您的静态数据。

您的 API 应该强制所有连接都使用 SSL。使用[严格传输安全头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)通知您的客户使用 SSL。如果协议是 HTTP，不要传递任何数据，它应该只通过 HTTPS 用[位置头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location)和一个 3XX 状态码重定向。也许，在正文中给出一些“不支持 HTTP”的信息。

如果您不能加密数据，但是有其他机制可以阻止对数据的访问，那么您应该记录下来。

### 认证 ePHI 的完整性控制和机制

您需要一种方法来确保医疗保健数据不会以未经授权的方式被更改或破坏。

这意味着您需要检查数据的完整性，例如，使用校验和或数字签名。如果经过身份验证的用户签署了更改的数据，那么未经授权的用户所做的任何后续更改都将是显而易见的。

为了防止未经授权的数据破坏，您必须实现一个不会同步未经授权用户数字签名的更改的备份。如果恶意用户删除数据，它要么会显示在审核日志中，要么删除不会传播到备份。

同样，事件源可以是解决方案。因为数据永远不会被隐式删除，所有的更改都会被设计记录下来。

## 如何从其他 API 访问 ePHI 数据？

如果您自己的数据已经符合 HIPAA，第三方 API 不是大问题，但是如果您不符合 HIPAA，它会导致额外的成本。

当您想要使用来自第三方 API 的 ePHI 数据时，如 [EPIC](https://open.epic.com/) ，您还需要应用安全措施。并非所有由 ePHI API 交付的数据都在 HIPAA 的保护之下，但是如果您请求的数据在 HIPAA 的保护之下，您就需要实现它。这还包括本文中没有提到的物理和管理保护措施。

## Moesif 如何符合 HIPAA？

Moesif 提供了一个安全代理，您可以将它托管在自己的基础设施上。这个代理将在把你的数据发送给 Moesif 之前对其进行加密，并在显示给你之前对其进行解密，所有这些都是用你自己的私人加密密钥完成的。主加密密钥从不存储在 Moesif 服务器上；相反，安全代理支持流行的密钥存储并自动处理密钥轮换。

最后，Moesif 永远不会看到你的未加密的 ePHI 数据。

这种方法也是您可以在自己的 API 中记住的。如果客户想要向您发送他们的 ePHI 数据，告诉他们应该首先加密这些数据，或者向他们提供您自己的安全代理，他们可以在自己的基础架构上托管这些数据。

## 摘要

如果你想实现一个 HIPAA 兼容的 API，你最好的做法是实现所有的安全措施，甚至是可选的。今天就降低你的风险，避免以后不舒服的技术和可能的法律问题。

一些安全措施实际上是分层的，例如，实现唯一的用户标识符将遵循独立的身份验证。加密和解密也是如此，两者都被认为是独立的安全措施，但实际上是一起工作的。

此外，备份、身份验证和加密是保护您整个企业的良好实践，而不仅仅是您可能持有的与健康相关的数据。因此，您的 API 很有可能已经具备了大部分保护措施。

你也应该去看看像 AWS、Azure 或 GCP 这样的云提供商提供的托管服务。通常，这些提供商已经做了大量的工作来满足要求，您只需选择配置正确的资源。