---
name: promo-kit
description: |
  开源项目推广全链路工具包。覆盖：项目分析 → 文案生成(中/英) → AI痕迹去除 → 配图制作 → 多平台发布。
  中文：小红书/小黑盒。English: Twitter/LinkedIn/Reddit/Dev.to。
  Use when asked to "推广我的项目", "写一篇小红书", "发推特推广", "promote my project on social media",
  "make a Twitter thread for my repo", "在小黑盒宣传", or any open-source promotion request.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
  - Skill
  - AskUserQuestion
triggers:
  - 推广我的项目
  - 写一篇小红书
  - 发个推特推广
  - 在小黑盒宣传
  - 帮我做推广
  - promote my project
  - promote on social media
  - write a Twitter thread for my repo
  - 帮我写个推广文案
---

# /promo-kit — 开源项目推广全链路工具包

一键推广：分析项目 → 写文案 → 去 AI 味 → 做配图 → 发布。

## 引擎架构

```
promo-kit (调度器)
  ├── health-check     # 启动前健康检查
  ├── copywriter       # 文案引擎 (zh/en)
  ├── humanizer        # 外部: 去 AI 味 (humanizer-zh / humanizer)
  ├── image-gen        # 配图引擎 (HTML→$B→PNG)
  └── publisher        # 发布引擎 (薄层包装 agent-reach)
```

每个引擎独立维护，SKILL.md 在 `skills/<name>/SKILL.md`。

---

## 工作流

### Step 0: 健康检查

启动前调用 `Skill("health-check", args="check_level: quick")`。

- ALL_OK → 继续
- DEGRADED → 提示用户，确认后继续
- BLOCKED → 停止，报告具体失败项

### Step 1: 解析意图

从用户输入中提取：

| 维度 | 选项 | 默认 |
|------|------|------|
| 语言 | zh / en | zh |
| 平台 | xhs / xiaoheihe / twitter / linkedin / reddit / devto | xhs |
| 角度 | A-F 或 auto | auto（自动选择） |
| 需要配图 | yes / no | zh 平台 yes, en 平台 no |
| 需要发布 | yes / no | no（先问用户） |

如果用户未指定平台和语言，使用 AskUserQuestion 确认。

### Step 2: 项目分析

如果当前目录是项目仓库，使用 Explore subagent 自动分析：

```
提取:
- 一句话价值主张
- 核心功能 (3-5 个 bullet)
- 技术栈亮点
- 安装/使用方式
- 目标用户画像
```

如果用户指定了外部项目路径，读取该路径的 README 或代码。

### Step 3: 选择角度

如用户未指定角度，根据项目特征自动匹配：

| 项目特征 | 推荐角度 |
|---------|---------|
| 新项目首次推广 | D 开源推荐 |
| 有明确痛点场景 | A 痛点解决 |
| 技术架构独特 | B 技术解析 |
| 有已知竞品 | C 横向对比 |
| 踩过明显的坑 | E 踩坑记录 |
| 上手门槛高 | F 上手教程 |

### Step 4: 生成 Hook

根据角度和项目信息，构思 3 个备选 Hook 标题。用 AskUserQuestion 让用户选择或自定义。

### Step 5: 调用 copywriter

```
Skill("copywriter", args="
lang: {zh|en}
platform: {platform}
angle: {angle}
hook: {选定的标题}
project_name: {项目名}
project_summary: {一句话描述}
project_features: {功能亮点}
project_tech_stack: {技术栈}
output_dir: {输出目录}
")
```

### Step 6: 去 AI 味

copywriter 内部已自动调用 humanizer。调度器确认执行完毕。

### Step 7: 配图（可选）

仅中文平台或用户要求时执行：

```
Skill("image-gen", args="
content_path: {Step 5 输出的文案路径}
platform: {platform}
output_dir: {文案同目录}/images/
")
```

### Step 8: 发布（可选）

询问用户是否发布。如确认：

```
Skill("publisher", args="
platform: {platform}
content_path: {文案路径}
images: {配图路径列表}
project_name: {项目名}
")
```

### Step 9: 输出摘要

```
=== 推广任务完成 ===

项目: {project_name}
平台: {platform}
语言: {zh/en}

文稿: {content_path}
配图: {N} 张 → {images_dir}
发布: {已发布 / 已保存待手动发布}

下一步:
  - 审核文稿: {content_path}
  - 发布: 如未自动发布，可手动或重新运行 /promo-kit
```

---

## 增量模式

如果用户说"再写一篇"或基于已有项目继续：

1. 跳过 Step 0-4（健康检查仍执行）
2. 复用已有的项目分析信息
3. 换个角度或平台，从 Step 5 继续

---

## 输出目录结构

```
promo-content/
  {project}/
    {platform}/
      {YYYY-MM-DD}-{angle}.md
      images/
        01-cover.png
        02-terminal.png
        03-cta.png

execution-log/
  {project}/
    {YYYY-MM-DD}-{platform}.md
```
