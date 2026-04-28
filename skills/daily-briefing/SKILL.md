---
name: daily-briefing
description: "Stripe + GitHub + Linear → 7am Telegram digest. Daily founder briefing in one message."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [stripe, github, linear, telegram, cron, founder]
    related_skills: [ship-log, revenue-pulse]
---

# Daily Briefing

Send yourself one Telegram message at 7am with everything that changed yesterday: revenue, MRR delta, GitHub PRs, Linear issues, and any agent alerts. Replaces opening five tabs every morning.

## When to use

- You run a portfolio (or a single SaaS) and want a single consolidated daily snapshot
- You want to know about MRR drops, churn events, or stalled work *before* you start your day
- You already have Stripe, GitHub, and (optionally) Linear API keys configured

## Required environment

```bash
STRIPE_API_KEY=sk_live_…
GITHUB_TOKEN=ghp_…           # or use `gh auth login`
LINEAR_API_KEY=lin_api_…     # optional
TELEGRAM_BOT_TOKEN=…         # the bot that DMs you
TELEGRAM_CHAT_ID=…           # your DM chat id

TODOS_FILE=~/Documents/Obsidian/TODOs.md                   # optional, default shown
FORECAST_FILE=~/Documents/Obsidian/Topics/Forecasting\ Log.md   # optional, default shown
```

## How it works

1. Pull the last 24h of Stripe events: new subs, cancellations, failed charges → compute MRR delta.
2. Pull GitHub: open PRs assigned to you, PRs awaiting review, PRs merged in last 24h.
3. (Optional) Pull Linear: in-progress issues, issues stalled >3 days.
4. Pull `~/.hermes/logs/` and any agent watchdog alert dirs for unread P0 issues.
5. **Read your TODOs file** (`$TODOS_FILE`, default `~/Documents/Obsidian/TODOs.md`):
   - Today's items (any checkbox with today's date tag) → shown bold
   - Next 3 days of Upcoming → shown as a peek-ahead
   - Any item with a past date that's still unchecked → flagged **OVERDUE**
6. **Read your Forecasting Log** (`$FORECAST_FILE`, default `~/Documents/Obsidian/Topics/Forecasting Log.md`):
   - Any open prediction with `Resolves` matching today → surfaced as **🎯 Resolves Today**
   - Any open prediction with `Resolves` past and still unresolved → flagged **⚠️ OVERDUE RESOLUTION**
   - Goal: prevent the Forecasting Log from becoming a write-only graveyard. Resolution discipline is enforced by surfacing.
7. Compose a single Telegram message. Send.

## Forecasting Log format

The skill parses the open-predictions table in `Topics/Forecasting Log.md`. Expected row format:

```markdown
| Date | Prediction | Probability | Resolves | Source |
|------|-----------|-------------|----------|--------|
| 2026-04-27 | Show HN reaches HN front page (top 30) | 30% | 2026-04-29 EOD | (source) |
```

Rules the parser enforces:
- `Resolves` column is parsed for `YYYY-MM-DD` (anything after the date, like "EOD", is ignored)
- Today's date in `Resolves` → surface in briefing as "🎯 Resolves Today"
- Past date + still in open-predictions table → "⚠️ OVERDUE RESOLUTION"
- Predictions in the **Resolved** section are skipped

## TODOs file format

The skill reads a plain markdown checkbox file (designed to live in your Obsidian vault as `TODOs.md`):

```markdown
## Today (YYYY-MM-DD)
- [ ] [2026-04-28] Post Show HN — 8am ET window
- [ ] [2026-04-28] Watch thread first hour

## Upcoming (next 7 days)
### Wednesday 2026-04-29
- [ ] [2026-04-29] Reply to Nous AMA thread with substance

## Backlog (no date)
- [ ] Add 2 more skills to operator-pack
```

Rules the skill enforces:
- `[YYYY-MM-DD]` date tags determine bucket (today / upcoming / overdue)
- Items in `Backlog` are **never** shown in the daily briefing
- Items in `Done — Archive` are skipped
- Parsing is regex-based, no LLM call required for the read step

## Suggested cron

```bash
hermes cron create "0 7 * * *" --name daily-briefing \
  --skill daily-briefing \
  --prompt "Run the daily briefing skill. Send the digest to my Telegram DM."
```

## Output template

```
☕ Daily Briefing — Wed Apr 29

📌 Today's TODOs
  ▸ Post Show HN — 8am ET window
  ▸ Watch thread first hour
  ▸ Reply to Nous AMA thread with substance

🎯 Resolves Today (2)
  ▸ Show HN front page top 30 — predicted 30%, score by 11:59pm
  ▸ Show HN top 5 — predicted 5%, score by 11:59pm

⚠️ OVERDUE (1)
  ▸ [2026-04-25] Reply to vendor invoice email

🔭 Coming up
  Thu: mid-week post-mortem
  Sat: weekly ship-log

💰 Revenue (24h)
  MRR: $1,247 (+$24 net)
  New: 2 ($16/mo, $8/mo)
  Churn: 0
  Failed: 1 (Stripe will retry)

🔧 GitHub
  PRs awaiting review: #142, #144
  Merged: #138, #139, #141

📋 Linear
  In progress: 3
  Stalled >3d: ENG-87, ENG-91

🚨 Agents
  EmbedProof watchdog: ✓ all green
  PMB: ✗ stale state, last update 6h ago — INVESTIGATE
```

## Pitfalls

- **Stripe sandbox vs live.** Confirm `STRIPE_API_KEY` points at live before you trust the MRR number.
- **Telegram chat ID.** DMs need the user-side chat id, not the bot id. Use `https://api.telegram.org/bot<TOKEN>/getUpdates` to find it.
- **GitHub rate limits.** Use a fine-grained PAT scoped to your repos to avoid bumping into the 60/h unauthenticated limit.
- **Don't run this from gateway** — long-running aggregations can exceed gateway timeouts. Use a cron job.
- **TODOs file parsing.** Date tags must be `[YYYY-MM-DD]` exactly — the parser is regex-based. Other date formats are ignored. If "today" looks empty in the briefing but you swear you have items due today, check the date tag format.
- **Forecasting Log resolution discipline.** This section's whole purpose is to **force** you to score predictions on their resolution date. If you see "🎯 Resolves Today" in the briefing and skip scoring, the log dies within 60 days (universal failure mode of forecasting practice). The auto-surface is load-bearing.
- **Forecasting Log path.** Default is `~/Documents/Obsidian/Topics/Forecasting Log.md` (note the space). Override via `$FORECAST_FILE` if your vault structure differs.
