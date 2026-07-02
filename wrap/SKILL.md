---
name: wrap
description: Session wrap-up in one word. Distills the current session into a digest (accomplishments, decisions, struggles, next steps), writes it to the project's wiki via /wiki log, updates STATE.md, and logs a breadcrumb. Works from any project repo. Use when Justin says "/wrap", "wrap up", "wrap this session", "done for the night", "close out", "hit the wiki and close out", or is clearly ending a work session.
bike-method-phase: 1  # Phase 1 — Training wheels. Run manually first. Hook automation comes after a validated week.
three-ms-attribution: |
  Adapted from The Three Ms of AI™ © 2026 Nate Herk. All rights reserved.
  The Three Ms of AI™ is a trademark of Nate Herk.
---

# Wrap

One word replaces the four commands Justin types ~23 times a week: "save this to the wiki," "update state," "write a handoff," "log everything." Scoped via /level-up 2026-06-12 from retro evidence (retros/retro-2026-06-12.md, pattern #1).

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

### 3. Distill the session (the one AI step — it's you, from context)

Build a digest from the conversation so far. Include any code review findings from step 2 in the **Next steps** section. Sections, all required (write "none" rather than omitting):

- **Accomplishments** — what shipped/changed, with file paths
- **Decisions** — calls made and why, one line each; flag any that belong in a decisions log
- **Struggles** — where we churned, corrected, or burned time. Be honest and specific; this section is mined by /retro (phase 2 input). Include tooling friction, not just code problems.
- **Next steps** — checkbox list, concrete enough to resume cold. Include code review findings if any.
- **Open questions** — anything needing Justin's call

Substance gate: if the session was trivial (a question or two, no work product), say so and stop — don't log noise.

### 3b. Research capture (conditional)

If the session was research-shaped — an external video/article/transcript discussed, a competitor or concept compared to Justin's approach, strategic thinking-out-loud — execute the capture procedure in `~/repos/claude-os/docs/wiki/research/CONVENTIONS.md`:

- **In claude-os:** write the note to `docs/wiki/research/notes/`, tag topics, update tagged synthesis pages, wire backlinks. Silent; one-line receipt in the close report (louder if a prior conclusion got reversed).
- **In any other repo:** project-level findings stay in this project's wiki (normal steps 4-5). But if the research was *business-level* (BGD strategy, market, pricing, competitive landscape — not project implementation detail), also drop a copy into `~/repos/claude-os/archives/raw/` named `research-YYYY-MM-DD-<gist>.md` for `/dispatch` to route. This is the one sanctioned cross-repo write — it targets the staging inbox, never curated pages.

If the session had no research flavor, skip silently.

### 4. Write to the wiki

Invoke `/wiki log` with the digest content. `/wiki log` handles curation: writing to the appropriate curated pages, updating log.md, and updating index.md.

If the repo has no wiki (no `docs/wiki/` with `WIKI-CLAUDE.md`), skip this step — STATE.md only.

### 5. Update state

Write/update `{repo}/STATE.md` (in `claude-os`, `state/<slug>.md` for each project touched, not a root STATE.md): bump Updated date and Status, then Accomplishments / Current Status / Next Steps matching the existing format. Thin — state is a springboard, not a transcript.

### 6. Breadcrumb

Append one line to `~/.claude/session-wrap.log`:
`<ISO date> | <slug> | digest: <path or "skipped"> | state: <path>`

This is the audit trail that proves wraps are happening (and feeds /retro).

### 7. Close

Report: files written, decisions that should be logged (suggest `decisions/log.md` in claude-os), compound note written (or skipped + why), and whether a git commit makes sense. Suggest the commit; don't push without being asked.

## Rules

- Write directly to the current repo's wiki via `/wiki log`. Never write outside the current repo — one exception: business-level research notes drop into `~/repos/claude-os/archives/raw/` (step 3b; staging inbox only, never curated pages). Cross-project context stays in the digest.
- Honest struggles. A digest that says "everything went great" when the session churned for an hour poisons the retro loop.
- Don't duplicate: if the session already wiki-logged its work (e.g. a /wiki log ran), note that in the breadcrumb and only fill gaps.

## Bike Method status

- **Phase 1 (now):** Justin runs `/wrap` manually at session end. Validates output format, digest quality, wiki writes.
- **Phase 2 (after ~a week of clean runs):** wire the trigger — Stop/SessionEnd hook spawning a detached digest pass (extend `claude-os/scripts/state-hook`; known issue: SessionEnd doesn't fire from the IDE extension, so the hook needs a breadcrumb check as fallback).
- **Advance only by editing `bike-method-phase` here, deliberately.**
