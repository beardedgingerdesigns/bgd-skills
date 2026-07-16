---
description: Extract a design blueprint from inspiration sites. Reads COMPETITOR-RESEARCH.md, extracts design DNA, selects components, and outputs a build-ready DESIGN-BLUEPRINT.md.
disable-model-invocation: true
---

# /design-extraction

Extract the design DNA from inspiration sites and distill it into a blueprint the build agent can follow. The blueprint is prescriptive — typography, colors, spacing, components, animation style — so the build agent executes rather than interprets.

Requires `COMPETITOR-RESEARCH.md` from `/competitor-research`. If it doesn't exist, tell the user to run that first.

## Step 1 — Gather inputs

Check for `COMPETITOR-RESEARCH.md` in the project root. Read it. Pull:
- The niche and site type
- The recommended inspiration sites (usually 2-3)
- The winner patterns (section order, CTA approach, hero style)

Then use `AskUserQuestion` — one call, all questions:

1. **Inspiration sites** — "The competitor research recommends [list them]. Want to use these, swap any, or add more?" (show the recommendations, let them override)
2. **Brand assets** — "Do you have existing brand guidelines? (Logo, colors, fonts — point me at a file, URL, or describe them. Leave blank if starting fresh.)"
3. **Vibe** — "Any sites you love the feel of, even outside your niche? (URLs or descriptions)"
4. **Constraints** — "Anything the design must NOT be? (e.g. 'no dark mode,' 'nothing too techy,' 'conservative — not startup-y')"

**Completion:** inspiration URLs finalized, brand constraints captured.

## Step 2 — Research award-winning design in the niche

Before extracting from the inspiration sites, research what award-winning sites in this niche look like. Use Exa and Firecrawl to scout Awwwards and similar galleries:

- `mcp__exa__web_search_exa` or `mcp__plugin_exa_exa__web_search_exa` — search for "[niche] site:awwwards.com", "[niche] website award winning", "[site type] awwwards nominee"
- `mcp__firecrawl__firecrawl_scrape` — scrape the top Awwwards results to extract what makes them award-worthy: interaction patterns, scroll behavior, 3D elements, typography choices, motion design

Surface the techniques that make the award winners stand out — could be bold typography, whitespace, scroll-driven animation, 3D elements, video, illustration, micro-interactions, or just impeccable restraint. Report what you find, not what you assume. The niche dictates what's appropriate.

**Completion:** Awwwards research done, 3-5 award-winning references collected with notes on what techniques they use and which are candidates for adoption.

## Step 3 — Extract design identity

For each inspiration site (2-3 max), use `mcp__firecrawl__firecrawl_scrape` to extract:

- Typography: font families, heading sizes, body size, weight usage, line heights
- Color palette: primary, secondary, accent, background, text colors (extract hex values)
- Spacing system: section padding, element gaps, whitespace ratios
- Layout patterns: grid structure, content width, alignment tendencies
- Imagery style: photography vs illustration vs abstract, treatment (rounded corners, shadows, overlays)
- Animation approach: subtle vs bold, scroll-triggered vs hover, speed/easing feel

Cross-reference with Refero for deeper pattern context:
- `mcp__claude_ai_Refero__refero_search_styles` — find the design system or style closest to each inspiration site
- `mcp__claude_ai_Refero__refero_get_screen_image` — grab reference screenshots of key sections

Where multiple inspiration sites conflict (one uses serif, another sans-serif), note both and resolve toward the client's brand constraints from Step 1. If no brand exists, pick the pattern the winner research supports.

Then run the three gates in [`BLUEPRINT-TEMPLATE.md`](BLUEPRINT-TEMPLATE.md) against what you extracted:

1. **Category reflex** — could you have guessed this palette or font from the niche name alone? If yes, you stopped reading the evidence. Rework it.
2. **Banned palette** — none of the listed premium-consumer hex values survive. Rotate to one of the seven families, and not the one the last project used.
3. **Font reflex** — anything on the reflex-reject list needs a written reason to survive.

The gates outrank your instinct, not the evidence. If a scraped winner genuinely uses a banned font well, say so and keep it — but you don't inherit it by default.

**Completion:** design identity extracted from every inspiration site, conflicts resolved, Refero references collected, all three gates passed with the reflex-check lines written.

## Step 4 — Select components

Using the extracted identity, the Awwwards research, and the winner patterns from `COMPETITOR-RESEARCH.md`, select specific components from:

- **Magic UI** (magicui.design) — animated marketing components, shadcn/ui compatible
- **21st.dev** — hero sections, navs, CTAs, feature grids
- **CodePen** — effects or animations that match the identity
For each selected component:
- Name and source URL
- Where it goes in the page structure (hero, features section, testimonials, footer)
- Any modifications needed to match the extracted identity (color swap, font override)

Target 4-8 components. Enough to define the feel, not so many the build agent drowns.

**Completion:** component list finalized with placement and modification notes.

## Step 5 — Write the blueprint

Write `DESIGN-BLUEPRINT.md` in the project root (or user-specified path), following the
template in [`BLUEPRINT-TEMPLATE.md`](BLUEPRINT-TEMPLATE.md). Copy the TEMPLATE section
and fill every field. The `▸ REFERENCE` sections are your source for real values —
the 4pt spacing scale, the easing curves and duration table, the named background modes,
the color-commitment axis. Don't invent values the reference already fixes.

Three sections carry the most weight and are the easiest to fudge:

- **Design System Variables** — token names get decided here, not at build time. This is
  the cleanest expression of "the build executes rather than interprets."
- **Do Not Use For** — a direction defined partly by its exclusions is much harder to
  drift from than one defined only by what it is.
- **Implementation Checklist** — the blueprint carries its own verification. The build
  self-checks against it, and stage 4 verifies it independently.

**Completion:** every field filled. Typography has real font names and a written reflex
check. Colors have real hex values and a written reflex check. Components have real URLs.
Motion has real cubic-beziers and durations, not "subtle fade-up." Assets name a background
mode from the reference list, not free text. Nothing says "TBD" or "choose later."

## Step 6 — Handoff

Tell the user:
- Where the blueprint was written
- Quick summary: "The design is [vibe in 3 words]. [Font]. [Color feel]. [Key component]."
- Anything the build agent will still need: product images, copy, brand assets not yet provided

Do not start building. This skill extracts; the next agent builds.
