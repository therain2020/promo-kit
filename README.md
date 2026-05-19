# promo-kit

[中文](README.zh-CN.md)

A Claude Code skill for open-source promotion. Give it a repo and it figures out the angles, writes the copy, kills the AI-sounding bits, makes the images, and posts it. Or hands you the files so you can post them yourself.

Chinese platforms: Xiaohongshu, Xiaoheihe, Juejin.
English platforms: Twitter/X, LinkedIn, Reddit, Dev.to.

## The pipeline

```
You: "write a RED post for my CLI tool"
  → health-check: verifies all 5 engines and external dependencies
  → copywriter: picks a narrative angle, drafts platform-native copy
  → humanizer-zh / humanizer: removes 29 English or 24 Chinese AI writing patterns
  → image-gen: fills HTML templates, screenshots via headless browser ($B)
  → publisher: hands off to agent-reach for posting
```

Each engine stands alone — its own SKILL.md under `skills/`. Run the whole pipeline or pick one.

## Platforms and formats

| Platform | Language | What it produces |
|----------|----------|-----------------|
| Xiaohongshu (RED) | zh | Note with cover, body, hashtags. 3-6 images at 1080×1440 or 750×1000 |
| Xiaoheihe | zh | Tech post with code blocks, architecture description |
| Juejin | zh | Technical article with cover, code blocks, topic tags |
| Twitter/X | en | Thread or single post with hooks |
| LinkedIn | en | Article or short post |
| Reddit | en | r/programming or topic-specific post |
| Dev.to | en | Tutorial or opinion article |

## Six narrative angles

The copywriter picks one. You can specify it.

| Angle | Chinese | English | Structure |
|-------|---------|---------|-----------|
| A | Pain point | Pain Point | Problem scenario → why nothing fixed it → how this does |
| B | Tech deep-dive | Technical | Background → architecture + key decisions → vs alternatives |
| C | Comparison | Comparison | Overview of options → side-by-side → when to use each |
| D | Open source | Open Source | Why built → highlights + tech stack → star/PR invitation |
| E | Pitfalls | Pitfalls | Context → per-pitfall (problem → cause → fix) → lessons |
| F | Tutorial | Tutorial | Target reader → step-by-step → common gotchas |

## Install

Clone the repo, then link the engines into your project:

```bash
git clone https://github.com/therain2020/promo-kit.git

# From your project directory:
mkdir -p .claude/skills
ln -s "$(pwd)/../promo-kit/skills/copywriter" .claude/skills/copywriter
ln -s "$(pwd)/../promo-kit/skills/image-gen" .claude/skills/image-gen
ln -s "$(pwd)/../promo-kit/skills/template-designer" .claude/skills/template-designer
ln -s "$(pwd)/../promo-kit/skills/publisher" .claude/skills/publisher
ln -s "$(pwd)/../promo-kit/skills/health-check" .claude/skills/health-check
```

On Windows, copy instead of symlink:

```powershell
Copy-Item ../promo-kit/skills/* .claude/skills/ -Recurse
```

Or open Claude Code inside the promo-kit repo. The root SKILL.md loads automatically.

### Dependencies

Install these globally:

```bash
npx skills add therain2020/humanizer-zh -g -y    # Chinese de-AI
npx skills add therain2020/humanizer -g -y       # English de-AI
npx skills add agent-reach -g -y                 # publishing to platforms
```

gstack browse (used by image-gen for screenshots) comes with any gstack install.

Run `Skill("health-check")` inside Claude Code to confirm everything is wired up.

## Usage

```
/promo-kit
```

Or in natural language:

> "Write a RED post promoting my CLI tool"

> "Make a Twitter thread about my open-source library"

> "在掘金上发一篇介绍我项目的技术文章"

The skill walks through:
1. Figuring out language, platform, and angle
2. Reading your project (README, source, config)
3. Drafting copy from templates in `references/`
4. Running it through humanizer or humanizer-zh
5. Generating images if the platform needs them
6. Asking whether to publish or save the files

Output lands in `promo-content/` under your project directory. Publishing logs go to `execution-log/`.

### Using engines individually

```
/copywriter         — Copy only. Params: lang, platform, angle, hook
/image-gen          — Images only. Feed it a markdown file, get PNGs
/template-designer  — Design templates from a style description
/publisher          — Publishing only. Content + images → agent-reach
/health-check       — Check if everything is working
```

## Engines

| Engine | What it does | Dependencies |
|--------|-------------|-------------|
| promo-kit (dispatcher) | Orchestrates the full 9-step pipeline | All engines below |
| copywriter | lang: zh→en, 6 angles × 7 platforms → markdown | humanizer-zh, humanizer |
| image-gen | markdown → HTML templates → $B screenshots → PNG | gstack browse daemon |
| template-designer | style description → design system → HTML templates | — |
| publisher | markdown + images → agent-reach → published post | agent-reach |
| health-check | 4-tier validation: engines, templates, dependencies, output | — |

## Image specs

Templates follow the Dark Editorial Tech design system: dark backgrounds, serif display headlines, neon accent colors, CSS noise texture.

| Platform | Size | Per post | Style |
|----------|------|----------|-------|
| RED (primary) | 1080 × 1440 | 3-6 images | Dark editorial |
| RED (auxiliary) | 750 × 1000 | 3-6 images | Dark editorial |
| Xiaoheihe | 750 × 1000 or 1200 × 800 | 2-4 images | Dark editorial or custom |
| Juejin | 1000 × 420 (cover) + flexible | 1-3 images | Dark editorial |

15 HTML templates are included: cover-xhs, xhs-cover-tech, xhs-pain, xhs-features, xhs-steps, xhs-cta, xhs-comparison, terminal, code-block, cover-juejin, comparison, steps, phone-text, cta-card, architecture.

## What's inside

```
promo-kit/
├── SKILL.md                          # dispatcher — 9-step pipeline entry point
├── skills/
│   ├── copywriter/SKILL.md           # zh|en → 6 angles × 7 platforms
│   ├── image-gen/SKILL.md            # 15 HTML templates → headless screenshot
│   ├── template-designer/SKILL.md    # style → design system → HTML templates
│   ├── publisher/SKILL.md            # agent-reach wrapper with retry and logging
│   └── health-check/SKILL.md         # 4-tier dependency validation
├── templates/                        # 15 HTML image templates (Dark Editorial Tech)
├── references/
│   ├── copywriter-cn.md              # CN templates (RED, Xiaoheihe, Juejin)
│   ├── copywriter-en.md              # EN templates (Twitter, LinkedIn, Reddit, Dev.to)
│   └── design-guide.md               # color, typography, layout specs
├── promo-content/                    # generated copy (gitignored)
└── execution-log/                    # publish records (gitignored)
```

## How engines talk to each other

The dispatcher passes structured parameters between engines. Each engine reads input, does one thing, writes output to a known path. The next engine picks it up.

```
copywriter:        lang, platform, angle, hook → markdown file
humanizer:         markdown file → de-AI-ed markdown file
image-gen:         markdown file, platform → PNG files
publisher:         markdown file, PNG files, platform → agent-reach call
```

Swap pieces. Use your own screenshot tool instead of image-gen. Skip publisher and post by hand. The pipeline doesn't care — it just passes data forward.

## Design system

Templates use "Dark Editorial Tech": `#090A0C` background with `#E4E4E6` text, serif display headlines (Georgia + Noto Serif CJK SC) at 96-108px for covers, neon cyan (`#3BC9DB`) and purple (`#845EF7`) accents, amber (`#FCC419`) highlights, and a CSS-only SVG noise texture overlay at 4% opacity.

The template-designer engine can generate new template sets from a style description. Ask Claude: "design a cyberpunk template set for RED" or "make warm retro-style templates."

## License

MIT
