---
name: web-scraping
description: >
  Scrape, search, and extract structured data from the web using Exa and Firecrawl.
  Use when the user wants to scrape websites, search the web semantically, crawl pages,
  extract structured data, or mentions Exa or Firecrawl.
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

## Usage notes

- Exa queries: describe the ideal page, not keywords ("blog post comparing React and Vue performance" not "React vs Vue")
- `web_fetch_exa` batches multiple URLs — always pass all URLs together instead of one at a time
- `enableHighlights: true` on advanced search gets relevant snippets without a separate fetch call
- Firecrawl with `formats: ["markdown"]` gives the cleanest LLM-consumable output
- For large crawls, start with `limit: 5` to verify scope before scaling up

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

For comprehensive Firecrawl usage (CLI, build integration, workflow deliverables), see the `/firecrawl` skill.
