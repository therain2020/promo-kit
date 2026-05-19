# promo-kit

[English](README.md)

一个 Claude Code skill，帮你把开源项目从零推到中英文开发者社区。一次对话走完整条流水线。

## 它做什么

跟它说说你的项目。它自己找切入点，用中文或英文写文案，把那股 AI 味儿去掉，生成配图，然后发布。或者把文件给你，你自己来。

中文平台：小红书、小黑盒、掘金。
英文平台：Twitter/X、LinkedIn、Reddit、Dev.to。

## 流水线

```
你: "帮我把这个项目推广到小红书"
  → health-check: 检查所有引擎和依赖是否就绪
  → copywriter: 选叙事角度，按平台风格写文案
  → humanizer-zh / humanizer: 去掉 AI 写作痕迹
  → image-gen: 渲染 HTML 模板，headless 浏览器截图
  → publisher: 交给 agent-reach 执行发布
```

每一步都是一个独立引擎，有各自的 SKILL.md 在 `skills/` 下。你可以走全流程，也可以单独调用某个引擎。

## 平台和格式

| 平台 | 语言 | 产出 |
|------|------|------|
| 小红书 | 中文 | 带封面图、正文、话题标签的笔记 |
| 小黑盒 | 中文 | 带代码块、架构描述的技术帖 |
| 掘金 | 中文 | 带封面图、代码块、话题标签的技术文章 |
| Twitter/X | 英文 | 推文串或单条推文 |
| LinkedIn | 英文 | 长文或短文 |
| Reddit | 英文 | r/programming 或特定子版块的帖子 |
| Dev.to | 英文 | 教程或观点文章 |

## 六种叙事角度

文案引擎自动选，你也可以指定：

A：痛点。什么让你难受，为什么以前没人解决，这个方案怎么搞定的。
B：技术深挖。架构怎么设计的，关键决策是什么，跟替代方案比差在哪好在哪。
C：横向对比。跟现有工具逐项比较。
D：开源故事。为什么要做这个，亮点在哪，来一起搞。
E：踩坑记录。哪里翻车了，怎么爬出来的，学到了什么。
F：上手教程。适合谁看，一步步来，常见坑在哪。

## 安装

克隆仓库，然后把引擎链到你的项目里：

```bash
git clone https://github.com/therain2020/promo-kit.git

# 在你的项目目录下执行：
mkdir -p .claude/skills
ln -s "$(pwd)/../promo-kit/skills/copywriter" .claude/skills/copywriter
ln -s "$(pwd)/../promo-kit/skills/image-gen" .claude/skills/image-gen
ln -s "$(pwd)/../promo-kit/skills/publisher" .claude/skills/publisher
ln -s "$(pwd)/../promo-kit/skills/health-check" .claude/skills/health-check
```

Windows 上用复制代替软链：

```powershell
Copy-Item ../promo-kit/skills/* .claude/skills/ -Recurse
```

或者直接在 promo-kit 仓库里打开 Claude Code，根目录的 SKILL.md 会自动加载。

### 依赖

这些需要全局安装：

```bash
npx skills add therain2020/humanizer-zh -g -y    # 中文去 AI 味
npx skills add therain2020/humanizer -g -y       # 英文去 AI 味
npx skills add agent-reach -g -y                 # 多平台发布
```

gstack browse（image-gen 做截图用的）随 gstack 安装就带了。

在 Claude Code 里跑一下 `Skill("health-check")` 确认所有东西都接好了。

## 使用

```
/promo-kit
```

或者用自然语言：

> "帮我把这个 CLI 工具推广到小红书"

> "写个 Twitter 串介绍我的开源库"

skill 会按这几步走：
1. 搞清你的语言、平台和角度
2. 读你的项目（README、源码、配置）
3. 用 `references/` 里的模板起草文案
4. 通过 humanizer/humanizer-zh 去 AI 味
5. 需要配图的话生成配图
6. 问你要直接发布还是先保存文件

产出文件放在你项目目录下的 `promo-content/`。发布记录写入 `execution-log/`。

### 单独使用某个引擎

你可以直接调用：

```
/copywriter   — 只写文案。参数：lang, platform, angle, hook
/image-gen    — 只做配图。给它 markdown 文件，还你 PNG
/publisher    — 只做发布。文案 + 配图 → agent-reach
/health-check — 检查所有东西是否正常
```

## 目录结构

```
promo-kit/
├── SKILL.md                          # 调度器，流水线入口
├── skills/
│   ├── copywriter/SKILL.md           # lang: zh|en → 6 角度 × 6 平台
│   ├── image-gen/SKILL.md            # 8 套 HTML 模板 → headless 截图
│   ├── publisher/SKILL.md            # agent-reach 包装，含重试逻辑
│   └── health-check/SKILL.md         # 四级依赖检查
├── templates/                        # 配图 HTML 模板
│   ├── cover-xhs.html                # 小红书封面：经典红色渐变版
│   ├── xhs-cover-tech.html           # 小红书封面：科技极简版 (1080×1440)
│   ├── xhs-pain.html                 # 痛点分析卡片
│   ├── xhs-features.html             # 功能亮点卡片
│   ├── xhs-steps.html                # 步骤教程卡片（科技极简）
│   ├── xhs-cta.html                  # 行动号召卡片（深色科技极简）
│   ├── xhs-comparison.html           # 方案对比（科技极简）
│   ├── cover-juejin.html             # 掘金封面：标题 + 价值主张
│   ├── comparison.html               # 横向对比图（经典版）
│   ├── terminal.html                 # 终端风格 CLI 展示
│   ├── steps.html                    # 编号步骤卡片（经典版）
│   ├── phone-text.html               # 模拟手机消息
│   ├── cta-card.html                 # 行动号召卡片（经典版）
│   ├── architecture.html             # 组件关系图
│   └── code-block.html               # 语法高亮代码
├── references/
│   ├── copywriter-cn.md              # 中文模板（小红书、小黑盒、掘金）
│   ├── copywriter-en.md              # 英文模板（Twitter、LinkedIn、Reddit、Dev.to）
│   └── design-guide.md               # 配图设计规范
├── promo-content/                    # 产出文件（gitignored）
└── execution-log/                    # 发布记录（gitignored）
```

## 引擎之间怎么协作

调度器在引擎之间传递结构化参数。每个引擎读自己的输入，做一件事，把结果写到约定路径。下一个引擎接着读。

```
copywriter:   lang, platform, angle, hook → markdown 文件
humanizer:    markdown 文件 → 去 AI 味后的 markdown 文件
image-gen:    markdown 文件, platform → PNG 文件
publisher:    markdown 文件, PNG 文件, platform → agent-reach 调用
```

这意味着你可以随时换零件。把 image-gen 换成你自己的截图工具，跳过 publisher 手动发布。流水线不关心这些。

## 配图规格

image-gen 按以下分辨率渲染：

| 平台 | 尺寸 | 每篇张数 |
|------|------|---------|
| 小红书（封面/内容） | 1080 x 1440 | 3-6 张 |
| 小红书（辅助卡片） | 750 x 1000 | 3-6 张 |
| 小黑盒 | 750 x 1000 或 1200 x 800 | 2-4 张 |
| 掘金 | 1000 x 420（封面）+ 灵活尺寸 | 1-3 张 |

模板是纯 HTML 加内联 CSS。没有外部字体，不用 JS 框架。headless 浏览器直接从磁盘加载，2x 分辨率截图。
