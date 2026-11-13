# Compound: record a solved problem as a lesson

Step 2 of `/wrap` runs this. It writes one lesson file to `{repo}/docs/solutions/` so the next session, any agent or tool, finds how a past problem was solved. The format matches the lessons already in these repos (written earlier by compound-engineering's `ce-compound`), so existing files and the workers that search them keep working.

## When to write one

Write a lesson only when the session solved a real problem or settled a non-obvious practice someone would otherwise rediscover the hard way: a bug with a diagnosed root cause, a setup or deploy trap, an integration quirk, a convention that took effort to work out. Routine feature work, copy changes and anything the code or its tests already make obvious get no lesson; say "no lesson: <why>" in the close report.

The two knowledge stores have different jobs. The wiki (`/wiki log`, step 4) records decisions, history and project state. `docs/solutions/` records how a specific problem was solved. Do not copy wiki content into a lesson or the other way round; link instead.

## Find an existing lesson first

Search `docs/solutions/` for the same problem (grep the module name, the error text, and likely tags). If a lesson already covers it, update that file: add what is new, correct what the session proved wrong, and bump `date`. Write a new file only when none fits. If the session shows an older lesson is now wrong, fix that lesson in the same pass and name it in the close report.

## Write it

Path: `docs/solutions/<category>/<short-kebab-name>.md`. Use the date resolved in step 1.

Pick `problem_type`, which sets the category folder:

| Track | problem_type -> folder |
|---|---|
| Bug | `build_error` build-errors, `test_failure` test-failures, `runtime_error` runtime-errors, `performance_issue` performance-issues, `database_issue` database-issues, `security_issue` security-issues, `ui_bug` ui-bugs, `integration_issue` integration-issues, `logic_error` logic-errors |
| Knowledge | `developer_experience` developer-experience, `workflow_issue` workflow-issues, `best_practice` best-practices, `documentation_gap` documentation-gaps, `architecture_pattern` architecture-patterns, `design_pattern` design-patterns, `tooling_decision` tooling-decisions, `convention` conventions |

If the repo already uses other folders (for example `pdf-pipeline/`), reuse the closest existing one rather than adding a near-duplicate.

Frontmatter, both tracks: `title`, `date`, `category`, `module` (area of the codebase), `problem_type`, `severity` (`critical`/`high`/`medium`/`low`), `tags` (lowercase, hyphenated). Bug track adds `symptoms` (1-5 observable symptoms), `root_cause`, `resolution_type` (short phrases). Knowledge track may add `applies_when`. In any list item, wrap the value in double quotes if it starts with `` ` `` `[` `*` `&` `!` `|` `>` `%` `@` `?` or contains `": "`, so strict YAML parsers accept it.

Body, bug track: `## Problem` (1-2 sentences and what it broke), `## Symptoms`, `## What Didn't Work`, `## Solution` (the fix, with the key snippet or command), `## Why This Works`, `## Prevention` (a test, guard or practice), `## Related` (wiki pages, PRs, other lessons).
Body, knowledge track: `## Context`, `## Guidance` (with a concrete example), `## Why This Matters`, `## When to Apply`, `## Related`.

Write what the session actually established: real error text, real commands, the PR or commit. No client contact details, secrets or customer data.

## Make sure agents can find lessons

If the repo's main instruction file (`AGENTS.md`, or `CLAUDE.md` when it is not just an `@AGENTS.md` pointer) never mentions `docs/solutions/`, do not edit it; put this suggested line in the close report for Justin to add:
`docs/solutions/` - solved problems and settled practices, one file each, with frontmatter (`module`, `tags`, `problem_type`); relevant when debugging or building in an area that has one.
