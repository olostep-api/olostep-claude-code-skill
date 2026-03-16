# olostep-claude-code-skill

A [Claude Code skill](https://code.claude.com/docs/en/skills) that gives Claude context about the [Olostep](https://www.olostep.com/) web scraping and data extraction API.

Once installed, Claude Code knows how to use every Olostep endpoint — Scrapes, Searches, Answers, Maps, Batches, Crawls, Files, and Schedules — without you needing to paste docs each time.

## Install

**Personal** (available in all your projects):

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/olostep/olostep-claude-code-skill ~/.claude/skills/olostep
```

**Project-scoped** (shared with anyone using the repo):

```bash
# From your project root
git clone https://github.com/olostep/olostep-claude-code-skill .claude/skills/olostep
```

Claude Code auto-detects the skill — no restart needed.

## What's included

The `SKILL.md` covers:

- **Scrapes** — turn any URL into markdown, HTML, text, screenshots, or structured JSON
- **Searches** — semantic web search with deduplicated results
- **Answers** — AI-powered web research with structured JSON output and sources
- **Maps** — get all URLs on a website with glob filtering and pagination
- **Batches** — run many scrapes in parallel
- **Crawls** — crawl websites following links
- **Files** — upload context for scrapes and answers
- **Schedules** — cron-based recurring jobs
- **SDKs** — Python and Node.js usage

## Links

- [Olostep Docs](https://docs.olostep.com)
- [Olostep Playground](https://www.olostep.com/playground)
- [Python SDK](https://pypi.org/project/olostep/)
- [Node.js SDK](https://www.npmjs.com/package/olostep)
