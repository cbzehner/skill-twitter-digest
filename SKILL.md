---
name: twitter-digest
description: Process exported Twitter/X bookmarks, categorize insights, and update the knowledge vault. Use this whenever the user mentions bookmarks, saved tweets, Twitter digest, or wants to process and categorize their saved social media content.
argument-hint: "[path to export file]"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
---

# Twitter Bookmark Digest

Process exported Twitter/X bookmarks, extract insights, and update the vault and CLAUDE.md.

## When NOT to Use

- **Posting or searching Twitter** — this skill processes exports, it doesn't interact with the Twitter API
- **Single articles or links** — just add them to the vault directly; this is for batch bookmark processing
- **Recalling past digests** — use `/seance` to find previous processing sessions

## Context

You are processing Twitter/X bookmarks for a startup founder and engineering leader interested in:
- **CS developments**: PLT, systems, algorithms, novel research
- **Agentic coding**: AI-assisted development, Claude Code techniques, LLM tooling, prompt engineering
- **Business & technology**: Startup automation, SaaS, developer tools, product strategy
- **Engineering leadership**: Team management, hiring, technical decision-making, org design

## Vault Location

```
VAULT_DIR=~/Developer/Personal/vault
INBOX_DIR=$VAULT_DIR/twitter-bookmarks/inbox
PROCESSED_DIR=$VAULT_DIR/twitter-bookmarks/processed
INSIGHTS_DIR=$VAULT_DIR/insights
```

## Step 1 — Find bookmark exports

Look for unprocessed bookmark files in `$INBOX_DIR/`. If `$ARGUMENTS` specifies a file path, use that instead. Supported formats:
- **JSON** (Twitter data export `bookmarks.js`, or browser extension exports)
- **CSV** (common extension format: columns like `text`, `url`, `author`, `created_at`)
- **Markdown** (manually saved threads or lists)

If no files found, tell the user and suggest how to export:
1. **X data export**: Settings → Your Account → Download an archive → extract `data/bookmarks.js`
2. **Browser extensions**: "Bookmark Bird", "Dewey", or similar → export as CSV/JSON
3. **Manual**: Copy-paste interesting threads into a `.md` file in the inbox folder

## Step 2 — Read and categorize

For each bookmark, determine:
1. **Category**: `cs-developments`, `agentic-coding`, `business-tech`, `engineering-leadership`, or `skip` (memes, noise, off-topic)
2. **Key insight**: One-line summary of what's valuable
3. **Actionable?**: Is there something concrete to try, adopt, or investigate?
4. **Source**: Author + link for attribution

## Step 3 — Update the vault

For each category, append new entries to the corresponding file in `$INSIGHTS_DIR/<category>/`. Use this format:

```markdown
## [Short Title] — @author
_Source: [link] | Date processed: YYYY-MM-DD_

Key insight summary.

**Action items** (if any):
- [ ] Concrete next step
```

Group related bookmarks into the same note if they cover the same topic. Create new files per topic rather than one giant file per category. Use descriptive filenames like `rust-async-runtimes.md` or `ai-code-review-patterns.md`.

Use `[[wikilinks]]` to connect related notes across categories. Tag notes with `#actionable`, `#tool`, `#pattern`, or `#question` as appropriate.

## Step 4 — Surface CLAUDE.md updates

If any bookmarks suggest:
- **New tools or libraries** the user should know about → suggest adding to CLAUDE.md
- **Workflow improvements** for Claude Code or agentic coding → suggest a feedback memory or CLAUDE.md update
- **Patterns to adopt** in their codebase → suggest a feedback memory

Present these as suggestions — don't auto-modify CLAUDE.md without confirmation.

## Step 5 — Archive processed files

Move processed files from `$INBOX_DIR/` to `$PROCESSED_DIR/` with a date prefix (e.g., `2026-03-27_bookmarks.json`).

## Step 6 — Summary

Output a brief digest:
- Total bookmarks processed
- Breakdown by category (count)
- Top 3-5 most actionable insights
- Any suggested CLAUDE.md or memory updates
