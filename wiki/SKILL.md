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

Discover project context (README, CLAUDE.md, package.json, git log), ask 2-3 setup questions (project name, exclusions, seed decisions), scaffold the canonical layout using `~/.claude/skills/wiki/assets/WIKI-CLAUDE-TEMPLATE.md` plus `~/.claude/skills/wiki/assets/schema-TEMPLATE.md` (as `schema.md` at the wiki root — the frontmatter/lint authority, ADR 0010 §8) and `~/.claude/skills/wiki/assets/open-questions-TEMPLATE.md` (as `open-questions.md` at the wiki root — ADR 0010 §4), seed index + overview + first log entry, add wiki pointer to CLAUDE.md if it exists.

If existing docs are found, suggest `/wiki convert` instead.

### convert — Reshape existing docs

Archive every existing doc verbatim to `raw/external/` (immutable), scaffold the wiki skeleton including `schema.md` and `open-questions.md` (see init, above), curate each file into the appropriate section. Curated versions may restructure and cross-reference but must not lose information from the originals.

Never overwrite `raw/external/`. Re-runs stage alongside as `raw/external/<folder>-YYYY-MM-DD/`.

### ingest — Process raw/ into curated pages

The core curation pipeline. Scan `raw/` for unprocessed files, process by authority level (see Source Authority above). Ingest means integrating — touching every page the source affects, not just creating one new page.

**Provenance stamping (ADR 0010 §5).** Assign `source` + `confidence` at promotion — never leave a promoted page untagged. `confidence: reported` when the raw source is faithfully captured (its own claims stay unvetted); `inferred` when the page is agent synthesis with no direct source statement; `verified` only when the claim restates an authority surface (`context/priorities.md`, `decisions/log.md`, `state/<slug>.md`, `clients.yaml`) or Justin confirms it in-session — never minted automatically otherwise. Full vocabulary in the wiki's `schema.md`.

### log — Session wrap-up

Extract wiki-worthy knowledge from the current session and **write directly to curated pages** — this is the one mode that skips `raw/` staging because the agent IS the source.

Scan recent commits, conversation decisions, deferrals, architecture changes. Categorize into decisions, architecture, capability, or deferral pages. Then run a staleness sweep: do overview pages match what was built? Did this session resolve any deferred decisions? Apply Write Safety rules.

### lint — Wiki health check

Nine checks: orphan pages, index gaps, dead links, stale claims, contradictions, missing cross-refs, raw/ orphans, lifecycle drift (decisions that should have moved status), log formatting.

Apply Write Safety: auto-fix mechanical issues, flag ambiguous semantic ones for the user.

**Scripted pass (ADR 0010 §1).** When the repo ships `scripts/wiki-lint.py`, run it first. Tier 1 (mechanical): frontmatter presence, type vocabulary, confidence backfill, dead links, index gaps, the 120-line budget, date drift, pointer-not-copy. Tier 2/3 (judgment): supersedence (a conclusion-like line changed in the window with no matching `## Superseded` entry → the block drafted from git history), transition markers older than ~a month, authority-order violations against `clients.yaml`, wiki-vs-wiki contradictions tie-broken by `confidence`, and op-vs-op drift. All config derives from the wiki's `schema.md`:

```
python3 scripts/wiki-lint.py --report state/wiki-lint.md          # read-only
python3 scripts/wiki-lint.py --report state/wiki-lint.md --fix    # + backfill + tier-3 escalation
```

Read-only by default; `--fix` applies the confidence backfill and appends tier-3 findings to the wiki's `open-questions.md` (deduped). Commit both with a receipt (`docs(wiki): lint auto-fix — <n> pages`). Tier-2 findings ship a drafted correction, never auto-applied — apply the unambiguous ones yourself, and remember the direction: the operational layer wins over the wiki, and between two wiki pages the *lower*-confidence side gets corrected.

The script only reaches what's mechanizable. Work the rest by hand in the same sweep: priorities and decisions claims against `context/priorities.md` and `decisions/log.md`, and value drift in prose (a figure quoted for a client that `clients.yaml` contradicts — the script deliberately doesn't guess, since pricing prose quotes numbers for a dozen honest reasons). Anything you can't adjudicate goes to `open-questions.md`, not into the report.

**Weekly sweep (ADR 0010 §2).** In claude-os, `scripts/wiki-sweep.py` runs that same lint across *every* wiki registered in `clients.yaml` `docs_paths` and rolls the findings per-repo into the one `state/wiki-lint.md`. `--fix` applies inside claude-os only — project repos are report-only at every tier, no cross-repo writes (ADR 0004). It also reports which repos need `graphify --update` (a `## Superseded` block gained an entry since that repo's last extract, ADR 0010 §10). Non-zero exit = a repo went unswept; never call the sweep complete in that case. Fired weekly by `scripts/wiki-sweep-cron.sh` (launchd `com.bgd.wiki-sweep`) via `~/.claude/scheduled-tasks/wiki-lint-weekly/`; run it by hand with:

```
python3 scripts/wiki-sweep.py --fix --report state/wiki-lint.md
```

Outside claude-os the authority-order layer silently skips (no `clients.yaml`) and the report has no `state/` to land in — pass `--report` somewhere the repo owns, and stay report-only in other people's repos.

### human-version — Browsable static site

Wrap the wiki in MkDocs Material so humans can browse it without AI. Set up `mkdocs.yml` with curated `nav:`, generate an onboarding landing page that **synthesizes** (not duplicates) key wiki content, and generate how-to guides grounded in **real git history** — actual commits, actual files, actual diffs.

Exclude wiki internals from nav (WIKI-CLAUDE.md, log.md, raw/). Onboarding page is always the landing page.

**Nav update protocol:** when wiki pages are added or moved after setup, flag the needed `mkdocs.yml` change to the user. Don't update nav silently. Content edits hot-reload automatically via `mkdocs serve`.

---

## Shared Behaviors

**Every write-producing mode:** log the operation, update affected indexes, emit a receipt (files created/updated/skipped, flagged items), commit with `docs(wiki): <mode> — <summary>`.

**Frontmatter on new pages:** when the wiki has a `schema.md` at its root (ADR 0010 §8), it is the authority — required keys, closed type vocabulary, provenance tags. Where no `schema.md` exists yet, fall back to the 2026-07-07 OKF-minimal rule: every NEW curated page gets at least `type:` and `timestamp:`. Never retrofit existing pages for format alone — wikis converge through normal churn, not a migration pass.

**Staleness guard:** during any session work, catch obvious staleness inline (you just built the thing the wiki says is deferred). During `/wiki log`, run a broader sweep across all overview and decision pages.

## Anti-Patterns

- Empty placeholder pages and premature folder taxonomies
- Asking the user to fill out forms — you do the bookkeeping
- Filing decisions without updating the pages they affect
- Letting index.md drift from reality
- Re-deriving the same synthesis every query — file it the second time
