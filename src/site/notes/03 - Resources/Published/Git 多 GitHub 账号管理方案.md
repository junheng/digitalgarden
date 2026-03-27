---
{"dg-publish":true,"dg-path":"Git 多 GitHub 账号管理方案.md","permalink":"/git-git-hub/","tags":["type/reference","tech/git","tech/devops"],"dg-note-properties":{"tags":["type/reference","tech/git","tech/devops"],"area":null,"aliases":[],"created":"2026-03-25"}}
---


## 场景

多个 GitHub 账号需要在不同项目中自动切换身份和凭据，无需手动操作。

## 当前配置

| 目录 | GitHub 账号 | SSH Host 别名 |
|------|------------|---------------|
| `~/Projects/work/` | work-account | github-work |
| `~/Projects/opensource/` | personal-account | github-personal |
| `~/Projects/personal/` | personal-account | github-personal |

## 方案：SSH Key 别名 + includeIf + insteadOf

一次配置，零维护。现有 HTTPS clone 的项目无需修改 remote URL。

## 1. SSH Key

```bash
ssh-keygen -t ed25519 -C "work-account" -f ~/.ssh/id_work
ssh-keygen -t ed25519 -C "personal-account" -f ~/.ssh/id_personal
```

通过 `gh` 添加到对应账号：

```bash
# 确保两个账号都已登录
gh auth login -h github.com  # 登录第一个账号
gh auth login -h github.com  # 登录第二个账号（gh 支持多账号）

# 个人账号（当前活跃）
gh auth switch -u personal-account
gh auth refresh -h github.com -s admin:public_key  # 添加 SSH key 管理权限
gh ssh-key add ~/.ssh/id_personal.pub --title "WSL-personal-ed25519"

# 公司账号
gh auth switch -u work-account
gh auth refresh -h github.com -s admin:public_key
gh ssh-key add ~/.ssh/id_work.pub --title "WSL-work-ed25519"

# 切回日常使用的账号
gh auth switch -u personal-account
```

> [!tip] `gh auth refresh -s admin:public_key` 会打开浏览器授权，每个账号只需做一次。授权后 `gh ssh-key add` 即可直接使用。

## 2. SSH Config

`~/.ssh/config`（仅 GitHub 相关部分）：

```
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_work
  IdentitiesOnly yes

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_personal
  IdentitiesOnly yes
```

`IdentitiesOnly yes` 防止 SSH agent 发送错误的 key。

## 3. Git includeIf

`~/.gitconfig`：

```ini
[user]
    name = your-name
    email = your-email@example.com

[includeIf "gitdir:~/Projects/work/"]
    path = ~/.gitconfig-work

[includeIf "gitdir:~/Projects/opensource/"]
    path = ~/.gitconfig-personal

[includeIf "gitdir:~/Projects/personal/"]
    path = ~/.gitconfig-personal
```

> [!warning] `gitdir:` 路径末尾的 `/` 表示递归匹配子目录，不要漏掉。

## 4. 子配置文件（url.insteadOf）

`~/.gitconfig-work`：

```ini
[url "git@github-work:"]
    insteadOf = git@github.com:
    insteadOf = https://github.com/
```

`~/.gitconfig-personal`：

```ini
[url "git@github-personal:"]
    insteadOf = git@github.com:
    insteadOf = https://github.com/
```

`insteadOf` 同时匹配 SSH 和 HTTPS URL，已有项目的 `.git/config` 中 remote 不需要改动。
[^1]
## 5. 验证

```bash
# SSH 连接
ssh -T git@github-work        # → Hi work-account!
ssh -T git@github-personal     # → Hi personal-account!

# Git 身份（进入对应目录）
git config user.email

# 实际 fetch
cd ~/Projects/work/some-repo && git fetch
cd ~/Projects/opensource/some-repo && git fetch
```

## 6. 防火墙封 SSH 22 端口

改用 443 端口：

```
Host github-work
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_work
  IdentitiesOnly yes
```

## 方案对比

| 方式 | 多账号支持 | 维护成本 | 备注 |
|------|-----------|---------|------|
| SSH key 别名 | ✅ 原生 | 低 | 推荐，一次配置零维护 |
| HTTPS + GCM | ✅ | 中 | Linux 上需额外配置 |
| HTTPS + PAT | 需 useHttpPath | 高 | token 过期需手动刷新 |
| gh CLI | 单账号为主 | 高 | 多账号支持差 |

## 新增账号

1. 生成新 key：`ssh-keygen -t ed25519 -f ~/.ssh/id_xxx`
2. `~/.ssh/config` 添加 Host 别名
3. `~/.gitconfig` 添加 `includeIf` 条目
4. 创建对应 `.gitconfig-xxx`，写 `url.insteadOf`
5. `gh ssh-key add` 或手动添加 public key 到 GitHub

## 相关链接

- [[03 - Resources/Tech/Git 与代理配置速查\|Git 与代理配置速查]]
- [[03 - Resources/Tech/设置 git 使用代理\|设置 git 使用代理]]

[^1]: 
