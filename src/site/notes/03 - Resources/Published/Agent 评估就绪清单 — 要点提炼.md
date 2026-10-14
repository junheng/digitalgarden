---
{"dg-publish":true,"dg-path":"Agent 评估就绪清单 — 要点提炼.md","permalink":"/Agent 评估就绪清单 — 要点提炼/","tags":["type/article","area/ai"],"created":"2026-03-28","updated":"2026-10-14","dg-note-properties":{"tags":["type/article","area/ai"],"created":"2026-03-28","updated":"2026-10-14","source":"https://blog.langchain.com/agent-evaluation-readiness-checklist/","description":"LangChain Agent 评估清单提炼：从错误分析到生产飞轮，恢复二元评分与短量表的适用边界，区分护栏职责和实现速度，同时验证输出、行为约束与外部状态。"}}
---


## 来源

- 原文：[Agent Evaluation Readiness Checklist](https://blog.langchain.com/agent-evaluation-readiness-checklist/)（LangChain Blog，Victor Moreira，2026-03-27）
- 本次复核使用[当前官方页面](https://www.langchain.com/blog/agent-evaluation-readiness-checklist)，重点对应 Grader design 的评分、护栏和路径评估部分。
- 配套阅读：[Agent Observability Powers Agent Evaluation](https://blog.langchain.com/agent-observability-powers-agent-evaluation/)

## 核心理念

**从最简单的 eval 开始，只在简单方法漏掉真实失败时才加复杂度。**

几个端到端 eval 测试 Agent 是否完成核心任务，就能立即建立基线——即使架构还在变。

## 一、构建 Eval 之前

### 手动审查 20-50 条真实 trace

原文建议先花 30 分钟读真实 Agent trace，把它作为建立失败分类的高回报起点。这是实践建议，不是手审在所有任务上都胜过自动化的比较结论。

### 60-80% 的精力花在 Error Analysis 上

流程：收集失败 trace → 开放编码（不预设分类）→ 归类为失败分类法 → 迭代直到不再发现新类别。

失败根因分类：

| 根因 | 含义 | 修复方向 |
|------|------|----------|
| Prompt problem | 指令不清晰导致误解 | 改 prompt |
| Tool design problem | 工具接口让 Agent 容易犯错 | 重新设计参数、加示例、明确边界 |
| Model limitation | 指令清晰但模型不泛化 | 加示例、换架构、换模型 |
| Data/infra issue | 超时、API 畸形响应、缓存过期 | 先修管道再评模型 |

> 原文转述：Witan Labs 修复一个 extraction bug 后 benchmark 从 50% 升到 73%。这是该团队的个案，不是通用提升幅度；提醒是先排查基础设施，别把管道故障直接归因于推理失败。

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
2. **Trajectory** — 路径是否合理（不要求精确匹配，但必须满足授权、范围和安全约束）
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
| 典型时机 | 执行中，相关动作或输出放行前 | 生成后或事后，常异步运行 |
| 延迟要求 | 需符合在线路径的延迟预算，不保证毫秒级 | 可接受较高成本，但也可快速运行 |
| 用途 | 拦截危险/畸形输出 | 衡量质量、抓回退 |

原文表格里的“毫秒”与“秒到分钟”描述常见工程配置，不是分类定义。Guardrail 可以调用昂贵模型，evaluator 也可以只是快速的确定性代码；区别在于是否承担运行时放行职责，而不在必然速度。

### 关键原则

- **默认偏好 binary pass/fail（二元通过/失败）** — 任务成功标准清楚时，二元判断便于人工审查和快速迭代。不要机械禁用量表：原文明确保留意见，特定 LLM-as-judge 研究中短量表 0-5 可能有更强的人类一致性。根据维度、标注一致性和统计目标选择，不能把二元一概说成样本需求更小
- **评结果，不强制无关的精确路径** — Agent 可能找到创造性路线，硬编码工具 A → B → C 会误杀有效方案。但授权、操作范围、安全行为和副作用仍须检查；“结果正确”不能豁免越权。这与上文 Trajectory、State changes 评估互补
- **给部分分** — 正确识别问题但最后一步失败的 Agent，比一开始就失败的好
- **用自定义 evaluator**，不用通用指标（"helpfulness""coherence"）——通用指标制造虚假信心
- **LLM-as-judge 需要校准**：从 20+ 标注样本开始，目标 ~100 个；要求 judge 输出推理过程；定期重新校准（judge 会漂移）

## 五、运行与迭代

- **三种评估模式都要用**：Offline（发布前）、Online（生产流量）、Ad-hoc（探索性分析）
- **多次试验**：模型输出有随机性，单次运行的 benchmark 可能混入较大噪声。考虑 pass@k（k 次至少成功一次）或 pass^k（k 次全部成功），并说明它们分别衡量机会性成功与重复可靠性
- **隔离环境**：每次试验必须干净环境，无共享状态（coding → 新容器；API → staging；DB → 快照恢复）
- **按能力分类打标签**：file_operations、retrieval、tool_use、memory、conversation 等，便于定向运行子集
- **追踪效率指标**：实际步数/理想步数、实际工具调用/理想调用、实际延迟/理想延迟
- **区分 task failure 和 evaluation failure** — grader 把超时标记为"推理错误"会污染信号
- **通过率饱和时进化测试集** — 同类型任务不再暴露新失败模式时，加更难的任务或测新维度
- **Tool interface design > Prompt optimization** — Anthropic 在 SWE-bench 上花更多时间优化工具接口而非 prompt

## 我的工程补充：校准信号，也治理信号的后续用途

以下是我在后续思考中增加的建议，不是 LangChain 原清单的直接引文。

人工审查应关联到对应 trace、产物与系统版本，保留标准、简短依据、错误类型和分歧。“用户喜欢”“业务结果好”“符合任务约定”是不同信号，不能混成一个通过标签。先用原文、实际状态与人工样本校准，再把评分用于修复；记录可观察事件即可，不要求采集模型完整内部思维。

修复轮数增加不保证质量更好。反馈误判、覆盖错目标或诱发局部补丁时，循环可能放大偏差；应联看误报、漏检、原有不变量和新回归，而不把“独立 reviewer”当正确性的保证。

若系统会自动提炼 memory、Skill 或提示规则，经验先作为带来源和适用条件的候选，不因一次高分就直接改变长期行为。进入生产定义之前，要有版本、独立 Eval、适用的批准与发布流程；上线后可撤回并观察回归。有效期可以触发复核，不能替代适用性判断。

对自主续跑，还应区分继续执行的权限与完成判定的依据：运行层可以在授权和预算内推进，但“完成”要由可核验产物、外部状态及必要验收支持。飞轮可能放大有用信号，也可能固化误判，不天然保证向好。

## 六、生产就绪

### CI/CD 集成流程

以下是一种门禁编排示例，不是原文要求每个系统照搬的固定顺序；根据风险和延迟预算选择实现。生产流量的 online eval 通常持续观察，不应把事后异步评分当成已经阻止了危险动作。

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
