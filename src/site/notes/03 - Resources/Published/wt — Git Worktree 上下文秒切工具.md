---
{"dg-publish":true,"dg-path":"wt — Git Worktree 上下文秒切工具.md","permalink":"/wt — Git Worktree 上下文秒切工具/","tags":["type/article","area/ai","tech/git","tech/tooling"],"created":"2026-05-26","updated":"2026-06-11T14:30","dg-note-properties":{"tags":["type/article","area/ai","tech/git","tech/tooling"],"created":"2026-05-26","updated":"2026-06-11T14:30","description":"wt：一行命令创建 git worktree 并启动 AI 工具，多项目并行零上下文切换成本。附带完整脚本和 Agent 实施指南。"}}
---


## TL;DR

- `wt <branch> <tool>` — 一键创建/进入 worktree 并启动 codex / claude / kiro（或任何命令）
- 仓库名作为命名空间，`~/.worktrees/<repo>/<branch>` 免冲突
- 工具退出后回到原位目录，主工作区零污染
- 50 行 bash，零依赖，即装即用

## 问题：AI 编码时代的上下文地狱

用 AI 编码工具时，最痛苦的不是写 prompt，是切换。

你正在 `feature/payment` 上重构支付模块，Claude Code 的对话已经跑了 20 轮，上下文里积累了大量设计决策和 trade-off。这时来了一个紧急 bug —— 线上登录页崩了。

你的选项：

1. **新开一个终端 tab**，切到 main，修 bug，切回来 → 但你只有一个 Claude Code session，修 bug 也需要 Claude 帮忙
2. **中断当前对话**，`git stash`，切分支，开新对话修 bug → 修完再 `git stash pop`，重新加载上下文，祈祷没冲突
3. **再 clone 一份仓库** → 磁盘浪费，而且两个 clone 之间的分支/commit 不共享，cherry-pick 都麻烦

三个选项都在浪费你的注意力。

## Worktree 是最佳解，但少了一步

`git worktree` 是 Git 原生解决这个问题的机制：

```bash
git worktree add ~/.worktrees/hotfix main
cd ~/.worktrees/hotfix
# 修 bug...
git worktree remove ~/.worktrees/hotfix
```

两个工作区共享同一个 `.git` 对象库，分支、commit、stash 全部互通。不需要 clone，不需要 stash。

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

**幂等进入**。同名 worktree 已存在就直接 `cd` 进去，不重复创建。你可以放心在脚本里多次调用，或者在忘记是否已创建时随手 `wt xxx`。

**仓库命名空间**。路径是 `~/.worktrees/<repo-name>/<branch-name>`。两个不同仓库都有 `main` 分支？各自是 `~/.worktrees/api-server/main` 和 `~/.worktrees/frontend/main`，不打架。

**工具退出即回到原位**。脚本用 `exec` 替换自身进程：`cd` 到 worktree，启动工具，工具退出后你回到调用 `wt` 时的原始目录和工作区。什么都不会被污染。

### 管理命令

```bash
wt -l              # 列出所有 worktree，按 repo 分组
wt -d experiment   # 删除 worktree，保留分支
wt -f -d experiment # 强制删除（丢弃未提交改动）
```

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `WT_BASE` | `~/.worktrees` | worktree 根目录 |
| `WT_DEFAULT_BASE` | 自动检测 main→master→HEAD | 新建分支的基准分支 |

## 为什么不用工具自带的 worktree 功能

一些 AI 编码工具内置了 worktree 管理。但独立脚本有几个优势：

**跨工具通用**。同一个 `wt` 命令启动 codex、claude、kiro 或任何其他工具。你不会被绑定到某个工具的 worktree 实现上。

**零延迟**。不需要先启动 AI 对话、再让 agent 帮你创建 worktree。`wt feature-x cc` 直接到位。

**可脚本化**。`wt` 是标准 CLI，可以嵌到 CI、pre-commit hook、tmux 配置等任何自动化场景。

**不和工具 session 绑定**。AI 工具内置的 worktree 通常和 session 绑定 —— 对话结束，worktree 也跟着清理。`wt` 创建的是普通 git worktree，不管你开多少个工具 session、关掉重开多少次，它一直在。

## 多项目并行的实际效率

Jim 有三个项目同时推进——不是他想并行，是现实就是这样的：

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

# 回到 api-server 的 tab，对话上下文完好无损
# 继续重构...
```

三个 worktree 同时存在，各自的 `node_modules`、IDE 窗口、AI 对话完全独立。切换成本从"stash → checkout → npm ci → 重开对话"变成了"换个 terminal tab"。

这不是多线程——人的大脑做不到多线程。这是**减少上下文切换的摩擦**，让你在不得不切换时尽可能少地丢失状态。

## 附件：完整脚本

> 脚本持续迭代中，文章内不再嵌入代码。最新版本见附件：[[attachments/wt.sh.attachment]] — 右键下载后 `mv wt.sh.attachment ~/.local/bin/wt && chmod +x ~/.local/bin/wt` 即可使用。确保 `~/.local/bin` 在 `PATH` 中。
>
> **不建议在文章中直接嵌入可执行脚本**。问题有三：(1) 代码演进而文章冻结，嵌入版本迅速过时；(2) 大段代码破坏文章阅读节奏，读者被迫跳过 200 行才到下一步；(3) 复制粘贴的代码没有可执行性保障。将脚本作为 `.sh.attachment` 附件引用，既保文章轻量，又保附件始终指向最新版本。

## Agent 实施指南

> 以下内容可以直接粘贴给任何 AI 编码助手，让它帮你完成部署。

---

请帮我安装一个名为 `wt` 的 CLI 工具。它的脚本在附件文件 `wt.sh.attachment` 中，功能说明在「wt 的设计」章节中。请：

1. 从附件 [[attachments/wt.sh.attachment]] 获取脚本内容，写入 `~/.local/bin/wt`
2. `chmod +x ~/.local/bin/wt`
3. 检查 `~/.local/bin` 是否在 `PATH` 中，如果不在则添加到 `~/.bashrc`（或对应的 shell 配置文件）
4. 执行 `wt --help` 验证安装成功
5. 在一个 git 仓库中测试 `wt -l` 确认无报错

附加配置（可选）：
- 脚本支持 `codex`、`claude`（`cc` 自动映射为 `claude`）、`kiro` 及任意 PATH 中的命令。如果你常用的 AI 工具是 `~/.bashrc` 中定义的 bash 函数（如 `codexd`、`gemini`），脚本已支持自动提取
- 如果希望 worktree 存放在其他位置，在 `~/.bashrc` 中设置 `export WT_BASE=~/your/path`
- 如果默认基准分支不是 main/master，设置 `export WT_DEFAULT_BASE=develop`

---

## 参考

- [Git Worktree 官方文档](https://git-scm.com/docs/git-worktree)
- [Script-as-Pipeline — Agent 管线中的脚本节点设计模式](https://nexus.sigmoid.cc/Script-as-Pipeline%20%E2%80%94%20Agent%20%E7%AE%A1%E7%BA%BF%E4%B8%AD%E7%9A%84%E8%84%9A%E6%9C%AC%E8%8A%82%E7%82%B9%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/)
