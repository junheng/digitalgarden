---
{"dg-publish":true,"dg-path":"Agent 评估就绪清单 — 要点提炼.md","permalink":"/Agent 评估就绪清单 — 要点提炼/","tags":["type/article","area/ai"],"created":"2026-03-28T17:28:11.784+08:00","updated":"2026-03-28T17:28:12.105+08:00","dg-note-properties":{"tags":["type/article","area/ai"],"created":"2026-03-28","source":"https://blog.langchain.com/agent-evaluation-readiness-checklist/","description":"LangChain 的 Agent 评估就绪清单要点提炼：从 error analysis 到飞轮机制，覆盖构建 eval 前的准备、评估层级选择、数据集构建、Grader 设计、运行迭代和生产就绪六个阶段。"}}
---


## 来源

- 原文：[Agent Evaluation Readiness Checklist](https://blog.langchain.com/agent-evaluation-readiness-checklist/)（LangChain Blog，Victor Moreira，2026-03-27）
- 配套阅读：[Agent Observability Powers Agent Evaluation](https://blog.langchain.com/agent-observability-powers-agent-evaluation/)

## 核心理念

**从最简单的 eval 开始，只在简单方法漏掉真实失败时才加复杂度。**

几个端到端 eval 测试 Agent 是否完成核心任务，就能立即建立基线——即使架构还在变。

## 一、构建 Eval 之前

### 手动审查 20-50 条真实 trace

花 30 分钟读真实 Agent trace，比任何自动化系统都能更快发现失败模式。这是最高 ROI 的投入。

### 60-80% 的精力花在 Error Analysis 上

流程：收集失败 trace → 开放编码（不预设分类）→ 归类为失败分类法 → 迭代直到不再发现新类别。

失败根因分类：

| 根因 | 含义 | 修复方向 |
|------|------|----------|
| Prompt problem | 指令不清晰导致误解 | 改 prompt |
| Tool design problem | 工具接口让 Agent 容易犯错 | 重新设计参数、加示例、明确边界 |
| Model limitation | 指令清晰但模型不泛化 | 加示例、换架构、换模型 |
| Data/infra issue | 超时、API 畸形响应、缓存过期 | 先修管道再评模型 |

> Witan Labs 修复一个 extraction bug 后 benchmark 从 50% 升到 73%。基础设施问题经常伪装成推理失败。

### 定义无歧义的成功标准

- ❌ "Summarize this document well"
- ✅ "Extract the 3 main action items, each < 20 words, include owner if mentioned"

如果两个专家对 pass/fail 无法达成一致，说明任务定义需要细化。

### 分离 Capability Evals 和 Regression Evals

| 类型 | 回答的问题 | 通过率 | 用途 |
|------|-----------|--------|------|
| Capability | "它能做什么？" | 低，逐步爬坡 | 推动进步 |
| Regression | "它还能做吗？" | ~100% | 防止退化 |

没有分离会导致：要么只守不攻（停止改进），要么只攻不守（频繁回退）。

## 二、评估层级

| 层级 | 粒度 | 测什么 | 建议 |
|------|------|--------|------|
| Single-step (Run) | 单个工具调用 | 选对工具了吗？API 调用合法吗？ | 架构稳定后再加 |
| Full-turn (Trace) | 完整一轮交互 | 最终输出 + 执行路径 + 状态变更 | **首选起点** |
| Multi-turn (Thread) | 多轮对话 | 上下文保持、跨轮一致性 | trace-level 稳定后再加 |

### Trace-level 的三个评估维度

1. **Final response** — 输出是否正确有用
2. **Trajectory** — 路径是否合理（不要求精确匹配，只要有效）
3. **State changes** — 副作用是否正确（文件写入、数据库更新、日历事件创建等）

> State change 评估常被忽略但至关重要。Agent 说 "Done!" 不代表事情真的做对了——要验证实际状态。

### Multi-turn 实用技巧：N-1 Testing

取生产中真实对话的前 N-1 轮作为前缀，只让 Agent 生成最后一轮。避免全合成多轮对话的误差累积问题。

## 三、数据集构建

- 每个任务必须无歧义，附带参考解证明可解
- 同时测正例（应该做）和反例（不应该做）——只测正例会训出过度执行的 Agent
- 数据集结构匹配评估层级（run-level 需要参考工具调用，trace-level 需要预期输出和状态变更）
- 按 Agent 类型定制：coding → 单元测试；conversational → 任务完成 + 交互质量；research → groundedness + 覆盖度
- 冷启动：手动创建 ~20 个覆盖关键变化维度的样本，**20-50 个高质量手审样本 > 数百个未验证的合成样本**

### 持续发现新 eval 的三个来源

1. **Dogfooding** — 团队每天用自己的 Agent，每个错误变成一条 eval
2. **外部 benchmark 挑选** — 从 Terminal Bench、BFCL 等挑选相关任务适配
3. **手写行为测试** — 针对特定行为（如"是否并行调用工具""是否对模糊请求追问"）

## 四、Grader 设计

### 按维度选择专门的 Grader

| 类型 | 适用场景 | 注意 |
|------|---------|------|
| Code-based | 确定性检查、格式验证、工具调用验证 | 可能误杀合法但非预期的格式 |
| LLM-as-judge | 主观质量、开放式任务 | 需要和人类校准 |
| Human | 校准、边界案例 | 贵、慢、不可规模化 |

> 有客观正确答案时默认用 code-based。LLM-as-judge 判客观题不可靠，会引入不一致性。

### Guardrails ≠ Evaluators

|  | Guardrails | Evaluators |
|---|-----------|------------|
| 时机 | 执行中，用户看到输出前 | 生成后，异步 |
| 速度 | 毫秒级 | 秒到分钟 |
| 用途 | 拦截危险/畸形输出 | 衡量质量、抓回退 |

### 关键原则

- **偏好 binary pass/fail**，不用 1-5 分制——二元判断迫使更清晰的思考，且需要更小的样本量
- **评结果不评路径** — Agent 可能找到创造性路线，硬编码路径检查会误杀有效方案
- **给部分分** — 正确识别问题但最后一步失败的 Agent，比一开始就失败的好
- **用自定义 evaluator**，不用通用指标（"helpfulness""coherence"）——通用指标制造虚假信心
- **LLM-as-judge 需要校准**：从 20+ 标注样本开始，目标 ~100 个；要求 judge 输出推理过程；定期重新校准（judge 会漂移）

## 五、运行与迭代

- **三种评估模式都要用**：Offline（发布前）、Online（生产流量）、Ad-hoc（探索性分析）
- **多次试验**：模型输出有随机性，单次运行的 benchmark 是噪声。考虑 pass@k（k 次至少成功一次）或 pass^k（k 次全部成功）
- **隔离环境**：每次试验必须干净环境，无共享状态（coding → 新容器；API → staging；DB → 快照恢复）
- **按能力分类打标签**：file_operations、retrieval、tool_use、memory、conversation 等，便于定向运行子集
- **追踪效率指标**：实际步数/理想步数、实际工具调用/理想调用、实际延迟/理想延迟
- **区分 task failure 和 evaluation failure** — grader 把超时标记为"推理错误"会污染信号
- **通过率饱和时进化测试集** — 同类型任务不再暴露新失败模式时，加更难的任务或测新维度
- **Tool interface design > Prompt optimization** — Anthropic 在 SWE-bench 上花更多时间优化工具接口而非 prompt

## 六、生产就绪

### CI/CD 集成流程

1. 代码/prompt 变更触发 pipeline
2. Offline eval 运行（code-based grader，便宜快速）
3. 通过 → 部署到 preview
4. Online eval 运行（LLM-as-judge，更贵更慢）
5. 全部通过 → 推生产；失败 → 路由到标注队列 + 告警

### 飞轮

生产失败 → 回流到数据集 → 更新 error analysis → 改进 eval → 改进 Agent → 生产表现提升

这个持续改进的飞轮本身比任何具体的 eval 实现都重要。

### 关键动作

- Capability eval 通过率稳定后提升为 regression eval
- 捕获用户反馈——自动化 eval 只能抓已知失败模式，用户会暴露未知的
- 定期手动探索生产 trace，不完全依赖自动化
- 版本化 prompt 和 tool definition，否则无法关联 eval 结果与具体变更
