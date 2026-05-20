---
name: health-check
description: |
  Promo-kit 健康检查引擎。检查 4 个引擎完整性、模板可用性、外部依赖可达性。
  产出结构化健康报告。由 promo-kit 调度器在工作流启动前调用。
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - Skill
triggers: []
---

# health-check — 健康检查引擎

promo-kit 的内部引擎。在调度器启动工作流前验证所有依赖可用。

## 调用协议

由 promo-kit 调度器通过 `Skill("health-check", args)` 调用：

```
check_level: quick|full
```

- `quick` — 文件完整性 + 外部依赖可达性（30 秒内完成）
- `full` — 包含产出验证（跑最小 pipeline）

## 检查清单

### L1: 引擎完整性

检查 5 个引擎 SKILL.md 文件存在且 frontmatter 合法：

```
skills/copywriter/SKILL.md
skills/image-gen/SKILL.md
skills/template-designer/SKILL.md
skills/publisher/SKILL.md
skills/health-check/SKILL.md
```

每个文件检查：
- 文件存在
- YAML frontmatter 包含 `name:` 字段

### L2: 模板完整性

文案模板检查：
- `references/copywriter-cn.md`
- `references/copywriter-en.md`
- `references/design-guide.md`

HTML 模板检查（15 个）：
- `templates/cover-xhs.html`
- `templates/cover-juejin.html`
- `templates/xhs-cover-tech.html`
- `templates/xhs-pain.html`
- `templates/xhs-features.html`
- `templates/xhs-steps.html`
- `templates/xhs-cta.html`
- `templates/xhs-comparison.html`
- `templates/comparison.html`
- `templates/terminal.html`
- `templates/steps.html`
- `templates/phone-text.html`
- `templates/cta-card.html`
- `templates/architecture.html`
- `templates/code-block.html`

### L3: 外部依赖可达性

检查 skill 可用性：

| 依赖 | 检查方式 | 用途 |
|------|---------|------|
| humanizer-zh | `Skill("humanizer-zh")` 可达 | 中文去 AI 味 |
| humanizer | `Skill("humanizer")` 可达 | 英文去 AI 味 |
| agent-reach | `Skill("agent-reach")` 可达 | 多平台发布 |
| gstack browse ($B) | `$B status` 返回 healthy | 配图截图 |

```
browse daemon 检查:
  1. Locate the binary — do NOT assume $B is set. The variable is defined
     only inside gstack's own SKILL.md preamble; it does not persist.
     Find it with:
       Glob **/browse/dist/browse* under ~/.claude/skills/gstack/
     On Windows the binary is browse.exe, not browse.
  2. Run "<found_path>" status → parse "Status: healthy"/"Status: unhealthy"
     If the daemon is not running, the binary auto-launches it on any command.
```

### L4: 产出验证（仅 full 模式）

跑一条最小 pipeline：

1. 用示例数据生成一篇推广文案
2. 用示例数据生成一张封面配图
3. 检查产出文件格式正确

验证 `promo-content/` 和 `execution-log/` 目录可写入。

## 输出格式

```
=== promo-kit 健康报告 ===
时间: {timestamp}
级别: {check_level}

L1 引擎完整性: {PASS/FAIL}
  OK  skills/copywriter/SKILL.md
  OK  skills/image-gen/SKILL.md
  OK  skills/template-designer/SKILL.md
  OK  skills/publisher/SKILL.md
  OK  skills/health-check/SKILL.md

L2 模板完整性: {PASS/FAIL}
  OK  references/copywriter-cn.md
  OK  references/copywriter-en.md
  OK  references/design-guide.md
  OK  templates/*.html ×15

L3 外部依赖: {PASS/FAIL}
  OK  humanizer-zh
  OK  humanizer
  OK  agent-reach
  OK  browse daemon

L4 产出验证: {PASS/FAIL/SKIPPED}
  OK  promo-content/ 可写入
  OK  execution-log/ 可写入

总评: {ALL_OK / DEGRADED / BLOCKED}
```

| 总评 | 条件 |
|------|------|
| ALL_OK | 所有检查通过 |
| DEGRADED | L3 部分失败（可降级运行） |
| BLOCKED | L1 或 L2 有 FAIL（不可继续） |
