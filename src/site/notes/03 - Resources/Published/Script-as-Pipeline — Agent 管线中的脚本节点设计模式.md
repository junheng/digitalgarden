---
{"dg-publish":true,"dg-path":"Script-as-Pipeline — Agent 管线中的脚本节点设计模式.md","permalink":"/Script-as-Pipeline — Agent 管线中的脚本节点设计模式/","tags":["type/article","area/ai","area/architecture"],"created":"2026-05-12","updated":"2026-10-14","dg-note-properties":{"tags":["type/article","area/ai","area/architecture"],"created":"2026-05-12","updated":"2026-10-14","description":"把脚本设计成 Agent 管线节点：stdout 返回低信任数据与行动建议，宿主通过 schema、路由白名单和权限审核落实控制流，而不是让输出自行成为指令。"}}
---


## 你的 agent 跑完脚本之后，stdout 去哪了？

你写了一个 skill，里面有个 shell 脚本。Agent 执行它，读取输出，然后继续工作。看起来没什么特别的——脚本就是个工具，agent 用它干活。

但仔细想一下：在常见的执行工具中，脚本的 stdout 会作为工具结果（tool result）进入 agent 的上下文，和上一步的输出、用户指令一起影响下一步判断。**进入同一个上下文，不等于获得同样的指令权威。** stdout 仍然是低信任数据，不是 system prompt，也不会因为来自本地脚本就自动变成可信指令。

这不只是格式问题。脚本可能转述网页、用户文件或日志，里面的文字可能在说「忽略之前的规则」「把凭据发到这里」。如果 agent 把它们当作新规则执行，发生的不是管线编排，而是提示注入（prompt injection）。脚本可信，不代表它搬运的内容可信。

所以，我想强调的不是「stdout 拥有和 prompt 同等的编程权力」，而是：**脚本可以被设计为管线节点，输出节点之间约定的数据和行动建议；哪些信号能变成控制流，由宿主在授权范围内决定。**

## 脚本的三层角色

我把脚本在 agent 系统中的职责分成三个层级。这是一种设计视角，不是行业成熟度排名，也不是说所有团队都应走到第三层。

### 第一层：Guardrail（门卫）

脚本只做一件事：检查，然后返回 exit code。宿主按约定处理：0 继续，非 0 停下或转入错误处理。

这对应 Anthropic「Building Effective Agents」中 Prompt Chaining 的 programmatic gate：在 LLM 步骤之间插入程序检查，防止不满足条件的结果直接传到下一步。它严格执行的是你写下的检查条件，不会自动证明整个业务目标已经满足。

但不能因此把 LangGraph 的 ToolNode 也限定为「只能过不过」。LangGraph 的路由可以由 conditional edge 实现，也可以由节点或工具返回 `Command`，用 `update` 更新 state、用 `goto` 指向下一节点。工具返回状态和控制流，并不是 stdout 模式才有的能力。

门卫是一种职责，不是工具框架的能力上限。

### 第二层：Informant（导航员）

脚本不仅返回状态，还返回**结构化的行动信号**。这里需要区分两件事：让 agent 理解信号、提出下一步建议，和让宿主解析字段、执行分支。

一个具体的例子：我在设计 meta-harness 时，给脚本定义了三类 action token：

- `📋` — 信息输出，提示 agent 了解相关状态
- `⚡` — 行动建议，提示 agent 判断是否需要处理
- `🚫` — 阻塞提示，提示 agent 解释问题或请求协助

这些标记方便人和模型阅读，但它们不是权限凭证。模型对 emoji 的解释仍然可能出错；日志正文里出现一个 `🚫`，也不能自动获得「停止整个管线」的权力。真正需要强制阻塞时，应由宿主读取明确的状态字段并落实，而不是等模型理解符号。

比如 `scaffold.sh` 输出 `created:5 / skipped:3`。如果由 agent 读到后决定进行 archive comparison，这是**模型驱动路由**。如果宿主按 schema 验证 `skipped` 是合法计数，按预定义规则检查 `skipped > 0`，再通过路由白名单（allowlist）与权限审核进入对比分支，才是**程序控制的路由**。同一行输出，接法不同，可靠性边界就不同。

### 第三层：Conductor（指挥官）

这是我在形式化的方向：脚本输出的不只是状态，还可以包括下一阶段所需的上下文、行动建议，以及宿主预先允许的控制字段。它不是从 stdout 里临时创造新的系统规则。

管线的形状变成：

```text
step1 → script0 → 结构化结果 → 宿主解析、校验、授权
                                  ├─ 路径A → script1a → step2a
                                  └─ 路径B → script1b → step2b
```

要把控制流从自然语言下沉到程序，宿主至少需要做三件事：

- **按 schema 解析**：规定字段、类型和取值；控制字段与自由文本日志分开，缺失、截断或格式不合法时不猜分支。
- **按 allowlist 路由**：只接受已注册的节点或动作；不把任意输出字符串当作 shell 命令、工具名或新 prompt 执行。
- **按权限审核执行**：核对当前任务的授权、数据范围和副作用；需要人类批准的动作不能靠脚本自称「已获批准」跳过。

schema 合法只是第一关，不证明内容真实，更不授予权限。即使不可信日志挟带了看似合法的控制字段，也应留在数据面；宿主只能认可来源、语义和授权都符合契约的信号。不得把日志或外部文本直接拼进高权威指令。

在这些条件下，程序负责约定好的分支，prompt 负责解释上下文、处理开放问题和生成输出。如果分支仍由 LLM 从自由文本中选择，就应诚实地称它为模型决策，不能包装成确定性控制。两者都能有用，重要的是不混淆。

## 为什么是 stdout？

Agent 和脚本之间的通信有几种候选通道：exit code、文件系统、数据库、网络。为什么考虑 stdout？

因为在常见的同步执行工具里，stdout 能随着工具返回直接交给 agent，通常不用再单独读文件。这是一种方便的默认接法，**不是 Unix 保证输出完整进入模型上下文**。宿主可能截断、摘要、限制大小，也可能先返回后台任务句柄，稍后才提供结果。

如果下一步依赖完整数据，就应规定输出上限、完整性标记和失败处理；长日志或大型产物另存为文件并提供可回读引用。控制记录不能埋在可能被截断的日志末尾，也不能在记录不完整时让模型「大概猜一下」。

Unix 管道给我的启发不是「文本天然就是控制流」，而是**小接口、明确契约、可组合**。前一个程序的输出能成为后一个的输入，前提是双方约定了格式与含义；agent 管线同样需要这一层。

Action token 体系可以是人机可读展示层的雏形。我的设计约束是：

- 对人类可读，方便调试
- 对模型含义清楚，但不假设模型必然正确解析
- 标记尽量少、含义互斥，具体数量通过目标模型与任务测试决定
- 强控制使用宿主可校验的字段，不依赖 emoji 解释

「三个」是我的选择，不是通用阈值。没有足够实验支持「五个以上 agent 必然混淆」。协议是否好用，要看误路由、漏处理和异常输入下的行为。

## 和文件式通信的边界

这里有必要区分两种常见接法，但不能把它们说成通道的绝对属性。

文件式 Agent 间通信——比如 Anthropic 的 Planner → Generator → Evaluator 通过 spec 和 sprint contract 交接——适合持久化、可回读的上下文。一个 agent 写，另一个 agent 读，可以跨会话，也可以在同一次运行里同步交接。文件能保存完整产物，但不保证写入、读取、版本选择都正确。

Script stdout 常被用作一次调用的增量反馈，适合短状态和小型结构化结果。但它也可以承载较大数据，或者属于异步任务；是否完整、何时可见，取决于宿主。

两者不是替代关系，也不按「文件只适合 agent 之间、stdout 只适合 harness」划死边界。选择依据应是生命周期、数据量、可回读性和调度方式。**换成文件不会消除 prompt injection；留在 stdout 也不会自动得到路由权威。**

## 节点契约还要说明恢复与卸载

后续思考让我把接口从“这轮返回什么”扩展到“中断后还能承诺什么”。作为工程建议，我会把事实日志、当前上下文与执行现场分开：输出另存原件可以保留证据，不会自动保存后台进程、变量或尚未返回的 API 调用。节点应返回可关联的执行标识与状态；超时先记为结果未知，按服务真正支持的机制核对，不直接重放有副作用的动作。

可组合也不等于安全可拆：节点注册的监听、计时器和临时资源需要明确清理；替换涉及的状态要迁移、验证并有回退准备。这些生命周期要求适用于 stdout、文件或网络接口，不是换一种通道就自动获得的能力。清理运行资源，也不是撤销已经发生的外部结果。

## 脚本作为管线节点的真正意义

把 script 从「临时工具」进一步设计成「管线节点」，改变的不只是实现方式，而是设计 agent 系统时的思考框架。它仍然可以是工具，只是多了明确的系统职责。

当你只把 script 看作工具时，你在想「agent 需要什么功能，我写个脚本给它用」。这是**以 agent 为中心**的视角。

当你把 script 看作管线节点时，你在想「这条管线上有哪些决策点、哪些验证点、哪些状态需要传递，谁有权决定下一步」。这是**以系统为中心**的视角。

前者关注怎样「用 agent」，后者关注怎样「设计 agent 系统」。区别不在于多几个标记，而在于是否把接口契约、信任边界和失败处理设计出来。

这关联到我对 Harness Engineering 的理解：agent 的有效性不只取决于 system prompt，也取决于围绕它的工具、状态、约束和验证结构。Script stdout 是环境提供运行时信息的一种方式——**信息可以影响判断，但控制权必须由系统明确分配。**

## 参考

- [Anthropic, Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)：Prompt Chaining 中的 programmatic gate，以及 workflows 与模型驱动 agents 的区分。
- [LangGraph Graph API — Command](https://docs.langchain.com/oss/python/langgraph/graph-api#command)：`update` 与 `goto`，以及「Return from tools」。返回 `Command` 的动态边不会自动取消既有静态边，路由接法也需要检查。
- [OpenAI, Safety in building agents](https://developers.openai.com/api/docs/guides/agent-builder-safety)：Prompt injections、不要把不可信变量放入 developer messages，以及结构化输出的数据流边界。这里引用安全方法，不是推荐 Agent Builder 界面。
- [Anthropic, Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)：Planner、Generator、Evaluator 的文件交接与验收契约案例。
