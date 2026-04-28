---
name: agent-watchdog
description: "Health-check + alert any launchd/cron agent loop. Catches stale runs, stuck processes, and runaway costs."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [launchd, cron, monitoring, watchdog, autonomous-agents, alerting]
    related_skills: [wake-cycle-template, reflexion-debrief]
---

# Agent Watchdog

Hourly health check for any autonomous agent loop you run via launchd or cron. Verifies the loop is alive, not stuck, not blowing through your cost cap, and that recent runs actually wrote logs.

This is the watchdog I run on four production autonomous agents. It catches:

- launchd/cron job unloaded or disabled
- Last run exited non-zero
- No recent log file (loop silently dead)
- Stuck `claude` / agent process running >2h with no progress
- Daily cycle count exceeded cost ceiling
- API tokens stale / expired

## When to use

- You run any long-lived autonomous agent (Claude Code, Hermes, Codex, custom)
- You've experienced an agent loop dying silently and not noticing for hours
- You want one canonical alert path (Telegram / email / log) for all agent health issues

## Required environment

```bash
TELEGRAM_BOT_TOKEN=…         # for push alerts (optional)
TELEGRAM_CHAT_ID=…
```

Plus a config: `~/.hermes/agent-watchdog.yaml`

```yaml
agents:
  - name: my-agent
    launchd_label: com.example.myagent      # OR cron_pattern: "*/30 * * * *"
    log_dir: ~/.local/var/log/myagent
    log_max_age_minutes: 180                # alert if no fresh log
    daily_cycle_cap: 20                     # alert if exceeded
    process_match: "claude.*myagent"         # for stuck-process detection
    process_max_runtime_minutes: 120

alert:
  telegram: true
  daily_summary_file: ~/.local/var/log/agent-watchdog/YYYY-MM-DD.md
```

## How it works

1. For each configured agent:
   - Check launchd is loaded (`launchctl list | grep <label>`) or cron entry exists.
   - Read the last exit status — alert if non-zero.
   - Find the newest file in `log_dir` — alert if older than `log_max_age_minutes`.
   - Check for stuck processes via `ps`.
   - Count today's cycle markers — alert if >`daily_cycle_cap` (cost protection).
2. Append every alert to a daily markdown file (one per day, easy to review).
3. Push critical alerts (P0) to Telegram.

## Suggested cron

```bash
hermes cron create "0 * * * *" --name agent-watchdog \
  --skill agent-watchdog \
  --prompt "Run the agent watchdog. Alert on any failures."
```

## Output template (daily alert file)

```markdown
# Agent Watchdog — 2026-04-28

## 09:00 [WARN] my-agent
Last log was 4h old (cap: 3h). Process likely dead. Reload via:
  launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.example.myagent.plist

## 13:00 [P0] my-agent
Daily cycle count = 23 (cap: 20). Cost runaway. Manual investigate.
```

## Pitfalls

- **macOS TCC.** launchd-spawned bash can't read `~/Documents` without Full Disk Access. Keep all watchdog logs under `~/.local/var/log/`.
- **launchctl exit codes.** Recent macOS deprecated `launchctl list` output format. Use `launchctl print gui/$(id -u)/<label>` instead.
- **Stuck-process detection.** `ps` matching is fuzzy. Test your `process_match` regex carefully — false positives kill running agents.
- **Don't auto-restart.** If you auto-restart a stuck agent, you risk infinite restart loops at 3am. Alert + manual investigate is the right default.

## Why this matters

A silently-dead agent is worse than no agent. You think work is happening. It isn't. By the time you notice, you've lost a day of progress AND lost trust in the loop. A dumb hourly health check fixes 80% of "why didn't my agent do X" panic.
