---
name: twitter-digest
description: Process exported Twitter/X bookmarks, categorize insights, and update the knowledge vault. Use this whenever the user mentions bookmarks, saved tweets, Twitter digest, or wants to process and categorize their saved social media content.
argument-hint: "[path to export file]"
arguments:
  - export_path
license: MIT
effort: medium
allowed-tools: Bash Read Write Edit Glob Grep Agent WebFetch Skill
# Note: Agent, WebFetch, and Skill are Claude Code tools. On other hosts,
# use equivalent capabilities or degrade gracefully.
---

# Twitter Bookmark Digest

Process exported Twitter/X bookmarks, extract insights, and update the vault plus any relevant agent guidance files.

## When NOT to Use

- **Posting or searching Twitter** — this skill processes exports, it doesn't interact with the Twitter API
- **Single articles or links** — just add them to the vault directly; this is for batch bookmark processing
- **Recalling past digests** — use `/seance` to find previous processing sessions

## Context

You are processing Twitter/X bookmarks for a startup founder and engineering leader. See [references/categorization-guide.md](references/categorization-guide.md) for interest categories, per-bookmark decision criteria, and vault entry format.

## Vault Location

```
VAULT_DIR=~/Developer/Personal/vault
INBOX_DIR=$VAULT_DIR/twitter-bookmarks/inbox
PROCESSED_DIR=$VAULT_DIR/twitter-bookmarks/processed
INSIGHTS_DIR=$VAULT_DIR/insights
```

## Step 0 — Fetch bookmarks (if inbox is empty)

If the inbox is empty and no file argument was provided, offer to fetch fresh bookmarks:

```bash
~/Developer/Personal/vault/twitter-bookmarks/fetch-bookmarks.sh
```

If cookies are missing, run with `--refresh-cookies` first (requires Chrome to be closed):

```bash
~/Developer/Personal/vault/twitter-bookmarks/fetch-bookmarks.sh --refresh-cookies
```

## Step 1 — Find bookmark exports

Look for unprocessed bookmark files in `$INBOX_DIR/`. If `$ARGUMENTS` specifies a file path, use that instead. Supported formats:
- **JSON from fetch-bookmarks.sh** (preferred — array of `{text, author, author_handle, url, date}`): the automated pipeline output
- **JSON** (Twitter data export `bookmarks.js`, or browser extension exports)
- **CSV** (common extension format: columns like `text`, `url`, `author`, `created_at`)
- **Markdown** (manually saved threads or lists)

If no files found and fetch script isn't available, suggest manual export:
1. **X data export**: Settings → Your Account → Download an archive → extract `data/bookmarks.js`
2. **Browser extensions**: "Bookmark Bird", "Dewey", or similar → export as CSV/JSON
3. **Manual**: Copy-paste interesting threads into a `.md` file in the inbox folder

## Step 1.5 — Enrich articles and threads

Some bookmarks are X Articles (Notes) where the `text` field is just a URL. These need enrichment before categorization.

See [references/twitter-api-enrichment.md](references/twitter-api-enrichment.md) for the GraphQL endpoint, auth details, and processing steps.

## Step 2 — Read and categorize

For each bookmark, determine category, key insight, actionability, and source. See [references/categorization-guide.md](references/categorization-guide.md) for the full decision criteria and category definitions.

## Step 3 — Update the vault

For each category, append new entries to the corresponding file in `$INSIGHTS_DIR/<category>/`. See [references/categorization-guide.md](references/categorization-guide.md) for the vault entry format, file naming conventions, and wikilink/tag guidance.

## Step 4 — Surface agent guidance updates

If any bookmarks suggest:
- **New tools or libraries** the user should know about → suggest adding to `AGENTS.md`, `CLAUDE.md`, or the relevant local guidance file
- **Workflow improvements** for agentic coding → suggest a guidance-file update or persistent note
- **Patterns to adopt** in their codebase → suggest a persistent note for future sessions

Present these as suggestions — don't auto-modify guidance files without confirmation.

## Step 5 — Archive processed files

Move processed files from `$INBOX_DIR/` to `$PROCESSED_DIR/` with a date prefix (e.g., `2026-03-27_bookmarks.json`).

## Step 6 — Summary

Output a brief digest:
- Total bookmarks processed
- Breakdown by category (count)
- Top 3-5 most actionable insights
- Any suggested guidance-file or memory updates
