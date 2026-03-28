# Twitter Digest

Process exported Twitter/X bookmarks, categorize insights, and update a knowledge vault.

## What It Does

Reads bookmark exports (JSON, CSV, or Markdown), categorizes each by topic, extracts key insights, and writes them to an Obsidian-style vault with wikilinks and tags.

## Usage

```
/twitter-digest                          Process files in inbox
/twitter-digest ~/Downloads/bookmarks.json   Process a specific file
```

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

## License

MIT
