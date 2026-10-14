---
{"dg-publish":true,"dg-path":"Scrum 的壳变薄，Agile 的心脏变强.md","permalink":"/Scrum 的壳变薄，Agile 的心脏变强/","title":"Scrum 的壳变薄，Agile 的心脏变强","tags":["type/article","area/ai","area/management","area/architecture"],"created":"2026-05-22","updated":"2026-10-14","dg-note-properties":{"title":"Scrum 的壳变薄，Agile 的心脏变强","tags":["type/article","area/ai","area/management","area/architecture"],"created":"2026-05-22","updated":"2026-10-14","description":"Agentic Engineering 让把 Scrum 当作两周排班系统的旧实践更难维持。持续交付和动态调整原本就在 Scrum Guide 中；AI 的增益仍需按任务和完整交付成本验证。","source_notes":["Agentic Engineering 时代，Scrum 被冲击，敏捷被增强"],"sources":["https://agilemanifesto.org/principles.html","https://scrumguides.org/scrum-guide.html","https://dora.dev/research/2024/dora-report/","https://blog.google/innovation-and-ai/technology/developers-tools/dora-report-2025/","https://www.microsoft.com/en-us/research/publication/the-impact-of-ai-on-developer-productivity-evidence-from-github-copilot/","https://arxiv.org/abs/2507.09089","https://arxiv.org/abs/2605.07717"]}}
---


## 核心观点

- Agentic Engineering 冲击的是“Scrum 化管理”，不是 Agile 的核心原则
- AI agent 让个人变成更完整的交付单元，但不会消灭团队，只会淘汰低价值协作
- 软件团队的瓶颈正在从 coding 转向 intent、verification、integration 和 judgment
- Scrum 实践需要从任务排班回到产品学习与系统健康节奏，这不是 AI 才发明的新原则
- AI 时代的敏捷不是更多会议，而是更短反馈环、更强工程护栏、更高密度的人类判断

## 先说结论

Agentic Engineering 正在冲击 Scrum，但它没有削弱 Agile。

更准确地说，它冲击的是很多组织里被称为 Scrum 的那套东西：把两周 sprint 当成固定排班表、提前锁死 backlog、每日站会逐人报工、用 story point 或 velocity 衡量产能、把开发者当作稀缺执行资源去排班。

这些不是《Scrum Guide》的定义。Guide 规定 Sprint 为一个月或更短，计划应随学习持续调整；在不危及 Sprint Goal 的前提下，范围可与 Product Owner 协商。一个 Sprint 可以产生多个 Increment，也可在结束前交付，Sprint Review 不是发布门禁。Guide 也不要求 story points 或 velocity。把任务锁死、到期才交付的实践，早在 AI 之前就该改。

但 Agile 的底层原则反而被增强了：更小批量、更短反馈环、更频繁交付、更强调 working software、更依赖自组织团队、更需要业务和开发持续协作。

这不是一个“AI 让敏捷过时”的故事，而是一个“AI 逼我们把敏捷从流程外壳里救出来”的故事。

## Agentic Engineering 改变了什么

这里说的 Agentic Engineering，不是简单用 ChatGPT 问几段代码，也不是让 IDE 自动补全。

它指的是工程师把 AI agent 纳入真实软件交付流程：让 agent 读代码库、理解任务、修改文件、运行测试、查日志、修 CI、写文档、提交 PR，甚至并行探索多个方案。工程师不再只是亲手写代码，而是在设计目标、约束、上下文、验证路径和质量门禁。

这会改变软件交付的成本结构。

过去，一个工程师的主要产出来自“亲自实现”。现在，一个工程师可以带着 coding agent、测试生成器、浏览器自动化、日志查询工具和 CI 修复能力工作。他不再只是一个前端、后端或测试角色，而更像一个微型交付系统：能理解需求、拆任务、生成实现、补测试、查报错、写文档、提交 PR。

这就是“超级个体”的含义。不是一个人突然懂了一切，而是一个人可以通过 agent 临时调用很多能力。

## 被冲击的是“Scrum 化管理”的旧假设

《Agile Manifesto》强调的是 working software、customer collaboration、responding to change，以及通过早交付和持续交付获得反馈。《Scrum Guide》强调的也是经验主义、自管理团队、跨职能团队，以及每个 Sprint 创建有价值、可用的 Increment；一个 Sprint 可以有多个 Increment，不必等到结束才发布。

从原教旨上看，Agentic Engineering 并不反敏捷，甚至很敏捷。

问题在于，很多公司实践的不是这个版本的 Agile，而是一个基于旧生产函数的管理框架。这个旧生产函数至少有几个隐含假设：

- 开发能力是稀缺资源，需要通过 sprint planning 预先分配
- 前端、后端、测试、运维、数据等技能边界相对稳定，需要通过团队协作补齐
- 需求拆解和实现之间有明显交接，需要用 ticket 保持控制
- 两周是一个合理的计划和反馈颗粒度
- “完成多少点数”在某种程度上可以代表团队进展

Agentic Engineering 正在逐条动摇这些假设。

当某些工作切片的 delivery cadence 被压缩到小时级，而团队仍然两周才重新看一次计划，这种“Scrum 化管理”就会显得迟钝。不是因为经验主义错了，而是执行节奏和实际反馈节奏脱节；Guide 原本就要求 Sprint 内持续调整计划。

## 超级个体不会消灭团队

一种常见直觉是：如果每个人都能靠 agent 变成全栈，那 Scrum 团队是不是就不需要了？

我认为答案是否定的。超级个体消灭的是低价值协作，不是协作本身。

过去很多协作发生在“技能缺口”上：前端等后端接口，后端等 DBA 建表，测试等开发提测，运维等部署脚本。Agentic Engineering 会显著压缩这类等待。一个人可以把更多横向工作先跑起来，团队不必为了每个小切面都开同步会。

但高价值协作会变得更重要：

- 产品方向是否值得做
- 用户问题是否被正确理解
- 架构边界是否还能承载变化
- 生成代码是否真的可维护
- 质量、合规、安全、成本是否被纳入 Definition of Done
- 多个 agent 并行产出的改动是否能被系统性集成

也就是说，瓶颈从“谁来写代码”转移到“谁来定义意图、验证结果、管理复杂性”。

执行会越来越便宜，判断会越来越贵。

## 研究信号并不单向

已有研究并不支持一个简单的“AI 让开发效率暴涨，所以流程都可以砍掉”的结论。

GitHub Copilot 的早期受控实验显示，在特定编程任务中，使用 Copilot 的开发者完成速度显著提升。这个结果解释了为什么个体层面会强烈感觉到“我变快了”。

但 METR 在 2025 年针对有经验开源开发者的随机对照实验给出了相反信号：在开发者熟悉的真实开源仓库任务上，使用当时的 AI 工具反而让完成时间变慢。更有意思的是，参与者事前和事后都主观认为 AI 会让自己更快。这个历史结果提醒我们验证体感，不代表后续所有工具和任务都会变慢。

METR 在 2026-02-24 更新了判断：根据参与者反馈等信息，当时的 AI 工具更可能带来加速；但新实验中，不愿无 AI 工作的开发者更少参加，一些预期高收益任务也未被提交。参与者与任务的选择偏差，以及并行 agent 下工时测量的困难，使他们无法可靠量化增益。不能用旧结果否定当前加速，也不能把新更新写成已证实的大幅提速。

DORA 2024 报告也提供了一个更系统的判断：AI adoption 和个人 productivity、flow、满意度存在正相关，但它对 software delivery performance 的影响不是单调变好。Google 对 DORA 2025 的解读进一步把 AI 称为 amplifier：它会放大组织已有的能力，也会放大已有的问题。

这个判断很关键。

AI 不是把弱流程自动变强。它是把流程里的反馈、质量、上下文、架构、协作问题放大到更快的速度。

如果一个团队本来就有清晰的产品判断、自动化测试、可观测性、架构边界和 code review 纪律，agent 会放大它的吞吐。如果一个团队本来就是模糊需求、弱测试、弱 review、弱 owner、弱反馈，agent 会更快地产生技术债。

## 社区讨论里的现场感

Reddit 和 Hacker News 上的讨论不应被当成严肃研究，但它们能提供一线现场感。

在 r/scrum 里，有人提出一个很典型的问题：团队用了 AI 之后，开发速度快到 Scrum 显得太慢。这个讨论里最有价值的判断是：问题不一定是 Scrum 太慢，而是 planning cadence 已经和 delivery cadence 脱节。

在一些把两周 sprint 当排班周期的团队里，实际节奏成了：两周计划一次、两周交付一次、两周复盘一次。这是实施习惯，不是 Scrum 的必然节奏。Agentic Engineering 则可能让某些 feature slice、spike、refactor、测试补齐、文档修复的周期缩短到小时级甚至分钟级，进一步暴露这种实施方式的问题。

如果 delivery 已经进入小时级，而 planning 仍然是两周级，backlog 就会变成历史书。ticket 还没写完，工程师已经把三个实现方案跑出来了。

Hacker News 上围绕 METR 研究的争论也指向同一个问题：AI 让人感觉更快，但真实成本可能转移到了 review、debug、理解、维护、回滚和长期复杂度上。

这就是 AI 工程管理最容易误判的地方：体感速度不是系统吞吐。

## Scrum 需要变薄

我说的“变薄”，是让 Scrum 实践里的排班、重复汇报和过早拆细变薄，不是随意删掉 Guide 的核心事件、责任和承诺，再把剩下的流程称为 Scrum。

Sprint 本来就不该是把任务锁死两周的容器，而应围绕目标提供检视和适应的节奏。我希望它更聚焦产品学习和系统健康，Sprint 内的工作流则可以结合 Kanban 或 continuous flow：小批量进入、小批量验证、小批量发布。这是在落实并扩展已有原则，不是 AI 才赋予 Scrum 的灵活性。

Backlog 不再应该是详细规格仓库，而应该是 intent backlog：问题、机会、假设、约束、成功指标、风险边界。实现细节可以由工程师和 agent 在更靠近执行的位置展开。

Daily Scrum 本来就不是向主管逐人报工，而是 Developers 检视 Sprint Goal 进展、调整计划的机会。在 agent 工作流里，我建议同时聚焦三个问题：当前最重要的意图是否清楚，系统性阻塞在哪里，agent 产出是否暴露了新的质量或架构风险。

Retrospective 也不只是团队协作复盘，而是 harness 复盘：哪些错误反复发生，哪些上下文缺失，哪些测试没有覆盖，哪些 review 规则可以自动化，哪些文档应该进入仓库。

Definition of Done 则必须变厚。AI 时代的 Done 不能只是“功能完成 + 测试通过”。它至少应该包括：

- 需求意图被记录，关键假设可追踪
- 自动化测试覆盖主要路径和边界条件
- Agent 生成内容经过人类 owner 审查
- 安全风险、prompt injection、权限边界被考虑
- 监控、日志、回滚路径存在
- 代码和文档没有明显漂移
- 变更的产品信号有后续观测方式

这不是为了增加流程负担，而是承认工作重心可能发生变化：实现加速以后，验证需要更高优先级和更明确的资源投入，但不意味着验证成本必须上涨。自动化与更好的门禁也可以降低单位验证成本。

我还会明确每类变更的验收者与后续维护 owner，避免一次审查后无人承接结果。并行度按测试和审查的实际容量调整：未验收变更达到约定上限时，先验证、整合或暂停续跑，不让未经确认的产出成为下游默认输入。这个在制品上限随风险和验证能力调整，不是所有团队共用的固定数字。管理节奏也要给方向判断保留不被调度与提醒占满的时间。

## Agile 需要变强

我倾向于把 Agentic Engineering 时代的团队流程抽象成六步：

1. Intent：人类定义问题、约束、成功指标和风险边界
2. Decomposition：人类和 agent 一起把目标拆成可验证的小切片
3. Agent Work：agent 并行完成实现、测试、文档、迁移、分析
4. Verification：确定性工具优先验证，包括测试、lint、类型、截图、日志、性能、合规检查
5. Human Judgment：人类判断是否符合产品意图、架构方向和长期可维护性
6. Product Learning：发布后观察真实反馈，把学习结果回写到 backlog、文档和 harness

这个循环比固定排班式的“Scrum 化管理”更像“意图驱动的持续流”。它不反对 Sprint：Sprint 提供目标和检视节奏，小切片的执行与发布不必等到周期边界；这与 Guide 的 Increment 原则并不冲突。

2026 年 arXiv 上有一篇 AI-Native Large-Scale Agile Software Development Manifesto，里面提到 parallel processes、intent-driven teams、living knowledge、verification-first assurance、orchestrated agent workforces。虽然这类新论文还需要时间检验，但它捕捉到了一个真实趋势：敏捷的对象正在从纯人类团队，转向人机混合系统。

所以 AI 时代的敏捷不是更多会议，而是更短反馈环、更强工程护栏、更高密度的人类判断。

## 角色会重组

Product Owner 会更像 intent curator。他的重点不是写更多 ticket，而是定义更清楚的问题、优先级、用户信号和商业约束。

Scrum Master 或 Engineering Manager 会更像 flow designer 和 harness designer。他的重点不是维护仪式，而是降低系统摩擦：让信息进入正确位置，让质量门禁可自动执行，让团队从重复错误中沉淀规则。

Engineer 会更像 agent operator、system integrator 和 quality owner。他不只是写代码，也要会写 spec、调上下文、设计验证路径、审查 agent 的假设。

QA 不会消失，但会从“末端验收”前移成 verification architect：把测试、监控、回归、评估、异常检测都变成 agent 可以调用的反馈系统。

架构师也不会消失，反而更重要。因为 agent 能更快地产生局部最优代码，而架构师负责维护全局约束。没有架构约束的 Agentic Engineering，会变成高速制造复杂度的机器。

## 度量也要换

AI 时代继续把 velocity 和 story point 当成产能或用户价值指标，会更容易失真。Guide 没有要求使用它们；作为团队内部的计划辅助，与跨团队排名和产能考核是两回事。

因为 agent 可以很快完成大量 ticket，但这不等于交付了更多用户价值。更危险的是，团队可能为了让 AI 看起来有效，把任务拆得更碎、把产出算得更多，却把 review load、缺陷逃逸、维护成本和认知负担藏起来。

更值得看的指标包括：

- lead time：从想法到用户可用的时间
- cycle time：从开始实现到可交付的时间
- review load：每个 reviewer 需要审查的变更量和复杂度
- change failure rate：发布后导致故障、回滚、热修的比例
- escaped defects：逃逸到生产或用户侧的问题
- rework rate：agent 产出需要返工的比例
- test confidence：关键路径的自动化验证覆盖度
- product outcome：功能是否真的改变用户行为或业务结果
- verification debt：未验收变更的数量、等待时长、风险和下游依赖范围

若团队还启用主动发现机会与追加提案的 Agent，我会另外观察净注意力收益：省下的寻找和跟进，是否超过新增确认、核对、纠错与收尾负担。普通受命任务不必机械套用；必要审批和失败也不能为了“少打扰”被隐去。这些是治理观察维度，不是统一 KPI 或本轮已验证的效率公式。

一句话：不要衡量 agent 写了多少代码，要衡量团队把多少可信变化送进了真实世界。

## 我的判断

Scrum 如果被理解为一套固定仪式，它会被 Agentic Engineering 持续冲击。

Scrum 如果被理解为一套经验主义反馈框架，它仍然有生命力。我主张让它的实施外壳变薄、反馈变快、Sprint 内的工作更接近 flow-based system，而不是把这些改变说成对 Guide 的全盘替代。

Agile 的核心不会过时，因为 AI 恰恰让 Agile 的原始承诺更可实现：更短反馈、更小批量、更快学习、更贴近用户。

真正过时的是把 Agile 当作人力资源排班系统。

AI 时代的软件团队，不再是“若干工程师围绕 backlog 工作”。它更像一个由人类意图、agent 执行、确定性验证、架构约束和产品反馈组成的控制系统。

Scrum 的壳正在变薄，Agile 的心脏正在变强。

## 参考

- [Principles behind the Agile Manifesto](https://agilemanifesto.org/principles.html)
- [The Scrum Guide](https://scrumguides.org/scrum-guide.html)
- [DORA 2024 Report](https://dora.dev/research/2024/dora-report/)
- [Google: DORA 2025 report on AI-assisted software development](https://blog.google/innovation-and-ai/technology/developers-tools/dora-report-2025/)
- [Microsoft Research: The Impact of AI on Developer Productivity](https://www.microsoft.com/en-us/research/publication/the-impact-of-ai-on-developer-productivity-evidence-from-github-copilot/)
- [METR: Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity](https://arxiv.org/abs/2507.09089)
- [METR: We are Changing our Developer Productivity Experiment Design（2026-02-24）](https://metr.org/blog/2026-02-24-uplift-update/)
- [AI-Native Large-Scale Agile Software Development Manifesto](https://arxiv.org/abs/2605.07717)
- [Reddit r/agile: How will agentic coding change agile software development?](https://www.reddit.com/r/agile/comments/1t0z08i/how_will_agentic_coding_change_agile_software/)
- [Reddit r/scrum: My devs are on AI steroids and Scrum is officially too slow](https://www.reddit.com/r/scrum/comments/1r4jbys/my_devs_are_on_ai_steroids_and_scrum_is/)
- [Hacker News discussion on the METR productivity study](https://news.ycombinator.com/item?id=44522772)
