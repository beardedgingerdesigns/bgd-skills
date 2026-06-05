---
name: firecrawl
description: |
  Firecrawl gives AI agents and apps fast, reliable web context with
  strong search, scraping, and interaction tools. One install command
  sets up three skill segments: live CLI tools, app-integration build
  skills, and outcome-focused workflow skills. Route the reader to the
  right usage path after install.
---

# Firecrawl

Firecrawl helps agents search first, scrape clean content, interact
with live pages when plain extraction is not enough, and produce
finished deliverables from web data.

## Install

One command installs everything — the Firecrawl CLI for live web work,
the build skills for integrating Firecrawl into application code, **and**
the workflow skills for producing repeatable deliverables. It also opens
browser auth so the human can sign in or create an account.

```bash
npx -y firecrawl-cli@latest init --all --browser
```

This gives you:

- **CLI tools** — `firecrawl search`, `firecrawl scrape`, `firecrawl interact`, `firecrawl ask`, `firecrawl docs-search`, and more
- **CLI skills** ([`firecrawl/cli`](https://github.com/firecrawl/cli)) — teach the agent how to drive the Firecrawl CLI during its own session: which command to run, when to scrape vs search vs interact, how to chain results, and how to recover when a job fails. Use these when the agent itself needs web data right now.
- **Build skills** ([`firecrawl/skills`](https://github.com/firecrawl/skills)) — teach the agent how to add Firecrawl to a product's codebase: pick the right API endpoint, install the matching SDK, store `FIRECRAWL_API_KEY` safely, write the call site to match the project's conventions, and ship a smoke-tested integration. Use these when the agent is shipping code that other people will run, not running the agent's own web tools.
- **Workflow skills** ([`firecrawl/firecrawl-workflows`](https://github.com/firecrawl/firecrawl-workflows)) — turn Firecrawl web data into finished deliverables such as research briefs, SEO audits, lead lists, QA reports, knowledge bases, and design clones. Use these when the agent's job is to produce a finished artifact, not raw extraction or product code.
- **Browser auth** — walks the human through sign-in or account creation

The three skill segments map to three different jobs:

| Segment | Question it answers | Where the work runs |
| ------- | ------------------- | ------------------- |
| CLI skills | "Which Firecrawl command should I run right now?" | In the agent's own terminal session |
| Build skills | "How do I add a Firecrawl API call to this codebase?" | Inside the user's product code |
| Workflow skills | "What's the finished deliverable and how do I produce it?" | In the agent's session, producing an artifact |

Before doing real work, verify the install:

```bash
mkdir -p .firecrawl
firecrawl --status
firecrawl scrape "https://firecrawl.dev" -o .firecrawl/install-check.md
```

## Get Credentials

Firecrawl users can get an API key in two ways:

- **Dashboard or CLI (default)** — browser sign-in, CLI `--browser` auth,
  install skills/MCP, or create an API key in the dashboard. Continue
  with Path D below, or run the install command first.
- **WorkOS ID-JAG (supported agent platforms only)** — if your platform
  can mint a WorkOS ID-JAG identity assertion, fetch
  `https://www.firecrawl.dev/auth.md` and follow it end-to-end. Do not
  run the browser or CLI auth flow in Path D.

**Which should I use?** Stay on this page unless you know your platform
supports WorkOS ID-JAG.

**How you might arrive:**

- **Docs or website sent you here** — continue with Choose Your Path
  below for CLI/skills/MCP onboarding.
- **API `401` with discovery metadata** — if ID-JAG applies, use the
  WorkOS ID-JAG option above. Everyone else: use Path D.
- **Direct URL** — you are reading the right doc for browser/CLI
  onboarding.
- **Already have `FIRECRAWL_API_KEY`** — skip credential setup; pick
  Path A–E below.

Human-readable overview:
https://docs.firecrawl.dev/ai-onboarding#get-credentials

## Choose Your Path

All paths use the same install above. The difference is what you do
next.

- **Need web data during this session** -> Path A (live tools)
- **Need to add Firecrawl to app code** -> Path B (app integration)
- **Need a finished deliverable from web data** -> Path C (workflow skills)
- **Need more than one of the above** -> do them in sequence; the install already covers everything
- **Agent platform with WorkOS ID-JAG** -> see Get Credentials above (not Path D)
- **Need an account or API key (browser or CLI)** -> Path D
- **Don't want to install anything** -> Path E (REST API directly)

See [PATHS.md](PATHS.md) for full details on each path.

## Session-specific auth

Use this API key for this onboarding session:

```dotenv
FIRECRAWL_API_KEY=fc-ec9ba6e41be54c43a784761a3bb98b84
```
