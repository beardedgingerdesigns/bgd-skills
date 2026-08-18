# bgd-skills

Custom Claude Code skills authored by Bearded Ginger Designs. Each skill dir is symlinked into `~/.claude/skills/` so Claude picks them up globally across all projects.

## Skills

### Orchestration & session

- **[dayshift](dayshift/SKILL.md)** — Governing contract for the human-in-the-loop orchestrator running a day of work on ONE project: delegate and verify, never implement. Load when Justin takes the command-ship seat (`/dayshift`). Night-shift/AFK is the separate `nightshift` skill (lives in claude-os).
- **[catchup](catchup/SKILL.md)** — Re-orient at the start of a session: reads the newest handoff in the repo's `/tmp` folder plus STATE/CLAUDE.md and recent git, presents a one-screen orientation. `/catchup`. Distinct from built-in `/resume`.

### Website pipeline

Four stages, run in order:

- **[competitor-research](competitor-research/SKILL.md)** — Stage 1. Scout the competitive landscape; output a structured `COMPETITOR-RESEARCH.md` blueprint.
- **[design-extraction](design-extraction/SKILL.md)** — Stage 2. Extract design DNA from inspiration sites into a build-ready `DESIGN-BLUEPRINT.md`.
- **[design-build](design-build/SKILL.md)** — Stage 3. Build the site from `COMPETITOR-RESEARCH.md` + `DESIGN-BLUEPRINT.md`.
- **[design-verify](design-verify/SKILL.md)** — Stage 4. Verify the built site against the blueprint: computed-style diff for mechanical claims, screenshot reads for visual ones. Fixes drift, re-verifies, reports.

### QA

- **[verify-ui](verify-ui/SKILL.md)** — Agent-side visual QA. Drives Playwright against the local dev server, logs in as each test role, screenshots key routes, captures console errors, then READS the screenshots and judges them. `/verify-ui`. Localhost/dev only.

### Web data

- **[web-scraping](web-scraping/SKILL.md)** — Scrape, search, and extract structured data using Exa (semantic search) and Firecrawl (JS-rendered page scraping). Routes between tools by job.
- **[firecrawl](firecrawl/SKILL.md)** — Bootstrap skill for the Firecrawl CLI + MCP. One install command sets up CLI, build, and workflow segments.

### Wiki & meta

- **[wiki](wiki/SKILL.md)** — Canonical project wiki curator. Modes: bootstrap, convert, ingest, research, log, lint, query, decide, human-version. The only writer to curated pages.
- **[refine-skill](refine-skill/SKILL.md)** — Critically review a skill for verbosity and over-specification, then rewrite it. Lens: "if I deleted this line, would the model do something worse?"

## Tracked, not a skill

- **[sropus](sropus/README.md)** — The `sr-opus-5.md` Opus 5 system prompt + implementation notes. Backup of record for `~/.claude/sr-opus-5.md`. Tracked here for versioning only — no symlink (it's an appended-system-prompt file, invoked by the `sropus()` zsh func, not a skill).

## Setup

Skills are symlinked from `~/.claude/skills/` into this repo. Edits in either location are the same files.

```bash
# Clone
git clone git@github.com:beardedgingerdesigns/bgd-skills.git ~/repos/bgd-skills

# Symlink all skills (sropus has no SKILL.md — it's tracked-only, skip it)
for skill in ~/repos/bgd-skills/*/; do
  name=$(basename "$skill")
  case "$name" in .git|sropus) continue;; esac
  ln -sfn "$skill" ~/.claude/skills/"$name"
done
```

### Adding a new skill

```bash
mkdir ~/repos/bgd-skills/my-skill
# write SKILL.md
ln -s ~/repos/bgd-skills/my-skill ~/.claude/skills/my-skill
cd ~/repos/bgd-skills && git add my-skill && git commit -m "feat: my-skill" && git push
```
