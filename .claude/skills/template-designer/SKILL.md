---
name: template-designer
description: |
  Promo-kit 模板设计引擎。用户描述视觉风格 → 引擎设计配色/字体/排版系统 → 生成对应 HTML 模板文件。
  支持小红书（1080×1440 / 750×1000）、掘金（1000×420）、小黑盒。可与 image-gen 联动配图。
  Use when user says "设计一套XX风格的模板", "create a template set in XX style",
  "帮我做一套配图模板", "换个风格做模板", or describes a visual direction for image templates.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
triggers:
  - 设计一套模板
  - 创建模板
  - 换个风格
  - 设计配图风格
  - create template
  - design template set
  - 做个XX风的模板
---

# template-designer — 模板设计引擎

promo-kit 的内部引擎。用户描述一个视觉风格，引擎设计完整的设计系统并输出 HTML 模板。

## 调用协议

由 promo-kit 调度器或用户直接通过 `Skill("template-designer", args)` 调用：

```
style: "风格描述（自然语言）"
platform: xhs|juejin|xiaoheihe|all
templates: "cover,pain,features,steps,cta,comparison"  (可选，不传则全部）
output_dir: "模板输出目录"  (可选，默认 templates/)
```

## 工作流

### Step 1: 解析风格意图

从用户输入中提取 / 追问确认：

| 维度 | 选项 | 来源 |
|------|------|------|
| 风格关键词 | 赛博朋克 / 极简白 / 暖复古 / 纸墨 / 杂志 / brutalist / ... | 用户描述 |
| 色调偏好 | 深色系 / 浅色系 / 用户指定 | 用户描述或 AskUserQuestion |
| 字体偏好 | 衬线 / 无衬线 / mono / 用户指定 | 用户描述或 AskUserQuestion |
| 目标平台 | xhs / juejin / xiaoheihe / all | AskUserQuestion (default: xhs) |
| 模板范围 | 全部 / cover-only / 用户指定 | AskUserQuestion (default: 全部) |

如果用户描述模糊（如"好看一点"、"专业一点"），追问澄清。

### Step 2: 设计系统

根据风格关键词推导完整设计系统：

**配色方案（5-8 色）：**
- 主背景色
- 卡片/次级背景色
- 标题文字色
- 正文文字色
- 辅助文字色
- 强调色 1（主 CTA）
- 强调色 2（高亮/编号）
- 正面/成功色（对比表）
- 负面/警示色（痛点）
- 边框/分割线色

**字体栈（2-3 套）：**
- 展示标题字体（封面大标题）
- 正文字体
- 等宽字体（代码/终端）

**排版规则：**
- 封面标题字号
- 页面标题字号
- 卡片标题字号
- 正文字号
- 安全区域

**纹理/装饰（可选）：**
- CSS 噪点纹理
- 网格线
- 渐变辉光
- 装饰线条
- 图案叠加

### Step 3: 生成模板

按平台规格生成 HTML 模板文件。每个模板注入对应的设计系统。

#### 小红书 (xhs) — 1080×1440 主模板组

| 文件名 | 用途 | 何时生成 |
|--------|------|---------|
| `xhs-cover-{slug}.html` | 封面 | 必选 |
| `xhs-pain-{slug}.html` | 痛点分析 | 按需 |
| `xhs-features-{slug}.html` | 功能亮点 | 按需 |
| `xhs-steps-{slug}.html` | 步骤教程 | 按需 |
| `xhs-cta-{slug}.html` | 行动号召 | 按需 |
| `xhs-comparison-{slug}.html` | 方案对比 | 按需 |

#### 通用辅助模板（750×1000，适用多平台）

| 文件名 | 用途 |
|--------|------|
| `comparison-{slug}.html` | 小尺寸对比图 |
| `steps-{slug}.html` | 小尺寸步骤 |
| `cta-card-{slug}.html` | 小尺寸 CTA |

#### 掘金 (juejin) — 1000×420

| 文件名 | 用途 |
|--------|------|
| `cover-juejin-{slug}.html` | 掘金封面 |

#### 终端/代码模板（1080×1440，深色系为主）

| 文件名 | 用途 |
|--------|------|
| `terminal-{slug}.html` | 终端截图风格 |
| `code-block-{slug}.html` | 代码块展示 |

`{slug}` 从风格关键词推导，如 `cyberpunk`、`warm-retro`、`paper-ink`。

### Step 4: 模板命名与冲突处理

1. 如果用户提供的风格名称与现有模板 slug 冲突，追加 `-v2` 或数字后缀
2. 生成后列出所有文件路径
3. 询问是否更新 `references/design-guide.md` 添加该风格的配色文档

### Step 5: 注册到 image-gen

生成后自动更新 `skills/image-gen/SKILL.md` 和 `.claude/skills/image-gen/SKILL.md`：

- 在模板表中新增生成的模板行
- 在自动选择规则中标注该风格的适用场景
- 在参数注入表中新增模板的参数映射

### Step 6: 输出摘要

```
=== 模板设计完成 ===

风格: {style_name}
配色: {primary_colors}
平台: {platforms}

生成模板 ({count} 个):
  {file_paths}

设计系统:
  配色见 references/design-guide.md 新增的 {style_name} 章节
  可在 image-gen 中通过 templates 参数指定使用

下一步:
  - 审查模板: 在浏览器中打开 HTML 文件预览
  - 测试配图: 调用 image-gen 并指定 templates="{template_names}"
  - 微调: "把{模板名}的{元素}改成{效果}"
```

---

## 设计系统参考库

### 已验证的配色方案

以下方案经过实际截图验证，可作为快速启动参考：

**Dark Editorial Tech（已内置）：**
- 背景 `#090A0C` / 标题 `#E4E4E6` / 强调 `#3BC9DB` `#845EF7`
- 衬线展示标题 + 无衬线正文 + mono 编号

**赛博朋克（参考）：**
- 背景 `#0D0221` / 卡片 `#15062E` / 标题 `#FF00FF` / 强调 `#00FFFF`
- 霓虹发光效果 + 网格线 + 等宽字体为主

**极简白（参考）：**
- 背景 `#FAFAF9` / 标题 `#171717` / 强调 `#2563EB`
- 无衬线字体 + 大留白 + 细分割线

**纸墨（参考）：**
- 背景 `#FDFBF7` / 标题 `#1A1A1A` / 强调 `#C2640D`
- 衬线标题 + 无衬线正文 + 边框装饰

**温暖复古（参考）：**
- 背景 `#FEF3C7` / 卡片 `#FDE68A` / 标题 `#78350F` / 强调 `#DC2626`
- 粗衬线标题 + 暖色调 + 复古纹理

---

## 模板结构规范

每个生成的 HTML 模板必须：

1. **内联所有 CSS** — 不依赖外部样式表
2. **使用 CSS 自定义属性不用** — 变量直接写值（headless browser 兼容性）
3. **不依赖外部字体** — 使用系统字体栈，可声明优先顺序
4. **不依赖 JavaScript** — 纯静态 HTML
5. **指定精确尺寸** — `width:1080px; height:1440px;` 或 `min-height:1440px;`
6. **预留占位符** — 使用 `{PLACEHOLDER_NAME}` 格式，与 image-gen 参数接口兼容
7. **保持 `<meta charset="UTF-8">`** — 确保中文渲染
8. **保持 `<meta name="viewport">`** — 确保正确缩放
