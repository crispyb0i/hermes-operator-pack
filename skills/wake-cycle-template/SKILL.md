---
name: wake-cycle-template
description: "Generalized scaffold for autonomous wake-cycle agents. launchd-callable, TCC-safe, with cost caps + idempotency guards."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [autonomous-agents, launchd, wake-cycle, scaffold, claude-code, hermes]
    related_skills: [agent-watchdog, reflexion-debrief]
---

# Wake-Cycle Template

A battle-tested scaffold for any "agent that wakes up every N minutes / hours, does work, goes back to sleep" loop. Extracts the patterns that survive 6 months of production:

- TCC-safe paths (everything outside `~/Documents/`)
- Per-day cost caps with idempotent already-ran-today guard
- Self-healing prompt sync from a canonical location
- Node version pin (launchd's PATH ships a stale node 18.14.2)
- FATAL-exit alerting (silent failure is the worst failure)
- Cycle-marker files for cost accounting + watchdog observability

## When to use

- You're building an autonomous agent loop on macOS via launchd (or Linux via systemd/cron)
- You want a starting scaffold instead of inventing the same boilerplate again
- You've already been bitten by silent failures, TCC blocks, or runaway costs

## Required environment

Whatever your agent CLI needs (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, custom keys). The scaffold itself just orchestrates; it doesn't call the LLM directly.

## What the scaffold gives you

```
~/.local/bin/<agent>-wake-cycle.sh    # launchd-callable wrapper
~/.local/etc/<agent>/                 # config + canonical prompt
~/.local/var/log/<agent>/             # logs (one per cycle)
~/Library/LaunchAgents/com.<agent>.plist
```

## Generated wrapper (excerpt)

```bash
#!/bin/bash
# <agent>-wake-cycle.sh — Autonomous wake cycle, loaded by launchd.

set -e

# ─── node version pin ────────────────────────────────────────────────
# launchd's plist PATH ships a stale /usr/local/bin/node v18.14.2 that
# breaks Playwright MCP and modern util.styleText. Prepend nvm v22.
NVM_NODE_BIN="$HOME/.nvm/versions/node/v22.14.0/bin"
[ -d "$NVM_NODE_BIN" ] && export PATH="$NVM_NODE_BIN:$PATH"

# ─── per-day cost cap ────────────────────────────────────────────────
TODAY=$(date +%Y-%m-%d)
COUNT_FILE="$HOME/.local/var/log/<agent>/cycles-$TODAY.count"
mkdir -p "$(dirname "$COUNT_FILE")"
COUNT=$(cat "$COUNT_FILE" 2>/dev/null || echo 0)
if [ "$COUNT" -ge "${WAKE_CYCLE_DAILY_CAP:-20}" ]; then
    echo "[<agent>] daily cap reached ($COUNT). Exiting."
    exit 0
fi

# ─── self-healing prompt sync ────────────────────────────────────────
VAULT_PROMPT="$HOME/Documents/Obsidian/<agent>/prompt.txt"
LOCAL_PROMPT="$HOME/.local/etc/<agent>/prompt.txt"
if [ -r "$VAULT_PROMPT" ] && [ "$VAULT_PROMPT" -nt "$LOCAL_PROMPT" ]; then
    cp "$VAULT_PROMPT" "$LOCAL_PROMPT" 2>/dev/null || true
fi

# ─── run the agent ───────────────────────────────────────────────────
TS=$(date +"%Y-%m-%dT%H-%M-%S")
LOG="$HOME/.local/var/log/<agent>/wake_${TS}.log"
exec > >(tee -a "$LOG") 2>&1

# Increment cycle counter only AFTER we've decided to actually run
echo $((COUNT + 1)) > "$COUNT_FILE"

# Hand off to claude / hermes / your agent CLI
"$HOME/.local/bin/claude" -p "$(cat "$LOCAL_PROMPT")" --max-turns 15
```

## Generated launchd plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.<agent>.wakecycle</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/<you>/.local/bin/<agent>-wake-cycle.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>1800</integer>  <!-- every 30 min -->
    <key>StandardOutPath</key>
    <string>/Users/<you>/.local/var/log/<agent>/launchd-stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/<you>/.local/var/log/<agent>/launchd-stderr.log</string>
</dict>
</plist>
```

## Pitfalls

- **macOS TCC is the silent killer.** launchd-spawned bash CANNOT read `~/Documents`, `~/Desktop`, `~/Downloads` without Full Disk Access. Symptom: random `Operation not permitted` 50% of the time. Fix: keep ALL paths under `~/.local/`.
- **Don't put the canonical prompt in the launchd path.** Edit it in your vault / git repo, sync to `~/.local/etc/` via the wrapper. Otherwise you'll edit the prompt and forget which copy is canonical.
- **Cost caps are non-negotiable.** Without a daily cap, a stuck loop can run 1000 times overnight. Daily cap + watchdog alert = sleep at night.
- **Test with `launchctl bootstrap` not `launchctl load`.** `load` is deprecated. Use:
  `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.<agent>.plist`

## Companion skills

Pair with:
- [`agent-watchdog`](../agent-watchdog) — hourly health check on this loop
- [`reflexion-debrief`](../reflexion-debrief) — daily debrief summarizing what the loop did
