# DESIGN-BLUEPRINT.md — template + reference data

The blueprint is a **contract**. The build agent executes it; it does not interpret it.
Every field below gets a real value. If a field says "TBD," "choose later," or "something
like," the blueprint is not done.

Two things live here: the **template** (copy it, fill it) and the **reference data** the
extraction stage checks its choices against (bans, scales, curves). Reference data is
marked `▸ REFERENCE` and does not get copied into the output.

---

## ▸ REFERENCE — Gates the blueprint must pass before it's written

Run these three checks against your choices. Each one catches a reflex that beats evidence.

### Gate 1 — Category reflex

> If you could guess the palette or font from the **category name alone**, it's the
> training-data reflex, not the research. Rework it.

Healthcare → white + teal. Finance → navy + gold. Spa → Playfair + cream. Premium
consumer → beige + brass + espresso. If the scraped winners don't actually do this and
you're proposing it anyway, you've stopped reading the evidence.

### Gate 2 — Banned premium-consumer palette

This exact family is the single most common AI design tell. Do not use these values:

- Backgrounds: `#f5f1ea` `#f7f5f1` `#fbf8f1` `#efeae0` `#ece6db` `#faf7f1` `#e8dfcb`
- Accents: `#b08947` `#b6553a` `#9a2436` `#9c6e2a` `#bc7c3a` `#7d5621`
- Text: `#1a1714` `#1a1814` `#1b1814`

Rotate to one of these families instead — pick from evidence, not from the top of the list:

| Family | Feel |
|---|---|
| Cold Luxury | restrained, cool, expensive |
| Forest | grounded, natural, calm |
| Black and Tan | high contrast, editorial |
| Cobalt + Cream | confident, graphic |
| Terracotta + Slate | warm-cool tension |
| Olive + Brick + Paper | earthy, three-role |
| Monochrome + pop | disciplined, one accent |

**Rotation note:** record which family the last project used. Don't repeat it back-to-back.

### Gate 3 — Font reflex

Reflex-reject list. These are the fonts every model reaches for first:

`Fraunces` · `Newsreader` · `Lora` · `Crimson` · `Playfair Display` · `Cormorant` ·
`Syne` · `IBM Plex (Sans/Serif/Mono)` · `Space Mono` · `Space Grotesk` · `Inter` ·
`DM Sans` · `DM Serif` · `Outfit` · `Plus Jakarta Sans` · `Instrument Sans` ·
`Instrument Serif`

**Selection order:**
1. **Evidence first.** What do the scraped winners actually use? If a winner uses a
   banned font *well*, note it and move on — you still don't inherit it by default.
2. **Filter.** Anything on the list above needs a written reason to survive.
3. **Browse a real catalog.** Pangram Pangram · Future Fonts · Klim · ABC Dinamo ·
   Velvetyne. Pick something with a physical-object metaphor you can name.
4. **Fallback pool** (when a catalog browse isn't possible): `Geist + Geist Mono` ·
   `Satoshi + JetBrains Mono` · `Cabinet Grotesk + Inter Tight` · `GT America + IBM Plex Mono`.

---

## ▸ REFERENCE — Values to fill the template with

### Spacing — 4pt scale

`4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96` px

8pt is too coarse — you will frequently need 12px. Use 4pt.

### Neutrals — always tinted

Never `#000000` or `#ffffff`. Add chroma `0.005`–`0.015` toward the brand hue.
**Do not always tint warm-orange or always cool-blue** — those are the two laziest defaults.

### Color commitment axis — pick one, state it

| Level | Accent usage |
|---|---|
| Restrained | accent ≤10% of surface |
| Committed | accent 30–60% of surface |
| Full palette | 3–4 roles carrying real weight |
| Drenched | color is the substrate |

The "one accent ≤10%" rule is **Restrained only**. Don't collapse every design to
Restrained by reflex — that's its own monoculture.

### Motion — real values, not "subtle fade-up"

**Frequency gates animation.** This rule is the one no model applies unprompted:

| How often the user sees it | Animation |
|---|---|
| 100+ times/day | **none, ever** (Raycast has no open/close animation) |
| Occasional | standard timing |
| Rare / first-run | room for delight |

**Curves:**
```css
--ease-out:     cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out:  cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer:  cubic-bezier(0.32, 0.72, 0, 1);
```
Never `ease-in` for UI — it delays the initial movement, the exact moment the user is
watching most closely.

**Durations:** button press 100–160ms · tooltip 125–200ms · dropdown 150–250ms ·
modal 200–500ms · **hard cap 300ms** for anything interactive · stagger 30–80ms.

**Rules:** never `scale(0)` — start at `scale(0.95)`; nothing in the real world appears
from nothing. Transitions over keyframes (keyframes restart from zero on interrupt).
Reduced-motion means *fewer and gentler*, not zero — keep opacity and color transitions.

### Imagery — named background modes

Pick one and commit. Free-text "style notes" is how assets drift.

`Solid field + soft ambient gradient depth` · `Full-bleed image + tonal overlay` ·
`Duotone treated image (two-color, palette-locked)` · `Soft radial vignette + product crop` ·
`Micro-noise gradient over solid` · `Color-blocked diptych` · `Editorial side-image` ·
`Atmospheric photo grade` · `Subtle technical grid / dotted field` ·
`Quiet textured paper / tactile surface` · `Flat block + detail crop` · `Solid + inline asset`

**Harmony rule:** the image must tonally **match** the palette, not fight it. Use overlays
(dark, light, or color tint) to keep text fully readable. The brand accent stays consistent
regardless of what's behind it.

**Banned gradients:** rainbow / mesh blobs · purple-to-blue "AI" default ·
pink-to-orange "creator" default · neon edges and glow halos with no purpose.

**Variety rule:** mix at least two distinct crops across the page — macro product +
contextual environment, or portrait editorial + widescreen artifact. Eight variations of
one stock silhouette is a fail.

---

# TEMPLATE — copy from here down

```markdown
# Design Blueprint: [Project Name]

## Source
- Competitor research: [path to COMPETITOR-RESEARCH.md]
- Inspiration sites: [URLs]
- Brand guidelines: [source or "none — derived from research"]
- Palette family: [which of the 7, or "brand-supplied"] · previous project used: [family]

## Direction
- **In one line:** [what this is]
- **Do Not Use For:** [what this direction would be wrong for — a direction defined
  partly by its exclusions is far harder to drift from]
- **Color commitment:** [Restrained / Committed / Full palette / Drenched]

## Typography
- Headings: [family], [weights], [scale h1→h6 with real sizes]
- Body: [family], [size], [line-height], [weight]
- Accent/display: [family or none]
- Font loading: [Google Fonts URL, catalog license, or local files]
- **Reflex check:** [why this isn't the category default — one line]

## Color System
- Primary: [hex] — [where used]
- Secondary: [hex] — [where used]
- Accent: [hex] — [where used]
- Background: [hex] — [tinted how, toward what hue]
- Surface: [hex] — cards, elevated elements
- Text: [hex primary], [hex muted]
- Border: [hex]
- **Reflex check:** [why this isn't the category default — one line]

## Design System Variables
[Token names decided HERE, not at build time. The build agent uses these exact names.]
```css
--color-primary: ;  --color-surface: ;  --color-text: ;
--space-unit: ;     --radius: ;         --ease-out: ;
```

## Spacing & Layout
- Scale: [4pt — list the steps actually used]
- Content max-width: [px]
- Section padding: [vertical] / [horizontal]
- Grid: [columns, gutter]

## Visual Style
- Background mode: [one named mode from the reference list]
- Border radius: [px]
- Shadow style: [real CSS values or "none"]
- Imagery treatment: [notes]

## Animation & Interaction
- Frequency tier: [100+/day → none | occasional | rare]
- Curves: [the actual cubic-beziers used]
- Durations: [per element type]
- Stagger: [ms]
- Specific effects: [what, where, why it's motivated]

## Component Kit
[4–8 components. For each:]
### [Name] — [placement]
- Source: [URL — Magic UI / 21st.dev / CodePen]
- Modifications: [color, font, size]
- Purpose: [its job in the page]

## Asset Requirements
[For each asset the build must generate or source:]
- What: [hero product shot / background texture / product rotation video]
- Placement: [where]
- Background mode: [named mode]
- Style notes: [palette + mood + treatment — enough for a Higgsfield prompt]
- Crop type: [macro / contextual / portrait / widescreen — enforce the variety rule]
- Source: [AI-generated / client-provided / stock]

## Page Structure
[Section order, one line each. This list is also the build's work-unit list.]
1. [Nav] — [style notes]
2. [Hero] — [approach]
3. ...

## Do / Don't
### Do
- [Specific positive rules from the extraction]
### Don't
- [Client constraints + anti-patterns from research]

## Implementation Checklist
[Mechanical pass/fail. The build self-checks against this; stage 4 verifies it.]
- [ ] Hero headline ≤2 lines; subtext ≤20 words; ≤4 text elements; fits viewport at 1440×900
- [ ] H1 uses `clamp(3rem, 5vw, 5.5rem)` in a `max-w-5xl`/`max-w-6xl` container (never wraps to 6 lines)
- [ ] Uppercase-tracking eyebrows: count ≤ ceil(sectionCount / 3)
- [ ] No 3rd consecutive image+text split (zigzag cap)
- [ ] Bento: N items → N cells, `grid-flow-dense` (no dead cells)
- [ ] Every color on the page traces to the Color System above
- [ ] Every font on the page traces to Typography above
- [ ] No banned palette hex values present
- [ ] Zero em-dashes in copy
- [ ] Max one marquee
- [ ] No duplicate-intent CTAs
- [ ] No fake round numbers in copy (`99.99%`, `50%`) — use organic values (`47.2%`)
- [ ] `min-height: 100dvh`, never `100vh`
- [ ] GSAP ScrollTrigger uses `start: "top top"`, not `"top center"`
- [ ] Focus rings present; alt text present; meta tags present; 404 exists
- [ ] Every import resolves in package.json

## Refero References
[Screen/flow references — what pattern, why included]
```
