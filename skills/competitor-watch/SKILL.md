---
name: competitor-watch
description: "RSS/HN/Reddit poller with weekly synthesis. Know what your competitors shipped before they tell you."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [rss, hackernews, reddit, monitoring, research, weekly-digest]
    related_skills: [reflexion-debrief]
---

# Competitor Watch

Poll a configured list of RSS feeds, HN keyword searches, and subreddits hourly. Match against keywords. Synthesize a weekly digest of what shifted in your space — what shipped, who raised, what's trending, what changed in pricing.

## When to use

- You compete in a fast-moving space (AI tooling, dev tools, indie SaaS)
- You don't want to read 30 blogs and 5 subreddits every day
- You want one Friday-afternoon digest, not 100 hourly Telegram pings

## Required environment

```bash
# All optional — skill works with whatever you provide
REDDIT_CLIENT_ID=…
REDDIT_CLIENT_SECRET=…
HN_USERNAME=…    # optional, for personalized HN
```

## Configuration: `~/.hermes/competitor-watch.yaml`

```yaml
keywords:
  - "social proof widget"
  - "embed testimonials"
  - "EmbedProof"          # also watch your own brand
  - "Senja"
  - "Famewall"

feeds:
  - https://hnrss.org/newest?q=social+proof
  - https://news.ycombinator.com/rss
  - https://blog.competitor.com/rss

subreddits:
  - SaaS
  - SideProject
  - indiehackers
  - LocalLLaMA

# Anything matching these is auto-discarded (cuts noise)
deny_keywords:
  - jobs hiring
  - cryptocurrency
```

## How it works

1. **Hourly cron** polls every feed/subreddit/HN search → SQLite cache in `~/.hermes/cache/competitor-watch.db`.
2. **Match** each item against keywords (case-insensitive substring + simple stemming).
3. **Tag** each match with source + matched keyword + score (engagement-weighted).
4. **Friday 4pm cron** synthesizes the week's matches into a digest:
   - Top 5 stories by engagement
   - New competitors detected (mentions of unknown product names)
   - Pricing / launch signals
   - Threats and opportunities

## Suggested crons

```bash
# Hourly poll (cheap, no LLM cost)
hermes cron create "0 * * * *" --name competitor-poll \
  --skill competitor-watch \
  --prompt "Run the polling step only. Cache new matches. No synthesis."

# Friday 4pm digest
hermes cron create "0 16 * * 5" --name competitor-digest \
  --skill competitor-watch \
  --prompt "Synthesize this week's competitor matches into a digest. Send to my Telegram."
```

## Pitfalls

- **Hourly LLM calls are expensive.** The polling step should be PURE PYTHON / regex matching — no LLM. Only the weekly synthesis should hit the LLM.
- **HNRSS aggressive caching.** Their RSS lags by ~10 minutes. That's fine for a weekly digest, not for breaking news.
- **Reddit OAuth.** PRAW requires a script-type app. Anonymous access is rate-limited brutally.
- **Brand-name dupes.** "Senja" is also a Norwegian island. Use phrases like `"Senja testimonials"` not bare `"senja"`.
