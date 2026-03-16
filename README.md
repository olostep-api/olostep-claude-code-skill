# olostep-claude-code-skill

A [Claude Code skill](https://code.claude.com/docs/en/skills) that gives Claude deep technical knowledge of the [Olostep](https://www.olostep.com/) web scraping and data extraction API.

Once installed, Claude Code can act as a principal engineer when integrating Olostep — it knows every endpoint schema, async orchestration flow, pagination paradigm, and edge case without you pasting docs.

## Install

**Personal** (all your projects):

```bash
git clone https://github.com/olostep-api/olostep-claude-code-skill ~/.claude/skills/olostep
```

**Project-scoped** (shared with the repo):

```bash
git clone https://github.com/olostep-api/olostep-claude-code-skill .claude/skills/olostep
```

Claude Code auto-detects the skill — no restart needed.

## What's covered

The `SKILL.md` is an exhaustive technical reference covering:

- **Scrapes** — full request/response schema, actions, parsers, LLM extraction, screen sizes
- **Answers** — AI agent with structured JSON output, `NOT_FOUND` handling, sources
- **Batches** — step-by-step async orchestration: submit → poll → list items → retrieve
- **Crawls** — step-by-step async orchestration: kickoff → poll → list pages → retrieve
- **Maps** — domain URL discovery with glob filtering and cursor pagination
- **Searches** — semantic web search with deduplication
- **Retrieve** — shared content extraction endpoint for batches/crawls
- **Files** — context upload lifecycle
- **Schedules** — serverless cron for any endpoint
- **Webhooks** — completion notifications with retry behavior
- **Metadata** — key-value pairs with PATCH semantics
- **SDKs** — Python (sync + async) and Node.js patterns

## Links

- [Olostep Docs](https://docs.olostep.com)
- [Olostep Playground](https://www.olostep.com/playground)
- [Python SDK](https://pypi.org/project/olostep/)
- [Node.js SDK](https://www.npmjs.com/package/olostep)
