# Olostep API

Base URL: `https://api.olostep.com`
Auth: `Authorization: Bearer <API_KEY>`
All request/response bodies are JSON.

---

## Scrapes — `/v1/scrapes`

Turn any URL into clean markdown, HTML, text, screenshots, or structured JSON.

### POST /v1/scrapes

```json
{
  "url_to_scrape": "https://example.com",
  "formats": ["markdown"],
  "country": "US",
  "wait_before_scraping": 0,
  "remove_css_selectors": "default",
  "actions": [],
  "parser": { "id": "@olostep/google-search" },
  "llm_extract": { "schema": { "title": "", "price": "" } },
  "links_on_page": { "include_links": ["/blog/**"] },
  "metadata": {}
}
```

Only `url_to_scrape` is required. Everything else is optional.

- `formats`: array of `html`, `markdown`, `text`, `json`, `raw_pdf`, `screenshot`
- `country`: US, CA, IT, IN, GB, JP, MX, AU, ID, UA, RU, RANDOM
- `actions`: array of `{ "type": "wait"|"click"|"fill_input"|"scroll", ... }`
- `parser.id`: use a built-in parser (e.g. `@olostep/google-search`, `@olostep/google-maps`) to get structured JSON
- `llm_extract.schema`: provide a JSON schema with empty values; the API fills them via LLM
- `remove_css_selectors`: `"default"`, `"none"`, or an array of selectors
- `links_on_page`: extract links from the page with optional include/exclude glob filters

**Response:**

```json
{
  "id": "scrape_abc123",
  "object": "scrape",
  "created": 1700000000,
  "metadata": {},
  "url_to_scrape": "https://example.com",
  "retrieve_id": "...",
  "result": {
    "markdown_content": "...",
    "html_content": "...",
    "text_content": "...",
    "json_content": "...",
    "markdown_hosted_url": "https://...",
    "html_hosted_url": "https://...",
    "links_on_page": [],
    "page_metadata": { "title": "...", "description": "..." }
  }
}
```

### GET /v1/scrapes/{scrape_id}

Retrieve a completed scrape by ID. Same response shape as POST.

**Pricing:** 1 credit (base). 1-5 with parsers. 20 with LLM extract.

---

## Searches — `/v1/searches`

Semantic web search. Send a natural language query, get deduplicated links with titles and descriptions. The API expands the query into multiple targeted searches, runs them in parallel, and deduplicates results.

### POST /v1/searches

```json
{
  "query": "Best Answer Engine Optimization startups"
}
```

Only `query` is required.

**Response:**

```json
{
  "id": "search_9bi0sbj9xa",
  "object": "search",
  "created": 1760327323,
  "metadata": {},
  "query": "Best Answer Engine Optimization startups",
  "result": {
    "json_content": "...",
    "json_hosted_url": "https://...",
    "links": [
      {
        "url": "https://example.com/article",
        "title": "Article Title",
        "description": "Short snippet describing the result."
      }
    ]
  }
}
```

### GET /v1/searches/{search_id}

Retrieve a completed search by ID. Same response shape.

**Pricing:** 5 credits per request.

---

## Answers — `/v1/answers`

AI-powered web research. Ask a question or give a data-enrichment task; get back structured answers grounded in real web sources.

### POST /v1/answers

```json
{
  "task": "What is the latest book by J.K. Rowling?",
  "json": { "book_title": "", "author": "", "release_date": "" }
}
```

Only `task` is required. `json` is optional — provide an object with empty values as a schema, or omit it to get a plain text answer.

- When data can't be verified, fields return `"NOT_FOUND"`
- Without `json`, the response contains `{ "result": "plain text answer" }`

**Response (with json schema):**

```json
{
  "id": "answer_9bi0sbj9xa",
  "object": "answer",
  "created": 1760327323,
  "metadata": {},
  "task": "What is the latest book by J.K. Rowling?",
  "result": {
    "json_content": "{\"book_title\":\"The Hallmarked Man\",\"author\":\"J.K. Rowling\",\"release_date\":\"September 2, 2025\"}",
    "json_hosted_url": "https://...",
    "sources": [
      "https://example.com/source1",
      "https://example.com/source2"
    ]
  }
}
```

`result.json_content` is a stringified JSON string — parse it to access the structured data.

### GET /v1/answers/{answer_id}

Retrieve a completed answer by ID. Same response shape.

**Pricing:** 20 credits per request. Execution time: 3-30s.

---

## Maps — `/v1/maps`

Get all URLs on a website. Useful for content discovery, site structure analysis, and preparing URLs for batch scraping.

### POST /v1/maps

```json
{
  "url": "https://example.com",
  "include_urls": ["/blog/**"],
  "exclude_urls": ["/careers/**"],
  "top_n": 1000,
  "include_subdomain": true,
  "search_query": "optional relevance sort",
  "cursor": null
}
```

Only `url` is required.

- `include_urls` / `exclude_urls`: glob patterns for path filtering
- `top_n`: limit number of URLs returned
- `cursor`: pagination token from a previous response (10MB max per response)
- `include_subdomain`: include subdomains (default true)

**Response:**

```json
{
  "id": "map_abc123",
  "urls_count": 245,
  "urls": [
    "https://example.com/page1",
    "https://example.com/page2"
  ],
  "cursor": "next_page_token_or_null"
}
```

When `cursor` is null or absent, all URLs have been retrieved.

**Pricing:** 1 credit + 1 credit per extra 1,000 URLs. Execution time: seconds to 120s for complex sites.

---

## Batches — `/v1/batches`

Run many scrapes in parallel as a single batch job.

- **POST /v1/batches** — create a batch with an array of URLs and scrape config
- **GET /v1/batches/{batch_id}** — get batch status and progress
- **PATCH /v1/batches/{batch_id}** — update a batch (e.g. add URLs)
- **GET /v1/batches/{batch_id}/items** — get individual scrape results

---

## Crawls — `/v1/crawls`

Crawl a website starting from a URL, following links automatically.

- **POST /v1/crawls** — start a crawl
- **GET /v1/crawls/{crawl_id}** — get crawl status
- **GET /v1/crawls/{crawl_id}/pages** — get crawled pages

---

## Files — `/v1/files`

Upload files to use as context in scrapes and answers.

- **POST /v1/files** — create a file upload
- **POST /v1/files/{file_id}/complete** — mark upload complete
- **GET /v1/files/{file_id}** — get file metadata
- **GET /v1/files/{file_id}/content** — get file content
- **GET /v1/files** — list files
- **DELETE /v1/files/{file_id}** — delete a file

---

## Schedules — `/v1/schedules`

Schedule recurring scrapes or answers on a cron schedule.

- **POST /v1/schedules** — create a schedule
- **GET /v1/schedules** — list schedules
- **GET /v1/schedules/{schedule_id}** — get a schedule
- **DELETE /v1/schedules/{schedule_id}** — delete a schedule

---

## Common patterns

- All object IDs are prefixed: `scrape_`, `search_`, `answer_`, `map_`, `batch_`, `crawl_`, `file_`, `schedule_`
- Most endpoints return a hosted URL (S3) for large content
- `metadata` can be passed on creation for your own tracking (key-value object)
- Errors: 400 (bad request), 401 (invalid key), 404 (not found), 500 (server error)

## SDKs

```bash
pip install olostep       # Python
npm install olostep       # Node.js
```

```python
from olostep import Olostep
client = Olostep(api_key="...")
scrape = client.scrapes.create(url_to_scrape="https://example.com", formats=["markdown"])
```

```javascript
import Olostep from 'olostep';
const client = new Olostep({ apiKey: '...' });
const scrape = await client.scrapes.create({ urlToScrape: 'https://example.com', formats: ['markdown'] });
```

## Docs

Full documentation: https://docs.olostep.com
