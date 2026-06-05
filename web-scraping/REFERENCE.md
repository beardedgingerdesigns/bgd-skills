# Web Scraping Reference

## MCP Server Setup

### Exa

Exa provides semantic web search. Requires an API key from [exa.ai](https://exa.ai).

```bash
# Add Exa MCP server
claude mcp add exa -e EXA_API_KEY=your-key-here -- npx -y exa-mcp-server
```

Available MCP tools once connected:
- `mcp__exa__web_search_exa` — simple semantic search
- `mcp__exa__web_search_advanced_exa` — search with filters (dates, domains, categories, highlights)
- `mcp__exa__web_fetch_exa` — batch fetch clean markdown from URLs

### Firecrawl

Two options for Firecrawl access:

**Option 1: MCP server** (if available)

```bash
# Add Firecrawl MCP server
claude mcp add firecrawl -e FIRECRAWL_API_KEY=your-key-here -- npx -y firecrawl-mcp
```

Provides `mcp__firecrawl__firecrawl_scrape`, `mcp__firecrawl__firecrawl_crawl`, `mcp__firecrawl__firecrawl_extract`, `mcp__firecrawl__firecrawl_search` tools.

**Option 2: CLI via `/firecrawl` skill** (current setup)

```bash
npx -y firecrawl-cli@latest init --all --browser
```

Provides `firecrawl search`, `firecrawl scrape`, `firecrawl interact`, `firecrawl crawl`, `firecrawl map` CLI commands. See the `/firecrawl` skill for full path routing (live tools, app integration, workflow deliverables).

## Advanced Patterns

### Competitive analysis

```
1. Exa advanced search: "companies building [product category]" with category: "company"
2. Expand: search again with competitor names from step 1 to find related companies
3. Fetch company pages with mcp__exa__web_fetch_exa (static) or Firecrawl scrape (JS-rendered)
4. Extract structured data with a schema: { name, pricing_tiers, key_features, target_audience }
```

### Research aggregation

```
1. Exa advanced search: topic query with category: "research paper" or "news"
2. Filter by startPublishedDate / endPublishedDate for recency
3. Batch fetch top result URLs with mcp__exa__web_fetch_exa
4. Summarize and cross-reference findings
```

### Price monitoring / data collection

```
1. Firecrawl extract with a product schema: { name, price, availability, specs }
2. Run against a list of product URLs
3. Structure results for comparison
```

### Site documentation extraction

```
1. Firecrawl crawl from docs root URL with maxDepth: 2-3
2. Filter to documentation paths with includePaths: ["/docs/*"]
3. Process each page's markdown for structured knowledge base
```

## Rate Limits & Cost

- **Exa**: 1000 searches/month on free tier; results include snippets by default
- **Firecrawl**: 500 credits/month on free tier; scrape = 1 credit, crawl = 1 credit per page
- Batch fetch calls with `mcp__exa__web_fetch_exa` (pass all URLs in one `urls` array)
- Use Exa fetch for static pages to save Firecrawl credits for JS-heavy sites

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Exa returns irrelevant results | Use `mcp__exa__web_search_advanced_exa` with `includeDomains` or `includeText` filters |
| Firecrawl scrape returns empty | Increase `waitFor` to 3000-5000ms for slow-loading SPAs |
| Crawl returns too many pages | Tighten `includePaths` and reduce `maxDepth` |
| Extract returns wrong structure | Simplify your JSON schema; add a clearer `prompt` |
| Exa MCP tools not found | Check MCP server is connected; tools are `mcp__exa__web_search_exa`, `mcp__exa__web_fetch_exa` |
| Firecrawl MCP tools not found | Firecrawl MCP may not be installed — fall back to `/firecrawl` CLI skill |
