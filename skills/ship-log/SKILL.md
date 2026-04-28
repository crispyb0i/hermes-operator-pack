---
name: ship-log
description: "Auto-generate weekly changelog from git history + Linear/GitHub issues. Public-ready, zero effort."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [changelog, git, github, linear, weekly, content]
    related_skills: [daily-briefing, reflexion-debrief]
---

# Ship Log

Generate a weekly changelog from git commits, merged PRs, and closed Linear issues — formatted for a public changelog page or Twitter thread. Run it Friday 5pm and you have a "what shipped this week" post ready by Monday.

## When to use

- You ship publicly and want a consistent changelog rhythm
- You want a draft Twitter thread / blog post seeded with real shipped work
- You hate writing "what I did this week" posts from scratch

## Required environment

```bash
GITHUB_TOKEN=ghp_…          # for PR + commit history
LINEAR_API_KEY=lin_api_…    # optional
```

Plus a config file: `~/.hermes/ship-log.yaml`

```yaml
repos:
  - org/embedproof
  - org/dvdshn-site
  - org/talkdrop

linear_team_ids:
  - "abc-123"

# Skip these commit-message prefixes
ignore_prefixes:
  - "chore:"
  - "wip:"
  - "ci:"

# Group commits by these labels
sections:
  - label: "Features"
    matches: ["feat:", "feature:"]
  - label: "Fixes"
    matches: ["fix:", "bugfix:"]
  - label: "Performance"
    matches: ["perf:"]
  - label: "Documentation"
    matches: ["docs:"]
```

## How it works

1. Pull last 7 days of merged PRs across all configured repos.
2. Pull last 7 days of closed Linear issues (if configured).
3. Pull commits on `main` not behind a PR (direct pushes).
4. Filter via `ignore_prefixes`.
5. Group by `sections`. LLM rewrites each line as user-facing English ("Added CSV export to /reports" — not "feat(reports): csv export, fix lint").
6. Output two formats:
   - **Long form** (markdown blog post): `~/Documents/changelogs/YYYY-WW.md`
   - **Twitter thread** (≤280 chars per tweet, threaded): `~/Documents/changelogs/YYYY-WW-thread.md`

## Suggested cron

```bash
hermes cron create "0 17 * * 5" --name ship-log-friday \
  --skill ship-log \
  --prompt "Generate this week's ship log. Draft a Twitter thread variant too. Save both, don't post."
```

## Pitfalls

- **Don't auto-post.** The point is a *draft* you review for vibe before shipping. LLM-generated weekly recaps with no human pass read like a robot.
- **Squash-merge PRs hide work.** If your repo squashes, the PR title is the only signal. Make sure PR titles are user-facing-meaningful, or this skill is GIGO.
- **Force-pushes break commit history.** Reset / rebase + push --force on `main` will make this skill see ghost commits. Don't force-push to main.
- **LLM cost.** Per-week run with 50 PRs ≈ ~$0.10 on Sonnet. Negligible, but worth noting.
