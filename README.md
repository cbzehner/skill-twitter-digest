# Twitter Digest

Process Twitter/X bookmark exports into categorized markdown notes. The skill extracts useful ideas, writes vault entries, and keeps cookies and tokens out of logs.

## Use It For

- Turning saved tweets into reusable notes
- Categorizing bookmarks by topic
- Updating a local markdown vault from exports
- Optionally fetching or enriching bookmarks through Hermes Tweet when installed

## Install

Clone the repo and run the installer:

```bash
git clone https://github.com/cbzehner/skill-twitter-digest.git
cd skill-twitter-digest
./install.sh all
```

Install targets:

- `./install.sh claude` installs to `~/.claude/skills/twitter-digest`
- `./install.sh codex` installs to `~/.codex/skills/twitter-digest`
- `./install.sh agents` installs to `~/.agents/skills/twitter-digest`
- `./install.sh opencode` installs to `~/.config/opencode/skills/twitter-digest`
- `./install.sh all --copy` copies files instead of symlinking

Manual install works too: symlink or copy `skills/twitter-digest` into your agent's skills directory.

## Agent Support

This repo uses the plain `skills/twitter-digest/SKILL.md` layout. Claude Code and Codex also get small plugin manifests at `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`.

Other agents can read the same `SKILL.md` file. If a host does not support a frontmatter field or tool name, ignore that field and follow the workflow text.

## Optional Hermes Tweet Backend

Install Hermes Tweet when you want live bookmark fetches or richer tweet/article/thread enrichment:

```bash
hermes plugins install Xquik-dev/hermes-tweet --enable
hermes tools list
```

Configure the Hermes runtime with `XQUIK_API_KEY`. Do not paste keys or cookies into chat. See `skills/twitter-digest/references/hermes-tweet-source.md`.

## Layout

```text
.claude-plugin/plugin.json
.codex-plugin/plugin.json
install.sh
skills/twitter-digest/SKILL.md
README.md
LICENSE
```

## Public Notes

These repos are public. Keep private repo names, secrets, customer data, raw logs, cookies, and absolute filesystem paths out of examples.

## License

MIT
