# bgd-skills

Custom Claude Code skills authored by Bearded Ginger Designs. Each skill is symlinked into `~/.claude/skills/` so Claude picks them up globally across all projects.

## Skills

### [wiki](wiki/SKILL.md)

Canonical project wiki curator. Nine modes: bootstrap (`/wiki init`), convert existing docs (`/wiki convert`), ingest raw sources (`/wiki ingest`), web research (`/wiki research`), session wrap-up (`/wiki log`), health checks (`/wiki lint`), queries (`/wiki query`), decision recording (`/wiki decide`), and browsable static site (`/wiki human-version`). The wiki skill is the only writer to curated pages -- external producers stage to `raw/<source>/` and the ingest pipeline curates.

### [ask-the-board](ask-the-board/SKILL.md)

Deliberate with a board of advisors. Each advisor is a subagent backed by ingested creator content -- books, talks, essays. Advisors reason independently in isolation, then the board synthesizes a unified response with agreements, tensions, and a bottom-line recommendation. Supports quick-take (default) and deep multi-round deliberation.

### [add-board-member](add-board-member/SKILL.md)

Onboard a new advisor or expand an existing one's knowledge base. Discovers content (local files, web, provided URLs), runs parallel knowledge extraction across sources, synthesizes a persona profile with voice patterns and decision-making heuristics, and registers the advisor in the board roster. Works with any project's advisory board or the global AIOS board.

### [web-scraping](web-scraping/SKILL.md)

Scrape, search, and extract structured data from the web using Exa (semantic search) and Firecrawl (JS-rendered page scraping). Routes between tools based on the job -- Exa for semantic discovery and research, Firecrawl for full-page extraction from known URLs and JS-heavy sites. Includes workflows for search-then-scrape, site crawling, and structured data extraction.

### [refine-skill](refine-skill/SKILL.md)

Critically review a skill for verbosity and over-specification, then rewrite it. Applies the grill-me standard: describe behavior and non-obvious constraints, trust the model for everything else. Core lens: "if I deleted this line, would the model do something worse?" Sorts every line into three buckets -- model already knows (delete), model would derive (delete), non-obvious constraint (keep).

### [firecrawl](firecrawl/SKILL.md)

Bootstrap skill for the Firecrawl CLI and MCP integration. One install command (`npx firecrawl-cli init --all --browser`) sets up three skill segments: CLI tools for live web work, build skills for adding Firecrawl to application code, and workflow skills for producing deliverables (research briefs, SEO audits, lead lists, knowledge bases).

## Setup

Skills are symlinked from `~/.claude/skills/` into this repo. Edits in either location are the same files.

```bash
# Clone
git clone git@github.com:beardedgingerdesigns/bgd-skills.git ~/repos/bgd-skills

# Symlink all skills
for skill in ~/repos/bgd-skills/*/; do
  name=$(basename "$skill")
  [ "$name" = ".git" ] && continue
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
