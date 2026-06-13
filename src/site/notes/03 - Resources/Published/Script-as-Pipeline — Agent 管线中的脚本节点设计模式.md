---
{"dg-publish":true,"dg-path":"Script-as-Pipeline — Agent 管线中的脚本节点设计模式.md","permalink":"/Script-as-Pipeline — Agent 管线中的脚本节点设计模式/","tags":["type/article","area/ai","area/architecture"],"created":"2026-06-02T10:09:03.731+08:00","updated":"2026-06-02T10:09:03.733+08:00","dg-note-properties":{"tags":["type/article","area/ai","area/architecture"],"created":"2026-05-12","updated":"2026-05-12","description":"Agent 调用脚本时，stdout 不是日志——是动态生成的 prompt 片段。把脚本从工具升级为管线节点，是从「用 agent」到「设计 agent 系统」的标志。"}}
---


## 你的 agent 跑完脚本之后，stdout 去哪了？

你写了一个 skill，里面有个 shell 脚本。Agent 执行它，读取输出，然后继续工作。看起来没什么特别的——脚本就是个工具，agent 用它干活。

但仔细想一下：脚本的 stdout 被注入到了 agent 的上下文中。它和你的 system prompt、上一步的输出、用户的指令混在一起，成为 agent 下一步决策的依据。

这可不是"工具返回了一个结果"那么简单。**脚本的 stdout 是一段动态生成的 prompt 片段。** 它可以在 agent 不知道的情况下，改变它的行为路径。

这意味着什么？意味着 script 不是 agent 的工具——script 是 agent 管线中的一个节点，它拥有和 prompt 同等的"编程"能力。

## 脚本的三层角色

业界对 script 在 agent 系统中的用法，大致可以分成三个层级。大多数团队停在第一层。

### 第一层：Guardrail（门卫）

脚本只做一件事：检查，然后返回 exit code。0 继续，非 0 停下。

这是 Anthropic "Building Effective Agents" 博客中描述的 Prompt Chaining 模式——在 LLM 步骤之间插入确定性检查，防止错误在链上传播。LangGraph 的 ToolNode 本质上也是这一层：tool 返回值写进 state，但路由逻辑写在 conditional edge 里，和 tool 本身是分离的。

这一层已经普及了。但它把脚本当成一个"门"——只能决定过不过，不能决定往哪走。

### 第二层：Informant（导航员）

脚本不仅返回状态，还返回**结构化的行动信号**。Agent 不是简单地"看到输出"，而是解析语义标记后决定行为。

一个具体的例子：我在设计 meta-harness 时，给所有脚本定义了三类 action token：

- `📋` — 信息输出，agent 应该了解
- `⚡` — 需要 agent 采取行动
- `🚫` — 阻塞，agent 必须停下来处理

这不是给人看的日志。这是给 agent 解析的语义标记——脚本在说"接下来你要做这个"，而不是"我执行完了，这是结果"。

更关键的是，stdout 里的数据可以直接路由管线。`scaffold.sh` 输出 `created:5 / skipped:3`——当 `skipped > 0` 时，agent 自动激活 archive comparison 分支，对比新旧文件差异。这不是 agent 自己"想到"的，是脚本的输出引导它走的。

### 第三层：Conductor（指挥官）

这是我在形式化的方向：脚本不只是输出状态标记，而是输出**下一段 prompt 的完整片段**——包括路由指令、约束条件、甚至临时的行为规则。

管线的形状变成：

```
step1 → script0 → stdout 输出路由标记
                     ├─ 路径A → script1a → step2a
                     └─ 路径B → script1b → step2b
```

控制流从 prompt 的自然语言描述，下沉到了脚本的确定性逻辑。Prompt 仍然重要，但它不再负责"流程控制"——它负责理解上下文和生成输出，脚本负责管线的路由。

## 为什么是 stdout？

Agent 和脚本之间的通信有几种候选通道：exit code、文件系统、数据库、网络。为什么是 stdout？

因为 stdout 有一个其他通道没有的特性：**它被自动注入到 agent 的上下文中。** 不需要 agent 主动去读文件、不需要额外的 tool call、不需要轮询。脚本执行完毕，输出就在 agent 的"视野"里了。

这听起来像是技术细节，但它的意义远超技术细节。Unix 管道能工作，是因为每个程序遵守同一个契约：stdin 是文本流，stdout 也是文本流，`\n` 分隔。这个极简的契约让任意程序可以组合——前一个的输出直接变成后一个的输入。

Agent 管线缺的就是这一层。Tool result 注入上下文时，没有任何格式约定——agent 自由解释，路由完全依赖 prompt 中的文字指令。当管线变复杂时，prompt 里的 if/else 越来越长，agent 的理解误差越来越大。

Action token 体系就是这个契约的雏形。它的设计约束很明确：
- 必须对人类可读（方便调试）
- 必须对 LLM 可解析（能在 prompt 中触发可靠的分支决策）
- token 数量必须极少（三个刚好，五个以上 agent 开始混淆）

## 和文件式通信的边界

这里有必要区分两种模式，因为它们容易混淆。

文件式 Agent 间通信——比如 Anthropic 的 Planner → Generator → Evaluator 通过 spec/sprint contract 文件交接——是**异步的、完整的上下文传递**。一个 agent 写完文件退出，另一个 agent 启动时读取。目标是信息不丢失。

Script stdout 是**同步的、增量的上下文注入**。Agent 不停下，脚本作为一个管线节点被嵌入到 agent 的执行流中。目标是控制流路由。

两者不是替代关系，而是互补。文件适合 agent 之间，stdout 适合 agent 与 harness 组件之间。

## 脚本作为管线节点的真正意义

把 script 从"工具"升级为"管线节点"，改变的不只是实现方式，而是设计 agent 系统时的思考框架。

当你把 script 看作工具时，你在想"agent 需要什么功能，我写个脚本给它用"。这是**以 agent 为中心**的视角。

当你把 script 看作管线节点时，你在想"这条管线上有哪些决策点、哪些验证点、哪些状态需要传递"。这是**以系统为中心**的视角。

前者是"用 agent"，后者是"设计 agent 系统"。两条路的能力天花板不同。

这直接关联到 Harness Engineering 的核心主张：agent 的有效性不来自它的 system prompt，而来自围绕它的环境结构。Script stdout 是环境向 agent "说话"的方式——不是通过人类预先写好的 prompt，而是通过运行时动态生成的信息。

## 参考

- Anthropic, "Building Effective Agents" — Prompt Chaining 与 programmatic gate 模式: https://www.anthropic.com/engineering/building-effective-agents
- LangGraph StateGraph 文档 — ToolNode 与 conditional edge: https://langchain-ai.github.io/langgraph/
- Boris Cherny, "Why I'm Building Claude Code" — thin harness 哲学
- Prithvi Rajasekaran, "Build to Delete" — harness 组件的临时性
