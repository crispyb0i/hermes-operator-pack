---
name: reflexion-debrief
description: "Daily debrief generator from session transcripts. What the agent did, what it learned, what's stuck."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [reflexion, debrief, autonomous-agents, learning, daily]
    related_skills: [agent-watchdog, wake-cycle-template, obsidian-vault-curator]
---

# Reflexion Debrief

End-of-day debrief for any autonomous agent loop. Reads the day's session transcripts, extracts what the agent actually did, what it learned, what's stuck, and what regrets it has. Saves to `Daily_Logs/YYYY-MM-DD.md` for human review and as memory for tomorrow's wake.

This is the "reflexion" pattern operationalized — but only the pieces that survive contact with reality.

## When to use

- You run an autonomous wake-cycle agent and want one rolled-up summary per day
- You want the agent to remember what it tried yesterday so it doesn't loop on the same thing
- You're tired of reading 30 raw session logs to figure out what happened

## What it produces

A single `Daily_Logs/YYYY-MM-DD.md` containing:

```markdown
---
type: daily-log
app: my-agent
date: 2026-04-28
status: complete
cycles: 14
---

# Daily Debrief — 2026-04-28

## What I shipped
- Sent 3 outreach emails (lead-1, lead-2, lead-3)
- Published a tweet about the new feature → 47 impressions
- Closed 2 GitHub issues

## What I tried that didn't work
- HN submission tanked at 2 points after 1h. Probably title was off.
- Cold email to acmecorp bounced (bad domain in target list)

## What I'm stuck on
- Stripe webhook not firing on test events. Need David to check Stripe dashboard config.

## What I learned (memory for tomorrow)
- HN submissions before 6am PT seem to die instantly — try later in the day
- The `/api/leads` endpoint requires `X-API-Key` not `Authorization: Bearer` (cost me 3 cycles to figure out)

## Cost
- Cycles: 14
- Estimated spend: $1.84
```

## Required environment

```bash
ANTHROPIC_API_KEY=…   # or whatever LLM provider drives the synthesis
```

Plus config: `~/.hermes/reflexion-debrief.yaml`

```yaml
agents:
  - name: my-agent
    log_dir: ~/.local/var/log/my-agent
    output_dir: ~/Documents/Obsidian/Experiments/my-agent/Daily_Logs
    memory_file: ~/Documents/Obsidian/Experiments/my-agent/MEMORY.md
```

## How it works

1. Glob all today's session log files in `log_dir`.
2. For each, extract: tool calls made, files changed, outcomes (success/failure), errors.
3. Send the consolidated trace to the LLM with a structured prompt asking for:
   shipped / tried-failed / stuck / learned / cost.
4. Write the debrief to `output_dir/YYYY-MM-DD.md`.
5. Append the "what I learned" section to `memory_file` (which the agent reads at next wake).

## Suggested cron

```bash
hermes cron create "0 23 * * *" --name reflexion-debrief \
  --skill reflexion-debrief \
  --prompt "Run the reflexion debrief for all configured agents. Save daily logs and update MEMORY."
```

## Why "reflexion only the pieces that work"

The original reflexion paper assumes the agent self-evaluates after every step. In production this is:
- Expensive (an extra LLM call per step)
- Noisy (the agent's per-step self-eval is often wrong)
- Slow (latency doubles)

What survives is **end-of-day reflexion**, where you have enough trace data to actually distinguish patterns. Per-step is theatre; per-day is signal.

## Pitfalls

- **MEMORY.md grows unbounded.** Cap it at ~200 lines and have the agent prune the oldest entries periodically.
- **Don't use the same model for synthesis as for the loop.** Cheaper model (Haiku, GPT-4o-mini) for synthesis is plenty.
- **Don't run debrief mid-day.** Wait until the day's loops are done. Otherwise the debrief reflects a partial day and the memory drift compounds.
- **Watch for hallucinated outcomes.** The synthesizer LLM will confidently say "successfully sent 3 emails" when the agent only drafted them. Verify against actual side effects (DB writes, HTTP responses) where possible.

## Companion skills

- [`agent-watchdog`](../agent-watchdog) — for *failure* alerts (this skill is for *daily summary*, different concern)
- [`wake-cycle-template`](../wake-cycle-template) — to scaffold the agent loop this debriefs
- [`obsidian-vault-curator`](../obsidian-vault-curator) — for the vault structure this writes into
