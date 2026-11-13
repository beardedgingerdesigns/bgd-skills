# Back-office steps (AIOS)

Back office: `~/repos/claude-os`

Wrap runs these steps only when that directory exists. They hand business-level material to the back office, where its scheduled `/dispatch` routes it. Inbox: `{back office}/archives/raw/`. Every drop is a new file there; wrap never edits anything else in the back office from another repo.

## Research (step 3b)

Follow the capture procedure in `{back office}/docs/wiki/research/CONVENTIONS.md`. Outside the back-office repo, business-level research (strategy, market, pricing, competitive landscape; not implementation detail) also drops a copy named `research-YYYY-MM-DD-<gist>.md` into the inbox.

## Business-level open questions (step 3c)

Outside the back-office repo, also drop each business-level question as `{inbox}/open-question-YYYY-MM-DD-<slug>.md` with `type: open-question` frontmatter, for `/dispatch` to route into the back office's own `open-questions.md`.

## State (step 5)

Inside the back-office repo, project state lives in `state/<slug>.md`, one file per project the session touched; there is no root `STATE.md`. Decisions worth logging go to `decisions/log.md` (step 7).

## State-update envelope (step 5b)

Outside the back-office repo, write `{inbox}/state-update-YYYY-MM-DD-<slug>.md` so `/dispatch` can sync the project's card:

```markdown
---
type: state-update
project: <slug>
origin: <repo>/wrap
date: YYYY-MM-DD
---

**Status:** {Enum} — {phase}

<1-5 lines of delta: what changed this session, not a retelling>

Next: <the top next steps>
```

- `project:` is the project slug in `{back office}/clients.yaml`, which may differ from the repo folder name (repo `inside-out` → slug `inside-out-website`). If none matches, use the folder name and say so in the body.
- Status is one of `Active` / `Blocked` / `Waiting` / `Launching` / `Paused` / `Done`, per `{back office}/state/README.md`; dates are absolute.
- Delta only, no content payload: nothing in it should be worth ingesting anywhere else if misrouted.

## Push (step 6b)

Only when the repo being wrapped is the back office (its `git rev-parse --show-toplevel` is the path above, or its `origin` remote is the back-office repo): after the session's final commit, run `git push origin main`.

- Push failed (no network, non-fast-forward, auth): report the error verbatim in the close report and stop. No retry, no `--force`, no rebase, no pull-and-merge; a failed push is Justin's call.
- Nothing to push: say so in one line.
