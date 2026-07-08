---
description: Build a website from COMPETITOR-RESEARCH.md and DESIGN-BLUEPRINT.md.
disable-model-invocation: true
---

# /design-build

Build the website. Read the research and the blueprint, close any gaps, then build the whole thing. The user refines from there.

Requires both `COMPETITOR-RESEARCH.md` and `DESIGN-BLUEPRINT.md` in the project. If either is missing, tell the user which skill to run first.

## Step 1 — Read the ground

Read both documents. Then read the codebase:

- Existing components, design tokens, layouts, shared styles — extend rather than duplicate.
- Package.json / dependencies already installed.
- Any CLAUDE.md, AGENTS.md, or project conventions.

**Completion:** both documents read, codebase scanned.

## Step 2 — Close gaps

Use `AskUserQuestion` for anything the documents don't answer. Batch into one call — this isn't a grill, just closing holes:

- **Stack** — if not obvious from the codebase or blueprint
- **Copy** — does the user have page copy, or draft from the competitor research patterns?
- **Assets** — does the user have product images, or generate with Higgsfield?
- **Deploy target** — if not already clear

If everything is answered by the documents and codebase, skip this step entirely.

**Completion:** no open questions.

## Step 3 — Build

Build the entire site in one pass. Follow the blueprint:

- Set up design tokens first (fonts, colors, spacing from the blueprint).
- Follow the Page Structure section top to bottom.
- Snap in components from the Component Kit, modify per the blueprint's style rules.
- Respect the Do/Don't rules.
- Draft copy from competitor research patterns if the user didn't provide it. Mark drafted copy with a comment so they know what to review.

Generate assets as needed:
- `mcp__higgsfield__generate_image` — product shots, hero imagery, textures. Prompt with the blueprint's color palette and style notes.
- `mcp__higgsfield__generate_video` — if the blueprint calls for motion (hero video, product rotation).
- `mcp__higgsfield__remove_background` — clean up product shots for placement.

Implement visual effects from the blueprint. Use Three.js, CSS animation, Framer Motion, or whatever fits — the blueprint specifies the *what*, you pick the *how*. If an effect isn't worth the weight, use the fallback and note why.

**Completion:** site builds and runs.

## Step 4 — Show it

Start the dev server. Tell the user:

- It's running — go look at it
- What was generated vs. what needs replacing (AI images, drafted copy)
- Anything that deviated from the blueprint and why

Then stop. The user drives refinement from here.
