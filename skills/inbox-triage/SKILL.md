---
name: inbox-triage
description: "Gmail label + draft replies via the gws CLI or Gmail API. Inbox-zero on autopilot."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [gmail, email, triage, gws, founder, automation]
    related_skills: [daily-briefing]
---

# Inbox Triage

Auto-label new Gmail threads (lead / customer / billing / vendor / spam) and draft replies for the easy ones. You wake up to an inbox where every thread already has context and a starting point.

## When to use

- You get >20 emails/day and want labels + draft responses ready by morning
- You sell a product and want leads / customer questions surfaced immediately
- You already authenticated `gws` (`hermes skills install google-workspace`) or have a Gmail OAuth token

## Required environment

Either:
```bash
# Option A: gws CLI (recommended — handled by google-workspace skill)
gws auth status   # should show authenticated
```

Or:
```bash
# Option B: Direct Gmail API
GMAIL_CREDENTIALS_PATH=~/.hermes/google_token.json
```

## How it works

1. List unread threads in `INBOX` from the last 12h.
2. For each, read subject + first ~500 chars + sender domain.
3. Classify into one of: `lead`, `customer`, `billing`, `vendor`, `personal`, `newsletter`, `spam`.
4. Apply the matching Gmail label (creates labels if missing).
5. For `lead` and `customer`, draft a reply (saved as a Gmail draft, NOT sent).
6. Append a one-line summary to a `triage-log.md` so you can review what was classified.

## Classification rules (you can override)

| Label | Trigger |
|-------|---------|
| `lead` | New sender + mentions your product/service + asks a question |
| `customer` | Sender's email is in your Stripe `customers` list |
| `billing` | From `noreply@stripe.com`, `billing@*` |
| `vendor` | From a domain you've previously sent invoices to |
| `personal` | From your contacts + non-product subject |
| `newsletter` | List-Unsubscribe header present |
| `spam` | Already in spam folder OR matches a deny-list pattern |

## Suggested cron

```bash
hermes cron create "*/30 * * * *" --name inbox-triage \
  --skill inbox-triage \
  --prompt "Triage new Gmail threads. Apply labels. Draft replies for leads and customers. Don't send anything."
```

## Pitfalls

- **NEVER send drafts automatically.** Always save as draft and let a human review. One bad LLM-drafted reply to a customer can cost more than the time saved.
- **Stripe customer matching.** You need read-only API access to Stripe to identify customer emails. Cache in `~/.hermes/cache/stripe-customers.json` and refresh daily.
- **Gmail label limits.** Gmail caps at 10,000 labels. If you create per-sender labels you'll hit it eventually — stick to category labels.
- **OAuth scope drift.** If gws was authenticated before this skill was added, you may need to re-auth with the right Gmail modify scope.

## Output: draft reply quality

Drafts should always include:
- A one-line acknowledgement that fits the thread context
- A specific question or next step (not just "Thanks, I'll get back to you")
- Your signature

If the agent can't draft a confident reply, it should label the thread but skip the draft step. Empty drafts are noise.
