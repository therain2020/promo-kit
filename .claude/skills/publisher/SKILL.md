---
name: publisher
description: |
  Promo-kit 发布引擎。薄层包装 agent-reach，提供发布前确认、失败重试、发布后记录。
  支持小红书、小黑盒、Twitter/X、LinkedIn、Reddit、Dev.to。
  由 promo-kit 调度器内部调用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Skill
  - AskUserQuestion
triggers: []
---

# publisher — 发布引擎

promo-kit 的内部引擎。将文案和配图转换为 agent-reach 调用，执行发布并记录结果。

## 调用协议

由 promo-kit 调度器通过 `Skill("publisher", args)` 调用：

```
platform: xhs|xiaoheihe|twitter|linkedin|reddit|devto
content_path: "文案 md 文件绝对路径"
images: "图片路径列表"  (可选，逗号分隔)
project_name: "项目名称"
```

## 平台 → agent-reach 路由

| 平台 | agent-reach 路由 | 发布模式 |
|------|-----------------|---------|
| 小红书 (xhs) | `social 小红书` | 图文笔记 |
| 小黑盒 (xiaoheihe) | `social 小黑盒` | 图文帖子 |
| Twitter/X (twitter) | `social 推特` | tweet/thread |
| LinkedIn (linkedin) | `career LinkedIn` | post/article |
| Reddit (reddit) | `social Reddit` | text post |
| Dev.to (devto) | `dev github` | 通过 GitHub API |

## 工作流

### Step 1: 发布前确认

读取文案文件，提取前 200 字作为预览。调用 AskUserQuestion 确认发布。

### Step 2: 转换格式

将文案 md 转换为 agent-reach 命令格式：

**小红书:**
```
Skill("agent-reach", args="用小红书发布一篇笔记，标题：{hook}，正文：{content}，配图：{image_paths}")
```

**Twitter/X:**
```
Skill("agent-reach", args="发一条推文/thread：{content}")
```

**LinkedIn:**
```
Skill("agent-reach", args="在 LinkedIn 发一篇帖子：{content}")
```

**Reddit:**
```
Skill("agent-reach", args="在 Reddit r/{subreddit} 发帖：标题：{title}，正文：{content}")
```

**小黑盒:**
```
Skill("agent-reach", args="在小黑盒发一篇帖子：标题：{title}，正文：{content}")
```

### Step 3: 执行发布

调用 agent-reach，捕获返回结果。

### Step 4: 失败重试

- 最多重试 3 次
- 每次重试间隔 30 秒
- 记录每次失败原因
- 3 次都失败 → 报告失败原因，保存内容到本地待手动发布

### Step 5: 记录结果

```
execution-log/{project}/{YYYY-MM-DD}-{platform}.md

# 发布记录

- 时间: {datetime}
- 平台: {platform}
- 内容: {content_path}
- 状态: success|failed
- 链接: {post_url} (如 agent-reach 返回)
- 备注: {notes}
```

## agent-reach 前置检查

发布前验证：
1. agent-reach skill 可用（检查 Skill 列表）
2. 目标平台登录态有效（通过 agent-reach 的健康检查命令）

如登录态失效，提示用户手动登录后重试。

## 错误处理

| 错误类型 | 处理 |
|---------|------|
| agent-reach 不可用 | 保存内容到本地，提示 `npx skills add agent-reach` |
| 登录态失效 | 提示用户重新登录目标平台 |
| 网络超时 | 自动重试，超过 3 次标记失败 |
| 平台拒绝（内容违规） | 记录原因，建议修改后重试 |
| 图片上传失败 | 降级为纯文本发布（AskUserQuestion 确认） |
