---
name: wiki
description: >
  Canonical project wiki curator. One skill for all wiki operations: bootstrap, convert,
  ingest, research, session wrap-up, lint, query, and record decisions. The wiki skill is
  the only writer to curated pages — external producers (GSD, AIOS, research tools) stage
  to raw/<source>/ and the wiki's ingest pipeline curates.
  Trigger phrases: "log everything to the wiki", "wrap up the wiki", "update the wiki",
  "save this session to the wiki", "done hit the wiki", "set up project memory",
  "add a wiki", "build a knowledge base", "ingest these docs", "remember our decisions",
  "ingest the raw files", "research X and save to the wiki", "research X for this project",
  "lint the wiki", "what does the wiki say about X", "record this decision",
  "convert this wiki", "bootstrap a wiki", "wiki health check".
  Also triggers on /wiki, /wiki init, /wiki convert, /wiki ingest, /wiki research,
  /wiki log, /wiki lint, /wiki query, /wiki decide.
---

# /wiki — Project Wiki Curator

The canonical interface for all wiki operations. One skill, eight modes.

## Principles

1. **Knowledge arrives, wiki curates.** Source-agnostic. Doesn't matter if knowledge came from GSD, AIOS, research, or conversation.
2. **The wiki skill is the only writer to curated pages.** External producers stage to `raw/<source>/`. The wiki skill's ingest pipeline is the single gateway.
3. **WIKI-CLAUDE.md governs.** Every project wiki has a schema file. Read it before every write operation.
4. **Curation over completeness.** Only write what future sessions need that they can't derive from code.

---

## Wiki Detection

Before any operation, resolve the wiki location. Five-path resolver, checked in order:

1. **Explicit path** — argument passed directly (e.g. `/wiki ingest /path/to/wiki`)
2. **`docs/wiki/WIKI-CLAUDE.md`** — standard project layout
3. **`wiki/WIKI-CLAUDE.md`** — alternate project layout
4. **`WIKI-CLAUDE.md` at repo root** — flat layout
5. **Structural markers only** — `decisions/` + `log/` directories exist but no `WIKI-CLAUDE.md`. Detect the wiki but warn: "Found wiki structure without a schema file. Run `/wiki convert` to add WIKI-CLAUDE.md and complete the canonical layout."

**If no wiki found:** suggest `/wiki init` to bootstrap one.

**Once found:** read `WIKI-CLAUDE.md` before any write operation. This is non-negotiable. The schema defines section structure, naming conventions, and curation rules for the specific project.

---

## Mode Routing

### Explicit mode

When the user passes a mode argument, route directly:

| Command | Mode |
|---|---|
| `/wiki init` | init |
| `/wiki convert` | convert |
| `/wiki ingest` | ingest |
| `/wiki research` | research |
| `/wiki log` | log |
| `/wiki lint` | lint |
| `/wiki query` | query |
| `/wiki decide` | decide |

### Smart routing (no args or natural language)

When the user says `/wiki` with no mode, or uses natural language ("update the wiki", "done hit the wiki"), detect from context:

- **Pending files in `raw/`** that haven't been ingested (no corresponding curated page or log entry) → suggest **ingest**
- **Session produced commits or file changes** visible in `git diff` or `git log --since` → suggest **log**
- **User asking a question** about what's in the wiki → route to **query**
- **"wrap up", "done", "log everything", "save this session"** → route to **log**
- **"record this decision", "we decided"** → route to **decide**
- **"research X and save to wiki"** → route to **research**
- **"lint", "health check", "wiki health"** → route to **lint**

For state-changing operations detected via smart routing, briefly announce the selected mode before proceeding:

> "Detected pending raw files. Running **ingest** mode."

> "Session has 4 new commits. Running **log** mode to capture changes."

This gives the user a chance to redirect before writes happen.

---

## Mode: init

Bootstrap a new project wiki from scratch. No AIOS dependency required.

### Prerequisites

- No existing wiki detected (all 5 detection paths return empty)
- If existing docs are found (`docs/`, `wiki/`, markdown files in root), suggest `/wiki convert` instead and stop

### Workflow

**Step 1 — Discover existing context.**
Scan for: README.md, CLAUDE.md, package.json, docs/, .planning/, git log (first 20 commits). Build a mental model of the project before asking questions.

**Step 2 — Ask 2-3 setup questions.**
- Project name and one-line description (pre-fill from README/package.json if possible)
- Anything to exclude from wiki tracking? (e.g. "don't track deployment configs")
- Any seed decisions to record? (e.g. "we chose Postgres over SQLite because...")

Keep it brief. Two questions minimum, three maximum. Don't interrogate.

**Step 3 — Scaffold directory structure.**

```
docs/wiki/
  WIKI-CLAUDE.md
  index.md
  overview.md
  log/
  raw/
    aios/
    external/
    research/
    session/
  sources/
  decisions/
    active/
    superseded/
    deferred/
  architecture/
  overview/
  strategy/
  plans/
  research/
```

**Step 4 — Write WIKI-CLAUDE.md.**
Use the template at `~/.claude/skills/wiki/assets/WIKI-CLAUDE-TEMPLATE.md`. Replace placeholder tokens (`{{PROJECT_NAME}}`, `{{PROJECT_DESCRIPTION}}`, etc.) with values from Steps 1-2.

**Step 5 — Write raw/README.md.**
Use the template at `~/.claude/skills/wiki/assets/raw-README.md`. Copy verbatim (no token replacement needed).

**Step 6 — Seed initial content.**
- `index.md` — wiki table of contents linking to all top-level pages
- `overview.md` — project overview synthesized from Step 1 discovery
- `decisions/index.md` — empty decisions index with column headers
- `log/YYYY-MM-DD-wiki-init.md` — initial log entry recording the bootstrap

**Step 7 — Emit receipt and commit.**
Print a summary:
```
Wiki bootstrapped at docs/wiki/
  - WIKI-CLAUDE.md (schema)
  - index.md, overview.md (seed content)
  - decisions/, log/, raw/ (structure)
  - 1 log entry recorded
```
Create a single git commit: `docs(wiki): bootstrap project wiki`.

---

## Mode: convert

Reshape existing docs into the canonical wiki layout. For projects that already have documentation but not in the llm-wiki structure.

### Prerequisites

- Existing docs detected (markdown files, docs/ folder, wiki/ folder, or similar)
- No prior conversion (check: WIKI-CLAUDE.md doesn't exist, no `sources/*-import.md` files)
- If a canonical wiki already exists, abort with message: "Wiki already exists. Use `/wiki ingest` to add new content."

### Workflow

**Step 1 — Detect and snapshot existing docs.**
Inventory every documentation file. Record paths, sizes, last-modified dates. Classify each file by likely destination (decisions, architecture, strategy, plans, research, overview).

**Step 2 — Scaffold wiki skeleton.**
Same directory structure as init (Step 3 above). Create all directories. Do not overwrite any existing files.

**Step 3 — Stage existing content verbatim.**
Copy every detected doc file into `raw/external/<source-folder>/` preserving relative paths. This is the immutable archive. Never modify files in `raw/external/`.

**Step 4 — Write source summary.**
Create `sources/YYYY-MM-DD-<slug>-import.md` documenting:
- Source location (original paths)
- File count and total size
- Classification decisions (which file goes where)
- Any content that was skipped and why

**Step 5 — Promote into curated sections.**
For each classified file, write a curated version into the appropriate section (`decisions/`, `architecture/`, `strategy/`, `plans/`, `research/`). Curated versions may restructure, deduplicate, and cross-reference, but must not lose information present in the raw originals.

**Step 6 — Write WIKI-CLAUDE.md.**
Use the template at `~/.claude/skills/wiki/assets/WIKI-CLAUDE-TEMPLATE.md`. Replace tokens with project-specific values discovered in Step 1.

**Step 7 — Register with AIOS.**
If the project is tracked in `/Users/justinlobaito/repos/claude-os/clients.yaml`, update the `docs_paths` entry to include the new wiki path. If not tracked, skip this step silently.

**Step 8 — Log the conversion.**
Write `log/YYYY-MM-DD-wiki-convert.md` recording:
- What was converted
- File count staged vs. promoted
- Any content that needs manual review

**Step 9 — Emit receipt.**
Print summary of what was created, staged, and promoted. Flag any files that need manual review.

**Step 10 — Commit.**
Single git commit: `docs(wiki): convert existing docs to canonical wiki layout`.

### Idempotency

Never overwrite `raw/external/`. If re-running a conversion (e.g. new docs appeared after initial convert), stage alongside the original as `raw/external/<folder>-YYYY-MM-DD/`. This preserves the original import while capturing the new batch.

---

## Mode: ingest

Process staged `raw/` files into curated wiki pages. This is the core curation pipeline.

### Source Authority Levels

| Source | Authority | Ingest behavior |
|---|---|---|
| `raw/gsd/` | **Trusted** | Work completed in this project. Auto-ingest with high confidence. |
| `raw/research/` | **Trusted provenance** | Research requested for this project. Auto-ingest, but cite and qualify external claims — not wiki-authoritative. |
| `raw/external/` | **Trusted** | Operator-provided docs. Ingest via promotion workflow. |
| `raw/aios/` | **Advisory** | AIOS thinks this might be relevant. Evaluate against wiki state before deciding. |

### AIOS Drop Evaluation

When the ingest pipeline encounters `raw/aios/` files:

1. Read the drop and its dispatch reason (frontmatter metadata from AIOS)
2. Read the project's WIKI-CLAUDE.md and current wiki state
3. Evaluate: scope fit? redundant? contradicts existing knowledge?
4. Four outcomes:
   - **Promote** — new, compatible information curated directly
   - **Adapt** — relevant but needs reframing for project context
   - **Flag** — incoming claim conflicts with existing wiki knowledge. Surface both claims to operator: existing page, incoming claim, suggested resolution. Do not silently overwrite.
   - **Skip** — redundant or irrelevant. Leave in raw/aios/ with frontmatter note: `status: skipped` and `reason: <why>`

The project wiki has final say. AIOS is the dispatcher, not the authority.

### Workflow

**Step 1 — Read WIKI-CLAUDE.md.**
Load the file-authority map, ground rules, and exclusions.

**Step 2 — Scan raw/ subdirectories.**
Identify unprocessed files: no corresponding `sources/` summary or no `status:` frontmatter.

**Step 3 — Process by authority level.**
- **Trusted** (`gsd/`, `external/`): read → draft curated entries → write → update index → cross-reference
- **Trusted provenance** (`research/`): same flow but cite and qualify external claims, note research date
- **Advisory** (`aios/`): run AIOS Drop Evaluation (above)

**Step 4 — Append to log/.**
One entry per ingest run.

**Step 5 — Emit receipt.**
Print summary: files ingested by source, pages created/updated, files skipped with reason, flagged/contested items.

**Step 6 — Commit.**
`docs(wiki): ingest <N> sources from raw/<subdirs>`

---

## Mode: research

Fetch web research and stage it for wiki ingestion. Chain: semantic search → source fetch → pre-analysis → staging to `raw/research/`.

### Prerequisites

- Wiki exists (at minimum: `WIKI-CLAUDE.md`, `overview.md`, `index.md`)
- Web search tools available (Exa or Firecrawl)

### Workflow

**Step 1 — Contextualize query.**
Read wiki `overview.md` and `index.md` to avoid duplicating existing coverage.

**Step 2 — Run web search.**
Use Exa (`mcp__exa__web_search_exa`) or Firecrawl (`mcp__firecrawl__firecrawl_search`).

**Step 3 — Present source candidates to operator.**
Ranked list with relevance notes. If unattended, select top 3-5.

**Step 4 — Fetch selected sources.**
Exa fetch for static pages, Firecrawl scrape for JS-heavy sites, WebFetch as fallback.

**Step 5 — Pre-analyze and structure each source.**
Markdown with frontmatter:
```yaml
source_url: <url>
fetched: YYYY-MM-DD
query: <original query>
status: staged
```
Sections: TL;DR, Key findings, Relevance to project, Raw content.

**Step 6 — Stage to `raw/research/YYYY-MM-DD-<slug>.md`.**

**Step 7 — Ingest decision.**
Ask operator: "ingest now" vs. "stage only". Default recommendation: ingest now.

**Step 8 — Log the research session.**
Write `log/YYYY-MM-DD-research-<slug>.md`.

**Step 9 — Emit receipt.**

**Step 10 — Commit.**
`docs(wiki): research — <query summary>`

### Multi-round research

For deep topics, the operator can run `/wiki research` multiple times with refined queries. Each run stages independently and logs separately. Prior research results appear in the wiki index, so subsequent rounds naturally avoid duplication and can build on earlier findings.

---

## Mode: log — Session wrap-up

Extract wiki-worthy knowledge from the current session and write directly to curated pages.

**When to invoke:** End of session ("log everything to the wiki", "wrap up", "done hit the wiki"), after significant changes, or `/wiki log` explicitly.

**Workflow:**

**Step 1 — Review the session.**
Scan: recent git commits since last wiki log or session start, files changed, decisions made in conversation, deferred items mentioned, architecture patterns established, capability changes.

**Step 2 — Extract and categorize wiki-worthy knowledge.**
- Decisions → `decisions/active/` or `decisions/deferred/`
- Architecture (new subsystems/patterns) → `architecture/`
- Capability changes → `overview/` pages
- Deferrals → `decisions/deferred/` with revisit trigger

**Step 3 — Write directly to curated pages.**
In-session knowledge = direct write. No staging to `raw/`. This is the one mode where you skip the staging inbox because the agent IS the source and has full context.

**Step 4 — Run staleness sweep.**
Check:
- `overview/` pages: do capability descriptions match what was built this session?
- `decisions/deferred/`: did this session resolve any deferred decisions?
- `architecture/`: do patterns match current implementation?

Apply Write Safety rules for each stale entry found (auto-fix when authority is clear, flag when ambiguous).

**Step 5 — Update affected section `index.md` files.**

**Step 6 — Write log entry.**
File: `log/YYYY-MM-DD-session-<slug>.md`

**Step 7 — Emit receipt.**
Report: decisions recorded, architecture pages created/updated, capability pages updated, deferred items resolved/added, stale entries fixed, flagged items needing operator attention.

**Commit:** `docs(wiki): session wrap-up — <1-line summary>`

---

## Mode: lint — Wiki health check

Run structural and semantic checks across the wiki. Fix what's mechanical, flag what's ambiguous.

**When to invoke:** `/wiki lint`, periodic maintenance, or when wiki feels stale.

**Checks (9 items):**

1. **Orphans** — pages on disk that no other page links to
2. **Index gaps** — pages not listed in their section `index.md`, or index entries pointing to missing files
3. **Dead links** — markdown links to non-existent files
4. **Stale claims** — `decisions/active/` preconditions changed; `architecture/` code refs don't match current codebase
5. **Contradictions** — pages claiming conflicting facts (especially `decisions/` vs `architecture/` vs `strategy/`)
6. **Missing cross-references** — pages that should link to each other based on content overlap
7. **raw/ orphans** — source material in `raw/` that no curated wiki page references
8. **Lifecycle drift** — `active/` decisions whose triggers fired but not moved to `superseded/`; `deferred/` decisions whose work shipped but not moved to `implemented/`
9. **Log formatting** — entries follow expected format (frontmatter, date, slug)

### Fix Rules (Write Safety)

**Auto-fix (mechanical):**
- Missing index entries
- Dead links (remove or update)
- Index entries pointing to missing files (remove from index)
- Missing cross-references (up to 10 per run)
- Log formatting issues

**Auto-fix only when authority is clear (semantic):**
- Deferred decisions where feature is demonstrably shipped in code → move to `implemented/`
- Architecture pages where code clearly contradicts AND no active decision protects the old pattern → update
- Capability descriptions factually wrong per codebase → update

**Flag for operator (ambiguous semantic):**
- Contradictions between `decisions/` and `architecture/` where both could be right
- Stale claims where the correct answer is unclear
- Active decisions whose triggers may have fired but evidence is ambiguous

### Workflow

**Step 1 — Read WIKI-CLAUDE.md.**

**Step 2 — Walk all wiki pages, build page graph.**
Map: links between pages, index memberships, section membership.

**Step 3 — Run all 9 checks.**

**Step 4 — Apply auto-fixes.**
Follow the fix rules above. Mechanical fixes applied silently. Semantic auto-fixes applied with brief justification in the log.

**Step 5 — Collect flagged items.**
Each flag includes: the page, the issue, why it's ambiguous, and a suggested resolution for the operator.

**Step 6 — Emit receipt.**
Report: issues found by category, auto-fixed by category, flagged for operator with details.

**Step 7 — Commit.**
`docs(wiki): lint — <N> fixes, <M> flags`

---

## Mode: query — Answer from wiki

Read-only unless synthesis is worth filing.

**When to invoke:** "What does the wiki say about X", `/wiki query`, or any question best answered from project memory rather than codebase grep.

**Workflow:**

**Step 1 — Find candidate pages.**
Read wiki `index.md` and relevant section indexes to locate pages likely to answer the question.

**Step 2 — Read and synthesize.**
Read candidate pages. Synthesize answer from wiki content.

**Step 3 — Cite sources.**
Every factual claim references its source wiki page. Format: `(source: <relative-path>)`.

**Step 4 — Offer to file reusable synthesis.**
If the synthesis is reusable (cross-section, significant work, would inform multiple future sessions), offer to file it under the appropriate section or `synthesis/`. If the operator agrees, follow standard ingest rules: update index, write log, add cross-references.

**Step 5 — Log only if wiki was updated.**
Append to `log/` only if step 4 produced a wiki write.

---

## Mode: decide — Record a decision

Capture a decision with full context, cross-references, and lifecycle tracking.

**When to invoke:** "Record this decision", `/wiki decide`, or when conversation reaches a clear decision point worth preserving.

**Workflow:**

**Step 1 — Read WIKI-CLAUDE.md.**
Load the decision template and ground rules for the project's wiki.

**Step 2 — Check for duplicates.**
Search `decisions/` for existing entries covering the same topic. If found, surface the existing entry. If the operator wants to update, follow immutability rules: supersede the old decision (move to `superseded/`), don't edit-in-place for substantive changes. Metadata-only fixes (typos, formatting) can be edited in place.

**Step 3 — Determine status.**
- **active** — governs current work
- **deferred** — pushed to later, with a revisit trigger
- **implemented** — fully built, decision now a fact of the system

**Step 4 — Write decision file.**
Path: `decisions/<status>/YYYY-MM-DD-<slug>.md`
Use the WIKI-CLAUDE.md template. Include:
- Decision statement (what was decided)
- Rationale (why)
- Scope (what it applies to)
- Cross-references (related decisions, architecture pages, strategy docs)
- "Do not relitigate without" trigger (what would need to change to reopen this)

**Step 5 — Update affected wiki pages.**
Walk `architecture/`, `overview/`, `strategy/`, and other `decisions/` pages. Add cross-references or update content where the new decision changes what those pages say.

**Step 6 — Update `decisions/index.md`.**

**Step 7 — Write log entry.**
File: `log/YYYY-MM-DD-decision-<slug>.md`

**Step 8 — Emit receipt.**
Report: decision file path, status, pages updated, cross-references added.

**Step 9 — Commit.**
`docs(wiki): decide — <slug>`

---

## Staleness Guard

Two mechanisms that keep the wiki current without a separate maintenance ritual.

**Inline Catch.** During any active session work (not just `/wiki log`), when the skill detects a capability change — new feature built, deferred item resolved, pattern changed — check wiki pages for stale references. Apply Write Safety rules: auto-fix when authority is clear, flag when ambiguous. This catches the obvious case: you just built the thing the wiki says is deferred.

**Wrap-up Sweep.** During `/wiki log`, run a broader pass across all overview pages, deferred lists, capability docs, and architecture entries. This catches cross-cutting staleness not touched directly during the session.

Together, inline + sweep means the wiki stays current without anyone remembering to run a cleanup pass.

## Operation Receipts

Every write-producing mode emits a consistent summary when it completes:

- Files created
- Files updated
- Files skipped (with reason)
- Decisions recorded
- Flagged/contested items requiring operator attention
- Suggested next action (if any)

This is terminal output, not a persistent file.

## Universal Conventions

- Markdown only. Frontmatter optional.
- Absolute dates everywhere. YYYY-MM-DD. Never relative.
- Markdown links between wiki pages. Repo-relative paths. No wikilinks.
- One topic per page. Split if covering two things.
- Promote when content grows. Start as rows in overview, promote to file when >5 lines of substance.
- Cite sources for claims. Reference source pages.
- `index.md` and `log/` are navigation, not knowledge.
- No empty placeholder pages. A page exists only when there's content.
- `raw/` is read-only. Never modify existing files.

## Anti-Patterns

- Creating a giant taxonomy of empty folders. Only create dirs when you have content.
- Asking the user to fill out forms. User sources and asks questions; wiki skill does bookkeeping.
- Treating ingest as one-shot retrieval. Ingest means integrating — touching every page the source affects.
- Filing decisions silently then forgetting to apply them. Page-level updates are part of the decision.
- Letting `index.md` drift from reality. Update every time a page is created/renamed/deleted.
- Re-deriving the same synthesis every query. File it if you've pieced together the same answer twice.
- Writing to curated pages from outside the wiki skill. If you're a GSD agent, kickoff-project, or any producer — stage to `raw/<source>/`. Let the wiki skill curate.
- Silently overwriting contested claims. When incoming info contradicts existing wiki and authority is ambiguous, flag it.
