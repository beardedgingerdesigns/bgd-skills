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

**Completion:** design identity extracted from every inspiration site, conflicts resolved, Refero references collected.

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

Write `DESIGN-BLUEPRINT.md` in the project root (or user-specified path). Structure:

```markdown
# Design Blueprint: [Project Name]

## Source
- Competitor research: [path to COMPETITOR-RESEARCH.md]
- Inspiration sites: [URLs]
- Brand guidelines: [source or "none — derived from research"]

## Typography
- Headings: [font family], [weights used], [scale: h1 → h6 sizes]
- Body: [font family], [size], [line-height], [weight]
- Accent/display: [if any]
- Font loading: [Google Fonts URL or local files]

## Color System
- Primary: [hex] — [where used]
- Secondary: [hex] — [where used]
- Accent: [hex] — [where used]
- Background: [hex]
- Surface: [hex] — cards, elevated elements
- Text: [hex primary], [hex muted]
- Border: [hex]

## Spacing & Layout
- Content max-width: [px]
- Section padding: [vertical], [horizontal]
- Element gap: [standard spacing unit]
- Grid: [columns, gutter]

## Visual Style
- Imagery: [photography / illustration / abstract / AI-generated — treatment notes]
- Border radius: [px]
- Shadow style: [description or CSS values]
- Background patterns: [gradients, textures, or clean]

## Animation & Interaction
- Style: [subtle / bold / minimal]
- Triggers: [scroll / hover / load]
- Speed: [fast / medium / deliberate]
- Specific effects: [parallax, fade-in, slide-up, etc.]

## Awwwards Research
[3-5 award-winning references with notes on techniques used and what's worth adopting]

## Visual Effects
[Recommended effects based on Awwwards research and identity extraction.]
[For each: what effect, where it goes, why it fits, and a simpler fallback.]
[Only if the research supports it. "None — clean execution is the effect" is a valid answer.]

## Component Kit
[For each selected component:]
### [Component name] — [placement]
- Source: [URL]
- Modifications: [color, font, size adjustments]
- Purpose: [what it does in the page]

## Asset Requirements
[What the build agent needs to generate or source. For each:]
- What: [hero product shot, background texture, product rotation video, etc.]
- Placement: [where in the page]
- Style notes: [color palette, mood, treatment — enough for Higgsfield prompting]
- Source: [AI-generated / client-provided / stock — recommendation]

## Page Structure
[Section order from top to bottom, one line each:]
1. [Nav] — [style notes]
2. [Hero] — [approach: product shot / video / text-led]
3. [Social proof bar] — [logos / stats / testimonial]
...

## Do / Don't
### Do
- [Specific positive rules derived from the extraction]

### Don't
- [Specific negative rules from client constraints + anti-patterns from research]

## Refero References
[Screen/flow references with annotation: what pattern, why included]
```

**Completion:** blueprint written with every section filled. Typography has actual font names. Colors have actual hex values. Components have actual URLs. Nothing says "TBD" or "choose later."

## Step 6 — Handoff

Tell the user:
- Where the blueprint was written
- Quick summary: "The design is [vibe in 3 words]. [Font]. [Color feel]. [Key component]."
- Anything the build agent will still need: product images, copy, brand assets not yet provided

Do not start building. This skill extracts; the next agent builds.
