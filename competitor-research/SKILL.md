---
description: Scout the competitive landscape before building a website. Outputs a structured blueprint that feeds design extraction and build agents.
disable-model-invocation: true
---

# /competitor-research

Scout a niche's web landscape — who wins, who loses, why — and produce a structured blueprint the design extraction agent can act on. The output is a landscape map, not a design. Design decisions happen downstream.

## Step 1 — Intake

Use `AskUserQuestion` to gather the brief. Ask all questions in one call:

1. **Niche** — "What industry or niche is this site for?" (e.g. "equipment distributor," "SaaS channel management platform," "roofing contractor")
2. **Site type** — "What kind of site are you building?" Options: `Marketing / landing page`, `Product / SaaS app`, `E-commerce / catalog`, `Service business`
3. **Location** — "Is there a geographic focus? (Leave blank for national/global)"
4. **Known competitors** — "Any competitors you already know about? (Names or URLs, comma-separated, or leave blank)"

If the client has an existing site, ask for its URL too — the landscape includes where they stand now.

**Completion:** all four answers collected. Proceed without re-asking — reasonable defaults for blanks (national scope, no known competitors).

## Step 2 — Find the field

Build two lists: **winners** and **losers**. Target 6-10 of each.

Use `mcp__exa__web_search_exa` or `mcp__plugin_exa_exa__web_search_exa` with semantic queries:
- "best [niche] websites [location]"
- "top [niche] companies"
- "[niche] website design award"

For losers, search for:
- "small [niche] [location]" — the long tail, not deliberately bad sites
- "[niche] near me [location]" — local results that haven't invested in design

Add any competitors the user named in Step 1.

If the client has an existing site, include it in the losers list for honest benchmarking (don't sugarcoat — the blueprint is internal).

**Completion:** two lists, 6-10 URLs each, with one-line descriptions. Log them before proceeding.

## Step 3 — Scrape and score

Use `mcp__firecrawl__firecrawl_scrape` on each URL. Extract:

- Page structure (section order from top to bottom)
- Hero approach (image, video, text-only, product shot, illustration)
- CTA count, placement, and wording
- Social proof (testimonials, logos, case studies, stats)
- Trust signals (certifications, awards, security badges, "as seen in")
- Navigation structure
- Above-the-fold content density
- Mobile responsiveness signals

Score each site against the matrix in [`SCORING.md`](SCORING.md). Every site gets a total score out of 100.

**Completion:** every site in both lists scraped and scored. No site skipped without a noted reason (e.g. paywall, JS-only render).

## Step 4 — Pull UI patterns

Use Refero to find screens and flows that match the strongest patterns found in Step 3.

- `mcp__claude_ai_Refero__refero_search_screens` — search for the niche and site type (e.g. "SaaS landing page hero," "equipment catalog product grid")
- `mcp__claude_ai_Refero__refero_get_similar_screens` — for any standout competitor screen, find similar patterns across other products
- `mcp__claude_ai_Refero__refero_search_flows` — search for key user flows (e.g. "pricing page flow," "product comparison flow," "dealer locator")

Collect 5-10 reference screens/flows. For each, note what pattern it demonstrates and why it's relevant.

**Completion:** at least 5 Refero references collected and annotated.

## Step 5 — Synthesize the landscape

Write `COMPETITOR-RESEARCH.md` in the project root (or the path the user specifies). Structure:

```markdown
# Competitive Landscape: [Niche]

## Brief
[One paragraph: niche, site type, location, client context]

## Scoring Matrix
[The criteria from SCORING.md with weights, rendered as a table]

## Winners (ranked by score)
[For each: URL, score, 2-3 sentence summary of what they do right]

## Losers (bottom 5 by score)
[For each: URL, score, 1-2 sentence summary of what they lack]

## Winner Patterns
[What the top 5 all share — section order, hero approach, CTA patterns, social proof placement. Be specific: "8 of 10 winners lead with a product hero image + single-line value prop + CTA above the fold."]

## Anti-Patterns
[What the bottom 5 all share — what to avoid]

## Page Structure Blueprint
[Recommended section order based on winner patterns, with content type for each section]

## UI Pattern References
[Refero screens/flows with annotations: what pattern, why relevant]

## Recommended Inspiration Sites
[Top 2-3 from the winners list that would make the best design extraction targets, with reasoning]
```

**Completion:** document written, includes all sections, winner patterns are specific and evidenced (not vague), and the recommended inspiration sites are explicitly chosen to feed the design extraction agent.

## Step 6 — Handoff

Tell the user:
- Where the file was written
- The top 2-3 recommended inspiration sites for design extraction
- Any gaps: sites that couldn't be scraped, thin niche with few competitors, client's existing site ranking

Do not start design extraction. This skill scouts; the next agent designs.
