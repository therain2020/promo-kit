# promo-kit

[English](README.md)

一个 Claude Code skill，给开源项目做推广。给它一个仓库，它自己找角度、写文案、去掉 AI 味、做配图、发出去。或者把文件给你，你自己发。

中文平台：小红书、小黑盒、掘金。
英文平台：Twitter/X、LinkedIn、Reddit、Dev.to。

## 流水线

```
你: "帮我把这个 CLI 工具推广到小红书"
  → health-check: 检查 5 个引擎和外部依赖是否就绪
  → copywriter: 选叙事角度，按平台风格写文案
  → humanizer-zh / humanizer: 去掉英文 29 种或中文 24 种 AI 写作痕迹
  → image-gen: 填充 HTML 模板，headless 浏览器（$B）截图
  → publisher: 交给 agent-reach 执行发布
```

每个引擎独立维护，各自在 `skills/` 下有 SKILL.md。走全流程也行，单独调某个引擎也行。

## 平台和格式

| 平台 | 语言 | 产出 |
|------|------|------|
| 小红书 | 中文 | 带封面图、正文、话题标签的笔记。3-6 张配图，1080×1440 或 750×1000 |
| 小黑盒 | 中文 | 带代码块、架构描述的技术帖 |
| 掘金 | 中文 | 带封面图、代码块、话题标签的技术文章 |
| Twitter/X | 英文 | 推文串或单条推文 |
| LinkedIn | 英文 | 长文或短文 |
| Reddit | 英文 | r/programming 或特定子版块的帖子 |
| Dev.to | 英文 | 教程或观点文章 |

## 六种叙事角度

文案引擎自动选，你也可以指定：

| 角度 | 中文 | 英文 | 结构 |
|------|------|------|------|
| A | 痛点解决 | Pain Point | 痛点场景 → 为什么没人解决 → 我的方案怎么搞定的 |
| B | 技术解析 | Technical | 技术背景 → 架构和关键决策 → 与替代方案的区别 |
| C | 横向对比 | Comparison | 方案概述 → 逐项对比 → 适用场景建议 |
| D | 开源推荐 | Open Source | 为什么做 → 亮点和技术栈 → Star/PR 邀请 |
| E | 踩坑记录 | Pitfalls | 背景 → 逐坑（问题 → 原因 → 解决）→ 经验总结 |
| F | 上手教程 | Tutorial | 目标读者 → 分步教程 → 常见问题 |

## 安装

克隆仓库，然后链到你的项目里：

```bash
git clone https://github.com/therain2020/promo-kit.git

# 在你的项目目录下执行：
mkdir -p .claude/skills
ln -s "$(pwd)/../promo-kit/skills/copywriter" .claude/skills/copywriter
ln -s "$(pwd)/../promo-kit/skills/image-gen" .claude/skills/image-gen
ln -s "$(pwd)/../promo-kit/skills/template-designer" .claude/skills/template-designer
ln -s "$(pwd)/../promo-kit/skills/publisher" .claude/skills/publisher
ln -s "$(pwd)/../promo-kit/skills/health-check" .claude/skills/health-check
```

Windows 用复制代替软链：

```powershell
Copy-Item ../promo-kit/skills/* .claude/skills/ -Recurse
```

或者直接在 promo-kit 仓库里打开 Claude Code，根目录的 SKILL.md 会自动加载。

### 依赖

全局安装这些：

```bash
npx skills add therain2020/humanizer-zh -g -y    # 中文去 AI 味
npx skills add therain2020/humanizer -g -y       # 英文去 AI 味
npx skills add agent-reach -g -y                 # 多平台发布
```

gstack browse（image-gen 截图用的）随 gstack 安装就带了。

在 Claude Code 里跑 `Skill("health-check")` 确认所有东西都接好了。

## 使用

```
/promo-kit
```

或者自然语言：

> "帮我把这个 CLI 工具推广到小红书"

> "写个 Twitter 串介绍我的开源库"

> "在掘金上发一篇介绍我项目的技术文章"

skill 会按这几步走：
1. 搞清语言、平台、角度
2. 读你的项目（README、源码、配置）
3. 用 `references/` 里的模板起草文案
4. 通过 humanizer 或 humanizer-zh 去 AI 味
5. 需要配图的话生成配图
6. 问你要直接发布还是先保存文件

产出文件放在你项目目录下的 `promo-content/`。发布记录写入 `execution-log/`。

### 单独使用某个引擎

```
/copywriter         — 只写文案。参数：lang, platform, angle, hook
/image-gen          — 只做配图。给它 markdown 文件，还你 PNG
/template-designer  — 根据风格描述生成模板
/publisher          — 只做发布。文案 + 配图 → agent-reach
/health-check       — 检查所有东西是否正常
```

## 引擎

| 引擎 | 做什么 | 依赖 |
|------|--------|------|
| promo-kit（调度器） | 编排完整的 9 步工作流 | 下面所有引擎 |
| copywriter | 语言：中→英，6 角度 × 7 平台 → markdown | humanizer-zh、humanizer |
| image-gen | markdown → HTML 模板 → $B 截图 → PNG | gstack browse daemon |
| template-designer | 风格描述 → 设计系统 → HTML 模板 | — |
| publisher | markdown + 图片 → agent-reach → 发布 | agent-reach |
| health-check | 4 级验证：引擎、模板、依赖、产出 | — |

## 配图规格

模板采用 Dark Editorial Tech 设计系统：深色背景、衬线展示标题、霓虹强调色、CSS 噪点纹理。

| 平台 | 尺寸 | 每篇 | 风格 |
|------|------|------|------|
| 小红书（主模板） | 1080 × 1440 | 3-6 张 | 深色编辑风 |
| 小红书（辅助卡片） | 750 × 1000 | 3-6 张 | 深色编辑风 |
| 小黑盒 | 750 × 1000 或 1200 × 800 | 2-4 张 | 深色编辑风或自定义 |
| 掘金 | 1000 × 420（封面）+ 灵活尺寸 | 1-3 张 | 深色编辑风 |

共 15 个 HTML 模板：cover-xhs、xhs-cover-tech、xhs-pain、xhs-features、xhs-steps、xhs-cta、xhs-comparison、terminal、code-block、cover-juejin、comparison、steps、phone-text、cta-card、architecture。

## 目录结构

```
promo-kit/
├── SKILL.md                          # 调度器，9 步工作流入口
├── skills/
│   ├── copywriter/SKILL.md           # zh|en → 6 角度 × 7 平台
│   ├── image-gen/SKILL.md            # 15 个 HTML 模板 → headless 截图
│   ├── template-designer/SKILL.md    # 风格 → 设计系统 → HTML 模板
│   ├── publisher/SKILL.md            # agent-reach 包装，含重试和日志
│   └── health-check/SKILL.md         # 4 级依赖验证
├── templates/                        # 15 个 HTML 配图模板（Dark Editorial Tech）
├── references/
│   ├── copywriter-cn.md              # 中文模板（小红书、小黑盒、掘金）
│   ├── copywriter-en.md              # 英文模板（Twitter、LinkedIn、Reddit、Dev.to）
│   └── design-guide.md               # 配色、字体、排版规范
├── promo-content/                    # 产出文件（gitignored）
└── execution-log/                    # 发布记录（gitignored）
```

## 引擎之间怎么协作

调度器在引擎之间传递结构化参数。每个引擎读自己的输入，做一件事，把结果写到约定路径，下一个引擎接着读。

```
copywriter:        lang, platform, angle, hook → markdown 文件
humanizer:         markdown 文件 → 去 AI 味后的 markdown 文件
image-gen:         markdown 文件, platform → PNG 文件
publisher:         markdown 文件, PNG 文件, platform → agent-reach 调用
```

中间任意环节都可以换。把 image-gen 换成你自己的截图工具，跳过 publisher 手动发。流水线不关心这些。

## 设计系统

模板采用 Dark Editorial Tech：深黑底 `#090A0C`、近白文字 `#E4E4E6`、衬线展示标题（Georgia + 思源宋体），封面字号 96-108px。霓虹青 `#3BC9DB` 和紫色 `#845EF7` 作为强调色，琥珀 `#FCC419` 做高亮。CSS-only SVG 噪点纹理 4% 透明度叠加。

template-designer 引擎可以根据风格描述生成新的模板套组。试试对 Claude 说："设计一套赛博朋克风格的模板"或"做个温暖复古风的配图模板"。

## License

MIT
