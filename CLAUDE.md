# CLAUDE.md — promo-kit

## Skill routing

When the user's request matches an available skill, invoke it via the Skill tool. When in doubt, invoke the skill.

Key routing rules:
- Promote open-source project / write promo copy → invoke /promo-kit
- Just the copy (CN or EN) → invoke /copywriter
- Just the images → invoke /image-gen
- Just publish to platforms → invoke /publisher
- Check engine health → invoke /health-check

## Project overview

promo-kit is a Claude Code skill that handles the full open-source promotion pipeline:
project analysis → copywriting (zh/en) → de-AI (humanizer/humanizer-zh) → image generation (HTML → headless screenshot) → publishing (agent-reach wrapper).

Chinese platforms: Xiaohongshu, Xiaoheihe.
English platforms: Twitter/X, LinkedIn, Reddit, Dev.to.

## Architecture

```
promo-kit/
├── SKILL.md              # Dispatcher — entry point for the pipeline
├── skills/               # 4 independent engines
│   ├── copywriter/       # lang: zh|en → 6 angles × 6 platforms
│   ├── image-gen/        # 8 HTML templates → headless screenshot
│   ├── publisher/        # agent-reach wrapper with retry
│   └── health-check/     # 4-tier dependency validation
├── templates/            # HTML image templates (×8)
├── references/           # Copy templates + design guide
├── promo-content/        # Generated output (gitignored)
└── execution-log/        # Publish records (gitignored)
```

## Engine dependencies

- copywriter → humanizer-zh (CN) or humanizer (EN) for de-AI pass
- image-gen → gstack browse daemon ($B) for headless screenshots
- publisher → agent-reach for multi-platform publishing
- health-check → validates all of the above
