# 不要把你的 API 数据塞进振幅里

> 原文：<https://www.moesif.com/blog/technical/api-analytics/Dont-Shove-API-Data-Into-Amplitude/>

当达到最小可行产品(MVP)时，只关注你的核心特性是谨慎的商业实践。微服务架构允许您将解决方案的非差异化部分外包给第三方提供商；使用其他人进行用户管理、计费和帐户管理。

乍一看，开发自己的 API 分析解决方案似乎很有吸引力，也许是通过构建在 web 分析工具之上，如 Amplitude、MixPanel 或 Segment。但是一旦你剥开洋葱皮，你很快就会意识到你会因为上传限制、最小的维度支持和有缺陷的可视化而不必要地削弱你的解决方案。

> 为工作使用正确的工具，并采用最佳案例分析解决方案，一个专为 API 产品构建的解决方案，如 Moesif API Analytics。

## Web 分析与 API 分析非常不同

Amplitude 非常适合跟踪用户旅程，并通过简单的 logEvent API 调用分析实时用户交互。当一个 Amplitude 开发者加入这个平台时，一些开发者会尝试使用 Amplitude 作为一个包罗万象的整体平台来满足他们的分析需求。然而，在实现阶段，开发人员会发现 Amplitude 平台在上传限制(批次/秒)、字段限制、维度限制和规模限制方面有严重的局限性。由于这些限制，开发人员将很难在 Amplitude 上构建他们的整个分析解决方案，并且需要购买其他解决方案来满足他们的分析需求。

最重要的是，产品经理(pm)和产品营销经理(pmm)需要报告 KPI，但不一定拥有开发知识或技术资源来实现复杂的自我报告仪表板。因此，振幅有时在大型组织中无法使用，因为 KPI 仪表板需要在业务单位的基础上进行定制。有了 Moesif，任何项目经理或 PMM 想要的所有 API/KPI 数据都可以在一个简单的下拉菜单界面中获得。在本文中，我们将讨论为什么不应该将 API 数据硬塞进 Amplitude，而应该考虑一个为 API 的独特需求定制的解决方案。

## 使用 Moesif 轻松跟踪 API 数据

Amplitude 的客户加入该平台时设想，他们将拥有创建图表来跟踪重要 KPI 和指标数据的无缝体验。然而，一旦实现了 Amplitude，项目经理和他们的开发团队很快就发现建立图表来跟踪这些度量是一项单调乏味的手工工作。

开箱即用的 Moesif 使您的团队能够跟踪对您的基础设施和平台/营销团队都很重要的 API 指标。无论您是在开发运维、产品管理、应用工程还是发展领域，Moesif 都旨在跟踪您关心的指标数据。在某种程度上，您的 API 数据最初并不是为您配置的，并且您跟踪单个用户数据或总体公司指标的选项非常有限。穆塞夫

这是一个指标数据的列表，可以用 Moesif 跟踪，并且必须在振幅中自定义创建:

*   API 使用增长
*   独特的 API 消费者(API DAU 和 MAU)
*   按 API 使用情况列出的顶级客户
*   API 保留
*   第一个 Hello World (TTFHW)的时间到了
*   正常运行时间、CPU 和内存使用量
*   每分钟请求数(RPM)、平均和最大延迟、每分钟错误数

您的团队应该跟踪的顶级 API 指标的完整概要在[companion]((https://www . moesif . com/blog/technical/API-Metrics/API-Metrics-That-Every-Platform-Team-Should-be-Tracking/？UTM _ campaign = Int-site & UTM _ source = blog & UTM _ medium = body-CTA & UTM _ term = API-solutions){:target = " _ blank " }博客文章。

借助 Moesif 的保存工作区功能，团队可以保存报告并在整个组织内共享。有没有一些只有你想看的报告？使用 Moesif 创建私有仪表板很容易，只需修改您工作区的“编辑共享”设置，并选择您希望您的工作区是私有的、与您的团队共享的还是公共的。要了解有关该功能的更多信息，[请点击此处](https://www.moesif.com/blog/announcements/features/Product-Update-Saved-Workspaces-to-Track-And-Share-API-KPI-Reports/?utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-solutions)。

![Saved Workspace - Dashboard](img/109dc4aed5e35f1e1c82b549ad593ef7.png)

## Moesif 与振幅 API 数据限制

Amplitude 对每月可以触发的事件数量以及每秒可以发送的事件数量有限制。Moesif 有一些限制，比如 JSON 负载大小，但是正如您从下表中看到的，Moesif 更具成本效益，并且不需要您注册昂贵的企业计划来访问超过 1000 个事件/秒。通过 Moesif 的现收现付定价，您只需为您使用的活动付费，如果您超出分配的计划，Moesif 不会阻止您的请求:

|   | 穆塞夫 | 振幅 |
| --- | --- | --- |
| 批量 | 12MB | 20MB |
| 上传限制 | 没有限制 | 100 批/秒和 1000 个事件/秒，超过 1000 个事件/秒需要企业计划 |
| 设备限制 | 没有限制 | 每个请求 2000 个事件，小于 1MB |
| 请求限制 | 没有限制 | 跨所有 REST API 端点的 5 个并发请求 |
| 事件类型 | 没有限制 | 每个项目 2000 个事件类型 |
| 事件属性 | 没有限制 | 每个项目 2000 个事件属性 |
| 用户属性 | 没有限制 | 每个项目 1000 个用户属性 |

使用 Moesif，在 API 数据跟踪方面没有上限。Moesif 允许开发人员收集客户想要发送的任意多的事件。该基础架构是围绕集群和节点设计的，可以轻松地进行动态扩展。使用 Moesif，负载在多个集群/节点之间平衡，这允许您的 API 数据无限扩展。

## 使用 Moesif 的 API 安全性和可伸缩性

Moesif 在其平台中实施了顶级的安全性，并且完全符合 GDPR 和 SOC 2 标准。所有的 API 令牌都使用 HMAC SHA-256 进行签名，Moesif 的 SSL 实现在 Qualsys 的 SSL 实验室分析中获得了 A+，所有 Moesif 的 SDK 和代理都被配置为默认使用 HTTPS。借助 Moesif 的安全代理，HIPAA 合规性与本地客户端加密和自带密钥(BYOK)无缝结合。这些安全特性将使您能够在跟踪 API 分析时，在任何行业或用例中实现 Moesif。

## 使用 Moesif vs Amplitude 的 API 分析管理

如您所见，使用 Moesif 管理您的 API 分析使您的企业能够快速创新，并使您的开发人员能够轻松构建完全可扩展和兼容的分析仪表板。Moesif 使组织能够创建强大的仪表板，跟踪常见的 KPI 和可根据您的业务需求定制的指标数据。此外，Moesif 不会对 API 的使用设置任意的限制，因为没有理由忽略一个 API 请求。准确的数据报告是为您的客户建立强大、可靠业务的基础，Moesif 非常乐意通过我们强大的 API 分析平台帮助您实现这些目标。要了解更多关于 Moesif 的信息并查看其工作原理的简要概述，请查看我们的[介绍视频](https://www.youtube.com/watch?v=sopAQL-QWIM&utm_campaign=Int-site&utm_source=blog&utm_medium=body-cta&utm_term=api-solutions)。