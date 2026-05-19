# promo-kit

开源项目推广全链路工具包 — Claude Code skill。

**一条命令完成：** 项目分析 → 文案生成 → AI 痕迹去除 → 配图制作 → 多平台发布。

## 覆盖平台

| 语言 | 平台 | 内容类型 |
|------|------|---------|
| 中文 | 小红书 | 图文笔记 |
| 中文 | 小黑盒 | 技术帖子 |
| English | Twitter/X | Thread / Single post |
| English | LinkedIn | Article / Short post |
| English | Reddit | r/programming showcase |
| English | Dev.to | Technical tutorial |

## 架构

```
promo-kit (统一调度)
  ├── copywriter    # 文案引擎 — lang:zh/en 路由，6 种叙事角度
  ├── image-gen     # 配图引擎 — 8 种 HTML 模板 + headless 截图
  ├── publisher     # 发布引擎 — 薄层包装 agent-reach
  └── health-check  # 健康检查 — 4 级检查项
```

## 安装

```bash
# 克隆仓库
git clone https://github.com/therain2020/promo-kit.git

# 链入 Claude Code（在目标项目目录下执行）
mkdir -p .claude/skills
ln -s /path/to/promo-kit/skills/copywriter .claude/skills/copywriter
ln -s /path/to/promo-kit/skills/image-gen .claude/skills/image-gen
ln -s /path/to/promo-kit/skills/publisher .claude/skills/publisher
ln -s /path/to/promo-kit/skills/health-check .claude/skills/health-check
```

或者直接在这个仓库里打开 Claude Code，所有子 skill 自动加载。

## 使用

在 Claude Code 中：

```
/promo-kit
```

或自然语言：

> "帮我推广这个项目到小红书"

> "Write a Twitter thread promoting my CLI tool"

### 外部依赖

| 依赖 | 用途 | 安装 |
|------|------|------|
| humanizer-zh | 中文去 AI 味 | `npx skills add therain2020/humanizer-zh -g` |
| humanizer | English de-AI | `npx skills add therain2020/humanizer -g` |
| agent-reach | 多平台发布 | `npx skills add agent-reach -g` |
| gstack browse | 配图截图 | 自动随 gstack 安装 |

运行 `Skill("health-check")` 验证所有依赖就绪。

## 目录

```
promo-kit/
├── SKILL.md              # 调度器
├── skills/               # 4 个独立引擎
│   ├── copywriter/SKILL.md
│   ├── image-gen/SKILL.md
│   ├── publisher/SKILL.md
│   └── health-check/SKILL.md
├── templates/            # 配图 HTML ×8
├── references/           # 文案模板 + 设计规范
├── promo-content/        # 产出（gitignored）
└── execution-log/        # 发布记录（gitignored）
```
