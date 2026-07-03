---
name: refine-skill
description: >
  Refine a skill — cut verbosity, collapse duplication, surface missed elegance.
  Use when the user wants to tighten, simplify, or review a skill's quality.
  Also triggers on /refine-skill.
---

Read the skill. Then apply this lens:

**The only things worth writing in a skill are things the model wouldn't do on its own.** Everything else is noise that costs tokens and dilutes the signal.

## The cut

For every line, ask: if I deleted this, would the model do something worse? Three buckets:

1. **Behavior the model already knows** — delete. The model knows how to install packages, read files, run builds, structure markdown, write shell commands. Numbered step-by-step procedures for obvious workflows, exact config keys, boilerplate templates — if a competent developer could derive it, the model can too.

2. **Repeated meaning** — collapse. Anything said more than once belongs in one place or not at all. "Emit receipt" repeated per mode → say it once. Config templates inlined across sections → one assets file or trust the model.

3. **Non-obvious constraints and judgment calls** — keep. Decision frameworks (when to do X vs Y), authority hierarchies, when to flag vs auto-fix, surprising behaviors ("this is the one mode that skips X"), anti-patterns, things that contradict what the model would default to. This is what earns a skill's existence.

## The test

Compare against [grill-me](~/.claude/skills/grill-me/SKILL.md) — 4 sentences that drive an entire interview process. Not every skill can be 4 sentences, but every skill can answer: what's my grill-me version? Start there, then add back only what's genuinely non-obvious.

Read the skill end-to-end, extract the non-obvious constraints per section, rewrite as behavioral description + constraints only, present before/after line count and what was cut, ask the user to review before writing.
