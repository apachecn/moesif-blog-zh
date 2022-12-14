# 在非 API 优先公司中启动 API 程序

> 原文：<https://www.moesif.com/blog/podcasts/developer-marketing/Podcast-Launching-API-Programs-in-Non-API-First-Companies/>

Ep。11: Jeannie Hawrysz，SAS API 项目负责人

### Moesif 的播客网络:为 API 产品经理和其他 API 专业人士提供可操作的见解。

加入我们的是拥有 22，000 名员工的商业分析公司 SAS 的首席 API 架构师 Jeannie Hawrysz。在此之前，她在 IBM 工作了 18 年，是 IBM 的 API Connect 微网关的技术开发经理。在我们的播客中，她分享了如何在非 API 优先的公司中成功启动 API 项目。

莫里斯国际基金会的 CMO 拉里·埃布林格是今天的主持人。

[https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1006998298&color=%23ff4f00&auto_play=false&hide_related=true&show_comments=true&show_user=true&show_reposts=false&show_teaser=true](https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1006998298&color=%23ff4f00&auto_play=false&hide_related=true&show_comments=true&show_user=true&show_reposts=false&show_teaser=true)

[Moesif](https://soundcloud.com/moesifhq "Moesif") · [11\. Launching API Programs in Non API-First Companies](https://soundcloud.com/moesifhq/11-launching-api-programs-in-non-api-first-companies "11\. Launching API Programs in Non API-First Companies")

在 SoundCloud 上听上面的剧集，或者在苹果或谷歌上下载。

[![Listen on Apple Podcasts](img/0f035c1005fd9c71c8f40242b9519f3d.png)](https://podcasts.apple.com/us/podcast/11-launching-api-programs-in-non-api-first-companies/id1545437670?i=1000513009354) [![Listen on Google Podcasts](img/5dde28e33453b3096c5f996f6d62b5af.png)](https://podcasts.google.com/feed/aHR0cHM6Ly9mZWVkcy5zb3VuZGNsb3VkLmNvbS91c2Vycy9zb3VuZGNsb3VkOnVzZXJzOjg4ODY5ODI0NS9zb3VuZHMucnNz/episode/dGFnOnNvdW5kY2xvdWQsMjAxMDp0cmFja3MvMTAwNjk5ODI5OA?sa=X&ved=0CAUQkfYCahcKEwiA97nJ1rLvAhUAAAAAHQAAAAAQAw)

目录

*   [1:14 从事 API 管理工作](#work-in-api-management)
*   [4:45 形成卓越中心](#form-a-center-of-excellence)
*   [8:58 搭建跨越生命周期的桥梁](#build-bridges-across-the-lifecycle)
*   [10:30 汗发人物](#sweat-dev-personas)
*   [12:25 工作流程指南开发者](#workflows-guide-devs)
*   [15:50 外部 API 的商业原因](#biz-reasons-for-external-apis)
*   [18:29 外化的最低标准](#minimum-criteria-for-externalizing)
*   [22:43 产品管理负责](#product-management-is-accountable)
*   [24:28 原料药赚钱](#apis-make-money)
*   [27:34 4 技术注意事项](#4-technical-considerations)
*   [30:15 与业务需求保持一致](#align-with-a-business-need)
*   [31:37 有耐心，有眼光，有同理心](#have-patience-a-vision-and-empathy)

拉里·埃布林格(Moesif): 欢迎来到 Moesif 的 APIs over IPAs 播客网络第 11 集。我是劳伦斯·埃布林格，今天的主持人，也是 Moesif 的首席营销官，API 可观测性平台。和我一起的是[珍妮·霍利斯](https://www.linkedin.com/in/jeannie-kristufek-hawrysz-70a0a85/)，首席 API 架构师和 SAS 经理，SAS 是一家拥有 22，000 名员工的商业分析公司。在此之前，她在 IBM 工作了 18 年，是 IBM 的 API Connect 微网关的技术开发经理。

珍妮，欢迎来到我们简陋的播客网络。我们在哪里能找到你？"

珍妮·霍利斯(SAS): 谢谢。所以我住在北卡罗来纳州的卡里。我在特种空勤团工作。事实上，我现在在家和家人一起工作。

拉里:和我们一样。

珍妮:我住在北卡罗来纳州，我喜欢这里。山和海滩，你不会错的。

## 从事 API 管理工作

“我的项目被取消了……我应该做些什么……你应该试试这个 API 管理的东西”,他递给我一本奥莱利的书，名为《API:战略指南》。我看了一夜。我爱不释手。我就是这样进入 API 和 API 管理的。

拉里:太棒了。好了，开始吧，你为什么不和我们分享一下你过去 18 年在 IBM 使用 API gateway 产品的经历，然后让我们了解你在 SaaS 运行所有 API 的情况。

Jeannie: 所以我会试着很快地讲述 18 年。我真的非常非常幸运，在我的职业生涯中有很多不同的机会，无论是在 IBM 还是在 SAS。我最初是一名从事大型机 TCP/IP 协议栈的工程师，后来我从事支持、性能和开发方面的工作。我第一次有机会成为跨部门的技术领导，去开发一个 SDK，人们可以用它来开发软件和虚拟硬件设备。不幸的是，这个项目实际上在 2012 年夏天因为商业优先的原因而终止了。这是一个相当大的败笔，因为这是我第一次领导的大好机会。但是所有从事这项工作的人都被邀请来从事数据电源设备的工作，这些设备做得非常好。有许多不同的技术领域需要工程帮助。当时 data power 的 CTO 实际上是我实习时的导师，这是一种偶然。所以我马上去找他说，“我的项目被取消了，我要去你的地区，我应该做些什么呢？”

他说“你知道，我认为你应该在 API 管理方面下功夫。”我不知道那是什么，我不知道。但他递给我一本名为 [APIs: A Strategy Guide](https://www.oreilly.com/library/view/apis-a-strategy/9781449321628/) 的书，那是一本奥莱利的书，我一夜之间就看完了。我爱不释手。我就是这样进入 API 和 API 管理的。

在接下来的几年里，我作为一名工程师为 IBM 开发 API 管理解决方案的监控组件。后来，我成为了一个软件开发经理，负责开发 data power RestAPIs 的团队、几个云集成团队，以及最终的 IBM API Connect Microgateway。

奇怪的是，我离开 IBM 并开始寻找 IBM 之外的工作的原因之一是我真的厌倦了 API。我“精疲力尽”了，准备开始看别的东西。我有宏伟的计划。我要成为一名安全专家，彻底重塑自己。但不知何故，我最终又回到了 SAS，从事 API 方面的工作。但是，这是不同的。这与我在 IBM 的经历非常非常不同。我不得不学习、成长和拥抱更多的 API 生命周期，而不仅仅是技术方面。学到了很多我职业生涯早期没有学到的东西。所以这很有趣。

我的职业生涯实际上是围绕着网络的，但是所有的一切似乎总是被 API 所吸引。这真的很令人兴奋，就像我在 SAS 的经历一样，能够在一个项目的底层工作。

## 形成一个卓越的中心

API 工程卓越中心管理标准、指南和通用模式，提供目录，采用最新版本的 OpenAPI，拥有模式验证和样式规则，并审查每个版本

Larry: 嗯，这是 API 领域的丰富经验。我知道奥赖利那本书，它开启了你的旅程。这非常令人兴奋。无论是在 IBM 还是在 SAS，你已经构建了听起来相当多的 API 程序。据你估计，在一个现有的企业软件公司中，建立一个 API 程序的关键步骤是什么，根据定义，显然不是 API 优先的。

Jeannie: 所以，我要说清楚的是，在 IBM，我并不是真正的木偶师、幕后操纵者，我更像是一名软件开发经理。但是我利用这个机会尽可能多地了解了生命周期的剩余部分，我真的很感激这段经历。如果我只是尝试领导一个关于 API 的项目，而没有技术经验，我不认为我会成功。

当我在 2017 年以这个角色来到 SAS 时，实际上有一些 API 计划的好元素。他们确实有一个卓越中心，这是非常好的。他们有标准和指导方针，以及一些共同的模式。它们都被写了下来，并且已经被讨论过了。他们有一个内部 API 目录，这很棒。他们使用斯瓦格·V2，这是 2017 年的选择。他们有模式验证和样式规则，当时是自动化的。他们对每个版本都进行 API 审查。所有这些东西都非常非常棒。当我到达那里时，我想我不会有我的工作，因为他们已经解决了所有的问题。但是当我开始梳理事情的时候，我发现我们真的只关注了项目的技术方面。在业务方面也不是很好，有些东西不工作，或者不会扩展，或者不会与 CI/CD 一起工作。在我加入 SAS 的时候，我们真的没有大规模地进行 CI/CD，我们正在开始将公司向那个方向转变的过程。使用集中的团队来审查每个版本是行不通的，所以我们需要开始考虑如何分散。

一旦我了解到我们在开始一个程序的时候比我想象的要多，我肯定会害怕，因为我以前从来没有运行过 API 程序或任何这种规模的程序。我真的认为，作为一名架构师，这份工作更多的是在架构上工作，并为所有这些事情编写平台支持代码。结果是，那些东西可能处于最佳状态，而不是当时的优先事项。我们需要开始思考如何与业务的其他部分保持一致，并查看生命周期中的差距并填补它们。我问自己“我该怎么做？我怎么转大船。”我以前从未做过这种事，而且对公司来说还很陌生，当我担任这个角色时，我已经在公司工作了一年多一点，我需要建立一些信誉，因为我是谁来告诉人们他们认为他们都在正确的轨道上，有些事情需要改变？我不想成为那个从外面进来说“一切都很糟糕”的人，因为事情并不糟糕，但如果我们要让这个项目成功，肯定有一些事情需要关注。

所以过去三年我一直在解决这个问题。我真的相信 API，我相信 API 是一个商业基础。在 IBM 和 API 经济中拥有这样的背景真的激励我去实现它。我真的相信可以。所以我认为这里有机会，SAS 有如此丰富的分析能力，我知道我们可以用我们的 API 来区分。

## 搭建跨越生命周期的桥梁

*找到其他部门的信徒，帮助在整个公司内搭建桥梁，不仅仅是在工程部门，还包括文档、产品管理和支持部门*

一开始确实感觉很慢，因为最重要的是我需要开始了解我们在 API 作为一个程序的真正位置以及我们需要去哪里。我们需要获得领导地位，不仅是在工程部门，还包括文档、产品管理和支持等其他领域。我们需要让整个公司的人都明白商业价值和如何前进。我非常非常幸运地在其他部门有一些信徒，他们帮助我在那里建立了领导地位，并开始建立跨越生命周期的桥梁。肯定有一些人很早就从 API 程序的角度理解了我们想要做的事情。

最重要的是，我们开始安排关键角色，这样他们就可以帮我实现这个目标，因为我不可能一个人完成所有的事情。我们添加的第一件事是开发人员倡导者，因为我们需要有人能够围绕 API 在内部和外部帮助宣传。我们增加了一个项目经理，他现在实际上更像是一个程序而不是项目经理。文档负责人，我们组建了我们所谓的 API 优先核心团队，这是一个跨部门的领导团队，负责提供战略、方向和支持，这是所有 SAS 实现 API 所必需的。

## 汗水 Dev 人

*开发者应该是你软件的一流用户，他们应该拥有出色的体验*

有两件事我想提一下，这两件事给了我们最大的飞跃:首先，作为一个整体公司，我们在技术和成为真正的产品领导者方面转向了一个更加由外向内的视角，并支持这些设计优先的开发，不仅仅是 API 设计优先，而是设计优先的开发。事实上，无论如何，公司正在朝那个方向发展，这与我们需要 API 的方向非常吻合。我们的 API 优先倡议完全建立在这样一个理念上，即开发人员与 API 交互的用户体验是值得设计思考的，也是值得使用它们的角色付出汗水的。开发人员将是我们软件的一流用户，他们应该得到出色的体验。

第二个改变游戏规则的因素是我们从 SAS 设计团队获得的支持。因此，他们毫不犹豫地接受挑战，利用他们的用户体验设计知识，帮助找出我们如何与产品团队和利益相关者一起将用户体验转化为 API 设计研讨会。因此，当我们开始举办 API 设计研讨会，并开始让人们在整个生命周期中开始思考人们在 API 方面真正关心的是什么时，这是一件大事。如果没有跨所有学科的协调，我们不可能取得过去三年的进步。这也是我们领导层的支持，让我们的 CTO 在会议上谈论 API 的重要性。我们经历的这段旅程真是太神奇了。真的很棒。

## 工作流指南开发人员

*为了满足治理预期，使用护栏来引导开发人员*

**Larry:** 好的，感谢您讲述了现有企业软件公司必须经历的关键要素，正如您所说，慢慢地将大船转向攻击一个新兴市场。深入研究不同工程团队中设计 API 的最佳实践，您发现哪些标准是应该遵循的最佳标准？

Jeannie: 因此，我们的最终目标是，通过设计优先的方法，团队将从一开始就设计和记录他们的 API，以满足标准以及性能和安全期望。我们还没有走到那一步，但肯定会有团队在这条路上走得更远，我们非常乐观，现在觉得在影响团队遵循我们的标准作为最佳实践方面得到了支持。我们确实有一套记录在案的可搜索标准，比如:如何进行版本控制，什么样的 HTTP 状态响应代码是可接受的，以及可搜索的通用设计模式。对于如何进行不同类型的互动，有指导方针、建议和模式。我们也有一个单一的存储库，存放我们所有的 API 文档，这些文档现在仍在 V2，但我们正在向 OpenAPI 3 迁移。我们颁布了一项法令，要求人们在年底前采用 OpenAPI 3。这构成了我们的 API 目录的基础，以便于 Greg。

但是，就像我前面提到的，人们能够自己解析这些标准并解释它们可能需要很多信息。正因为如此，我们实际上想做一些更主动的工作来帮助我们的开发人员从一开始就能够遵循这些标准。因此，我们创建了一个我们称之为 API 优先的工作流，它本质上是一个 Jenkins 管道，接受我们的 OpenAPI 文档，这些文档存放在我们的开发人员的存储库中，以及他们编写的任何支持性降价，以帮助指导开发人员使用 API 文档。它为开发团队提供了护栏，以确保他们的 API 满足治理期望。像正确解析引用这样的事情，或者我们有大量的光谱规则，我们必须执行样式和写作指南。比如结尾没有句号的摘要，这有助于我们在 API 文档中更加一致。作为工作流程的一部分，我们设置了基于 OpenAPI 文档的模拟服务器。它是成功的，我们仍然处于采用的早期，我们正在将 OpenAPI 转换、OpenAPI 3 现代化与采用这种工作流相结合。但到目前为止，我们从领养者那里得到的反馈非常好。我们不断地接受反馈和学习，并试图让开发者更容易地建立他们的 API，以更快地满足期望。

## 外部 API 的商业原因

所有的 API 在设计时都应该考虑到有一天它们会被外部化，所以产品经理的商业理念必须回答这样一个问题:“这个 API 适合我的目标受众吗？”

拉里:太棒了。您当然已经构建了大量的支持基础设施和文档来设计您的 API 接口。你似乎篡夺了我的下一个问题，那就是你认为 OpenAPI 3.0 有多重要，但你肯定已经回答了这个问题。因此，我们为什么不继续下一节，讨论内部和外部 API。因此，我们最初没有涉及它，但是您在 SAS 的项目非常成功，您已经向开发人员社区公开了 50 多个 API 服务。所以，跟我们分享一下，一个产品经理把内部 API 作为外部产品发布的流程。

Jeannie: 所以最重要的是，如果我们有一个现有的内部 API，而产品经理确实想把它发布出去，那么他们的责任就是想出一个让它发生的商业理由。这是因为，向前发展，我们真正希望发生的是 API 从一开始就被构建，从一开始就被设计，被外部化。在过去，我们的所有服务并不总是如此，所以我们确实需要很好地关注“这个 API 是否适合他们所瞄准的受众”，或者它是否需要一个抽象层才能对他们所瞄准的受众有用。

因此，这方面的权威是产品经理，他最终决定是否发布。他们可以超越我们，但我有一个总体产品经理，我在业务方面与她一起工作，在她和我之间，我们查看 API，我们讨论它，我们与产品经理交谈，以真正理解用例是什么，我们讨论风险。如果你正在发布一个内部 API，并且你认为它不会持续很长时间，那么你必须考虑它的寿命将在什么时候结束。你是否考虑过他们是否会对这个 API 进行突破性的修改。我们向他们建议风险，在某些情况下，他们接受这些风险，他们发布 API 供外部使用。但在其他情况下，人们会说“哦，现在我明白了，我们真的想构建另一个需要外部化的 API。”

## 外部化的最低标准

*发布 API 的最低验收标准包括:准确的&测试文档，来自 PM 的不做突破性更改的保证，满足适当的安全&性能预期，以及发布它的真实用例*

我们有外部化 API 的标准，我们称之为最低接受标准。我们对 API 的最低接受标准是，它们必须符合我们的 SAS API 标准和指导方针，围绕它们所服务的产品的一致性和可用性的基本行业期望。他们需要有完整的书面文件，然后该文件需要准确，它需要满足消费者的需求和期望。我们希望他们对文档进行合同测试，以确保 API 的功能符合实际记录的内容。我们希望他们基本上保证不做突破性的改变，除非将来有一些可怕的理由这样做，显然是一个可怕的安全问题或需要解决的事情。但是我们希望这些东西能够长久存在，我们真的考虑得很周到，注意不要引入突破性的变化，并建议我们的产品管理团队，这对使用 API 的人很重要。我们不能把它们从下面分开，因为如果你这样做，人们会非常不安，并在 Twitter 上写一些关于你公司的刻薄的东西。所以我们对此非常关注。如果我们不得不弃用或删除某个东西，我们会非常仔细地考虑弃用声明，并围绕它循环。我们需要他们满足产品安全办公室的所有期望。他们需要满足为产品设定的任何性能预期。它需要有一个真正的用例，它不能只是像“我写了这个很酷的 API，我想把它放在那里，有人可能会使用它，或者没有人会像那里一样使用它。”把它放在那里一定有真正的商业原因。

最后一件事有点深奥，但我们希望确保在自有产品管理团队、内部消费者和面向客户的团队之间有一个反馈循环，这样我们就能不断地获得关于 API 的信息。因为如果 API 不起作用，那么我们可能需要构建其他东西。但是，所有团队之间的反馈循环，让产品经理真正了解人们希望如何使用 API，或者正在使用 API，这确实是我们知道我们最好地服务于将使用这些的社区的方式。

**Larry:** 没错，了解您的客户使用您的 API 的目的和用途，以及它的部署是否成功非常重要。这有点自私，因为我为一家分析公司工作，这就是我们所做的，但我完全理解这个问题。有趣的是，你提出了外部化内部 API 的标准之一，因为在之前与[迈克·阿蒙森](https://www.moesif.com/blog/podcasts/api-product-management/Podcast-With-Mike-Amundsen/#internal-apis-lack-resiliency)的播客中，他说当创建一个新的外部 API 时，你应该考虑的最重要的问题之一是确保它遵循你希望你的公司面向外部的服务的所有合规性和安全性问题。确保这一点的最好方法是，当你第一次构建你的内部 API 时，要有这样的心态，有一天这可能会成为一个面向外部的服务。

珍妮:是的。最重要的是，我们已经有了一堆 API，有些情况下，我们的客户会立即需要它们，从商业角度来看，我完全理解这一点，但我们的目标实际上是从一开始就开始设计这些 API，以满足这些期望。生产可以外化的东西，即使有人从一开始就不想这么做。

## 产品管理是负责任的

*产品管理应严格确保所有检查都已完成，这样整个产品才能交付*

Larry: 是的，所以你至少从第一天起就拥有内置于 API 中的能力，以防你想在外部部署它。深入研究一下您之前所说的业务案例，根据您的经验，谁需要支持一个 API 来开发或对外发布它？它必须有销售和营销人员的参与吗？他们需要一个铁证如山的商业案例吗，或者一个产品管理方面的冠军就足够了吗？

**Jeannie:** 所以对于内部 IP 的外化，你需要一个产品经理。你需要一个人来担当产品所有者的角色，这个人将从业务的角度接管 API 的所有权，从业务的角度对它负责，并确保它从业务的角度满足所有那些生命周期的期望。正如我所说，它们肯定是明确的业务需求。你知道，我们已经有工程团队提出了他们已经编写的 API，他们希望将这些 API 具体化。我们只需要确保所有的检查标记都完成了，不仅仅是关于满足标准，而是关于以下内容:支持团队是否经过培训，以便他们能够在这个 API 发布后支持它，它是否经过测试，我们是否有好的文档？所有这些都很重要。因此，我们真的需要某种产品管理的严格性，或者某个担任产品管理角色的人对此负责。

## API 赚钱

*决策者应该通过向 API 展示来自该领域的用例，并向该领域的领导者展示这些用例，来衡量 API 成为利润中心的能力*

拉里:对。很有道理。就价值创造而言，从你的 API 中赚钱还是赔钱——你如何让大型组织中的决策者理解 API 的价值潜力，他们可以赚钱也可以赔钱，这取决于做得对不对？

**Jeannie:** 所以这肯定是一场艰苦的战斗，战斗是一个错误的词，因为像商业中的任何事情一样，专注于对业务来说是全新的、有潜力但尚未被证明的东西，特别是在一个有 40 多年历史的公司中，这自然会与其他业务优先事项竞争。随着时间的推移，我们的许多其他优先事项已经开始真正赚钱了。我们只是在推理，对吗？而且，在它成为现实之前，我们理论上认为 API 会赚钱。尽管如此，潮流确实在改变。我们已经从我们的领域中获得了许多用例。我们一直与该领域的领导者保持联系。他们一直在寻求高价值、易于使用的 API。我们将那些来找我们并告诉我们他们有 SAS APIs 用例的人与决策者、产品管理人员和我们的执行领导联系起来，他们实际上可以确定工作的优先级并雇佣人员来创建这些东西。我们提出了我们的愿景，我们一直非常坚持，希望不会令人讨厌，但我们非常坚持地试图证明这样一个事实，即有许多人正在从他们的 API 中赚钱。我认为在我们的业务中也有这样做的真正机会。

我确实认为我们确实在改变，我只是想分享一个非常酷的故事，实际上就发生在上周。在我的职业生涯中发生的最酷的事情之一是，我们公司有了一位全新的 CTO，他要求我们的 API 优先项目团队来演示我们的新重启愿景。我们正在重新启动我们的 developer.sas.com 开发者门户，这也是非常令人兴奋的，他想做一个概述，说明我们希望以服务的形式提供 API。他用自己的研究开始了会议，这让我们感到惊讶，他带来了几篇行业文章，与他的执行团队分享了 API 优先的公司和 API 优先的成功公司。听到他把 API 说成是人们愿意为之付费的东西，真的非常非常酷。并且知道我们已经得到了自上而下的支持来推进这个计划。除此之外，我很快将有机会雇用一群新人，所以请在 LinkedIn 上留意一下。

拉里:嗯，也许我们可以在播客的文字稿里放一个链接，链接到你的[公开简历](https://www.linkedin.com/in/jeannie-kristufek-hawrysz-70a0a85/opportunities/hiring-opportunities/view/)。

珍妮:我们肯定能做到。

## 4 技术考虑

在现有的企业软件公司中建立一个 API 程序时，有四个主要的技术考虑:制定设计标准&文档指南，因为标准会改变而具有灵活性，为你的开发人员提供 guardrails &工具，坚持使用 OpenAPI

拉里:很好。很好，你有来自自上而下的支持，来自新 CTO 的支持，来发展和扩展你的 API 程序。最后，请与我们分享您在现有企业软件公司中建立和发展 API 程序的最佳实践。

Jeannie: 所以，我认为你需要关注三个不同的领域。第一个领域是技术方面。将设计标准记录下来是非常重要的。制定游戏规则至关重要。我非常幸运的是，我们最初有一个卓越的中心，它确实考虑到了这一点。我继承了一个标准。我不喜欢标准的一切。有些事情我不能改变，因为那样会让一切都不一致，但我喜欢有一个标准，有一个准则可以遵循。这对需要使用 API 的人来说有很大的不同，这对他们来说非常非常重要。

认识到标准必须不断发展，这是第二件事。我已经看到了为什么我必须为低代码/无代码 API 或 API 即服务开发第二个标准的原因。我们今天的标准实际上专注于我们的微服务 RestAPIs，我发现其中一些模式对于我们未来想要进入的其他领域并不适用。所以，只要知道有些关于微服务的标准的事情我不能改变，或者会变得不一致，但是这些事情对于这些其他类型的 API 可能没有意义，没关系，向前发展没关系，发展也没关系。

技术方面的第三件事是，给制造商，也就是提供 API 的人提供护栏和工具。让他们尽可能容易地遵循风格、标准和指导方针。我有一个非常小的团队，我希望我能做得更多，给他们更多的工具，但我们非常关注这一点。我知道这很难。开发人员和工程师在构建这些 API 时，在质量方面需要担心所有这些事情。你越容易让人们做到这一点，他们就会做得越好，坚持下去。然后我只需要插入 OpenAPI，因为如果你不使用 OpenAPI，我不知道该告诉你什么。你会感谢你自己，无论是做设计还是做文档。如果你不想做，那就做吧。

## 符合业务需求

*在现有的企业软件公司中建立一个 API 程序时，拿出证据证明这个 API 程序是值得的——面向客户的团队举足轻重*

在商业方面，尤其是当你刚开始组建一个项目时，寻找同盟。真正让我们在 SAS 的 API 上向前迈进的事情是，我们真正开始与 SAS 的其他人联系，这些人相信 API 不仅仅是管道，他们不应该要求 10 倍的开发人员来执行集成。拥有 API 崇拜是一回事，我们内部称自己为“API 很重要”，但是当其他团队，尤其是面向客户的团队，开始支持我们时，事情才真正开始加速。这也非常符合我的下一个想法，你必须做的一切都必须符合某种业务需求。你需要了解你在整体优先级中的位置，并在你的限制范围内尽你所能去做。即使你努力游说以提高优先级，并获得你需要的帮助来真正使项目启动。你必须坚持不懈，你必须不断努力。如果你认为这件事对公司很重要，那就继续做下去。我知道我不会说每个公司都会得到它，或者扭转局势，但我会说，如果你带来足够的证据，最终事情会开始移动。

## 要有耐心、远见和同情心

构建 API 程序时要有耐心。对那些和你有不同看法的人感同身受——试着搭建桥梁帮助他们理解。有一个愿景，并为之努力。

然后在个人方面，我只想说要有耐心，我一点都没有耐心，我不是一个有耐心的人。当你发现自己刚开始做某件事，或者走进一个并不像你想象的那样糟糕的情况，你很容易就可以离开。你知道，这是我早就想过的事情，我就像“哇，这将需要很多工作，这些事情我不知道如何去做。”但我真正发现的是，这是一个创造奇迹的机会。我仍然不知道所有的事情。我还在想很多事情。但是如果你处在我的位置，你正试图找出如何构建一个程序，坚持下去，因为首先这是很好的体验，其次，API 很重要。API 值得。我认为你会发现这是一次有意义的经历，即使最初可能会有一些挫折。

第二件事是要有远见。如果你没有远见，你就有麻烦了。即使你所拥有的愿景在今天看起来真的不可能，或者看起来可能需要很长时间，拥有一个愿景多少会给你一个努力的目标。所以我认为这真的很重要，最后一件事就是对那些和你看问题的方式不同的人产生共鸣，然后试着搭建桥梁帮助他们理解。

关于这个我要说的最后一件事，就是相信 API。毫不夸张地说，API 是我们今天互联存在的基础。随着时间的推移，它们可能会采用新的协议和流行的新风格，以及各种各样的用例，但总会有集成的需求。机器总是需要能够相互交流，人类也需要能够与机器交流，这种以精心设计的无缝方式进行交流的能力具有巨大的价值。鉴于如此多的 API 优先业务的出现，围绕 API 进行商业案例变得越来越容易。所以，如果你在挣扎，或者沮丧，或者你觉得自己没有进步，坚持下去，然后“改变世界”我认为如果你继续在 API 上努力，你会成功的。

拉里:嗯。这是对这一领域的认可。我相信我们所有的听众都很高兴你没有离开 API 的世界，正如你所说，你仍然是“API 的崇拜者”。Jeannie，今天和你聊得很愉快，解释了一家大型现有企业软件公司是如何拥抱新兴的 API 世界的。以及如何通过采用正确的程序和计划来实现这一目标并获得成功和有利可图的服务。所以非常感谢你参加我们今天的播客网络。

Jeannie: 当然，谢谢你给我这个机会来谈论这个

拉里:谢谢。