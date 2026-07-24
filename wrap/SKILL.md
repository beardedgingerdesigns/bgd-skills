---
name: wrap
description: Wrap the session — digest to the project's wiki, update state, log a breadcrumb. Use when Justin types /wrap, or is clearly ending a work session.
---

# Wrap

## Steps

### 1. Resolve where you are

- Repo = current working directory's git root. Slug = repo folder name.
- Wiki = `{repo}/docs/wiki/` if it exists (check for `WIKI-CLAUDE.md` inside). No wiki → skip step 4, STATE.md only.

### 2. Review + compound (conditional — code sessions only)

Run this BEFORE the digest so findings appear in the digest's Next steps.

Check whether the session produced code changes: run `git diff --name-only "$BASE_SHA"...HEAD` (where `BASE_SHA` is passed by the caller, e.g. nightshift; default to `HEAD~5` for interactive sessions) and look for non-doc, non-planning source files (`.ts`, `.tsx`, `.js`, `.py`, `.rb`, `.go`, `.css`, `.html`, `.svelte`, `.vue`, `.swift`, `.rs`, etc. — not `.md`, not `.planning/`).

**If code changes exist:**

1. **Code review** — invoke `/ce-code-review`. Captures findings for inclusion in the digest.
2. **Compound** — invoke `/ce-compound` (the Compound Engineering skill). Records what the codebase learned.

**If no code changes** (research, decisions, planning, AIOS work): skip both silently.

### 3. Distill the session

Build a digest from the conversation so far. Include any code review findings from step 2 in the **Next steps** section. Sections, all required (write "none" rather than omitting):

- **Accomplishments** — what shipped/changed, with file paths
- **Decisions** — calls made and why, one line each; flag any that belong in a decisions log
- **Struggles** — where we churned, corrected, or burned time; this section is mined by /retro. Include tooling friction, not just code problems.
- **Next steps** — checkbox list, concrete enough to resume cold. Include code review findings if any.
- **Open questions** — anything needing Justin's call

Substance gate: if the session was trivial (a question or two, no work product), say so and stop — don't log noise.

### 3b. Research capture (conditional)

If the session was research-shaped — an external video/article/transcript discussed, a competitor or concept compared to Justin's approach, strategic thinking-out-loud — execute the capture procedure in `~/repos/claude-os/docs/wiki/research/CONVENTIONS.md`. In a repo other than claude-os, project findings stay in this repo's wiki (steps 4-5), but *business-level* research (BGD strategy, market, pricing, competitive landscape — not implementation detail) also drops a copy named `research-YYYY-MM-DD-<gist>.md` into `~/repos/claude-os/archives/raw/` for `/dispatch` to route.

If the session had no research flavor, skip silently.

### 3c. Park open questions

Take the digest's **Open questions** section (step 3). If it says "none" or is empty, skip silently —
no receipt, no file touch.

- Wiki exists → append each unanswered question to `{repo}/docs/wiki/open-questions.md`, using the
  entry format documented in that file (bold question, then `Added` / `Source` / optional `Client` /
  optional `Context`). Create the file first (same header as claude-os's copy) if this repo doesn't
  have one yet. `Source: wrap`.
- No wiki → skip (nothing to park).
- Repo is NOT claude-os AND a question is business-level (BGD strategy, pricing, market — not
  implementation detail) → also drop a copy for AIOS, same mechanism as step 5b:
  `~/repos/claude-os/archives/raw/open-question-YYYY-MM-DD-<slug>.md` with `type: open-question`
  frontmatter, for `/dispatch` to route into claude-os's own `open-questions.md`.

### 4. Write to the wiki

Invoke `/wiki log` with the digest content. Provenance stamping (ADR 0010 §5): new/edited pages default `confidence: inferred` — `/wrap` is the agent synthesizing the session, not a human or source directly stating the claim. Bump to `verified` only if the content restates an authority surface (`state/`, `decisions/log.md`, `context/priorities.md`, `clients.yaml`) or Justin confirmed it in-session; never auto-mint `verified` otherwise.

### 5. Update state

Write/update `{repo}/STATE.md` (in `claude-os`, `state/<slug>.md` for each project touched, not a root STATE.md): bump Updated date and Status, then Accomplishments / Current Status / Next Steps matching the existing format. Thin — state is a springboard, not a transcript.

### 5b. Notify the AIOS (conditional — non-claude-os repos only)

If the repo is NOT claude-os, drop a state-update envelope into the AIOS inbox so scheduled `/dispatch` can sync the springboard card (decision 2026-07-07). Skip silently inside claude-os (you just wrote `state/` directly).

Write `~/repos/claude-os/archives/raw/state-update-YYYY-MM-DD-<slug>.md`:

```markdown
---
type: state-update
project: <slug>
origin: <repo>/wrap
date: YYYY-MM-DD
---

**Status:** {Enum} — {phase}

<1-5 lines of delta — what changed this session, not a retelling>

Next: <the top next steps>
```

- `project:` must be the `clients.yaml` project slug (check `~/repos/claude-os/clients.yaml` — it may differ from the repo folder name, e.g. repo `inside-out` → slug `inside-out-website`). If it doesn't resolve, use the repo folder name and note the mismatch in the envelope body.
- Follow the `state/README.md` contract: status enum (`Active`/`Blocked`/`Waiting`/`Launching`/`Paused`/`Done`), absolute dates.
- Delta only — no content payload. Nothing in this file should be worth ingesting anywhere else if misrouted.

### 6. Breadcrumb

Append one line to `~/.claude/session-wrap.log` (feeds /retro):
`<ISO date> | <slug> | digest: <path or "skipped"> | state: <path>`

### 7. Close

Report: files written, decisions that should be logged (suggest `decisions/log.md` in claude-os), compound note written (or skipped + why), and whether a git commit makes sense. Suggest the commit; don't push without being asked.

## Rules

- Write directly to the current repo's wiki via `/wiki log`. Never write outside the current repo — two exceptions, both drops into `claude-os/archives/raw/`: business-level research (step 3b) and the state-update envelope (step 5b). Cross-project context stays in the digest.
- Honest struggles. A digest that says "everything went great" when the session churned for an hour poisons the retro loop.
- Don't duplicate: if the session already wiki-logged its work (e.g. a /wiki log ran), note that in the breadcrumb and only fill gaps.

---

> *Adapted from The Three Ms of AI™. © 2026 Nate Herk. All rights reserved.*
> *The Three Ms of AI™ is a trademark of Nate Herk.*
