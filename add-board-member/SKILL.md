---
name: add-board-member
description: >
  Onboard a new advisor to the board or expand an existing advisor's knowledge base.
  Use when the user wants to add a board member, onboard an advisor, or add content to an advisor's knowledge base.
  Also triggers on /add-board-member.
---

# Add Board Member

## Prerequisites

- **Web search available.** At least one of Exa (`mcp__exa__web_search_advanced_exa`) or Firecrawl (`mcp__firecrawl__firecrawl_search`) must be accessible. Check with ToolSearch if unsure.
- **Web fetch available.** At least one of Exa (`mcp__exa__web_fetch_exa`) or Firecrawl (`mcp__firecrawl__firecrawl_scrape`) must be accessible.
- If neither search nor fetch tools are available, stop: "Web search tools required. Connect Exa or Firecrawl MCP servers first."

## Execution

### Step 1 — Parse input

Extract from the user's message:
- **Advisor name** (required) — the person to add or update
- **Seed URLs** (optional) — specific content to start from
- Derive the **advisor slug** — kebab-case of the name (e.g., `alex-hormozi`)

### Step 2 — Resolve target board

Determine which board to write to:

1. Detect the current working directory's project.
2. Ask the user: "Adding to **this project's board** or the **global AIOS board**?"
   - If in the claude-os repo, skip the question — default to AIOS board.
   - If in a project repo, present both options.
3. Set the **board root** based on the answer:
   - **Project-local:** `{project-root}/docs/wiki/advisors/`
   - **Global AIOS:** `/Users/justinlobaito/repos/claude-os/docs/wiki/advisors/`

### Step 3 — Detect mode (new vs. existing)

Check if `{board-root}/roster.yaml` exists:
- **No roster.yaml:** This is a new board. Will scaffold the full directory structure.
- **Roster exists:** Read it. Check if the advisor slug already has an entry.
  - **Not in roster:** New advisor mode.
  - **Already in roster:** Content addition mode. Read their existing `persona.md` and list their `knowledge/*.md` files to understand what's already covered.

### Step 4 — Content discovery

Search for the advisor's public content. Use Exa (`mcp__exa__web_search_advanced_exa`) as the primary search tool, Firecrawl (`mcp__firecrawl__firecrawl_search`) as fallback.

**If seed URLs were provided:** Include them as known sources. Search for additional content to supplement.

**If no seed URLs:** Cast a wide net. Run 3-5 searches across different content types:
- `"{advisor name}" AI framework OR methodology site:youtube.com` — video content
- `"{advisor name}" blog OR article OR guide` — written content
- `"{advisor name}" podcast OR interview` — podcast appearances
- `"{advisor name}" talk OR conference OR keynote` — talks and presentations

**For content addition mode:** Read existing knowledge file names and their `## Key Frameworks and Patterns` / `## Key Concepts` headers. Exclude sources that cover already-ingested topics. Prioritize gaps listed in the persona.md `Gaps` section.

**Target:** Identify 10-15 candidate sources for new advisors. For content additions, identify 3-8 sources covering gaps.

### Step 5 — Select sources

From the discovered content, select the best sources for knowledge extraction:
- Prioritize content where the advisor explains their own frameworks, mental models, and positions
- Prefer long-form content (talks, interviews, blog posts) over short social posts
- Prefer primary sources (their own channel/blog) over third-party summaries
- Deduplicate — if the same framework appears in multiple sources, pick the richest one
- **New advisor target:** 8-10 sources to yield 8-10 knowledge files
- **Content addition target:** 3-8 sources to fill identified gaps

### Step 6 — Scaffold directory (if needed)

**New board (no roster.yaml):**
```
{board-root}/
├── roster.yaml          # create empty: "advisors:\n"
└── {advisor-slug}/
    ├── persona.md        # placeholder, written in Step 8
    └── knowledge/        # empty, filled in Step 7
```

**New advisor on existing board:**
```
{board-root}/{advisor-slug}/
├── persona.md
└── knowledge/
```

**Content addition:** No scaffolding needed — directory already exists.

Create these directories and any placeholder files using the Write tool. Do not modify any other project files (no CLAUDE.md or clients.yaml changes).

### Step 7 — Parallel knowledge extraction

Before spawning agents, read one example knowledge file from the AIOS board to use as a format reference: `/Users/justinlobaito/repos/claude-os/docs/wiki/advisors/nate-herk/knowledge/three-ms-framework.md`. This ensures agents produce files consistent with the established format.

Spawn one Agent call per selected source. Run agents in parallel (include all Agent calls in a single message). Use the extraction agent prompt from [PROMPTS.md](PROMPTS.md#extraction-agent-prompt), adapted per source. Every selected source must produce a written `knowledge/*.md` file before proceeding to Step 8.

### Step 8 — Persona synthesis

After all knowledge agents complete, spawn a synthesis agent using the persona synthesis prompt from [PROMPTS.md](PROMPTS.md#persona-synthesis-prompt).

### Step 9 — Register in roster

**New advisor:** Append an entry to `{board-root}/roster.yaml`:

```yaml
  {advisor-slug}:
    name: {Full Name}
    lens: "{one-line description of their advisory lens}"
    sources:
      - {source type}: "{handle or domain}"
    knowledge_path: docs/wiki/advisors/{advisor-slug}/
```

Use the Edit tool to append under the `advisors:` key.

**Content addition:** No roster change needed.

**New board:** The roster.yaml was created in Step 6. Edit it to add the advisor entry.

### Step 10 — Report

Output a report in this format:

```markdown
## Board Member Added: {Name}

**Board:** {project name} / AIOS (global)
**Mode:** New advisor / Content addition
**Knowledge files created:** {count}

| File | Topic | Source |
|------|-------|--------|
| {filename}.md | {topic description} | {source type}: {title or URL} |
| ... | ... | ... |

**Persona:** {full path to persona.md}
**Roster:** {full path to roster.yaml}
**Gaps identified:**
- {gaps from persona.md}

**Sources used:** {count} sources ({breakdown by type})
```

After the report, remind the user: "Review the knowledge files and persona for accuracy. Use `/ask-the-board` to test the new advisor."

### Shutdown

Do not summarize what the skill does. Do not offer to do more. The report is the final output.
