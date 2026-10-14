---
{"dg-publish":true,"dg-path":"wt — Git Worktree 上下文秒切工具.md","permalink":"/wt — Git Worktree 上下文秒切工具/","tags":["type/article","area/ai","tech/git","tech/tooling"],"created":"2026-05-26","updated":"2026-10-14","dg-note-properties":{"tags":["type/article","area/ai","tech/git","tech/tooling"],"created":"2026-05-26","updated":"2026-10-14","description":"wt：把 Git worktree 与 AI 工具启动合成统一入口，减少跨项目切换摩擦。说明原生 worktree 的保留机制、共享环境边界与强删风险；附件下载链接待发布者提供。"}}
---


## TL;DR

- `wt <branch> <tool>` — 一键创建/进入 worktree 并启动 codex / claude / kiro（或任何命令）
- 仓库名作为命名空间，`~/.worktrees/<repo>/<branch>` 减少跨项目命名冲突；同名仓库仍需另行区分
- 工具在独立工作副本里运行，不必在主工作副本切分支；进程、端口和外部服务仍需单独隔离
- Bash CLI 的统一入口；需 Git、Bash 及要启动的工具，安装与兼容性以实际附件版本为准

本文介绍设计与使用接口，不代表已经对附件脚本做过安全审计或运行测试。

## 问题：AI 编码时代的上下文地狱

用 AI 编码工具时，最痛苦的不是写 prompt，是切换。

你正在 `feature/payment` 上重构支付模块，Claude Code 的对话已经跑了 20 轮，上下文里积累了大量设计决策和 trade-off。这时来了一个紧急 bug —— 线上登录页崩了。

你的选项：

1. **新开一个终端 tab**，在同一工作副本切到 main → 新 session 可以开，但旧 session 仍可能读写这个目录，两个任务容易互相干扰
2. **中断当前对话**，`git stash`，切分支，开新对话修 bug → 修完再 `git stash pop`，重新加载上下文，祈祷没冲突
3. **再 clone 一份仓库** → 多一份仓库与工作副本；分支/commit 可以通过 fetch 等方式交换，但不如共享对象库直接

三个选项都在浪费你的注意力。

## Worktree 是最佳解，但少了一步

`git worktree` 是 Git 原生解决这个问题的机制：

```bash
git worktree add -b hotfix/login-bug ~/.worktrees/hotfix main
cd ~/.worktrees/hotfix
# 修 bug...
git worktree remove ~/.worktrees/hotfix
```

两个工作区共享 Git 对象库和通常的分支引用、stash，各自有工作文件、HEAD 和 index。你不必为了换任务 stash 当前工作，也不必再 clone。Git 默认不允许同一分支同时被多个 worktree 检出；示例从 main 新建 hotfix 分支，而不是强行在两处检出 main。

但手动敲这三行命令的方式在 AI 编码时代不够用。缺了关键一步：**启动你的 AI 工具**。

## wt 的设计

`wt` 做的事情很简单：把 `git worktree add` 和工具启动合并成一行。

```bash
# 从 main 创建 bugfix 分支的 worktree，启动 Claude Code
wt hotfix/login-bug cc

# 启动 Codex
wt refactor-auth codex

# 不加工具 → 开一个 shell 在 worktree 里
wt experiment
```

三个核心原则：

**幂等进入**。设计目标是同名 worktree 已存在就直接进入，不重复创建。实际使用前仍要核对目标目录属于预期仓库和分支；同名路径存在不等于状态正确。

**仓库命名空间**。路径是 `~/.worktrees/<repo-name>/<branch-name>`。`api-server` 和 `frontend` 都有 `main` 分支时，路径可以分别是 `~/.worktrees/api-server/main` 和 `~/.worktrees/frontend/main`。但两个不同位置的仓库若都叫 `api-server`，仓库名本身不够唯一，需要另外区分存放位置。

**工具退出后继续在原位工作**。作为独立可执行命令运行时，脚本可以在子进程里 `cd` 到 worktree，再用 `exec` 启动工具；父 shell 的目录本来就没有改变。这不是 `exec` 自动恢复目录，也不意味着工具不会写共享配置或操作外部系统。具体实现仍以附件为准。

### 管理命令

```bash
wt -l              # 列出所有 worktree，按 repo 分组
wt -d experiment   # 删除 worktree，保留分支
wt -f -d experiment # 强制删除（丢弃未提交改动）
```

**强删是破坏性操作。** 删除前确认目标仓库、分支和目录，检查 `git status`、未跟踪文件及仍在运行的工具或服务，先提交、备份或另行保存需要的工作。未跟踪文件也可能一起消失，不能假设“没有 commit 的内容都能靠 Git 找回”。不要把 `-f` 当成普通清理选项；这些是使用提示，不是对脚本参数安全性的测试结论。

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `WT_BASE` | `~/.worktrees` | worktree 根目录 |
| `WT_DEFAULT_BASE` | 自动检测 main→master→HEAD | 新建分支的基准分支 |

## 为什么不用工具自带的 worktree 功能

一些 AI 编码工具内置了 worktree 管理。但独立脚本有几个优势：

**跨工具通用**。同一个 `wt` 命令启动 codex、claude、kiro 或任何其他工具。你不会被绑定到某个工具的 worktree 实现上。

**直接启动**。不需要先启动 AI 对话、再让 agent 帮你创建 worktree。`wt feature-x cc` 合并了入口操作；不意味着 worktree 创建、依赖初始化或工具启动没有耗时。Claude Code 自己也支持 `claude --worktree <name>` 直接启动，不能把这一点说成 wt 独有。

**可脚本化**。`wt` 是标准 CLI，可以嵌到 CI、pre-commit hook、tmux 配置等任何自动化场景。

**统一生命周期管理**。`wt` 的价值是用同一套目录与管理接口承载不同工具，而不是因为工具内置 worktree 一结束就必定删除。按 Claude Code 当前官方文档，交互会话退出时，干净且未命名的会话可自动清理；命名会话会先询问，有改动、未跟踪文件或新提交等工作时也会询问保留还是删除，无法核实状态时不会直接自动删。保留后可用退出提示的 `--worktree <name> --resume` 返回。非交互 `-p` 运行没有退出提示，也不会在退出时清理 worktree。

Claude Code 的子 agent 与后台会话 worktree 另有按保留策略进行的周期清理，仍会跳过有工作或被占用等情况。手工用 `git worktree add` 创建的 worktree 不属于它的自动清理对象。选择 `wt`，主要是为了跨工具的统一入口和显式清理，不是填补 Claude 已经支持的保留与恢复能力。具体行为应随工具版本复核。

## 多项目并行的实际效率

用一个示例看这个工作流：Jim 有三个项目同时推进——不是他想并行，是现实就是这样的：

- `api-server`：重构认证中间件（分支 `refactor/auth-middleware`）
- `frontend`：修线上按钮样式 bug（分支 `hotfix/button-style`）
- `docs`：更新 API 文档（分支 `docs/api-v2`）

用 `wt`：

```bash
# 9:00 — 开始重构
cd ~/Projects/api-server
wt refactor/auth-middleware cc

# 10:30 — 紧急 bug
# 另外开个 terminal tab
cd ~/Projects/frontend
wt hotfix/button-style cc

# 11:00 — bug 修完，清理
wt -d hotfix/button-style

# 回到 api-server 的 tab，继续使用原 session（前提是进程仍在）
# 继续重构...
```

三个 worktree 可以同时存在，各自维护工作文件、IDE 窗口和 AI session。依赖仍需按项目初始化，目录不同也不自动创建或恢复工具对话。准备好这些以后，切换可以从"stash → checkout → 装依赖 → 重开对话"缩短为"换个 terminal tab"。

隔离的是工作副本，不是整台机器：仓库元数据、默认 Git 配置和 stash 仍可能共享；凭据、工具配置、依赖缓存也可能共享。后台进程、监听端口、数据库和外部服务不会因目录不同自动隔离，需要独立端口、环境配置或容器。Claude Code 文档还提醒，进入 worktree 后 hook 的 `${CLAUDE_PROJECT_DIR}` 可能仍指向启动时的项目根，而 hook 输入的 `cwd` 才跟随当前目录；hook 是否操作主副本要单独核对。

这不是多线程——人的大脑做不到多线程。这是**减少上下文切换的摩擦**，让你在不得不切换时尽可能少地丢失状态。

工作副本隔离也不解除审查和整合上限。我建议按实际验收能力决定并行任务数，限制待审 worktree 的数量和等待时长；更多分支不是更多已验收交付。目录仍在，只能说明文件保留，不能保证工具会话或执行现场恢复。重启后要通过工具真实支持的恢复机制，取回目标、决策与待办记录，并核对进程、端口和外部动作是否仍有效，不宣称任务能从中断点原样续跑。

## 附件：完整脚本

> 脚本持续迭代中，文章内不再嵌入代码。附件名：`wt.sh.attachment`。**当前公开页面未提供可确认的实际下载 URL，请由发布者提供附件链接；不要猜测路径。** 取得可信版本后，先阅读脚本并核对环境，再安装到 `~/.local/bin/wt`，赋予执行权限，确认 `~/.local/bin` 在 `PATH` 中。本文未运行或审计附件。
>
> **不建议在文章中直接嵌入可执行脚本**。问题有三：(1) 代码演进而文章冻结，嵌入版本可能过时；(2) 大段代码破坏阅读节奏；(3) 复制粘贴没有可执行性或安全保障。单独提供附件能保文章轻量，但必须同步维护链接和版本，扩展名本身不保证最新或可信。

## Agent 实施指南

> 以下内容可以直接粘贴给任何 AI 编码助手，让它帮你完成部署。

---

请帮我安装一个名为 `wt` 的 CLI 工具。它的脚本在附件文件 `wt.sh.attachment` 中，功能说明在「wt 的设计」章节中。请：

1. 请发布者提供 `wt.sh.attachment` 的可信下载链接或附件文件；没有附件时先停止，不自行编造下载 URL 或补写替代脚本。取得附件后先阅读并说明会执行的操作，再写入 `~/.local/bin/wt`（若已存在，先确认是否覆盖）
2. `chmod +x ~/.local/bin/wt`
3. 检查 `~/.local/bin` 是否在 `PATH` 中，如果不在则添加到 `~/.bashrc`（或对应的 shell 配置文件）
4. 在确认来源和脚本操作后，执行 `wt --help` 检查入口，并报告真实输出
5. 在可丢弃的测试仓库中验证 `wt -l` 和创建/进入流程，核对原工作副本未被修改；不要在真实仓库执行强删测试。本文列的是待安装者执行的步骤，不声称这些测试已经完成

附加配置（可选）：
- 本文接口约定支持 `codex`、`claude`（`cc` 映射为 `claude`）、`kiro` 及 PATH 中的命令，实际支持情况以附件版本为准。若工具是 `~/.bashrc` 中的函数（如 `codexd`、`gemini`），需先检查附件如何加载或提取函数，不能把 shell 函数当作天然可执行命令
- 如果希望 worktree 存放在其他位置，在 `~/.bashrc` 中设置 `export WT_BASE=~/your/path`
- 如果默认基准分支不是 main/master，设置 `export WT_DEFAULT_BASE=develop`

---

## 参考

- [Git Worktree 官方文档](https://git-scm.com/docs/git-worktree)
- [Claude Code Worktrees 官方文档](https://code.claude.com/docs/en/worktrees)（清理、恢复、hook 路径与隔离边界）
- [Script-as-Pipeline — Agent 管线中的脚本节点设计模式](https://nexus.sigmoid.cc/Script-as-Pipeline%20%E2%80%94%20Agent%20%E7%AE%A1%E7%BA%BF%E4%B8%AD%E7%9A%84%E8%84%9A%E6%9C%AC%E8%8A%82%E7%82%B9%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/)
