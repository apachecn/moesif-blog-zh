# 如何使用 Mailgun 和 Moesif API Analytics 发送行为邮件

> 原文：<https://www.moesif.com/blog/developer-marketing/behavioral-emails/How-to-Send-Behavioral-Emails-With-Mailgun-And-Moesif-API-Analytics/>

在本指南中，你将学习如何使用 Mailgun 发送 Moesif 行为邮件。

[Moesif 行为电子邮件](https://www.moesif.com/features/user-behavioral-emails)是一项根据客户的 API 使用情况自动向客户发送电子邮件的功能。这可用于通知客户有关技术问题，如达到速率限制、使用废弃的 API 或集成中断。您甚至可以使用它来触发与业务相关的事件，比如某个商品何时发货。如果某个东西可以映射到一个 API 调用，那么你可以从它发送一封电子邮件。

在[的配套文章](https://www.moesif.com/blog/technical/behavioral-emails/How-To-Accelerate-API-Integration-with-Behavioral-Emails-and-Developer-Segmentation/)中，我们介绍了如何在 Moesif 仪表板中配置行为电子邮件。

[Mailgun](https://www.mailgun.com/) 是一种托管电子邮件服务。他们提供了一个 API 来向你的用户发送电子邮件，并为你托管 SMTP 服务器。

## 先决条件

该操作指南需要一个 [Moesif 账户](https://www.moesif.com/wrap?onboard=true)和一个 [Mailgun 账户](https://signup.mailgun.com/new/signup)。

## 设置电子邮件服务器

设置电子邮件服务器的第一步是从 Mailgun 获取 SMTP 凭证。

为此，您必须登录到 Mailgun [仪表板](https://app.mailgun.com/app/dashboard)。

下面的屏幕截图说明了如何导航到凭据。

![ Mailgun Dashboard](img/58c47bd729facd16ede37dc7dd822287.png)

这些凭证必须输入到 Moesif 电子邮件服务器配置表中。

要导航到该表单，请遵循下一个屏幕截图中的步骤。

![ Moesif Email Server Form](img/ce6a5a5c5d77c09cb5b5b8aff5d8765c.png)

您可以将凭据从 Mailgun 站点复制并粘贴到 Moesif 站点。

Moesif 表单如下所示:

![ Moesif Email Server Form](img/64fd910032316f031fde3fff5095c2a7.png)

## 测试设置

然后，您可以使用 Moesif 通过新配置的 SMTP 服务器发送一封示例电子邮件。

配置完邮件服务器后，您必须创建一个新的电子邮件模板:

![Moesif Create a Blank Email Template](img/5584b9181cef3a64d55931d070f69d7a.png)

对于新模板，您必须填写必填字段:模板名称、主题行和发件人电子邮件地址。

完成后，点击“测试”按钮并输入目标电子邮件。

![Moesif Add From Email](img/8d05925902b6dff21ab2280fdf6bcfa9.png)

如果一切正常，你会在收件箱里看到一封新邮件，同时 Mailgun 仪表板会显示它已经发送了该邮件。

![ Moesif Email Server Form](img/cbad273be0ce6e0663ae79ab18192416.png)

## 摘要

用 Mailgun 设置 Moesif 行为邮件只需要几分钟。

使用 Mailgun 这样的服务可以缓解发送电子邮件时可能出现的许多常见问题，例如验证问题和电子邮件被错误地识别为垃圾邮件。

通过将 Moesif 的行为电子邮件服务与 Mailgun 相结合，您可以让您的 API 消费者及时了解重要问题。