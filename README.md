# promo-kit

[中文](README.zh-CN.md)

A Claude Code skill that takes your open-source project from zero to published across Chinese and English developer communities. One conversation handles the whole pipeline.

## What it does

You tell it about your project. It figures out the interesting angles, writes copy in Chinese or English, strips out the AI-sounding bits, renders companion images, and publishes to the platform. Or hands you the files to post yourself.

Chinese platforms: Xiaohongshu (RED), Xiaoheihe, and Juejin.
English platforms: Twitter/X, LinkedIn, Reddit, and Dev.to.

## The pipeline

```
You: "promote my project on Xiaohongshu"
  → health-check: verifies all engines and dependencies are ready
  → copywriter: picks narrative angle, drafts platform-native copy
  → humanizer-zh / humanizer: removes AI writing patterns
  → image-gen: renders HTML templates, screenshots via headless browser
  → publisher: hands off to agent-reach for actual posting
```

Each step is a standalone engine with its own SKILL.md under `skills/`. You can use the full pipeline or invoke individual engines directly.

## Platforms and formats

| Platform | Language | What it produces |
|----------|----------|-----------------|
| Xiaohongshu (RED) | zh | Note with cover image, body, hashtags |
| Xiaoheihe | zh | Tech post with code blocks, architecture |
| Juejin | zh | Technical article with cover, code blocks, tags |
| Twitter/X | en | Thread or single post with hooks |
| LinkedIn | en | Article or short post |
| Reddit | en | r/programming or topic-specific post |
| Dev.to | en | Tutorial or opinion article |

## Six narrative angles

The copywriter picks one (or you specify it):

A: Pain point. What hurt, why nothing fixed it, how this does.
B: Technical deep-dive. Architecture, key decisions, vs alternatives.
C: Comparison. Side-by-side with existing tools.
D: Open source story. Why you built it, highlights, call for contributors.
E: Pitfalls journal. What went wrong, how you dug out, what you learned.
F: Tutorial. Who this is for, step-by-step, common gotchas.

## Install

Clone the repo, then link the engines into your project:

```bash
git clone https://github.com/therain2020/promo-kit.git

# From your project directory:
mkdir -p .claude/skills
ln -s "$(pwd)/../promo-kit/skills/copywriter" .claude/skills/copywriter
ln -s "$(pwd)/../promo-kit/skills/image-gen" .claude/skills/image-gen
ln -s "$(pwd)/../promo-kit/skills/publisher" .claude/skills/publisher
ln -s "$(pwd)/../promo-kit/skills/health-check" .claude/skills/health-check
```

On Windows, copy instead of symlink:

```powershell
Copy-Item ../promo-kit/skills/* .claude/skills/ -Recurse
```

Or just open Claude Code inside the promo-kit repo. The root SKILL.md loads automatically.

### Dependencies

These need to be installed globally:

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

> "Write a Xiaohongshu post promoting my CLI tool"

> "Make a Twitter thread about my open-source library"

The skill walks through:
1. Figuring out your language, platform, and angle
2. Reading your project (README, source, config)
3. Drafting copy from templates in `references/`
4. Running it through humanizer/humanizer-zh
5. Generating images if the platform needs them
6. Asking whether to publish or just save the files

Output lands in `promo-content/` under your project directory. Publishing logs go to `execution-log/`.

### Using engines by themselves

You can call individual engines directly:

```
/copywriter   — Just the copy. Parameters: lang, platform, angle, hook
/image-gen    — Just the images. Feed it a markdown file, get PNGs back
/publisher    — Just the publishing. Content + images → agent-reach
/health-check — Check if everything is working
```

## What's inside

```
promo-kit/
├── SKILL.md                          # dispatcher — entry point for the pipeline
├── skills/
│   ├── copywriter/SKILL.md           # lang: zh|en → 6 angles × 6 platforms
│   ├── image-gen/SKILL.md            # 8 HTML templates → headless screenshot
│   ├── publisher/SKILL.md            # agent-reach wrapper with retry logic
│   └── health-check/SKILL.md         # 4-tier dependency validation
├── templates/                        # HTML image templates
│   ├── cover-xhs.html                # RED cover: classic red gradient
│   ├── xhs-cover-tech.html           # RED cover: tech-minimalist (1080×1440)
│   ├── xhs-pain.html                 # Pain point card
│   ├── xhs-features.html             # Feature highlights card
│   ├── xhs-steps.html                # Numbered steps (tech-minimalist)
│   ├── xhs-cta.html                  # CTA card (dark, tech-minimalist)
│   ├── xhs-comparison.html           # Comparison table (tech-minimalist)
│   ├── cover-juejin.html             # Juejin cover: title + tagline
│   ├── comparison.html               # side-by-side comparison (classic)
│   ├── terminal.html                 # terminal-style CLI window
│   ├── steps.html                    # numbered step cards (classic)
│   ├── phone-text.html               # simulated phone message
│   ├── cta-card.html                 # call-to-action card (classic)
│   ├── architecture.html             # component diagram
│   └── code-block.html               # syntax-highlighted code
├── references/
│   ├── copywriter-cn.md              # CN templates (Xiaohongshu, Xiaoheihe, Juejin)
│   ├── copywriter-en.md              # EN templates (Twitter, LinkedIn, Reddit, Dev.to)
│   └── design-guide.md               # image design system
├── promo-content/                    # generated copy (gitignored)
└── execution-log/                    # publish records (gitignored)
```

## How the engines talk to each other

The dispatcher passes structured parameters between engines. Each engine reads its input, does one thing, and writes output to a known path. The next engine picks it up.

```
copywriter:   lang, platform, angle, hook → markdown file
humanizer:    markdown file → de-AI-ed markdown file
image-gen:    markdown file, platform → PNG files
publisher:    markdown file, PNG files, platform → agent-reach call
```

This means you can swap pieces out. Replace image-gen with your own screenshot tool. Skip publisher and post manually. The pipeline doesn't care.

## Image specs

image-gen renders at these resolutions:

| Platform | Size | Per post |
|----------|------|----------|
| Xiaohongshu (covers) | 1080 x 1440 | 3-6 images |
| Xiaohongshu (classic cards) | 750 x 1000 | 3-6 images |
| Xiaoheihe | 750 x 1000 or 1200 x 800 | 2-4 images |
| Juejin | 1000 x 420 (cover) + flexible | 1-3 images |

Templates are plain HTML with inline CSS. No external fonts, no JS frameworks. The headless browser loads them directly from disk and screenshots at 2x resolution.
