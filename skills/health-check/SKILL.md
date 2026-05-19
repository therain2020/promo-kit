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

检查 4 个引擎 SKILL.md 文件存在且 frontmatter 合法：

```bash
SKILL_DIR="$(cd "$(dirname "$0")" && pwd)"
REPO_ROOT="$(cd "$SKILL_DIR/../.." && pwd)"
ERRORS=0

for engine in copywriter image-gen publisher health-check; do
  SKILL_FILE="$REPO_ROOT/skills/$engine/SKILL.md"
  if [ ! -f "$SKILL_FILE" ]; then
    echo "FAIL: skills/$engine/SKILL.md 不存在"
    ERRORS=$((ERRORS + 1))
  else
    # 检查 YAML frontmatter
    FM=$(head -15 "$SKILL_FILE")
    if ! echo "$FM" | grep -q "^---$" && ! echo "$FM" | grep -q "^name:"; then
      echo "WARN: skills/$engine/SKILL.md frontmatter 可能不合法"
    else
      echo "OK: skills/$engine/SKILL.md"
    fi
  fi
done
```

### L2: 模板完整性

```bash
# 文案模板
for ref in "$REPO_ROOT/references/copywriter-cn.md" "$REPO_ROOT/references/copywriter-en.md" "$REPO_ROOT/references/design-guide.md"; do
  if [ -f "$ref" ]; then
    echo "OK: $(basename "$ref")"
  else
    echo "FAIL: $(basename "$ref") 不存在"
    ERRORS=$((ERRORS + 1))
  fi
done

# HTML 模板
EXPECTED_TEMPLATES="cover-xhs comparison terminal steps phone-text cta-card architecture code-block"
for tmpl in $EXPECTED_TEMPLATES; do
  if [ -f "$REPO_ROOT/templates/$tmpl.html" ]; then
    echo "OK: templates/$tmpl.html"
  else
    echo "FAIL: templates/$tmpl.html 不存在"
    ERRORS=$((ERRORS + 1))
  fi
done

echo "模板检查: $ERRORS 个错误"
```

### L3: 外部依赖可达性

检查 skill 可用性。通过在调用时尝试引用目标 skill 来判断：

| 依赖 | 检查方式 | 用途 |
|------|---------|------|
| humanizer-zh | `Skill("humanizer-zh")` 可达 | 中文去 AI 味 |
| humanizer | `Skill("humanizer")` 可达 | 英文去 AI 味 |
| agent-reach | `Skill("agent-reach")` 可达 | 多平台发布 |
| gstack browse ($B) | `$B status` 返回 healthy | 配图截图 |

```bash
# browse daemon 检查
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B="$HOME/.claude/skills/gstack/browse/dist/browse"

if [ -x "$B" ]; then
  $B status 2>/dev/null && echo "OK: browse daemon" || echo "WARN: browse daemon 未运行"
else
  echo "FAIL: browse binary 不存在"
fi
```

### L4: 产出验证（仅 full 模式）

跑一条最小 pipeline：

1. 用示例数据生成一篇小红书笔记
2. 用示例数据生成一张 cover-xhs 配图
3. 检查产出文件格式正确

```bash
# 检查 promo-content/ 目录可写入
echo "test" > "$REPO_ROOT/promo-content/.write-test" 2>/dev/null && \
  rm "$REPO_ROOT/promo-content/.write-test" && \
  echo "OK: promo-content/ 可写入" || \
  echo "FAIL: promo-content/ 不可写入"

echo "test" > "$REPO_ROOT/execution-log/.write-test" 2>/dev/null && \
  rm "$REPO_ROOT/execution-log/.write-test" && \
  echo "OK: execution-log/ 可写入" || \
  echo "FAIL: execution-log/ 不可写入"
```

## 输出格式

```
=== promo-kit 健康报告 ===
时间: {timestamp}
级别: {check_level}

L1 引擎完整性: {PASS/FAIL}
  OK  skills/copywriter/SKILL.md
  OK  skills/image-gen/SKILL.md
  OK  skills/publisher/SKILL.md
  OK  skills/health-check/SKILL.md

L2 模板完整性: {PASS/FAIL}
  OK  references/copywriter-cn.md
  OK  references/copywriter-en.md
  OK  references/design-guide.md
  OK  templates/*.html ×8

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
