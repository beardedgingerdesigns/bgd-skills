---
name: web-scraping
description: >
  Scrape, search, and extract structured data from the web using Exa (semantic search)
  and Firecrawl (JS-rendered page scraping). Use when user asks to scrape a website,
  extract data from URLs, search the web semantically, crawl pages, get content from
  JS-heavy sites, collect structured data from the web, or mentions Exa or Firecrawl.
---

# Web Scraping

Two-tool strategy: **Exa** finds the right pages and fetches content, **Firecrawl** handles JS-rendered scraping, crawling, and structured extraction.

## Tool routing

| Need | Tool | Why |
|------|------|-----|
| Find pages by meaning, not keywords | `mcp__exa__web_search_exa` | Semantic search, simple interface |
| Search with filters (dates, domains, categories) | `mcp__exa__web_search_advanced_exa` | Full control over date ranges, domain scoping, content options |
| Get clean text from one or more URLs | `mcp__exa__web_fetch_exa` | Fast batch fetch, returns clean markdown |
| Scrape JS-rendered / SPA pages | `mcp__firecrawl__firecrawl_scrape` | Headless browser renders JS before extraction |
| Crawl multiple pages from a root | `mcp__firecrawl__firecrawl_crawl` | Follows links, respects depth limits |
| Extract structured data from a page | `mcp__firecrawl__firecrawl_extract` | Returns JSON matching a provided schema |
| Search + scrape in one shot | `mcp__firecrawl__firecrawl_search` | Google search with full-page content extraction |

## Workflow: semantic search → scrape

1. **Search** with Exa to find relevant URLs:
   - `mcp__exa__web_search_exa` for straightforward queries — just `query` and optional `numResults`
   - `mcp__exa__web_search_advanced_exa` when you need filters:
     - `type`: `"auto"` (recommended), `"fast"`, or `"instant"`
     - `includeDomains` / `excludeDomains` to scope by site
     - `startPublishedDate` / `endPublishedDate` (ISO 8601: YYYY-MM-DD)
     - `category` to focus results (see parameters below)
     - `enableHighlights: true` + `highlightsMaxCharacters` for inline snippets
     - `enableSummary: true` for AI-generated summaries per result
   - Set `numResults: 5-10` to balance breadth vs. cost

2. **Triage** results — check titles, highlights, and summaries; pick the most relevant URLs

3. **Scrape** each URL for full content:
   - Static pages → `mcp__exa__web_fetch_exa` with `urls: [...]` (faster, batches multiple URLs)
   - JS-heavy / SPA pages → `mcp__firecrawl__firecrawl_scrape` with `formats: ["markdown"]`
   - Need structured JSON → `mcp__firecrawl__firecrawl_extract` with a JSON schema

4. **Process** the extracted content (summarize, compare, structure) in your response

## Workflow: crawl a site

1. `mcp__firecrawl__firecrawl_crawl` with the root URL
   - Set `maxDepth` (1-3 for focused crawl, higher for deep)
   - Set `limit` to cap total pages
   - Use `includePaths` / `excludePaths` globs to scope
2. Process results — each page returns as markdown with metadata

## Workflow: extract structured data

1. Define a JSON schema for the data you want
2. `mcp__firecrawl__firecrawl_extract` with `urls` and `schema`
3. Firecrawl returns structured JSON matching your schema

## Key parameters

### Exa simple search (`web_search_exa`)
- `query`: natural language — describe the ideal page, not just keywords ("blog post comparing React and Vue performance" not "React vs Vue")
- `numResults`: number of results (default 10)
- Tip: append `category:people` or `category:company` to the query string for LinkedIn-style searches

### Exa advanced search (`web_search_advanced_exa`)
- `query`: search query (question, statement, or keywords)
- `type`: `"auto"` (recommended) | `"fast"` | `"instant"`
- `numResults`: 1-100 (default 10)
- `includeDomains` / `excludeDomains`: domain filters (e.g., `["arxiv.org", "github.com"]`)
- `includeText` / `excludeText`: require or exclude specific text strings in results
- `startPublishedDate` / `endPublishedDate`: ISO 8601 date filters (YYYY-MM-DD)
- `startCrawlDate` / `endCrawlDate`: filter by when Exa crawled the page
- `category`: `"company"` | `"research paper"` | `"news"` | `"pdf"` | `"github"` | `"personal site"` | `"people"` | `"financial report"`
- `enableHighlights`: `true` to get relevant snippets per result
- `highlightsMaxCharacters`: max total characters across all highlights per URL
- `enableSummary`: `true` for AI-generated summary per result
- `subpages`: number of subpages to crawl from each result (1-10)
- `subpageTarget`: keywords to guide subpage selection
- `maxAgeHours`: max cache age (0 = always fresh)
- `userLocation`: ISO country code for geo-targeted results (e.g., `"US"`)

### Exa fetch (`web_fetch_exa`)
- `urls`: array of URLs to read (batch multiple in one call)
- `maxCharacters`: max characters per page (default 3000)

### Firecrawl scrape
- `url`: target URL
- `formats`: `["markdown"]` for clean text, `["html"]` for raw, `["screenshot"]` for visual
- `onlyMainContent`: `true` to skip nav/footer/ads (default true)
- `waitFor`: milliseconds to wait for JS rendering

### Firecrawl extract
- `urls`: array of URLs to extract from
- `schema`: JSON Schema defining the output structure
- `prompt`: natural language instruction for the extraction

## Fallbacks

If Exa MCP is not connected:
- Use `mcp__firecrawl__firecrawl_search` for search + scrape in one call (if Firecrawl MCP is available)
- Use `WebSearch` for basic web search, then scrape with Firecrawl or `WebFetch`

If Firecrawl MCP is not connected:
- Use the `/firecrawl` skill for CLI-based scraping (`firecrawl scrape`, `firecrawl search`, `firecrawl interact`)
- Use `mcp__exa__web_fetch_exa` for static page content (batch URLs in one call)
- Use `WebFetch` for basic HTTP fetch (no JS rendering)

If neither MCP is connected:
- Use `WebSearch` + `WebFetch` as baseline (no semantic search, no JS rendering)
- Install Exa MCP: `claude mcp add exa -e EXA_API_KEY=... -- npx -y exa-mcp-server`
- Install Firecrawl: either MCP (`claude mcp add firecrawl -e FIRECRAWL_API_KEY=... -- npx -y firecrawl-mcp`) or CLI (`npx -y firecrawl-cli@latest init --all --browser`)

## Tips

- Exa semantic search is much better than keyword search for "find me pages about X" — describe the ideal page in your query
- Use `web_search_exa` for quick lookups, `web_search_advanced_exa` when you need date/domain/category filters
- `web_fetch_exa` batches multiple URLs in one call — always pass all URLs together instead of one at a time
- Firecrawl with `formats: ["markdown"]` gives the cleanest LLM-consumable output
- For large crawls, start with `limit: 5` to verify scope before scaling up
- Use `mcp__firecrawl__firecrawl_extract` when you need tabular or structured data — it's more reliable than scraping + parsing
- Set `enableHighlights: true` on advanced search to get relevant snippets without a separate fetch call
- For comprehensive Firecrawl usage (CLI, build integration, workflow deliverables), see the `/firecrawl` skill
