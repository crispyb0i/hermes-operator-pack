# Skills

| Skill | What it does |
|-------|--------------|
| [`daily-briefing`](daily-briefing) | Stripe + GitHub + Linear → 7am Telegram digest |
| [`inbox-triage`](inbox-triage) | Gmail labelling + drafted replies |
| [`competitor-watch`](competitor-watch) | RSS / HN / Reddit poller with weekly synthesis |
| [`ship-log`](ship-log) | Auto-generate weekly changelog from git + Linear |
| [`agent-watchdog`](agent-watchdog) | Health-check + alert any launchd / cron agent loop |
| [`wake-cycle-template`](wake-cycle-template) | Scaffold for autonomous wake-cycle agents |
| [`obsidian-vault-curator`](obsidian-vault-curator) | Maintain an Obsidian wiki to a strict LLM-friendly schema |
| [`reflexion-debrief`](reflexion-debrief) | Daily debrief generator from session transcripts |

Each skill is a self-contained `SKILL.md` with YAML frontmatter, conforming to the [Hermes skills spec](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills).

## Install one

```bash
hermes skills install https://raw.githubusercontent.com/crispyb0i/hermes-operator-pack/main/skills/<name>/SKILL.md
```

## Install all

```bash
hermes skills tap add crispyb0i/hermes-operator-pack
```
