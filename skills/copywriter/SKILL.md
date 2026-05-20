---
name: copywriter
description: |
  Promo-kit 文案引擎。根据语言和平台生成推广文案，自动调用 humanizer/humanizer-zh 去除 AI 痕迹。
  支持中文（小红书/小黑盒/掘金）和英文（Twitter/LinkedIn/Reddit/Dev.to）。
  由 promo-kit 调度器内部调用，不直接对用户暴露。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Skill
  - AskUserQuestion
triggers: []
---

# copywriter — 文案引擎

promo-kit 的内部引擎。接收结构化参数，产出平台适配的推广文案。

## 调用协议

由 promo-kit 调度器通过 `Skill("copywriter", args)` 调用：

```
lang: zh|en
platform: xhs|xiaoheihe|juejin|twitter|linkedin|reddit|devto
angle: A|B|C|D|E|F
hook: "已写好的标题文案"
project_name: "项目名称"
project_summary: "一句话项目描述"
project_features: "功能亮点（bullet list）"
project_tech_stack: "技术栈"
output_dir: "输出目录绝对路径"
```

## 语言路由

| lang | 模板文件 | humanizer |
|------|---------|-----------|
| zh | `references/copywriter-cn.md` | humanizer-zh |
| en | `references/copywriter-en.md` | humanizer |

## 角度 → 叙事结构

| Angle | 中文 | English | 结构 |
|-------|------|---------|--------|
| A | 痛点解决 | Pain Point | 痛点场景 → 为什么以前解决不了 → 本方案如何解决 |
| B | 技术解析 | Technical | 技术背景 → 方案架构+关键实现 → 与替代方案区别 |
| C | 横向对比 | Comparison | 主流方案简述 → 逐项对比 → 适用场景建议 |
| D | 开源推荐 | Open Source | 为什么做 → 核心亮点+技术栈 → Star/PR 邀请 |
| E | 踩坑记录 | Pitfalls | 踩坑背景 → 逐坑(问题→原因→解决) → 经验总结 |
| F | 上手教程 | Tutorial | 目标读者 → 分步教程 → 常见问题 |

## 平台模板路由

### 中文平台

| platform | 模板（来自 copywriter-cn.md） |
|----------|------------------------------|
| xhs | 小红书：痛点解决型 / 教程型 / 开源推荐型 |
| xiaoheihe | 小黑盒：技术分享型 / 踩坑记录型 |
| juejin | 掘金：技术分享型 / 开源推荐型 |

### 英文平台

| platform | 模板（来自 copywriter-en.md） |
|----------|------------------------------|
| twitter | Thread / Single-post |
| linkedin | Article / Short-post |
| reddit | r/programming / Topic sub |
| devto | Tutorial / Opinion |

## 工作流

### Step 1: 加载模板

根据 `{lang}` 选择模板文件:
- zh → `references/copywriter-cn.md`
- en → `references/copywriter-en.md`

根据 `{platform}` 和 `{angle}` 在模板文件中匹配对应章节。

### Step 2: 生成文案

按照模板结构和角度叙事框架，填入项目信息生成完整文案。

生成规则：
- 标题使用传入的 `{hook}`，不做修改
- 正文按角度的叙事结构展开
- 平台格式特点严格遵守（中文：emoji、短段落；英文：hook-first、简洁）
- 小红书平台特殊约束：
  - 输出纯文本，禁止使用 markdown 代码块（```）。代码/命令用缩进或自然嵌入段落
  - 正文（标题+内容，不含末尾🏷️标签区）≤1000 字，超了必须裁切
- CTA 必须包含
- 标签/话题按模板规则添加

### Step 3: 保存文稿

```
{output_dir}/
  {platform}/
    {YYYY-MM-DD}-{angle}.md
```

### Step 4: 去 AI 味

根据语言选择 humanizer-zh 或 humanizer 对输出文件进行处理。

## 质量门禁

生成后自检（10 点）：

1. 标题长度 — 小红书 ≤20 字，掘金 ≤50 字，Twitter tweet 1 ≤280 字符
2. 前三秒吸引力 — 首句能否制造好奇或紧迫感
3. 真实感 — 读起来像开发者分享，不是产品说明书
4. 技术准确性 — 所有声称可验证，不夸大
5. CTA 存在 — 明确的下一步行动
6. 标签数量 — 小红书 8-10，掘金 1-5，Twitter 2-3，LinkedIn 3-5，Reddit 0
7. 配图标记 — 如需要配图，标明位置和类型（文案中的 [配图：...] 占位符由 image-gen 生成配图后自动清理）
8. humanizer 已执行 — 无 AI 写作痕迹
9. 小红书无 markdown — 正文不含 ``` 代码块、** 加粗、> 引用等 markdown 语法，纯文本输出
10. 小红书字数 — 正文（标题+内容，不含末尾🏷️标签区）≤1000 字

## 输出格式

```
文案输出:
  - promo-content/{project}/{platform}/{YYYY-MM-DD}-{angle}.md
  - 字数: {count}
  - humanizer: 已处理
```
