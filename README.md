# Twitter Digest

Process exported or Hermes Tweet-fetched Twitter/X bookmarks, categorize insights, and update a knowledge vault.

## What It Does

Reads bookmark exports (JSON, CSV, or Markdown), or fetches bookmarks through an optional Hermes Tweet route, categorizes each by topic, extracts key insights, and writes them to an Obsidian-style vault with wikilinks and tags.

## Usage

```
/twitter-digest                          Process files in inbox
/twitter-digest ~/Downloads/bookmarks.json   Process a specific file
```

If Hermes Tweet is installed in Hermes Agent, the skill can fetch fresh bookmarks and enrich X Articles or threads through the `tweet_explore`, `tweet_read`, and approval-gated `tweet_action` tools before processing.

## Categories

- **CS developments** - PLT, systems, algorithms, research
- **Agentic coding** - AI-assisted development, LLM tooling
- **Business & technology** - Startups, SaaS, developer tools
- **Engineering leadership** - Teams, hiring, org design

## Installation

### From Marketplace

```bash
/plugin marketplace add cbzehner/skill-twitter-digest
/plugin install twitter-digest@cbzehner
```

### Manual Installation

```bash
cd ~/.claude/skills/
git clone https://github.com/cbzehner/skill-twitter-digest.git twitter-digest
```

### Optional Hermes Tweet Backend

Install Hermes Tweet when you want live bookmark fetches or richer tweet/article/thread enrichment:

```bash
hermes plugins install Xquik-dev/hermes-tweet --enable
hermes tools list
```

Configure the Hermes runtime with `XQUIK_API_KEY`. Do not paste keys or cookies into chat.

## License

MIT
