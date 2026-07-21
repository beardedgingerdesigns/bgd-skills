---
name: catchup
description: Re-orient at the start of a session — find the latest handoff for the current repo and summarize where work left off. Use when Justin says "/catchup", "catch me up", "where did we leave off", "pick up where we left off", "what were we doing", or opens a session cold in a repo. Reads the newest handoff in the repo's /tmp folder plus STATE/CLAUDE.md and recent git, and presents a one-screen orientation. Distinct from the built-in /resume (which reloads a prior conversation) — this orients a fresh session from the handoff.
bike-method-phase: 1
three-ms-attribution: |
  Adapted from The Three Ms of AI™ © 2026 Nate Herk. All rights reserved.
  The Three Ms of AI™ is a trademark of Nate Herk.
---

# /catchup — pick up where you left off

Kills the start-of-session re-orientation tax. Stop hunting for "where did we leave off" or pasting a handoff path — one command surfaces the latest handoff plus current repo state.

Not to be confused with the built-in `/resume`, which reloads a previous *conversation*. `/catchup` starts fresh and orients from the handoff file `/handoff` left behind.

## Why handoffs live in /tmp (don't move them)

`/handoff` writes to a per-repo folder in the OS temp dir — `/tmp/<repo-name>/handoff-<timestamp>.md`. This is on purpose: the OS auto-deletes `/tmp` after ~3 days, so handoffs self-clean and never bloat the repo. A handoff older than 3 days is gone — which is correct, because it would be stale anyway. **Never** store handoffs in the repo.

## Steps

1. **Identify the repo.** `git rev-parse --show-toplevel`; take its basename as the repo name (e.g. `brandos`, `claude-os`, `tonequest`). Not a git repo → use the cwd basename.

2. **Find the newest handoff** in this repo's temp folder (glob-free so it doesn't error in zsh when the folder is empty):
   ```
   ls -t /tmp/<repo-name>/ 2>/dev/null | grep -m1 '^handoff-.*\.md$'
   ```
   That returns the newest handoff's filename — read `/tmp/<repo-name>/<filename>`. `/handoff` writes there with timestamped names, so newest-by-mtime is the latest handoff. Empty output = no handoff (none written, or aged out past the ~3-day temp TTL) → use the fallback.

3. **Read the handoff** if one was found.

4. **Read current repo state regardless:** the repo's `CLAUDE.md`, any `STATE.md` or `state/` it keeps, the current branch (`git branch --show-current`), uncommitted changes (`git status --short`), and recent commits (`git log --oneline -10`).

5. **Present a one-screen orientation** in chat:
   - **Where we left off** — the gist of the handoff (or, if none, inferred from recent commits + STATE).
   - **Repo state now** — branch, uncommitted changes, last few commits.
   - **Next** — the open thread / next step the handoff or STATE names.

   Keep it tight. This is orientation, not a report.

## Fallbacks

- **No handoff found** (none written, or aged out past the ~3-day `/tmp` TTL): say so plainly, then orient from `STATE.md` / `CLAUDE.md` + recent git instead. A missing handoff is normal, not an error.
- **Not a git repo:** orient from `CLAUDE.md` + whatever state files exist; skip the git steps.

## Bike Method

**Phase 1 — run it manually at session start.** Do NOT wire it to an auto-on-start hook yet; prove the manual version across a few real sessions first. Done-state: `/catchup` self-orients in 3 consecutive sessions across 2+ repos with no path-paste. Advance phases only by explicit edit to `bike-method-phase`.

---

> *Adapted from The Three Ms of AI™. © 2026 Nate Herk. All rights reserved.*
> *The Three Ms of AI™ is a trademark of Nate Herk.*
