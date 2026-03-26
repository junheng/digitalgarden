---
{"dg-publish":true,"permalink":"/01-projects/personal/digital-garden/","tags":["type/project"],"dg-note-properties":{"tags":["type/project"],"project_status":"进行中","priority":2,"start_date":"2026-03-26"}}
---

## 目标

- 使用 Obsidian Digital Garden 插件 + Vercel 搭建个人知识分享站点
- 选择性发布 Zettelkasten 永久笔记和技术资源

## 方案

- 插件：[Digital Garden](https://github.com/oleeskild/obsidian-digital-garden)
- 托管：Vercel（免费）
- 源码：GitHub 私有仓库
- 发布控制：frontmatter `dg-publish: true`

## 里程碑

| 里程碑 | 截止日期 | 状态 |
|--------|----------|------|
| 基础搭建（GitHub + Vercel + 插件） |  | 待办 |
| 首批笔记发布 |  | 待办 |
| 自定义主题和外观 |  | 待办 |
| 绑定自定义域名（可选） |  | 待办 |

## 任务


```base
filters:
  and:
    - file.hasTag("type/task")
    - '!file.inFolder("Templates")'
formulas:
  _score: |-
    (if(priority, priority * 2, 0) + 3 * if(due_date.isEmpty(), 0,
        if(today() == due_date, 1,
          if((today() - due_date).days > 0,
            2 + 0.2 * min((today() - due_date).days, 60),
            1 / ((due_date - today()).days + 1))))
    + if(scheduled_date.isEmpty(), 0,
        if(today() == scheduled_date, 2,
          if((today() - scheduled_date).days > 0,
            1 + 0.2 * min((today() - scheduled_date).days, 60),
            1 / ((scheduled_date - today()).days + 1))))).round(1)
  urgency: |-
    if(formula._score >= 8, "🔴 紧急",
      if(formula._score >= 4, "🟡 较急",
        if(formula._score >= 1, "🟢 一般", "⚪ 宽松")))
properties:
  file.name:
    displayName: 任务
  note.task_status:
    displayName: 状态
  note.priority:
    displayName: 优先级
  note.due_date:
    displayName: 截止日期
  note.scheduled_date:
    displayName: 计划日期
  note.related_project:
    displayName: 所属项目
  note.done_date:
    displayName: 完成日期
  formula.urgency:
    displayName: 紧急度
views:
  - type: table
    name: 全部任务
    order:
      - file.name
      - task_status
      - priority
      - formula.urgency
      - due_date
      - scheduled_date
      - related_project
    columnSize:
      file.name: 260
      note.task_status: 80
      note.priority: 70
      formula.urgency: 80
      note.due_date: 110
      note.scheduled_date: 110
      note.related_project: 150
  - type: table
    name: 今日任务
    filters:
      and:
        - task_status != "已完成"
        - or:
            - due_date == today()
            - scheduled_date == today()
            - due_date < today()
    order:
      - file.name
      - task_status
      - formula.urgency
      - due_date
    sort:
      - property: file.name
        direction: ASC
      - property: task_status
        direction: ASC
    columnSize:
      file.name: 300
      note.task_status: 80
      formula.urgency: 80
      note.due_date: 110
  - type: table
    name: 逾期任务
    filters:
      and:
        - task_status != "已完成"
        - "!due_date.isEmpty()"
        - due_date < today()
    order:
      - file.name
      - due_date
      - related_project
    columnSize:
      file.name: 300
      note.due_date: 110
      note.related_project: 150
  - type: table
    name: 本周任务
    filters: task_status != "已完成"
    order:
      - file.name
      - task_status
      - priority
      - formula.urgency
      - due_date
      - scheduled_date
      - related_project
    columnSize:
      file.name: 260
      note.task_status: 80
      note.priority: 70
      formula.urgency: 80
      note.due_date: 110
      note.scheduled_date: 110
      note.related_project: 150
  - type: table
    name: 本月已完成
    filters: task_status == "已完成"
    order:
      - file.name
      - done_date
      - related_project
    columnSize:
      file.name: 300
      note.done_date: 120
      note.related_project: 180

```


## 相关资源

- [Digital Garden 官方文档](https://dg-docs.ole.dev/)
- [Digital Garden GitHub](https://github.com/oleeskild/digitalgarden)
- [[06 - Zettelkasten/Zettelkasten 卡片盒笔记法 — 使用指南\|06 - Zettelkasten/Zettelkasten 卡片盒笔记法 — 使用指南]]

## 日志

### 2026-03-26

- 项目启动，确定使用 Digital Garden + Vercel 方案