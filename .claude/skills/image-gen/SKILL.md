---
name: image-gen
description: |
  Promo-kit 配图引擎。读取文案内容，用 HTML 模板渲染后通过 headless 浏览器截图输出 PNG。
  支持小红书（1080×1440 / 750×1000）、小黑盒（灵活比例）和掘金（1000×420 封面）。由 promo-kit 调度器内部调用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Skill
triggers: []
---

# image-gen — 配图引擎

promo-kit 的内部引擎。读取文案 → 选模板 → HTML 渲染 → $B 截图 → PNG。

## 调用协议

由 promo-kit 调度器通过 `Skill("image-gen", args)` 调用：

```
content_path: "文案 md 文件绝对路径"
platform: xhs|xiaoheihe|juejin
templates: "cover,terminal,cta-card"  (逗号分隔，可选，不传则自动选择)
output_dir: "图片输出目录绝对路径"
```

## 平台规格

| 平台 | 尺寸 | 张数/篇 |
|------|------|---------|
| 小红书（封面/内容） | 1080×1440 | 3-6 张 |
| 小红书（辅助卡片） | 750×1000 | 2-4 张 |
| 小黑盒 | 750×1000 或 1200×800 | 2-4 张 |
| 掘金 | 1000×420（封面），灵活（正文） | 1-3 张 |

## 可用模板

### 主模板（1080×1440，Dark Editorial）

| 模板名 | 用途 | HTML 文件 |
|--------|------|----------|
| cover-xhs | 小红书封面 | templates/cover-xhs.html |
| xhs-cover-tech | 小红书封面（备用布局） | templates/xhs-cover-tech.html |
| xhs-pain | 痛点分析 | templates/xhs-pain.html |
| xhs-features | 功能亮点 | templates/xhs-features.html |
| xhs-steps | 步骤教程 | templates/xhs-steps.html |
| xhs-cta | 行动号召 | templates/xhs-cta.html |
| xhs-comparison | 方案对比 | templates/xhs-comparison.html |
| terminal | 终端截图 | templates/terminal.html |
| code-block | 代码块展示 | templates/code-block.html |

### 辅助模板（750×1000，同设计系统）

| 模板名 | 用途 | HTML 文件 |
|--------|------|----------|
| comparison | 方案对比图 | templates/comparison.html |
| steps | 步骤指南 | templates/steps.html |
| phone-text | 手机文案模拟 | templates/phone-text.html |
| cta-card | CTA 卡片 | templates/cta-card.html |
| architecture | 架构图 | templates/architecture.html |

### 其他平台

| 模板名 | 用途 | HTML 文件 |
|--------|------|----------|
| cover-juejin | 掘金封面（1000×420） | templates/cover-juejin.html |

## 自动模板选择规则

| 内容特征 | 推荐模板 |
|---------|---------|
| 必选（封面） | cover-xhs 或 xhs-cover-tech |
| 有痛点描述 | xhs-pain |
| 有功能亮点 | xhs-features |
| 有分步教程 | xhs-steps |
| 有 CLI/安装命令 | terminal |
| 有对比表 | xhs-comparison（1080w）或 comparison（750w） |
| 有架构描述 | architecture |
| 有代码片段 | code-block |
| 末尾 CTA | xhs-cta 或 cta-card |
| 掘金必选（封面） | cover-juejin |

小红书每篇 3-6 张：封面 + 痛点/功能 + 教程/终端 + 对比 + CTA。

## 工作流

### Step 1: 解析文案

Read `{content_path}`，提取：
- 标题/Hook
- 项目名称
- 核心卖点（3-5 个 bullet）
- 安装命令或 CLI 输出
- 对比表（如有）
- 教程步骤（如有）
- CTA 文案 + URL
- 代码片段（如有）

### Step 2: 选取模板

如已传 `templates` 参数，使用指定列表。否则按自动规则选择。

### Step 3: 渲染 HTML

每个模板构造对应参数注入 HTML：

```
cover-xhs: {TITLE}, {PROJECT}, {TAGLINE}, {EMOJI}
cover-juejin: {TITLE}, {PROJECT}, {TAGLINE}, {EMOJI}
xhs-cover-tech: {TAG}, {TITLE}, {SUBTITLE}, {ICONS}, {PROJECT}
xhs-pain: {SECTION_NUM}, {EMOJI}, {HEADLINE}, {DESC}, {PAIN_ITEMS}
xhs-features: {SECTION_NUM}, {HEADLINE}, {FEATURES}
xhs-steps: {SECTION_NUM}, {HEADLINE}, {STEPS}
xhs-cta: {STARS}, {HEADLINE}, {SUBTITLE}, {BTN_TEXT}, {URL}, {FEATURES}
xhs-comparison: {SECTION_NUM}, {HEADLINE}, {ROWS}, {NOTE}
comparison: {ROWS}, {DATE}
terminal: {TERMINAL_TITLE}, {LINES}
steps: {SECTION_TITLE}, {STEPS}
phone-text: {CONTEXT}, {MESSAGES}, {CAPTION}
cta-card: {PROJECT}, {STARS}, {TAGLINE}, {CTA_TEXT}, {FEATURES}, {URL}
architecture: {DIAGRAM_TITLE}, {BLOCKS}
code-block: {FILENAME}, {CODE}
```

读取模板 HTML，用 Read → Edit 做字符串替换注入内容。

### Step 4: $B 截图

定位 browse binary 后用 `$B goto` 渲染后的 HTML → `$B screenshot {output_dir}/NN-name.png`

### Step 5: 输出

```
{output_dir}/
  01-cover.png
  02-terminal.png
  03-cta.png
  ...
```

## 质量检查

每张图生成后检查：
1. 文字在安全区域内（上下 40px margin）
2. 对比度 ≥ 4.5:1
3. 信息密度 ≤ 3 个信息点
4. emoji 正确渲染
5. 代码块可读（等宽字体、语法高亮）
6. PNG 分辨率正确（1080×1440 或 750×1000，按模板类型）

## 设计规范

详见 `references/design-guide.md`。
