---
name: twitter-digest
description: Process exported Twitter/X bookmarks, categorize insights, and update the knowledge vault. Use this whenever the user mentions bookmarks, saved tweets, Twitter digest, or wants to process and categorize their saved social media content.
argument-hint: "[path to export file]"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, WebFetch, Skill
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

Some bookmarks are X Articles (Notes) where the `text` field is just a URL like `x.com/i/article/...`. These need enrichment before categorization.

### Fetching X Article content

Use Twitter's internal GraphQL API with the `TweetResultByRestId` endpoint and `fieldToggles.withArticlePlainText: true`:

```bash
TWEET_ID="<tweet_id_from_bookmark_url>"
VARIABLES='{"tweetId":"'$TWEET_ID'","withCommunity":false,"includePromotedContent":false,"withVoice":false}'
FEATURES='{"articles_preview_enabled":true,"responsive_web_twitter_article_tweet_consumption_enabled":true,"longform_notetweets_consumption_enabled":true,"longform_notetweets_rich_text_read_enabled":true}'
FIELD_TOGGLES='{"withArticleRichContentState":false,"withArticlePlainText":true}'

curl -sG "https://x.com/i/api/graphql/qxWQxcMLiTPcavz9Qy5hwQ/TweetResultByRestId" \
  --data-urlencode "variables=$VARIABLES" \
  --data-urlencode "features=$FEATURES" \
  --data-urlencode "fieldToggles=$FIELD_TOGGLES" \
  -H "authorization: Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttfk8LF81IUq16cHjhLTvJu4FA33AGWWjCpTnA" \
  -H "x-csrf-token: $CT0" \
  -H "x-twitter-auth-type: OAuth2Session" \
  -H "cookie: auth_token=$AUTH_TOKEN; ct0=$CT0"
```

Response path: `data.tweetResult.result.article.article_results.result.plain_text` (full text) and `.title`.

Auth: uses the same `cookies.txt` from fetch-bookmarks.sh. The bearer token is hardcoded in Twitter's JS bundle (same for everyone). The GraphQL query ID may rotate — check gallery-dl releases if it breaks.

### Processing enriched articles

1. For each article bookmark, extract the tweet ID from the URL and fetch via the endpoint above
2. Replace the URL-only `text` with the fetched `plain_text` content
3. If fetch fails (rate limit, auth expired), categorize based on author and any surrounding text
4. Rate limit: ~2 requests/second is safe

For thread references (`x.com/.../status/...` links in the text), fetch the referenced tweet if the bookmark text alone lacks enough context to categorize.

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
