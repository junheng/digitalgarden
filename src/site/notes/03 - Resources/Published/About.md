---
{"dg-publish":true,"dg-path":"About.md","permalink":"/About/","tags":["type/reference"],"created":"2026-05-12","dg-note-properties":{"tags":["type/reference"],"created":"2026-05-12","description":"关于本网站及其作者：研究方向、技术理念、可协作领域。"}}
---


## 关于本网站

Diomgis' Nexus 是一个关于 **Agentic Engineering** 的实践笔记。

这里不讨论"AI 会不会取代人类"这类话题。这里讨论的是更具体的问题：怎么设计一个能持续运转的 agent 系统？怎么在 agent 管线中分配人类和脚本的职责？怎么让 harness 的厚度随模型能力同步演化？

如果你在落地 agent 系统时遇到了类似的问题，这里的一些笔记可能对你有用。

## 作者的研究方向

作者的日常工作涵盖以下领域：

**Agent 系统设计与落地**。不满足于"用 agent 写代码"——关注的是 agent 系统的工程化：harness 架构、管线设计、多 agent 协作、评估体系。核心信念是 HIL（Human-in-the-Loop）：系统由 agent 自动驱动，人类充当关键决策节点，以及 AI 尚不成熟环节的临时替代。

**软件架构**。云端架构设计、领域驱动设计（DDD）、微服务治理。在 agent 时代，"好架构"的定义正在从"人容易理解"扩展为"agent 容易在其中工作"——好的架构约束 agent 的解空间，让正确的做法成为最简单的做法。

**技术管理**。20-40 人团队的管理经验。关注 agent 对团队结构、招聘标准、组织形态的深层影响。核心观点：agent 不是在替代人，是在替代不做判断的人。

**当前探索**：agent 管线的形式化设计（script-as-pipeline-node）、meta-harness（让 AI 设计自己的 harness）、评估飞轮的工程实现。

## 技术理念

- **HIL 优先**：不是让 AI 替代人，而是让 AI 最大程度自动执行，人类在最关键的节点做判断
- **Build to Delete**：harness 组件应该为模型缺陷而存在，模型变强后应该被移除，不是被保留
- **约束优于指令**：告诉 agent 不能做什么，比告诉它应该做什么更有效
- **异步沟通优先**：更深的思考，更少的打断

## 协作与交流

如果你在以下方向有实践经验或不同观点，欢迎交流：

- [GitHub Discussions](https://github.com/junheng/digitalgarden/discussions)（文章讨论）
- 邮件：`diomgis@icloud.com`（私下交流）

- Agent 系统的工程化落地（尤其是 harness 架构设计）
- 团队的 AI 转型（招聘标准、工作流重构、评估体系）
- Script-as-pipeline 或类似管线的实践
- 对本站文章的技术讨论或反驳

---
*本网站文章不构成任何形式的咨询服务。所有观点均为个人观点，与其他组织无关。*
