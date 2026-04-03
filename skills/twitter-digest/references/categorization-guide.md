# Categorization Guide

## User Interest Categories

- **CS developments**: PLT, systems, algorithms, novel research
- **Agentic coding**: AI-assisted development, agent workflow techniques, LLM tooling, prompt engineering
- **Business & technology**: Startup automation, SaaS, developer tools, product strategy
- **Engineering leadership**: Team management, hiring, technical decision-making, org design

## Per-Bookmark Decisions

For each bookmark, determine:
1. **Category**: `cs-developments`, `agentic-coding`, `business-tech`, `engineering-leadership`, or `skip` (memes, noise, off-topic)
2. **Key insight**: One-line summary of what's valuable
3. **Actionable?**: Is there something concrete to try, adopt, or investigate?
4. **Source**: Author + link for attribution

## Vault Entry Format

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
