---
name: wrap
description: Wrap the session — digest to the project's wiki, update state, log a breadcrumb. Use when Justin types /wrap, or is clearly ending a work session.
---

# Wrap

Close a work session so the next one starts warm: what changed, what was decided, what is next, written where the project keeps its memory. Works in any git repo. Steps marked **(back office)** come from `aios.md` in this skill's folder and run only when the back-office repo it names exists on this machine.

## Steps

### 1. Resolve where you are

- Repo = current working directory's git root. Slug = repo folder name.
- Wiki = `{repo}/docs/wiki/` if it exists (check for `WIKI-CLAUDE.md` inside). No wiki → skip steps 3c and 4, STATE.md only.
- Back office = the path named in `aios.md`. If that directory does not exist, skip every **(back office)** step silently.
- **Resolve today's date once, now, in the machine's local time** (`date +%F`) and reuse that single value for every date a person will read: STATE.md `Updated:`, log headings, filenames, `date:` fields. Never derive it from a UTC timestamp and never re-resolve mid-wrap: an evening session in a timezone behind UTC would otherwise stamp tomorrow's date. Machine timestamps (`created_utc` and the like) may stay UTC.

### 2. Review and lesson (code sessions only)

Run this before the digest so findings land in its Next steps.

Check whether the session produced code changes: `git diff --name-only "$BASE_SHA"...HEAD`, where `BASE_SHA` is passed by the caller (for example a worker wrapping a merged PR); default `HEAD~5` for interactive sessions. Count non-doc, non-planning source files (`.ts`, `.tsx`, `.js`, `.py`, `.rb`, `.go`, `.css`, `.html`, `.svelte`, `.vue`, `.swift`, `.rs`, …; not `.md`, not `.planning/`).

If code changed:

1. **Code review** — invoke `/mattpocock-skills:code-review` (report only). If that skill is not installed, use the built-in `/code-review` without `--fix`. Its findings go into the digest.
2. **Lesson** — follow `compound.md` in this skill's folder: write or update one lesson in `docs/solutions/` when the session solved a real problem; otherwise note "no lesson" and why.

No code changes (research, decisions, planning): skip both silently.

### 3. Distill the session

Build a digest from the conversation. Sections, all required (write "none" rather than omitting):

- **Accomplishments** — what shipped or changed, with file paths
- **Decisions** — calls made and why, one line each; flag any that belong in a decisions log
- **Struggles** — where the session churned, corrected course or lost time, tooling friction included; `/retro` mines this section
- **Next steps** — checkbox list, concrete enough to resume cold, including code review findings
- **Open questions** — anything needing Justin's call

Substance gate: if the session was trivial (a question or two, no work product), say so and stop.

### 3b. Research capture (back office)

If the session was research-shaped (an external video, article or transcript discussed, a competitor or concept compared, strategic thinking out loud), follow the research step in `aios.md`. Otherwise skip silently. Project-level findings always stay in this repo's wiki (step 4).

### 3c. Park open questions

If the digest's **Open questions** is "none", skip silently. Otherwise append each unanswered question to `{repo}/docs/wiki/open-questions.md` in the entry format that file documents (bold question, then `Added` / `Source: wrap` / optional `Client` / optional `Context`). The file is created on demand, never scaffolded: if this repo has none, instantiate the `/wiki` skill's `assets/open-questions-TEMPLATE.md` and append the entry in the same write, then add the "Open questions" bullet to that wiki's `WIKI-CLAUDE.md` conventions.

**(back office)** Business-level questions (strategy, pricing, market; not implementation detail) also get the back-office drop in `aios.md`.

### 4. Write to the wiki

Invoke `/wiki log` with the digest. `/wiki` owns the wiki's filing and provenance rules (`WIKI-CLAUDE.md`, `schema.md`); wrap does not read or restate them, and uses `WIKI-CLAUDE.md` only as the marker that a wiki exists.

### 5. Update state

Write or update `{repo}/STATE.md`: bump the Updated date and Status, then Accomplishments / Current Status / Next Steps in the file's existing format. Keep it thin: state is a springboard, not a transcript. **(back office)** In the back-office repo itself, `aios.md` says where state lives instead.

### 5b. Notify the back office (back office)

Outside the back-office repo, write the state-update envelope described in `aios.md`.

### 6. Breadcrumb

Append one line to `~/.claude/session-wrap.log` (feeds `/retro`):
`<ISO date> | <slug> | digest: <path or "skipped"> | state: <path>`

### 6b. Push (back office)

Only when the repo being wrapped is the back-office repo: follow the push step in `aios.md`. Every other repo: suggest the commit, never push without being asked.

### 7. Close

Report: files written; decisions worth logging (in the back office, `aios.md` names the log); the lesson written or updated, or "no lesson" and why, plus the suggested `docs/solutions/` line if the instruction file lacks one; whether a commit makes sense (suggest it); and the back-office push result when step 6b ran.

## Rules

- Write only inside the current repo, through `/wiki log` for the wiki. The back-office drops in `aios.md` are the only writes outside it. Cross-project context stays in the digest.
- Honest struggles. A digest that says "everything went great" after an hour of churn poisons the retro loop.
- Don't duplicate: if the session already ran `/wiki log`, note that in the breadcrumb and only fill gaps.
