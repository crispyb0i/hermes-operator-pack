---
name: obsidian-vault-curator
description: "Maintain an Obsidian vault to a strict schema: frontmatter, wikilinks, MOCs, index.md + log.md. The schema my LLMs follow."
version: 1.0.0
author: David Shin (dvdshn.com)
license: MIT
metadata:
  hermes:
    tags: [obsidian, knowledge-base, llm-context, frontmatter, wikilinks]
    related_skills: [reflexion-debrief]
---

# Obsidian Vault Curator

A complete `AGENTS.md` template + curator skill that turns your Obsidian vault into a high-signal, LLM-friendly knowledge base. Loads as a Hermes skill so any agent operating on the vault follows the same conventions.

## When to use

- You use Obsidian as a knowledge base for LLM agents (Claude Code, Hermes, Codex)
- You want consistent frontmatter, wikilinks, MOCs, and an index.md / log.md discipline
- You're tired of agents creating orphan pages with no frontmatter and no backlinks

## What the schema enforces

1. **Required YAML frontmatter** on every page (`type`, `app`, `date`, `status`).
2. **Sources/ folder is immutable** — raw material is never edited; synthesis happens in wiki layer.
3. **Aggressive cross-linking** — every product/topic mention links to its MOC.
4. **MOCs (Maps of Content)** prefixed with 🗺️ as navigation hubs.
5. **`index.md` + `log.md` always current** — every ingest, every new page, every lint pass.
6. **Prefer updating existing pages over creating new ones.**
7. **Debriefs are append-only history** — never edit past debriefs.

## Setup

```bash
# Install the skill
hermes skills install https://raw.githubusercontent.com/crispyb0i/hermes-operator-pack/main/skills/obsidian-vault-curator/SKILL.md

# Copy the AGENTS.md template into your vault root
cp ~/.hermes/skills/obsidian-vault-curator/AGENTS.md.template "$OBSIDIAN_VAULT/AGENTS.md"
# Then customize the products/agents/folder layout to fit your vault
```

## Operating mode

The skill teaches the agent four workflows:

- **Ingest** — new source → save raw → update 5–15 wiki pages → update index + log
- **Query** — answer a question → optionally file as new wiki page → update index + log
- **Lint** — check for contradictions, orphans, stale claims, frontmatter gaps
- **Sync** — after a coding session, update Architecture / Tech Debt / Debriefs

## Required environment

```bash
OBSIDIAN_VAULT_PATH=/Users/you/Documents/Obsidian      # absolute path to vault
```

## Why this works

Most "LLM + Obsidian" setups fail because the LLM creates pages with inconsistent structure, no backlinks, and no maintenance discipline. The vault becomes write-only — adding pages but never linking them. Three months in, search-by-grep is the only way to find anything.

The curator skill solves this by **front-loading the schema as a constraint** the agent reads at the start of every session. The schema is short enough to fit in context but strict enough that the agent stops generating slop. The result is a vault that gets *more* useful over time, not less.

## Pitfalls

- **The schema is opinionated.** If you already have a vault, migrating means adding frontmatter to every page. Worth it long-term, painful upfront.
- **MOCs need maintenance.** They drift. Run a monthly lint pass that checks every page is reachable from at least one MOC.
- **Don't let the agent edit Sources/.** It will try to "fix typos" in raw transcripts. Block this in the schema and reinforce in every session.

## Linked file

- `AGENTS.md.template` — the full schema template, ready to drop into a vault root
