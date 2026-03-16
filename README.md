# olostep-claude-code-skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that gives Claude context about the [Olostep](https://www.olostep.com/) web scraping and data extraction API.

Once installed, Claude Code will know how to use every Olostep endpoint — Scrapes, Searches, Answers, Maps, Batches, Crawls, Files, and Schedules — without you having to paste docs into every conversation.

## Install

Run this inside any project where you want Claude to have Olostep API knowledge:

```bash
claude install-skill https://github.com/olostep/olostep-claude-code-skill
```

## What's included

The `CLAUDE.md` file contains a concise reference covering:

- **Scrapes** (`/v1/scrapes`) — turn any URL into markdown, HTML, text, screenshots, or structured JSON
- **Searches** (`/v1/searches`) — semantic web search with deduplicated results
- **Answers** (`/v1/answers`) — AI-powered web research with structured JSON output and sources
- **Maps** (`/v1/maps`) — get all URLs on a website with glob filtering and pagination
- **Batches** (`/v1/batches`) — run many scrapes in parallel
- **Crawls** (`/v1/crawls`) — crawl a website following links automatically
- **Files** (`/v1/files`) — upload context files for scrapes and answers
- **Schedules** (`/v1/schedules`) — schedule recurring scrapes or answers
- **SDKs** — Python and Node.js client usage

## Links

- [Olostep Documentation](https://docs.olostep.com)
- [Olostep Playground](https://www.olostep.com/playground)
- [Python SDK](https://pypi.org/project/olostep/)
- [Node.js SDK](https://www.npmjs.com/package/olostep)
