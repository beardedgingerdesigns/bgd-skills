# Agent Prompts — Add Board Member

## Extraction Agent Prompt

Adapt per source (fill in `{name}`, `{slug}`, `{url}`, `{source type}`, `{board-root}`):

```
You are extracting knowledge from a single source for an advisor knowledge base.

**Advisor:** {name}
**Source:** {url}
**Source type:** {YouTube video / blog post / podcast / etc.}

**Your job:** Fetch this content and extract a structured knowledge file.

Use the web fetch tool to retrieve the content at the URL above. If it's a YouTube video, fetch the page to get available transcript/description content.

Write a knowledge file in this exact format:

---
source: add-board-member
advisor: {slug}
captured: {today's date YYYY-MM-DD}
url: {url}
status: ingested
---

# {Topic Title — descriptive, based on the content's core subject}

**URL:** {url}
**Published:** {year or date if known}
**Type:** {content type}

## Key Frameworks and Patterns

{Extract the core frameworks, mental models, and structured thinking. Use ### subheadings for distinct frameworks. Include specific terminology the advisor uses.}

## Positions and Opinions

{Extract clear positions — what they advocate for, what they argue against, and why. Use bullet points.}

## Relevant Quotes

{3-5 direct quotes with context. Format: "quote" -- context: where/why they said it}

## How This Applies as a Decision Lens

{One paragraph: when advising on a business decision, how would this content inform the recommendation? What kinds of questions does this content help answer?}

**Guidelines:**
- Extract THEIR frameworks and positions, not generic advice
- Use their terminology and language
- Be specific — "charge $5K-15K/mo for retainers" not "charge appropriately"
- If the content is thin or low quality, say so in the output rather than padding
- Name the file: {slug for the topic in kebab-case}.md

Write the knowledge file to: {board-root}/{advisor-slug}/knowledge/{topic-slug}.md
```

For content addition mode, append:
```
**Already covered topics (avoid overlap):**
{list of existing knowledge file names and their main topics}
```

## Persona Synthesis Prompt

Adapt per advisor (fill in `{name}`, `{board-root}`, `{advisor-slug}`):

```
You are synthesizing an advisor persona from their extracted knowledge base.

**Advisor:** {name}
**Knowledge files:** Read all .md files in {board-root}/{advisor-slug}/knowledge/

Read every knowledge file. Then write persona.md in this exact format:

# {Name} — Advisor Lens

## Role on the Board
{One sentence — what perspective this advisor brings.}

## Lens
{2-3 sentences — how this advisor sees problems. What they optimize for. What they notice that others miss.}

## Voice Notes
- {Communication style markers}
- {Tone, patterns, verbal habits from their content}
- {How they structure arguments}

## Known Positions

### {Theme 1 — e.g., "Core Frameworks"}
- **{Framework/position name}.** {Concise description with specifics.}

### {Theme 2}
- **{Position name}.** {Description.}

{Continue for all major theme clusters. Group related positions. 4-6 theme sections typical.}

## Gaps

### Content not yet ingested
- {Specific content that would strengthen the knowledge base}
- {Topics they're known for that aren't yet captured}

### Known limitations of this knowledge base
- {Time period covered}
- {Content types missing — e.g., paid courses, live workshops}
- {Context about the advisor that should filter how advice is applied}

**Guidelines:**
- Known Positions must be grounded in the knowledge files — don't invent positions
- Gaps should be specific and actionable — "their content on X" not "more content"
- Voice Notes come from HOW they communicate in the source content
- Keep it under 80 lines total

Write to: {board-root}/{advisor-slug}/persona.md
```

For content addition mode: the agent reads existing persona.md first, then all knowledge files (old + new), and updates the persona — refreshing Known Positions, updating Gaps, adding new themes.
