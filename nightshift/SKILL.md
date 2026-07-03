---
name: nightshift
description: AFK agent loop — pulls GitHub issues labeled ready-for-agent, implements each in an isolated worktree, commits sequentially, closes on success, labels needs-human on failure. The night shift half of the day/night shift workflow. Use when Justin says "/nightshift", "run the night shift", "pick up the issues", "AFK loop", or "let the agent take over".
---

# /nightshift

Pull issues from GitHub, implement them AFK, report back to AIOS.

## Prerequisites

- `gh` CLI authenticated
- At least one open issue labeled `ready-for-agent` in the current repo
- Tests runnable (the agent needs a feedback loop)

## Process

### 0. Pre-flight

```bash
# Verify gh is authenticated and issues exist
gh auth status
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner')
ISSUES=$(gh issue list --label "ready-for-agent" --state open --json number,title,body,labels)
```

If no issues found, report "Queue empty — nothing to do" and stop.

Read `CONTEXT.md` if it exists (domain glossary for agent context).
Detect the project slug from the repo directory name (for AIOS reporting).

Detect test command — **hard fail if none found:**

```bash
if [ -f "package.json" ] && grep -q '"test"' package.json; then
  TEST_CMD="npm test"
elif [ -f "Makefile" ] && grep -q '^test:' Makefile; then
  TEST_CMD="make test"
elif [ -f "pytest.ini" ] || [ -f "pyproject.toml" ]; then
  TEST_CMD="pytest"
else
  echo "No test command detected. Nightshift requires a runnable test suite. Aborting."
  exit 1
fi
```

### 1. Resolve dependency order

Parse each issue body for a `Blocked by` section. Extract referenced issue numbers (e.g., `#12`, `#13`).

**Dependency resolution happens at the TOP of each loop iteration, not once at the start.** After completing issue #9, re-check which issues are now unblocked (because #9 is closed) and add them to the queue. The queue is dynamic — it grows as blockers get resolved.

For every referenced blocker, check its actual state:

```bash
gh issue view $BLOCKER_NUMBER --json state -q '.state'
```

- An issue is **ready** if all its blockers are `CLOSED` (including issues closed earlier in this run)
- An issue is **blocked** if any blocker is `OPEN` (whether in this batch or not)

On each iteration:
1. Re-evaluate all remaining issues (not yet completed or failed in this run)
2. Pick the next READY issue (lowest number)
3. If no issues are READY and blocked issues remain, log "all remaining issues are blocked — stopping" and exit the loop
4. If no issues remain at all, exit the loop

Sort READY by issue number ascending. If circular dependencies exist among READY issues, warn and process in creation order.

### 2. Create the worktree

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
RUN_ID="$(date +%Y-%m-%d-%H%M)"
BRANCH="nightshift/$RUN_ID"
BASE_SHA=$(git rev-parse HEAD)
WORKTREE_PATH="$REPO_ROOT/.worktrees/$BRANCH"
RUN_LOG="$REPO_ROOT/nightshift-runs/$RUN_ID.md"
LAST_GOOD_SHA="$BASE_SHA"

# Guard: abort if branch or worktree path already exists
if git rev-parse --verify "$BRANCH" >/dev/null 2>&1 || [ -e "$WORKTREE_PATH" ]; then
  echo "Branch or worktree $BRANCH already exists — another nightshift may be running. Aborting."
  exit 1
fi

git worktree add -b "$BRANCH" "$WORKTREE_PATH" HEAD

# Initialize per-run log
mkdir -p "$REPO_ROOT/nightshift-runs"
cat > "$RUN_LOG" << EOF
# Nightshift Run: $RUN_ID
Base SHA: $BASE_SHA
Branch: $BRANCH
Test command: $TEST_CMD
EOF

# Initialize progress file — cross-issue memory for this run
cat > "$WORKTREE_PATH/progress.md" << EOF
# Nightshift Progress — $RUN_ID

Append-only notes for the next issue in this run.
Write what the next person needs to know, not what you did.
EOF

# Initialize fix_plan — discovered work that isn't the current task
cat > "$WORKTREE_PATH/fix_plan.md" << EOF
# Fix Plan — $RUN_ID

Discovered issues that need attention but are NOT the current task.
Append here instead of scope-creeping into unrelated fixes.
Format: one item per entry with a title and 1-2 lines of context.
EOF
```

All issues commit sequentially to this one branch. Later issues see earlier work. The `progress.md` file accumulates learnings across issues — each agent session reads it for context and appends after completing its issue.

### 3. Signal AIOS

Write a lightweight state entry so `/brief` and morning sessions know a nightshift is running (or ran and died):

```bash
STATE_FILE="$HOME/repos/claude-os/state/${PROJECT_SLUG}.md"
# Append or update a nightshift section
cat >> "$STATE_FILE" << EOF

## nightshift active
Run: $RUN_ID
Branch: $BRANCH
Issues: $(echo "${READY_ISSUES[@]}" | tr ' ' ', ')
Started: $(date -u +%Y-%m-%dT%H:%M:%SZ)
EOF
```

`/wrap` at the end overwrites this with the full picture.

### 4. Loop: implement each issue

For each issue in dependency order:

**a. Health check**

Before starting a new issue, verify the worktree is in a clean state. Run the test suite cold:

```bash
cd "$WORKTREE_PATH" && $TEST_CMD
```

- Tests pass → proceed to step b.
- Tests fail → the previous issue left breakage. Attempt a one-shot fix: re-run the agent with "Tests are failing before any new work. Fix them." If tests still fail after one attempt, log to run file and abort the loop (don't build on a rotten foundation).

Skip this check for the first issue in the run (the worktree was just created from HEAD, which should already be green).

**b. Label in-progress**
```bash
gh issue edit $NUMBER --add-label "in-progress" --remove-label "ready-for-agent"
```

**c. Build the prompt**

Assemble the agent prompt:

```
You are implementing a single issue in an existing codebase.

## Project glossary
{contents of CONTEXT.md, if it exists}

## Sprint progress so far
{contents of progress.md}

## Issue #{number}: {title}
{issue body}

## Instructions
- Read CLAUDE.md for project conventions
- Implement the issue as described
- Run tests after implementation — they must pass
- Make atomic commits with clear messages referencing #{number}
- Do NOT modify files outside the scope of this issue
- After completing the issue, APPEND to progress.md: what would help the next issue? (patterns found, gotchas, files created, architectural decisions). Not a status update — context for the next agent.
- If you discover something broken or wrong that is NOT part of this issue, APPEND it to fix_plan.md instead of fixing it. Do not scope-creep. One item per entry: title + 1-2 lines of context.
- If you get stuck after multiple attempts, explain what's blocking you and stop
```

**d. Run the agent (up to 3 attempts)**

```bash
(cd "$WORKTREE_PATH" && claude --print \
  --permission-mode bypassPermissions \
  --verbose \
  "$PROMPT")
```

After the agent finishes, run tests in the worktree:

```bash
cd "$WORKTREE_PATH" && $TEST_CMD
```

- Tests pass AND `HEAD` differs from `$LAST_GOOD_SHA` → **success**. Move to step e.
- Tests pass but `HEAD` unchanged (agent didn't commit) → retryable attempt. Re-run with: "Tests passed but no commit was created. Stage and commit the scoped changes referencing #{number}. This is attempt {N}/3."
- Tests fail, attempt < 3 → re-run the agent with: "Tests failed. Here's the output: {test output}. Fix the failing tests. This is attempt {N}/3." Retries are incremental — keep changes between attempts.
- Tests fail, attempt = 3 → **failure**. Move to step f.

**e. On success**

Record the checkpoint SHA. Do NOT close the issue yet — defer until after push (step 5):

```bash
LAST_GOOD_SHA=$(git -C "$WORKTREE_PATH" rev-parse HEAD)
gh issue edit $NUMBER --remove-label "in-progress"
COMPLETED_ISSUES+=("$NUMBER")
```

Log to run file: `echo "✓ #${NUMBER}: ${TITLE} — $(git -C "$WORKTREE_PATH" rev-parse --short HEAD)" >> "$RUN_LOG"`

**f. On failure**

Reset to the last known good state, then label:

```bash
# Log the failure diff before resetting
git -C "$WORKTREE_PATH" diff >> "$RUN_LOG"

# Reset to last successful commit
git -C "$WORKTREE_PATH" reset --hard "$LAST_GOOD_SHA"
git -C "$WORKTREE_PATH" clean -fd

gh issue edit $NUMBER --add-label "needs-human" --remove-label "in-progress"
gh issue comment $NUMBER --body "nightshift: failed after 3 attempts.

**Last error:**
{test output or agent's final message}

**What was tried:**
{summary of the 3 attempts}"
```

Log: `echo "✗ #${NUMBER}: ${TITLE} — needs-human" >> "$RUN_LOG"`

Continue to the next issue.

### 5. Cleanup worktree

After all issues are processed:

```bash
# If any commits were made, push the branch then close completed issues
if [ "$(git -C "$WORKTREE_PATH" rev-parse HEAD)" != "$BASE_SHA" ]; then
  git -C "$WORKTREE_PATH" push -u origin "$BRANCH"

  # Close issues directly after successful push — no subprocess needed.
  for NUMBER in "${COMPLETED_ISSUES[@]}"; do
    gh issue close "$NUMBER" --comment "Implemented by nightshift run $RUN_ID. Branch: $BRANCH"
  done
else
  echo "No commits made — skipping push and issue closure."
fi
```

### 6. File discovered work

Read `fix_plan.md`. If it has entries beyond the initial header, auto-file each as a new GitHub issue:

```bash
# For each entry in fix_plan.md, create a GitHub issue
gh issue create \
  --title "nightshift-discovered: {entry title}" \
  --body "{entry context}

---
_Auto-filed by nightshift run $RUN_ID. Discovered while working on #{source_issue}._" \
  --label "ready-for-agent"
```

Label them `ready-for-agent` so the next nightshift picks them up. If an entry is too vague to be actionable (no clear fix), label it `needs-human` instead.

Log filed issues to run file: `echo "📋 Filed #{NEW_NUMBER}: {title}" >> "$RUN_LOG"`

If `fix_plan.md` is empty (just the header), skip this step.

### 7. Synthesize sprint learnings

Before tearing down the worktree, read `progress.md` and evaluate whether anything is worth keeping. The bar: "would this change how a future session approaches this codebase?"

- **Yes** → synthesize into a single markdown file and stage to `{project-wiki}/raw/aios/nightshift-learnings-{RUN_ID}.md`. The wiki's ingest pass promotes what's worth keeping.
- **No** → discard. Most runs produce nothing worth staging. The agent found a hook, used it, moved on — that's doing the job, not a learning.

What qualifies: architectural patterns discovered, non-obvious gotchas, conventions that emerged across multiple issues, dependencies or coupling that surprised the agent. What doesn't: status updates, per-issue summaries, anything already in the commit messages.

### 8. Wrap the session

Run `/wrap` from the **main repo root** (not inside the worktree) so wiki/state writes land in the right place and the worktree stays clean for removal:

```bash
(cd "$REPO_ROOT" && BASE_SHA=$BASE_SHA claude --print \
  --permission-mode bypassPermissions \
  "/wrap")
```

Then clean up:
```bash
git worktree remove "$WORKTREE_PATH"
```

`/wrap` handles everything:
- **Code review** — detects code changes, runs `/ce-code-review` to catch bugs
- **Compound** — runs `/ce-compound` to record what the codebase learned
- **Wiki** — stages digest to the project wiki (or writes directly per project convention)
- **State** — updates `state/<slug>.md` in claude-os
- **Breadcrumb** — logs to `~/.claude/session-wrap.log`

After `/wrap` completes, append the QA todo to `~/repos/claude-os/todos/pending.md`:

```markdown
- [ ] QA nightshift run for {project} — {N} completed, {M} need human — branch: nightshift/{date}
```

### 9. Print summary

```
nightshift complete for {repo}

Completed: {N}
  ✓ #12: Add user avatar upload
  ✓ #13: Wire avatar to profile page
  ✓ #14: Add avatar fallback for no-image users

Failed: {M}
  ✗ #15: Resize avatar on upload — needs-human

Branch: nightshift/2026-06-28
Next: QA the branch, file bugs as new issues
```

## Labels

Uses Matt Pocock's triage vocabulary:

| Label | Meaning |
|---|---|
| `ready-for-agent` | Input: issue is ready for AFK implementation |
| `in-progress` | Agent is actively working on this issue |
| `needs-human` | Output: agent couldn't solve it in 3 attempts |

Ensure these labels exist in the repo. If missing, create them:
```bash
gh label create "ready-for-agent" --color "0E8A16" --description "Ready for AFK agent implementation" 2>/dev/null
gh label create "in-progress" --color "FBCA04" --description "Agent is working on this" 2>/dev/null
gh label create "needs-human" --color "D93F0B" --description "Agent couldn't solve — needs human review" 2>/dev/null
```

## What this skill does NOT do

- **No PRs.** Commits to a feature branch. You QA and merge.
- **No scheduling.** You kick it manually. Add scheduling later if the manual trigger becomes friction.
- **No multi-repo.** Runs against the current repo only.
- **`/wrap` runs automatically** after the loop — handles compound, wiki writes, state updates, breadcrumb.

## Part of the day/night shift workflow

```
Day shift:  /grill-with-docs → /to-prd → /to-issues
Night shift: /nightshift (this skill)
Next day:   QA branch (review + compound results waiting) → file bugs → /nightshift again if needed
```
