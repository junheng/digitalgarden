---
{"dg-publish":true,"dg-path":"难以完成的任务 — LLM 的信息容量天花板.md","permalink":"/难以完成的任务 — LLM 的信息容量天花板/","tags":["type/article","area/ai","area/architecture"],"created":"2026-06-13T14:54:11.268+08:00","updated":"2026-06-13T14:54:11.270+08:00","dg-note-properties":{"tags":["type/article","area/ai","area/architecture"],"created":"2026-06-13","description":"有些任务 AI 就是难以一次完成——不是因为不够聪明，而是 LLM 的单次推理存在信息容量上限。理解这个物理约束，才能设计绕过它的策略。"}}
---


## 你让它改第 3 轮，它开始删之前写对的东西

你正在用 AI 做一个中等复杂度的任务——写一个模块、设计一套接口、或者重构一段逻辑。

第一轮：AI 产出了一个看起来不错的版本。你 review 发现几个问题，指出来，让它改。

第二轮：AI 修好了大部分问题，但又冒出了两个新的。你耐着性子再指出来。

第三轮：AI 开始做一些奇怪的事——它删掉了一段之前写对的逻辑，把一个你明确说过不要改的接口改了，或者在两个方案之间反复横跳。

第四轮、第五轮……你发现每轮都在前进，但前进的步伐越来越小，而且每修复一个问题似乎都会引入一个新的问题。你不是在收敛，你是在打地鼠。

如果你有过这种体验，你不是一个人。而且**这不是你的 prompt 没写好，也不是 AI 不够聪明**。这是 LLM 计算本质决定的一个结构性约束。

我叫它**信息容量天花板**。

## 每个 pass 都是一条有宽度限制的信息通道

要理解这个问题，需要先理解 LLM 的一次前向传播（一个 pass）在计算上到底是什么。

Wan et al. (2025) 从信息论的角度给出了一个精确的描述：LLM 的单次推理相当于**信息通过一个有限容量的通道**。从 Fano 不等式出发，他们推导出了一个理论精度上界——一旦任务的信息需求超过这个通道容量，准确率不是线性下降，而是**崩溃**（Accuracy Cliff）。

> "LLMs have a **finite per-pass output capacity**, beyond which the integration of task-relevant evidence proves unreliable." — Wan et al., *A Fano-Style Accuracy Upper Bound for LLM Single-Pass Reasoning*, ICLR 2026

翻译成直觉：你的代码有 20 个约束需要同时满足——类型正确、接口一致、业务规则完整、边界条件覆盖、跨模块数据流正确、隐式假设不冲突……但模型在一个 pass 中只能可靠验证其中的 12 个。剩下的 8 个中，有些被随机丢弃了，有些被注意力分配机制忽略了。

**每次 pass 丢弃的约束不同，所以每次 review 发现的问题也不同。** 这不是 Reviewer 不够仔细，是物理上不可能在一个 pass 中完成所有验证。

这就是为什么一轮 review 不够——不是"再来一轮就好"，而是"一轮根本装不下"。

## 自修正盲区：为什么 AI 看不到自己的错误

还有更糟糕的。

Tsui (2025) 做了一个残酷的实验：让 14 个开源模型审查自己产出的代码，同时让它们审查别人（外部来源）产出的代码——内容完全相同，只是"作者"不同。

结果：模型修正别人错误的能力远高于修正自己错误的能力。**平均 64.5% 的自产出错误被漏掉了。** 他把这个现象称为 Self-Correction Blind Spot（自修正盲区）。

> "LLMs **cannot correct errors in their own outputs** while successfully correcting identical errors from external sources … an average **64.5% blind spot rate**." — Tsui, *Self-Correction Bench*, 2025

注意，这个实验是在非推理模型上做的。RL 训练的推理模型（如 Claude extended thinking）通过 outcome feedback 学会了纠错，可能没有这个盲区。但如果你用的不是推理模型，或者推理深度不够，这个效应仍然存在。

更有意思的是 Tsui 发现的一个细节：在 prompt 末尾加一个简单的 "Wait" 就能减少 89.3% 的盲区。这说明**纠错能力是存在的，只是没有被触发**。模型在"自我审查"模式下，某种激活路径被绕过了。

这解释了另一个常见体验：你把同一段代码给同一个模型的另一个对话窗口看，它能发现问题；但在同一个对话里让它 review 自己刚写的代码，它说"看起来没问题"。

## Tunnel Vision：为什么更深的思考不能解决所有问题

你可能会问：那推理模型呢？Extended thinking 不是可以思考得更深吗？

可以，但有上限。而且这个上限不是模型能力的上限，是**串行策略本身的结构性缺陷**。

Wen et al. (2025, 清华大学) 发现了一个现象，他们称之为 Tunnel Vision（隧道视野）：

> "This ceiling is **not an inherent limit of the model's capability but a flaw in the scaling strategy itself**." — Wen et al., *ParaThinker*, 2025

关键实验：把 thinking token budget 从 32K 扩到 128K，准确率**几乎不再提升**。原因是 **early token commitment**——模型在 thinking 早期如果做出了一个次优判断，所有后续 thinking token 都 condition 在这个判断上，没有能力回溯。错误前缀越长，恢复的概率越低。

用人的体验来类比：你拿到一个需求，脑海中冒出的第一个方案往往会影响你后续所有的思考方向，即使那个方案后来被证明不是最优的。LLM 面临同样的问题，而且更严重——因为它不能像人一样"睡一觉重新想"。

这解释了为什么即使使用 extended thinking，多轮外部审查仍然有价值：**extended thinking 是串行加深，外部多轮是并行加宽。** 每轮 compact/clear 后独立采样，不共享上一轮的早期决策，天然 decorrelate 盲区。

## 并行 vs 串行：一个数学上的根本差异

为什么"独立采样"这么重要？

Chen et al. (2024, NeurIPS 2025) 从理论上证明了：在合理假设下（模型能以非零概率生成正确解 + pairwise 比较优于随机），**并行生成多个独立候选 + knockout 聚合**可以让失败概率**指数衰减到零**。

而串行加深 thinking 做不到这一点——因为所有步骤共享同一条初始轨迹，不满足独立采样假设。

这个对比解释了为什么"给 AI 更多时间思考"和"重新开一轮对话让 AI 重新看"的效果截然不同。前者增加了深度但被困在同一条推理路径中；后者获得了独立视角。

这也意味着：**对于复杂到超出单条推理链覆盖能力的任务，最优策略不是"更深的 extended thinking"，而是"extended thinking + 多轮独立 review"。** 两者是互补的，不是替代的。

## Lost in the Middle：你的代码中间部分被"物理上"忽略了

还有一个更接地气的因素。

Liu et al. (2023) 发现 LLM 对长文本的注意力分布是 U 型的：开头和结尾 recall 好，**中间严重退化**。而且这个效应即使在使用显式长上下文模型时也无法消除。

> "Performance is often **highest when relevant information occurs at the beginning or end** of the input context, and **significantly degrades when models must access relevant information in the middle** of long contexts, **even for explicitly long-context models**." — Liu et al., *Lost in the Middle*, TACL 2024

这意味着当你把一份 500 行的代码文件交给 AI review 时，位于文件中间部分的逻辑天然获得了更少的注意力。那些"藏在中段"的隐式约束、边界条件、数据流假设 —— 它们在物理上更难被"看到"。

下一轮 review 时，由于 attention 的随机性，分布略有不同，于是发现了不同的问题。你以为是自己 review 得更仔细了，实际上是随机采样覆盖了不同的盲区。

## 修复债务：每次修改都在引入新的问题

最后，还有一个不那么理论化但同样重要的因素：修复操作本身不是无损的。

Editor 在修复 Reviewer 指出的问题时，受限于自己的单 pass 容量——它无法同时验证所有已有约束是否仍然满足。修复问题 #3 可能破坏问题 #1 的修复，但 Editor 在当前 pass 中没有注意到。这成为下一轮 Review 的"新发现"。

Lukaxiya (2026) 把这种现象称为**修复债务（Over-correction）**：

> "有时候前一次修正本身引入了新的错误，Agent 继续修正反而让情况更糟。这就是'修复债务'的来源。"

他和 Anthropic 的工程团队都观察到了同一个问题：在没有人为审查的情况下，AI agent 可以在一轮轮的 self-correction 中让代码越改越差。Anthropic 在 2026 年 3 月的工程报告中坦承其最优三 Agent Harness（planner + generator + evaluator）仍然存在 "remaining headroom" —— 即使用上最好的实践，Evaluator 仍然漏掉人类观察者立即可见的问题。

## 知道了这些，你能做什么

这些限制不是"AI 不好"的论据。它们是工程约束，就像你知道数据库有 CAP 定理、分布式系统有 FLP 不可能性一样。知道约束在哪里，你才能设计绕过它们的策略。

以下是被理论和实验支持的实践：

### 1. 分层 Review —— 一次只验证一个维度

这直接对应 InfoQA 的 capacity-aware decomposition 策略。不要在一次 review 中让 AI 同时检查类型、接口、业务规则、边界条件。拆成多个聚焦 pass，每次只验证一个维度。每个 pass 的信息负载被控制在容量以内，漏检率显著降低。

### 2. 多轮独立 review 不是浪费，是必须

如果你的任务复杂度 N_total 远超单 pass 容量 C，那么至少需要 N_total / C 轮才能覆盖所有约束。不是 AI 不努力，是数学不允许一轮搞定。接受多轮收敛，但监控收敛趋势——如果每轮发现的问题数不递减，或问题层级不递进（总是在表层打转），说明初始方案有结构性问题。

### 3. 用独立对话做审查，不要在同一个对话里让 AI review 自己

这直接对抗 Self-Correction Blind Spot。compact/clear 清空上下文只是第一步——你还需要一个**不受早期推理路径影响**的独立审查者。最理想的情况是用不同模型架构做 generation 和 review，次优方案是同模型但完全独立的对话。

### 4. 限制单次修复的范围

告诉 Editor："只修改这 3 处，不要做额外重构。" 这降低了 Editor 单 pass 的负载，减少了修复回归率。更激进的做法是：每次修复前先 git commit，形成可回退的基线。

### 5. Extended thinking + 外部多轮，而不是二选一

Extended thinking 扩大了每轮的有效深度，外部多轮提供了独立重新采样。对于复杂任务，两者需要组合使用——而不是指望"thinking 开到最大就能一轮搞定"。

## 这不是 bug，这是物理定律

最后我想说一个心态上的转变。

很多人在遇到"AI 改代码改到后面就乱了"时，第一反应是怀疑自己的 prompt 写得不对、或者用的工具不够好、或者模型版本不够新。这些因素确实有影响，但它们不是根本原因。

根本原因是：**LLM 的信息处理受到一个类似于香农极限的结构性约束。** 你的任务信息需求超过了通道容量，多出来的那部分不是"处理得不好"——是根本没通过通道。

理解了这一点，你就不会执着于"调好 prompt 让 AI 一次搞定"。你会把精力花在真正有效的事情上：**分解任务、独立采样、分层验证、接受多轮收敛。**

这不是 AI 的失败。这是我们从"AI 什么都能做"的盲目乐观，走向"AI 是一个有明确物理约束的工具"的必经之路。而知道工具的物理约束，恰恰是专业使用者的标志。

---

*本文的分析基于 Wan et al. (2025, ICLR 2026)、Tsui (2025)、Wen et al. (2025)、Chen et al. (2024, NeurIPS 2025)、Liu et al. (2023, TACL 2024)、Anthropic Engineering (2026)、CooperBench (Stanford/SAP, 2026)、Lukaxiya (2026) 等来源。完整引用段落与摘要见另一篇分析笔记（本 vault 内），本文仅保留关键引文以保持可读性。*
