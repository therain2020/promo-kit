---
name: publisher
description: |
  Promo-kit 发布引擎。薄层包装 agent-reach，提供发布前确认、失败重试、发布后记录。
  支持小红书、小黑盒、掘金、Twitter/X、LinkedIn、Reddit、Dev.to。
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
platform: xhs|xiaoheihe|juejin|twitter|linkedin|reddit|devto
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
| 掘金 (juejin) | `social 掘金` | 技术文章 |
| Dev.to (devto) | `dev github` | 通过 GitHub API |

## 工作流

发布策略：**优先全自动。** 全自动失败才降级为帮助用户手动发布。

### Step 1: 发布前确认

读取文案文件，提取前 200 字作为预览。调用 AskUserQuestion：

> 确认发布到 {platform}？
>
> 预览:
> {首 200 字}
>
> 配图: {N} 张

选项：
- A) 确认发布
- B) 只看不发布（保存内容，手动发）
- C) 取消

### Step 2: 转换格式并全自动发布

将文案 md 转换为 agent-reach 命令格式，优先尝试全自动发布：

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

**掘金:**
```
Skill("agent-reach", args="在掘金发布一篇技术文章：标题：{title}，正文：{content}，封面图：{cover_image}")
```

### Step 3: 失败处理 — 记录原因 + 降级

若全自动发布失败：

1. **记录尝试信息到 execution-log：**
   ```
   execution-log/{project}/{YYYY-MM-DD}-{platform}.md
   
   尝试方案: {调用的 agent-reach 命令 / CLI 工具}
   失败原因: {具体错误信息，如 "xhs-cli POST 返回 406" / "登录态失效" / "网络超时"}
   降级: 手动发布
   ```

2. **降级为手动发布 — 打开三类资源辅助用户：**
   - PowerShell `Start-Process "<平台创作中心URL>"` — 在系统默认浏览器打开发布页
   - `explorer <图片文件夹>` — 打开配图所在文件夹
   - `notepad <文案 .md 路径>` — 打开文案文件

   平台创作中心 URL：
   | 平台 | URL |
   |------|-----|
   | 小红书 | `https://creator.xiaohongshu.com/` |
   | 掘金 | `https://juejin.cn/editor/drafts/` |
   | 小黑盒 | 通过 App 发布，无 Web 创作中心 |

   告知用户三步操作：粘贴正文 → 上传配图 → 加标签 → 发布。

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
| CLI 不支持 POST/发布 | 1. PowerShell: `Start-Process "<创作者URL>"` 在系统浏览器打开 2. `explorer` 打开产出文件夹 3. `notepad` 打开文案 .md 4. 告知用户手动操作 |
| 图片上传失败 | 降级为纯文本发布（AskUserQuestion 确认） |
