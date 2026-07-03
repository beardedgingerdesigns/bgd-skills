---
name: ask-the-board
description: >
  Deliberate with your board of advisors — independent analysis from each advisor's lens, then a unified synthesis.
  Use when the user wants to ask the board, run something by advisors, or get multi-perspective counsel on a decision.
  Depth: defaults to quick take. Say "deliberate deeply" or "deep" for full multi-round deliberation.
  Also triggers on /ask-the-board.
---

# Ask the Board

Pose a question to your board of advisors. Each advisor is an independent agent
backed by ingested public content from real creators. They deliberate and return
a unified perspective — not individual reports.

## Prerequisites

- **Advisor roster** must exist. Check these locations in order:
  1. `docs/wiki/advisors/roster.yaml` in the current project
  2. `/Users/justinlobaito/repos/claude-os/docs/wiki/advisors/roster.yaml` (global fallback)
- If no roster is found, stop: "No advisor roster found. Set up advisors in your project wiki or in claude-os."
- Each advisor in the roster must have a `persona.md` and at least some files in `knowledge/`.

## Detect deliberation depth

- **Quick take (default):** one round of independent analysis + synthesis.
- **Deep deliberation:** user includes "deliberate deeply", "deep", or "full deliberation" in the prompt. Multi-round: independent analysis → cross-examination → synthesis.

---

## Execution

### Phase 1 — Frame the Question

Before spawning advisor agents, frame the question:

1. **Restate the question clearly** — remove ambiguity, identify the decision type.
2. **Surface missing context** that would materially change the answer:
   - Decision type (one-way door vs. reversible)
   - Stakes (what's on the line)
   - Time horizon (this week vs. this quarter vs. this year)
   - Budget or revenue impact
   - Reversibility (how hard to undo)
   - Audience/customer (who this affects)
   - Constraints (time, money, dependencies, contracts)
   - What Justin is leaning toward
   - What failure would look like
3. **Do not block on missing info.** Make reasonable assumptions and label them clearly so advisors can challenge the assumptions as part of their analysis.

**Prospect doc resolution (D-06):** Extract kebab-case identifiers from the question. For each candidate, check if `prospects/<candidate>.md` or `prospects/converted/<candidate>.md` exists. If found, read the prospect doc and include it in the advisor context files (Phase 2). This gives advisors the full dossier for pricing, positioning, and deal strategy advice.

Present the framing to Justin for a quick gut check before proceeding. If he corrects an assumption, update and continue.

### Phase 2 — Isolate (Independent Advisor Analysis)

Load the roster. For each advisor:

1. Read their `persona.md` for role framing.
2. Read all files in their `knowledge/` directory for reference material.
3. Spawn an agent with:
   - **System context:** the persona.md content + the following instructions:
     "You are an advisor on Justin's board. Your lens: [lens from roster].
     Use your knowledge base to inform your analysis.
     RULES:
     - Do not impersonate this person. Do not claim they would definitely say something.
     - Use their documented ideas, patterns, and public positions as a decision lens.
     - When your knowledge doesn't cover the topic, say so and reason from adjacent
       principles — label it as extrapolation.
     - Never fabricate quotes, anecdotes, or positions not grounded in the ingested content.
     Produce your independent analysis of the question. Focus on what your lens reveals
     that other perspectives might miss."
   - **Knowledge context:** all knowledge/*.md files concatenated
   - **The framed question** from Phase 1 with assumptions

Each advisor produces independent analysis. No cross-talk during this phase.

**Agent teams path (primary):**
Use ToolSearch to check if TeamCreate is available. If so:
1. TeamCreate with team_name: "board-deliberation"
2. Spawn each advisor as a team member via the Agent tool with name set to the advisor slug and team_name: "board-deliberation"
3. Collect their independent analyses

**Parallel agent fallback:**
If TeamCreate is not available, spawn each advisor using the Agent tool in parallel. Collect results.

### Phase 3 — Deliberate + Synthesize

**Quick take:**
Spawn a synthesis agent that receives all Phase 2 positions and produces the unified output. One pass.

**Deep deliberation:**
1. Send each advisor the other advisors' Phase 2 positions via SendMessage (agent teams) or spawn new agents with prior context (fallback).
2. Each advisor responds: where they agree, where they push back, what the other missed.
3. Synthesis agent receives the full exchange (Phase 2 positions + Phase 3 responses) and produces the unified output.

**Synthesis agent instructions:**
"You are synthesizing a board deliberation into a unified response. Do NOT attribute
positions to individual advisors by name. Write as a single coherent perspective that
weaves together the board's thinking. Surface genuine disagreements as tensions, not
as 'Advisor A vs. Advisor B.' Follow this output format exactly:"

### Output Format

The final response to Justin must follow this structure:

## Bottom Line
[One sentence — what to do]

## The Thinking
[Narrative synthesis — where the board aligned, where they diverged,
and how the tension resolved. Written as a unified perspective,
not individual advisor attributions.]

## Stress Test
- What could be wrong about this
- The hidden assumption
- The risk if Justin is overconfident

## How to Act
[Concrete next steps]

## Your Call
[If there's a genuine fork the board can't resolve, frame it as
a clear choice with trade-offs. Omit this section if consensus is clear.]

### Shutdown

After delivering the output:
- **Agent teams:** send shutdown_request to all team members.
- **Parallel agents:** agents return naturally, no cleanup needed.
