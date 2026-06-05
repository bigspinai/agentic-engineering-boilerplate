---
name: pr-loop
description: >
  Iteratively fix PR review comments from the Claude and Codex reviewer bots
  until all issues are resolved and CI passes. Invoke once after pushing to a
  PR; the agent handles the full loop internally and terminates when clean.
user-invocable: true
---

# PR Loop — Automated Review-Fix Cycle

## Purpose

Single-invocation skill that automates the PR feedback loop: push → wait for
CI + reviews → parse issues → fix → repeat. Terminates when all checks pass
and no actionable review issues remain, or after a max number of rounds.

## When to Use

Invoke `/pr-loop` after pushing to a PR when you want the agent to autonomously
fix CI failures and review-bot feedback until the PR is clean.

## Watched checks (parameterize for your repo)

This skill watches a fixed list of checks. The defaults below match the two
reviewer workflows that ship with this boilerplate plus a generic `CI` gate.
**Edit this list** if your repo uses different check names.

| Check name | Type | What failure means |
|------------|------|--------------------|
| `CI` | CI gate | Lint, type-check, unit tests, or build failed |
| `pr-review-claude` | Review bot | Claude reviewer posted blocking issues |
| `pr-review-codex` | Review bot | Codex reviewer posted blocking issues |

Adjust the JSON-query selectors below to match your check names exactly as
they appear in `gh pr checks`.

## How It Works

The agent runs an internal loop (max **10 rounds**). Each round:

1. Ensure changes are pushed.
2. Poll `gh pr checks` until all relevant checks complete.
3. Read CI logs (if failed) and review-bot comments.
4. Parse actionable issues (skip nice-to-have suggestions).
5. Pull latest main and resolve any merge conflicts.
6. Fix issues, verify locally, commit, push.
7. Repeat — or exit if clean.

---

## Step 1: Detect the PR

```bash
gh pr view --json number -q '.number'
```

If no PR exists for the current branch, stop with:
> "No PR found for the current branch. Push your branch and open a PR first,
> then re-run /pr-loop."

Store the PR number for all subsequent commands.

## Step 2: Push pending changes

If the working tree is dirty (unstaged or staged changes, or unpushed commits):

1. Stage changed files by name (never use `git add -A` or `git add .` — this
   can accidentally include sensitive files).
2. Commit with message: `fix: address PR review feedback (round N)` (or an
   appropriate message for the first push).
3. `git push`.

If the tree is clean and up-to-date with the remote, skip this step.

## Step 3: Poll checks

Wait for all relevant checks to complete by polling `gh pr checks`.

### Polling strategy

```bash
# Wait 2 minutes before first poll (nothing finishes before this)
sleep 120

# Then poll every 60s until all relevant checks complete (max 15 polls = ~17 min total)
```

On each poll:
```bash
gh pr checks $PR_NUMBER --json name,state,bucket \
  --jq '[.[] | select(
    .name == "CI" or
    .name == "pr-review-claude" or
    .name == "pr-review-codex"
  )]'
```

A check is "done" when `state` is `SUCCESS`, `FAILURE`, or `SKIPPED` (i.e.,
not `IN_PROGRESS` or `PENDING`). Wait until all present relevant checks have a
terminal state.

**Important**: Not all checks run on every PR. If a check does not appear in
the checks list at all, treat it as not applicable — do not wait for it.

If all relevant checks are still pending after 17 minutes, report the timeout
and stop.

Print status each poll:
```
Polling... (3m elapsed) CI: pending, pr-review-claude: completed, pr-review-codex: pending
```

## Step 4: Read results

Once all checks complete, gather results:

### CI failures

```bash
gh pr checks $PR_NUMBER --json name,state,bucket \
  --jq '[.[] | select(.name == "CI" and .bucket == "fail")]'
```

If CI failed, get the failing run logs:
```bash
# List runs for the PR's head branch to find the run ID
gh run list --branch $(git branch --show-current) --limit 5 --json databaseId,name,conclusion
# Then fetch failed logs
gh run view $RUN_ID --log-failed
```

### Review comments

Fetch **all** comments from the PR — bot reviews, human feedback, and inline
comments. Do not filter by user; human comments contain important context,
requests, and corrections that must be incorporated.

**Top-level comments** (bot reviews + human discussion):
```bash
gh api repos/$OWNER/$REPO/issues/$PR_NUMBER/comments \
  --jq '.[] | {user: .user.login, body: .body, created_at: .created_at}'
```

**Inline PR review comments** (code-level feedback from bots and humans):
```bash
gh api repos/$OWNER/$REPO/pulls/$PR_NUMBER/comments \
  --jq '.[] | {user: .user.login, body: .body, path: .path, line: .line, created_at: .created_at}'
```

**PR review bodies** (review-level summaries from bots and humans):
```bash
gh api repos/$OWNER/$REPO/pulls/$PR_NUMBER/reviews \
  --jq '.[] | {user: .user.login, body: .body, state: .state, submitted_at: .submitted_at}'
```

### Prioritizing comments

- **Human comments take priority over bot comments.** If a human requests a
  change, fix it — even if a bot didn't flag it.
- **Bot review comments from the latest round** — match the commit SHA in the
  comment body against the current HEAD to find the most recent review from
  each bot.
- **Human comments from any point in the PR** — read all of them. Earlier
  human comments may contain requirements, context, or style preferences that
  still apply.

## Step 5: Parse review comments

Both reviewer bots (`pr-review-claude` and `pr-review-codex`) post a structured
comment that includes a JSON summary plus a human-readable section. Look for:

- **Actionable issues**: items under `### Issues` (or `### Issues Found`) —
  these must be fixed.
- **Skip**: items under `### Suggestions` — fix only if they describe a clear
  bug.

Count the number of actionable issues and suggestions separately for
reporting.

## Step 6: Decide — fix, dispute, or done?

For each actionable item — whether from review bots or human commenters —
make a judgment call:

### 6a. Agree — fix it
If the issue is valid, fix it in Step 8.

### 6b. Disagree — dispute with evidence
If you believe a review issue is **incorrect** (e.g., the reviewer is
hallucinating, misreading the code, or recommending something that
contradicts actual behavior), **do not silently skip it**. Instead:

1. **Verify your position** — look up documentation, read the actual source
   code, run the command, or check the API to confirm the reviewer is wrong.
2. **Post a PR comment** with your reasoning and evidence so the review bots
   can take it into account on re-review:

```bash
gh pr comment $PR_NUMBER --body "$(cat <<'COMMENT'
### Disputing [bot name] issue: "[issue summary]"

**Claim**: [what the reviewer said]

**Evidence**: [paste actual documentation, command output, or code that proves the claim is wrong]

**Conclusion**: [why no change is needed]
COMMENT
)"
```

This creates a paper trail. Review bots read prior comments, so providing
evidence helps them avoid repeating the same hallucinated issue in subsequent
rounds.

#### Escalation to human reviewers

If the same issue was disputed with evidence in a **previous round** and the
reviewer raises it **again** in the current round despite the evidence,
escalate by posting a final summary comment and tagging the appropriate
GitHub team (e.g., `@your-org/maintainers`) for human arbitration. Continue
the loop — do not block on the disputed issue.

### 6c. All clear — exit

```
IF CI passed AND no actionable issues (or all disputed with evidence):
  -> Report success and EXIT the loop
```

## Step 7: Sync with main

Before fixing, pull the latest main and merge to avoid drift:

```bash
git pull origin main
```

If there are merge conflicts, resolve them before proceeding. After resolving,
commit the merge. If the merge is clean, git will auto-commit.

## Step 8: Fix, verify, push

1. **Fix** all identified issues (CI failures + review issues).
2. **Verify locally** — run the appropriate checks for your project:
   - Type-check: `pnpm run typecheck` (or the equivalent in your repo)
   - Lint: `pnpm run lint`
   - Tests: `pnpm test`
3. **Commit**: `fix: address PR review feedback (round N)`.
4. **Push**: `git push`.
5. **Go back to Step 3**.

---

## Safety Guards

1. **Max rounds: 10** — prevents infinite loops. If still failing after 10
   rounds, report what remains and stop.
2. **Adaptive polling** — polls actual check status via `gh pr checks`,
   adapts poll count to real CI duration (initial 2-min wait before first
   poll, then 60s intervals).
3. **Max poll time: ~17 min per round** — generous buffer over typical CI
   run times.
4. **No force push** — never rewrite history. Each round gets its own commit.
5. **Local verification** — run type-check + tests before each push to catch
   issues early.
6. **No amending** — every fix is a new commit.

## Output Format

Print clear status for each round:

```
=== Round 1/10 ===
Pushed: abc1234
Polling checks...
  2m: CI: pending, pr-review-claude: pending, pr-review-codex: pending
  4m: CI: pending, pr-review-claude: completed, pr-review-codex: completed
  12m: CI: completed
All checks complete.

Results:
  CI:                  PASS
  pr-review-claude:    1 issue, 2 suggestions
  pr-review-codex:     1 issue, 0 suggestions

Actionable issues (2):
  [claude] 1. config list missing --json guard
  [codex]  1. require() in ESM test file
Skipping 2 suggestions.

Fixing 2 issues...
<fix, verify, commit, push>

=== Round 2/10 ===
Pushed: def5678
Polling checks...
  ...
All checks complete.

Results:
  CI:                  PASS
  pr-review-claude:    0 issues
  pr-review-codex:     0 issues

=== Done ===
All checks green, no actionable issues. PR is clean after 2 rounds.
```

## Edge Cases

- **No review comments yet**: The polling step waits for review checks to
  complete. If a bot doesn't post (e.g., the workflow was skipped), its check
  won't appear in `gh pr checks` — skip it.
- **Bot comments from a previous commit**: Only parse the most recent comment
  from each bot that matches the current HEAD SHA.
- **CI passes but reviews have issues**: Fix review issues only.
- **CI fails but reviews are clean**: Fix CI issues only.
- **Flaky tests**: If the same test fails across 2 rounds with different
  errors, note it and ask the user.
