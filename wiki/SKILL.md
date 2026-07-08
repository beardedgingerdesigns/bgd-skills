---
name: wiki
description: >
  Canonical project wiki curator — bootstrap, curate, and maintain a project wiki.
  Use when the user wants to set up project memory, ingest docs, log a session,
  lint wiki health, convert existing docs, or make the wiki browsable.
  Also triggers on /wiki and any /wiki subcommand.
---

# /wiki — Project Wiki Curator

One skill, six modes. The single gateway for all wiki reads and writes.

## Ground Rules

1. **Wiki skill is the only writer to curated pages.** External producers stage to `raw/<source>/`.
2. **WIKI-CLAUDE.md governs.** Read it before every write. Non-negotiable.
3. **raw/ is immutable.** Never modify files in `raw/`.
4. **Curation over completeness.** Only persist what future sessions can't derive from code.
5. **Absolute dates.** YYYY-MM-DD everywhere. Never relative.
6. **Cite sources.** Every claim references its source page.
7. **Cross-reference liberally.** Update index.md on every write.
8. **Never silently overwrite contested claims.** Surface both versions, let the user decide.
9. **File reusable synthesis.** When answering a question from wiki content and the synthesis is non-trivial, offer to file it as a new wiki page rather than re-deriving it next time.

## Write Safety

Two tiers govern every fix:

- **Auto-fix (mechanical):** missing index entries, dead links, log formatting, missing cross-refs.
- **Flag for user (semantic):** contradictions between pages, stale claims where the correct answer is unclear, decisions whose triggers may have fired ambiguously.

When authority is clear (feature demonstrably shipped, code clearly contradicts docs, no protecting decision), auto-fix semantic issues too. When ambiguous, always flag.

## Source Authority

| Source | Authority | Behavior |
|---|---|---|
| `raw/gsd/`, `raw/external/` | **Trusted** | Auto-curate with high confidence |
| `raw/research/` | **Trusted provenance** | Auto-curate but cite and qualify external claims |
| `raw/aios/` | **Advisory** | Evaluate against wiki state: promote, adapt, flag, or skip |

AIOS drops get four outcomes: **promote** (new, compatible), **adapt** (relevant, needs reframing), **flag** (conflicts with existing wiki — surface both claims), **skip** (redundant — mark `status: skipped` with reason). The project wiki has final say.

## Wiki Detection

Resolve location in order: explicit path → `docs/wiki/WIKI-CLAUDE.md` → `wiki/WIKI-CLAUDE.md` → root `WIKI-CLAUDE.md` → structural markers (`decisions/` + `log/` without schema file). No wiki found → suggest `/wiki init`.

---

## Mode Routing

| Command | Mode |
|---|---|
| `/wiki init` | init |
| `/wiki convert` | convert |
| `/wiki ingest` | ingest |
| `/wiki log` | log |
| `/wiki lint` | lint |
| `/wiki human-version` | human-version |

**Smart routing** (no args or natural language): pending `raw/` files → ingest. Session produced commits → log. Question about wiki content → query. "wrap up" / "done" → log. "health check" → lint. "browsable" / "onboard" / "mkdocs" → human-version.

Announce the detected mode before proceeding so the user can redirect.

---

## Modes

### init — Bootstrap a new wiki

Discover project context (README, CLAUDE.md, package.json, git log), ask 2-3 setup questions (project name, exclusions, seed decisions), scaffold the canonical layout using `~/.claude/skills/wiki/assets/WIKI-CLAUDE-TEMPLATE.md`, seed index + overview + first log entry, add wiki pointer to CLAUDE.md if it exists.

If existing docs are found, suggest `/wiki convert` instead.

### convert — Reshape existing docs

Archive every existing doc verbatim to `raw/external/` (immutable), scaffold the wiki skeleton, curate each file into the appropriate section. Curated versions may restructure and cross-reference but must not lose information from the originals.

Never overwrite `raw/external/`. Re-runs stage alongside as `raw/external/<folder>-YYYY-MM-DD/`.

### ingest — Process raw/ into curated pages

The core curation pipeline. Scan `raw/` for unprocessed files, process by authority level (see Source Authority above). Ingest means integrating — touching every page the source affects, not just creating one new page.

### log — Session wrap-up

Extract wiki-worthy knowledge from the current session and **write directly to curated pages** — this is the one mode that skips `raw/` staging because the agent IS the source.

Scan recent commits, conversation decisions, deferrals, architecture changes. Categorize into decisions, architecture, capability, or deferral pages. Then run a staleness sweep: do overview pages match what was built? Did this session resolve any deferred decisions? Apply Write Safety rules.

### lint — Wiki health check

Nine checks: orphan pages, index gaps, dead links, stale claims, contradictions, missing cross-refs, raw/ orphans, lifecycle drift (decisions that should have moved status), log formatting.

Apply Write Safety: auto-fix mechanical issues, flag ambiguous semantic ones for the user.

### human-version — Browsable static site

Wrap the wiki in MkDocs Material so humans can browse it without AI. Set up `mkdocs.yml` with curated `nav:`, generate an onboarding landing page that **synthesizes** (not duplicates) key wiki content, and generate how-to guides grounded in **real git history** — actual commits, actual files, actual diffs.

Exclude wiki internals from nav (WIKI-CLAUDE.md, log.md, raw/). Onboarding page is always the landing page.

**Nav update protocol:** when wiki pages are added or moved after setup, flag the needed `mkdocs.yml` change to the user. Don't update nav silently. Content edits hot-reload automatically via `mkdocs serve`.

---

## Shared Behaviors

**Every write-producing mode:** log the operation, update affected indexes, emit a receipt (files created/updated/skipped, flagged items), commit with `docs(wiki): <mode> — <summary>`.

**OKF-minimal frontmatter on new pages (decision 2026-07-07):** every NEW curated page gets YAML frontmatter with at least `type:` and `timestamp:` (Open Knowledge Format v0.1 fields; add title/description/tags where useful; page-specific schemas like research notes keep their existing fields alongside). Never retrofit existing pages for format alone — existing wikis converge through normal churn. Full conversion of a wiki happens only on a consumer trigger (an OKF tool in the workflow, a client bundle export).

**Staleness guard:** during any session work, catch obvious staleness inline (you just built the thing the wiki says is deferred). During `/wiki log`, run a broader sweep across all overview and decision pages.

## Anti-Patterns

- Empty placeholder pages and premature folder taxonomies
- Asking the user to fill out forms — you do the bookkeeping
- Filing decisions without updating the pages they affect
- Letting index.md drift from reality
- Re-deriving the same synthesis every query — file it the second time
