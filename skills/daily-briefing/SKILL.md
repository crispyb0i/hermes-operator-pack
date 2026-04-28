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
```

## How it works

1. Pull the last 24h of Stripe events: new subs, cancellations, failed charges → compute MRR delta.
2. Pull GitHub: open PRs assigned to you, PRs awaiting review, PRs merged in last 24h.
3. (Optional) Pull Linear: in-progress issues, issues stalled >3 days.
4. Pull `~/.hermes/logs/` and any agent watchdog alert dirs for unread P0 issues.
5. Compose a single Telegram message. Send.

## Suggested cron

```bash
hermes cron create "0 7 * * *" --name daily-briefing \
  --skill daily-briefing \
  --prompt "Run the daily briefing skill. Send the digest to my Telegram DM."
```

## Output template

```
☕ Daily Briefing — Mon Apr 28

💰 Revenue (24h)
  MRR: $1,247 (+$24 net)
  New: 2 ($16/mo, $8/mo)
  Churn: 0
  Failed: 1 (Stripe will retry)

🔧 GitHub
  PRs awaiting review: #142, #144
  Merged: #138, #139, #141
  Yours stalled >3d: #131 ("Add CSV export")

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
