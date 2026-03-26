---
{"dg-publish":true,"permalink":"/01-projects/personal/digital-garden/tasks/digital-garden/","tags":["type/task"],"dg-note-properties":{"tags":["type/task"],"task_status":"已完成","priority":1,"related_project":"[[Digital Garden 搭建]]"}}
---

## 描述

完成 Digital Garden 插件 + Vercel 的初始搭建。

## 步骤

### 1. GitHub 准备

- 打开 [Digital Garden 模板仓库](https://github.com/oleeskild/digitalgarden)
- 点击 "Deploy to Vercel" 按钮
- Vercel 会自动 fork 仓库到你的 GitHub 账号，命名建议：`digital-garden`

### 2. Vercel 部署

- 用 GitHub 账号登录 Vercel（如果没有账号会自动注册）
- 跟着 Vercel 引导完成部署
- 部署完成后记下站点 URL（形如 `https://digital-garden-xxx.vercel.app`）

### 3. 生成 GitHub Access Token

- 打开 [GitHub Token 页面](https://github.com/settings/tokens/new?scopes=repo)
- Expiration 选 "No expiration"（或按安全偏好选择）
- 点 "Generate token"，复制 token（只显示一次！）
- key -  `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 4. 安装和配置插件

- Obsidian → Settings → Community Plugins → 搜索 "Digital Garden" → 安装并启用
- 插件设置中填入：
  - GitHub Username：你的 GitHub 用户名
  - GitHub Repo Name：`digital-garden`（或你在第 1 步取的名字）
  - GitHub Token：粘贴第 3 步的 token

### 5. 发布测试笔记

- 给任意一篇笔记添加 frontmatter：
  ```yaml
  dg-publish: true
  dg-home: true  # 仅首页笔记需要
  ```
- 打开命令面板（Ctrl+P）→ "Digital Garden: Publish Single Note"
- 等待 1-2 分钟，访问 Vercel URL 确认显示

## 备注

- 整个过程约 15-30 分钟
- 需要 GitHub 账号（免费）
- Vercel 免费额度：100GB 带宽/月，个人站完全够用