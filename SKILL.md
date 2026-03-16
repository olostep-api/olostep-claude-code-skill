---
name: olostep
description: Olostep web scraping and data extraction API reference. Use when building with Olostep, calling /v1/scrapes, /v1/searches, /v1/answers, /v1/maps, /v1/batches, /v1/crawls, /v1/retrieve, /v1/files, /v1/schedules, or working with web scraping, search, or AI answer endpoints.
---

# Olostep API — Complete Technical Reference

Base URL: `https://api.olostep.com` | Auth: `Authorization: Bearer <KEY>` | JSON in/out

---

## 1. Scrapes: `POST /v1/scrapes`

Synchronous. Extract markdown, HTML, text, screenshots, or structured JSON from any URL. Returns instantly.

### Request

```json
{
  "url_to_scrape": "https://example.com",        // REQUIRED. The only mandatory field.
  "formats": ["markdown", "html", "text", "json", "screenshot", "raw_pdf"],
  "wait_before_scraping": 2000,                   // ms to wait for JS rendering
  "country": "US",                                // US|CA|IT|IN|GB|JP|MX|AU|ID|UA|RU|RANDOM
  "remove_css_selectors": "default",              // "default"|"none"|["nav",".popup"]
  "transformer": "postlight",                     // "postlight"|"none" — Mercury parser for cleaning
  "remove_images": false,
  "remove_class_names": ["ad-banner"],
  "actions": [
    { "type": "wait", "milliseconds": 1500 },
    { "type": "click", "selector": "#load-more" },
    { "type": "fill_input", "selector": "#search", "value": "query" },
    { "type": "scroll", "direction": "down", "amount": 1000 }
  ],
  "parser": { "id": "@olostep/google-search" },   // Structured JSON via built-in parser
  "llm_extract": {                                 // OR: LLM fills empty-value schema (20 credits)
    "schema": { "title": "", "price": "" },
    "prompt": "Extract product info"               // Optional natural language guidance
  },
  "links_on_page": {
    "absolute_links": true,
    "include_links": ["/blog/**"],
    "exclude_links": ["*.pdf"],
    "query_to_order_links_by": "pricing"
  },
  "screen_size": { "screen_type": "desktop" },     // "default"(1024x768)|"mobile"(414x896)|"desktop"(1920x1080)
  "screenshot": { "full_page": true },
  "metadata": {}
}
```

### Response

```json
{
  "id": "scrape_6h89o8u1kt",
  "object": "scrape",
  "created": 1745673871,
  "retrieve_id": "6h89o8u1kt",                    // Used with /v1/retrieve for batches/crawls
  "url_to_scrape": "https://example.com",
  "result": {
    "markdown_content": "# Header...",             // null if not in formats
    "html_content": "<main>...</main>",
    "text_content": null,
    "json_content": "{\"title\":\"...\"}",         // STRINGIFIED JSON — must be parsed
    "screenshot_hosted_url": "https://olostep-storage.s3...",
    "html_hosted_url": "https://olostep-storage...",
    "markdown_hosted_url": "https://olostep-storage...",
    "json_hosted_url": null,
    "text_hosted_url": null,
    "links_on_page": ["https://example.com/page1"],
    "page_metadata": { "status_code": 200, "title": "Example Domain" }
  }
}
```

`GET /v1/scrapes/{scrape_id}` — retrieve by ID. Same shape, plus `size_exceeded` boolean (use hosted S3 URLs if true).

**Pricing:** 1 credit base. 1-5 with parsers. 20 with `llm_extract`.

---

## 2. Answers: `POST /v1/answers`

Synchronous (3-30s). High-level AI agent: searches the web, scrapes pages, validates data, returns structured answers with sources. Replaces manual search+scrape orchestration.

### Request

```json
{
  "task": "Who is the CEO of Stripe, total funding, and HQ location?",  // REQUIRED
  "json": {                    // OPTIONAL — empty-value object as schema guide
    "ceo": "",
    "total_funding": "",
    "headquarters": ""
  }
}
```

Without `json`, returns plain text in `result.json_content`.

### Response

CRITICAL: Code must handle `"NOT_FOUND"` — Olostep returns this string for fields it cannot independently verify across web sources.

```json
{
  "id": "answer_99xx",
  "object": "answer",
  "created": 1760327323,
  "task": "Who is the CEO of Stripe...",
  "result": {
    "json_content": "{\"ceo\":\"Patrick Collison\",\"total_funding\":\"$8.7B\",\"headquarters\":\"SF and Dublin\"}",
    "json_hosted_url": "https://...",
    "sources": [                // URLs the agent physically scraped to assemble this answer
      "https://stripe.com/about",
      "https://crunchbase.com/stripe"
    ]
  }
}
```

`result.json_content` is **stringified JSON** — parse it. Without the `json` param: `{ "result": "Plain text answer" }`.

`GET /v1/answers/{answer_id}` — retrieve by ID. Same shape.

**Pricing:** 20 credits.

---

## 3. Batches Pipeline: `POST /v1/batches`

Async. Process 1-10,000 URLs in parallel. Completes in constant ~5-8 min.
New/unverified accounts capped at 100 items/batch.

### ORCHESTRATION — Execute in this exact sequence:

**Step 1: Submit** `POST /v1/batches`

```json
{
  "items": [
    { "custom_id": "db_hash_1", "url": "https://google.com/search?q=saas" },
    { "custom_id": "db_hash_2", "url": "https://google.com/search?q=b2b" }
  ],
  "parser": { "id": "@olostep/google-search" },   // Applied to every item
  "country": "US",
  "webhook": "https://your-server.com/hook",       // Optional — POST on completion
  "metadata": { "project": "q1-research" }         // Arbitrary key-value, max 50 keys
}
```

Returns: `{ "id": "batch_xxx", "status": "in_progress", "total_urls": 2, "completed_urls": 0 }`

**Step 2: Poll** `GET /v1/batches/{batch_id}`

Poll every ~10s. Inspect `total_urls`, `completed_urls`, `failed_urls`.
Proceed when: `status == "completed"`.

**Step 3: List items** `GET /v1/batches/{batch_id}/items?limit=50&cursor={cursor}`

Cursor-paginated. Returns:
```json
{
  "items": [
    { "custom_id": "db_hash_1", "url": "https://...", "retrieve_id": "abc890", "status": "completed" },
    { "custom_id": "db_hash_2", "url": "https://...", "retrieve_id": "def456", "status": "completed" }
  ],
  "cursor": "next_token"      // null when done
}
```

**Step 4: Retrieve content** `GET /v1/retrieve?retrieve_id=abc890&formats=markdown,json`

See [Retrieve endpoint](#7-retrieve-get-v1retrieve) below.

**Update:** `PATCH /v1/batches/{batch_id}` — add URLs or update metadata on an in-progress batch.

---

## 4. Crawls Pipeline: `POST /v1/crawls`

Async (1-10 min). Agent-based recursive website exploration from a root URL.

### ORCHESTRATION — Execute in this exact sequence:

**Step 1: Kickoff** `POST /v1/crawls`

```json
{
  "start_url": "https://docs.company.com",         // REQUIRED
  "max_pages": 500,                                 // REQUIRED — enforce strict ceiling
  "max_depth": 3,                                   // Optional link-depth limit
  "include_urls": ["/blog/**", "/product/**"],      // Glob patterns — restrict agent to these routes
  "exclude_urls": ["/old/**", "**/tags/**"],         // Supersedes include_urls
  "include_external": false,                        // Crawl first-degree external links
  "include_subdomain": false,                       // Include subdomains (default false)
  "search_query": "contact pricing SLA",            // Semantic relevance filter within crawl
  "top_n": 10,                                      // Hard cap on relevance yield per page
  "follow_robots_txt": true,                        // Respect robots.txt (default true)
  "timeout": 300,                                   // End crawl after N seconds
  "webhook": "https://your-server.com/hook",        // POST on completion
  "scrape_options": {
    "formats": ["markdown", "screenshot"],           // Per-page scrape config
    "parser": "@olostep/extract-emails"              // Auto-adds json to formats
  }
}
```

Returns: `{ "id": "crawl_xxx", "status": "in_progress", "pages_count": 0 }`

**Step 2: Poll** `GET /v1/crawls/{crawl_id}`

Poll every ~5-10s. Track `status`, `pages_count`, `current_depth`.
Proceed when: `status == "completed"`. Or use `webhook`.

**Step 3: List pages** `GET /v1/crawls/{crawl_id}/pages?limit=10&cursor={cursor}`

Cursor is integer-based (start at 0). Iterate until `cursor` absent from response.

```json
{
  "pages": [
    { "id": "page_1", "url": "https://...", "retrieve_id": "xyz789", "is_external": false }
  ],
  "cursor": 10,               // Next offset. Absent when all pages returned.
  "pages_count": 47,
  "metadata": {
    "external_urls": ["https://external.com/..."],
    "failed_urls": ["https://broken.com/404"]
  }
}
```

NOTE: `html_content`/`markdown_content` fields on pages response are **deprecated**. Use `/v1/retrieve`.

**Step 4: Retrieve content** `GET /v1/retrieve?retrieve_id=xyz789`

See below.

**Pricing:** 1 credit per page crawled.

---

## 5. Maps: `POST /v1/maps`

Synchronous (seconds to 120s). Returns ALL distinct URLs on a domain. Yields 100k+ URLs via internal scraping + sitemap harvesting. Max 10MB per response, paginated via cursor.

### Request

```json
{
  "url": "https://stripe.com",                     // REQUIRED
  "include_urls": ["/guides/**"],                   // Glob path filters
  "exclude_urls": ["/de-de/**", "/fr-fr/**"],
  "top_n": 50000,                                   // Limit total URLs
  "include_subdomain": true,                        // Default true
  "search_query": "pricing enterprise",             // Sort by relevance
  "cursor": null                                    // Pagination token from previous response
}
```

### Response & Pagination

```json
{
  "id": "map_abc123",
  "urls_count": 50000,
  "urls": ["https://stripe.com/guides/...", "..."],
  "cursor": "next_page_token"   // null when all retrieved
}
```

To get next page: `POST /v1/maps` with `{ "url": "https://stripe.com", "cursor": "next_page_token" }`. Stop when `cursor` is null/absent.

**Pricing:** 1 credit + 1 per extra 1,000 URLs.

---

## 6. Searches: `POST /v1/searches`

Synchronous. Semantic web search. Expands query into multiple targeted searches, runs in parallel, deduplicates results. Bypasses need for raw Google/Bing parser scraping.

### Request

```json
{
  "query": "Who manufactures NVDA semiconductor fabs in Q1 2026?"  // REQUIRED
}
```

### Response

```json
{
  "id": "search_9bi0sbj9xa",
  "object": "search",
  "created": 1760327323,
  "query": "Who manufactures...",
  "result": {
    "links": [
      {
        "url": "https://bloomberg.com/...",
        "title": "TSMC remains primary NVDA foundry in early 2026",
        "description": "An analysis into TSMC production scaling..."
      }
    ],
    "json_content": "...",
    "json_hosted_url": "https://..."
  }
}
```

Code action: Feed `result.links[].url` values into `POST /v1/scrapes` if deep content extraction is needed from a given result.

`GET /v1/searches/{search_id}` — retrieve by ID. Same shape.

**Pricing:** 5 credits.

---

## 7. Retrieve: `GET /v1/retrieve`

Critical shared endpoint for extracting content from Batches and Crawls. Uses `retrieve_id` from batch items or crawl pages.

```
GET /v1/retrieve?retrieve_id=abc890&formats=markdown,json
```

- `retrieve_id` — REQUIRED. From `/v1/batches/{id}/items` or `/v1/crawls/{id}/pages` or `/v1/scrapes/{id}`
- `formats` — OPTIONAL. `html`, `markdown`, `json`. Omit to get all available.

### Response

```json
{
  "html_content": "<main>...</main>",
  "markdown_content": "# Page...",
  "json_content": "{\"extracted\": \"data\"}",    // Stringified — parse it
  "html_hosted_url": "https://olostep-storage...",
  "markdown_hosted_url": "https://olostep-storage...",
  "json_hosted_url": "https://olostep-storage...",
  "size_exceeded": false       // If true, inline content is truncated — use hosted S3 URLs
}
```

S3 hosted URLs expire in 7 days.

---

## 8. Files: `/v1/files`

Upload JSON context files for use in answers/scrapes. Max 200MB. Expire after 30 days.

Lifecycle: `POST /v1/files` (get presigned URL + file_id) → upload to presigned URL → `POST /v1/files/{file_id}/complete` → use in requests.

- `POST /v1/files` — create upload. `purpose`: `"context"` (default) or `"batch"`
- `POST /v1/files/{file_id}/complete` — finalize
- `GET /v1/files/{file_id}` — metadata
- `GET /v1/files/{file_id}/content` — content (accepts `expires_in`)
- `GET /v1/files` — list (filter by `purpose`)
- `DELETE /v1/files/{file_id}` — delete

**Free.** Only charged for API calls that consume the file.

---

## 9. Schedules: `/v1/schedules`

Serverless cron. Schedule any Olostep endpoint or external URL.

```json
{
  "endpoint": "v1/scrapes",              // Short-form auto-prefixed, or full URL
  "method": "POST",
  "payload": { "url_to_scrape": "https://example.com", "formats": ["markdown"] },
  "cron_expression": "0 10 * * ? *",     // OR "text": "every day at 10am"
  "timezone": "America/New_York",        // IANA timezone
  "execute_at": "2026-04-01T10:00:00Z"   // For one-time execution instead of cron
}
```

- `POST /v1/schedules` — create
- `GET /v1/schedules` — list
- `GET /v1/schedules/{schedule_id}` — get
- `DELETE /v1/schedules/{schedule_id}` — delete

One-time schedules auto-delete after execution. Cron format: 6 fields (min hour day month day-of-week year).

**Free.** Only charged for the executed API calls.

---

## 10. Webhooks

Pass `webhook` (or `webhook_url`) on batch/crawl creation. Receives HTTP POST on completion.

Envelope: `{ "id": "evt_...", "object": "batch.completed"|"crawl.completed", "timestamp": "...", "delivery_attempt": "1/5", "data": { ...resource... } }`

Retries: 5 attempts, exponential backoff over 30 minutes. 30s per-request timeout.
Best practice: respond 200 immediately, process async. Use `id` for idempotency.

---

## 11. Metadata

Available on Batches (and coming soon for other resources). Arbitrary string key-value pairs.

Constraints: max 50 keys, key max 40 chars, value max 500 chars, strings only (numbers/booleans auto-coerced).

PATCH behavior: `{ "metadata": { "new_key": "val" } }` adds/updates. `{ "metadata": { "key": null } }` deletes. `{ "metadata": null }` clears all.

---

## SDKs

```bash
pip install olostep    # Python 3.11+
npm install olostep    # Node.js
```

### Python

```python
from olostep import Olostep

client = Olostep(api_key="...")

# Scrape
scrape = client.scrapes.create(url_to_scrape="https://example.com", formats=["markdown"])

# Batch — SDK handles polling internally
batch = client.batches.create(urls=["https://a.com", "https://b.com"])
for item in batch.items():
    content = item.retrieve(["markdown"])

# Crawl
crawl = client.crawls.create(start_url="https://example.com", max_pages=100)
for page in crawl.pages():
    content = page.retrieve(["html"])

# Maps
maps = client.maps.create(url="https://example.com")
for url in maps.urls():
    print(url)

# Answers
answer = client.answers.create(task="What is X?", json={"answer": ""})

# Retrieve
content = client.retrieve.get(retrieve_id="abc123", formats=["markdown", "json"])
```

### Async Python

```python
from olostep import AsyncOlostep

async with AsyncOlostep(api_key="...") as client:
    scrape = await client.scrapes.create(url_to_scrape="https://example.com")
    async for item in (await client.batches.create(urls=[...])).items():
        content = await item.retrieve(["html"])
```

### Node.js

```typescript
import Olostep from 'olostep';
const client = new Olostep({ apiKey: '...' });

const scrape = await client.scrapes.create('https://example.com');
const batch = await client.batches.create(['https://a.com', 'https://b.com']);
await batch.waitTillDone();
for await (const item of batch.items()) { /* ... */ }
const content = await client.retrieve(retrieveId, 'markdown');
```

---

## Critical Implementation Rules

When building any architecture using Olostep:

1. **DO NOT invent paths, property names, or pagination structures.** Conform to the schemas above.
2. **Scrapes, Searches, Answers are synchronous.** Batches and Crawls are async and require the poll → list → retrieve pipeline.
3. **`json_content` is always a stringified JSON string.** Always `JSON.parse()` / `json.loads()` it.
4. **`NOT_FOUND`** is returned as a string value in Answers when data cannot be verified. Handle it.
5. **`size_exceeded`** — when true on retrieve/scrape responses, inline content is truncated. Use the `*_hosted_url` S3 links instead.
6. **Cursor pagination** — Batches/Maps use string cursors (null when done). Crawl pages use integer cursors (absent when done).
7. **`retrieve_id`** is the bridge between batch items / crawl pages and their actual content via `GET /v1/retrieve`.
8. S3 hosted URLs expire in **7 days**.

Docs: https://docs.olostep.com
