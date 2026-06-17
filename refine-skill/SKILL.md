---
name: refine-skill
description: >
  Critically review a skill for verbosity, over-specification, and missed elegance,
  then rewrite it. Applies the grill-me standard: describe behavior and non-obvious
  constraints, trust the model for everything else. Use when user says "refine this
  skill", "make this skill tighter", "this skill is too verbose", "simplify this skill",
  "review skill quality", or when evaluating whether a skill is over-engineered.
  Also triggers on /refine-skill.
---

Read the skill. Then apply this lens:

**The only things worth writing in a skill are things the model wouldn't do on its own.** Everything else is noise that costs tokens and dilutes the signal.

## The cut

For every line, ask: if I deleted this, would the model do something worse? Three buckets:

1. **Behavior the model already knows** — delete. Installing packages, reading files, making commits, structuring markdown, writing frontmatter. The model knows. Don't teach it to code.

2. **Implementation choices the model would make anyway** — delete. Exact config keys, shell commands, boilerplate templates, numbered step-by-step procedures for obvious workflows. If a competent developer could derive it, the model can too.

3. **Non-obvious constraints and judgment calls** — keep. Immutability rules, authority hierarchies, when to flag vs auto-fix, surprising behaviors ("this is the one mode that skips X"), things that contradict what the model would default to.

## The test

Compare against this standard: [grill-me](~/.claude/skills/grill-me/SKILL.md) — 4 sentences that drive an entire interview process. It works because it describes *what to do* and *the few rules that matter*, then trusts the model.

Not every skill can be 4 sentences. But every skill can answer: what's my grill-me version? Start there, then add back only what's genuinely non-obvious.

## How to apply

1. Read the skill end-to-end
2. Identify the repeated patterns — anything said more than once belongs in a shared section or not at all
3. For each mode/section, extract the 1-3 non-obvious constraints. Everything else is procedure the model can derive.
4. Rewrite: behavioral description + non-obvious constraints only
5. Present the before/after line count and what was cut (categories, not line-by-line)
6. Ask the user to review before writing

## Smell tests

- **Step 1, Step 2, Step 3...** — almost always over-specified. Describe the goal and constraints instead.
- **Exact shell commands** — the model knows how to install things and run builds.
- **"Emit receipt" / "Commit" repeated per mode** — say it once.
- **Config templates inlined** — put them in an assets file or trust the model.
- **Numbered lists of sections a page should have** — you're dictating one project's layout as universal truth.
- **Prerequisites sections** — if the prerequisite is "the thing this mode operates on exists," that's obvious.

## What to preserve

- Decision frameworks (when to do X vs Y)
- Authority and trust hierarchies
- Surprising behaviors that contradict model defaults
- Anti-patterns (what NOT to do — models learn well from these)
- The one sentence that makes each mode distinct from the others
