# Hermes Operator Pack

> Battle-tested Hermes skills for solo founders running a portfolio.

A free, open-source pack of 8 Hermes Agent skills extracted from a 6-month autonomous-agent stack at [dvdshn.com](https://dvdshn.com). These are the skills I actually use — generalized so you can drop them into your own Hermes setup.

## What's in the pack

| Skill | What it does |
|-------|--------------|
| [`daily-briefing`](skills/daily-briefing) | Stripe + GitHub + Linear → 7am Telegram digest |
| [`inbox-triage`](skills/inbox-triage) | Gmail labelling + drafted replies via the gws CLI |
| [`competitor-watch`](skills/competitor-watch) | RSS / HN / Reddit poller with weekly synthesis |
| [`ship-log`](skills/ship-log) | Auto-generate weekly changelog from git + Linear |
| [`agent-watchdog`](skills/agent-watchdog) | Health-check + alert any launchd / cron agent loop |
| [`wake-cycle-template`](skills/wake-cycle-template) | Generalized scaffold for autonomous wake-cycle agents |
| [`obsidian-vault-curator`](skills/obsidian-vault-curator) | Maintain an Obsidian wiki with strict frontmatter + index/log |
| [`reflexion-debrief`](skills/reflexion-debrief) | Daily debrief generator from session transcripts |

## Install

```bash
# Clone the pack
git clone https://github.com/crispyb0i/hermes-operator-pack ~/code/hermes-operator-pack

# Add as a Hermes skills tap (recommended)
hermes skills tap add crispyb0i/hermes-operator-pack

# Or install individual skills
hermes skills install https://raw.githubusercontent.com/crispyb0i/hermes-operator-pack/main/skills/daily-briefing/SKILL.md
hermes skills install https://raw.githubusercontent.com/crispyb0i/hermes-operator-pack/main/skills/agent-watchdog/SKILL.md
# ...
```

Each skill is a self-contained `SKILL.md` with YAML frontmatter. They follow the [Hermes skills spec](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills).

## Why these specifically

I run a portfolio of indie SaaS products plus four autonomous agents (an EmbedProof revenue agent, a Kalshi prediction-market trader, a site janitor, and a meta-orchestrator). The 8 skills here are the ones that came up over and over — the operations every solo operator with an agent stack ends up reinventing.

Read the [origin story / 6-month writeup](https://dvdshn.com/agents) for the context, the failures, and what the loops actually look like in production.

## What I do for a living

I build social-proof sections for SaaS founders for **$100, delivered in 48h**. If you're shipping something and need testimonials, case-study cards, or trust badges that actually convert — see [embedproof.app](https://embedproof.app) or DM me.

## License

MIT — fork it, ship it, change it. Attribution appreciated but not required.

## Credits

Built on [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research. Inspired by [awesome-hermes-usecases](https://github.com/ali-erfan-dev/awesome-hermes-usecases) and [hermes-skins](https://github.com/joeynyc/hermes-skins).
