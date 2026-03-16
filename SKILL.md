---
name: olostep
description: Olostep web scraping and data extraction API. Use when building with Olostep, calling /v1/scrapes, /v1/searches, /v1/answers, /v1/maps, /v1/batches, /v1/crawls, or working with web scraping, search, or AI answer endpoints.
---

# Olostep API

Base: `https://api.olostep.com` | Auth: `Authorization: Bearer <KEY>` | JSON in/out

## Scrapes `POST /v1/scrapes`

Extract markdown, HTML, text, screenshots, or structured JSON from any URL.

```json
{
  "url_to_scrape": "https://example.com",
  "formats": ["markdown", "html", "text", "json", "screenshot", "raw_pdf"],
  "country": "US",
  "wait_before_scraping": 1000,
  "remove_css_selectors": "default",
  "actions": [{ "type": "click", "selector": ".load-more" }],
  "parser": { "id": "@olostep/google-search" },
  "llm_extract": { "schema": { "title": "", "price": "" } },
  "links_on_page": { "include_links": ["/blog/**"] },
  "metadata": {}
}
```

Only `url_to_scrape` is required.

- `actions`: `wait`, `click`, `fill_input`, `scroll` — for JS-rendered / interactive pages
- `parser.id`: structured extraction — `@olostep/google-search`, `@olostep/google-maps`, etc.
- `llm_extract.schema`: pass empty-value JSON object, API fills it via LLM (20 credits)
- `country`: US, CA, IT, IN, GB, JP, MX, AU, ID, UA, RU, RANDOM

Response: `{ id, object, created, metadata, url_to_scrape, result: { markdown_content, html_content, text_content, json_content, *_hosted_url, links_on_page, page_metadata } }`

`GET /v1/scrapes/{scrape_id}` — retrieve by ID, same shape.

**1 credit** (base). 1-5 with parsers. 20 with `llm_extract`.

## Searches `POST /v1/searches`

Semantic web search. Expands query into multiple searches, runs in parallel, deduplicates.

```json
{ "query": "Best AEO startups" }
```

Response: `{ id, object, created, metadata, query, result: { links: [{ url, title, description }], json_content, json_hosted_url } }`

`GET /v1/searches/{search_id}` — retrieve by ID.

**5 credits.**

## Answers `POST /v1/answers`

AI-powered web research. Returns structured answers with sources.

```json
{
  "task": "What is the latest book by J.K. Rowling?",
  "json": { "book_title": "", "author": "", "release_date": "" }
}
```

Only `task` is required. `json` guides output shape — pass empty-value object as schema. Without it, returns `{ "result": "plain text" }`. Unverifiable fields return `"NOT_FOUND"`.

Response: `{ id, object, created, metadata, task, result: { json_content, json_hosted_url, sources: ["url1", "url2"] } }`

`result.json_content` is stringified JSON — parse it.

`GET /v1/answers/{answer_id}` — retrieve by ID.

**20 credits.** 3-30s execution.

## Maps `POST /v1/maps`

Get all URLs on a website.

```json
{
  "url": "https://example.com",
  "include_urls": ["/blog/**"],
  "exclude_urls": ["/careers/**"],
  "top_n": 1000,
  "include_subdomain": true,
  "cursor": null
}
```

Only `url` is required. Glob patterns for filtering. Cursor-based pagination (10MB max per response).

Response: `{ id, urls_count, urls: [...], cursor }`

`cursor` is null when all URLs retrieved.

**1 credit** + 1 per extra 1,000 URLs.

## Batches `POST /v1/batches`

Run many scrapes in parallel.

- `POST /v1/batches` — create batch with URLs + scrape config
- `GET /v1/batches/{batch_id}` — status and progress
- `PATCH /v1/batches/{batch_id}` — update (e.g. add URLs)
- `GET /v1/batches/{batch_id}/items` — individual results

## Crawls `POST /v1/crawls`

Crawl a website following links.

- `POST /v1/crawls` — start crawl
- `GET /v1/crawls/{crawl_id}` — status
- `GET /v1/crawls/{crawl_id}/pages` — crawled pages

## Files `POST /v1/files`

Upload context files for scrapes and answers.

- `POST /v1/files` — create upload
- `POST /v1/files/{file_id}/complete` — mark complete
- `GET /v1/files/{file_id}` — metadata
- `GET /v1/files/{file_id}/content` — content
- `GET /v1/files` — list
- `DELETE /v1/files/{file_id}` — delete

## Schedules `POST /v1/schedules`

Cron-based recurring scrapes or answers.

- `POST /v1/schedules` — create
- `GET /v1/schedules` — list
- `GET /v1/schedules/{schedule_id}` — get
- `DELETE /v1/schedules/{schedule_id}` — delete

## SDKs

```bash
pip install olostep
npm install olostep
```

```python
from olostep import Olostep
client = Olostep(api_key="...")
scrape = client.scrapes.create(url_to_scrape="https://example.com", formats=["markdown"])
answer = client.answers.create(task="Who founded Stripe?")
```

```javascript
import Olostep from 'olostep';
const client = new Olostep({ apiKey: '...' });
const scrape = await client.scrapes.create({ urlToScrape: 'https://example.com', formats: ['markdown'] });
```

## Patterns

- Object IDs are prefixed: `scrape_`, `search_`, `answer_`, `map_`, `batch_`, `crawl_`
- Large content is hosted on S3 via `*_hosted_url` fields
- `metadata` accepts arbitrary key-value pairs for tracking
- Errors: 400 bad request, 401 invalid key, 404 not found, 500 server error

Docs: https://docs.olostep.com
