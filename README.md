# Olostep Claude Code Skill

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/olostep-api/olostep-claude-code-skill)](https://github.com/olostep-api/olostep-claude-code-skill/stargazers)

<p align="center">
  <img src="media/thumbnail.png" alt="Olostep Claude Code Skill" width="100%" />
</p>

An exhaustive [Claude Code skill](https://code.claude.com/docs/en/skills) for the [Olostep](https://www.olostep.com/) web scraping and data extraction API. Once installed, Claude Code can act as a principal engineer when integrating Olostep — it knows every endpoint schema, async orchestration flow, pagination paradigm, and edge case without you pasting docs.

Works with [Claude Code](https://claude.ai/claude-code) and any agent that supports skills.

**Contributions welcome!** [Open a PR](#contributing) or [an issue](https://github.com/olostep-api/olostep-claude-code-skill/issues).

## What's covered

| Endpoint | What Claude knows |
|----------|-------------------|
| **Scrapes** | Full request/response schema, actions, parsers, LLM extraction, screen sizes |
| **Answers** | AI agent with structured JSON output, `NOT_FOUND` handling, sources |
| **Batches** | Step-by-step async orchestration: submit → poll → list items → retrieve |
| **Crawls** | Step-by-step async orchestration: kickoff → poll → list pages → retrieve |
| **Maps** | Domain URL discovery with glob filtering and cursor pagination |
| **Searches** | Semantic web search with deduplication |
| **Retrieve** | Shared content extraction endpoint for batches/crawls |
| **Files** | Context upload lifecycle |
| **Schedules** | Serverless cron for any endpoint |
| **Webhooks** | Completion notifications with retry behavior |
| **Metadata** | Key-value pairs with PATCH semantics |
| **SDKs** | Python (sync + async) and Node.js patterns |

## Installation

### Claude Code (Recommended)

**Personal** (available across all your projects):

```bash
git clone https://github.com/olostep-api/olostep-claude-code-skill ~/.claude/skills/olostep
```

**Project-scoped** (shared with anyone using the repo):

```bash
git clone https://github.com/olostep-api/olostep-claude-code-skill .claude/skills/olostep
```

Claude Code auto-detects the skill — no restart needed.

<details>
<summary><strong>Other Methods</strong></summary>

```bash
# Git submodule
git submodule add https://github.com/olostep-api/olostep-claude-code-skill .claude/skills/olostep
```

</details>

<details>
<summary><strong>Updating</strong></summary>

```bash
# If cloned directly
cd ~/.claude/skills/olostep && git pull

# If using git submodule
git submodule update --remote .claude/skills/olostep
```

</details>

## Usage

Once installed, Claude Code automatically loads the skill when you're working with Olostep. You can also invoke it directly:

```
/olostep
```

Or just ask naturally — the skill triggers when Claude detects Olostep-related work:

> *"Scrape this URL and extract the product title and price as JSON"*

> *"Set up a batch job to scrape these 500 Google Search URLs with the parser"*

> *"Crawl docs.example.com and retrieve markdown for all pages under /guides/"*

## Links

- [Olostep Docs](https://docs.olostep.com)
- [Olostep Playground](https://www.olostep.com/playground)
- [Python SDK](https://pypi.org/project/olostep/)
- [Node.js SDK](https://www.npmjs.com/package/olostep)

## Contributing

PRs and issues are welcome. If you notice an endpoint change, a missing field, or want to add examples, please open a pull request.

## License

[MIT](LICENSE)
