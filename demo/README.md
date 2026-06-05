# Demo — recording the boilerplate video

This directory exists for **tutorial / video walkthrough purposes only**. It is
not part of the agentic loop and not load-bearing on anything the boilerplate
ships. If you're adapting this template to your own project, delete the entire
`demo/` directory — nothing imports from it.

It contains the assets we use to record the "two AI agents read every PR"
walkthrough: an intentionally-broken Next.js API route that both reviewers will
catch, plus a checklist for the recording session.

## How to use this for the video

The demo flow is:

1. Clone the boilerplate to a fresh location (or use a clean branch).
2. Drop the broken route in.
3. Open a PR.
4. Record the screen while both reviewers post their comments live.
5. Narrate the loop closing.

### Step 1 — Set up the demo branch

```bash
git checkout -b demo/broken-api-route
mkdir -p src/app/api/example
cp demo/example-broken-route/route.ts.txt src/app/api/example/route.ts
git add src/app/api/example/route.ts
git commit -m "add example API route"
git push -u origin demo/broken-api-route
```

The file is shipped as a `.txt` so Next.js' file-system routing doesn't pick it
up at build time in the source tree — the copy step above is what turns it into
a real route on the demo branch.

### Step 2 — Open the PR

Suggested title and body:

- **Branch name**: `demo/broken-api-route`
- **PR title**: `Add /api/example user lookup route`
- **PR body**: keep it short and innocuous, e.g. *"Adds a quick `/api/example`
  endpoint to look up a user by id. Will wire up the UI in a follow-up."* —
  the deliberately mundane framing makes the reviewer comments land harder
  on camera.

```bash
gh pr create \
  --title "Add /api/example user lookup route" \
  --body "Adds a quick /api/example endpoint to look up a user by id. Will wire up the UI in a follow-up."
```

### Step 3 — What the reviewers should catch

Both `pr-review-codex` and `pr-review-claude` should flag at least the
following (and these are the points worth pausing on while narrating):

1. **No auth check** — the route is fully open.
2. **No input validation** — `userId` is read straight off the query string and
   used in a SQL query.
3. **SQL injection shape** — string concatenation into a `db.query(...)` call.
4. **Sensitive data leak** — the response returns `api_key` and
   `password_hash` columns plus `process.env.INTERNAL_API_KEY`.
5. **Logging untrusted input** — `console.log('POST body:', body)` will write
   whatever the caller sent into logs.
6. **Silently swallowed errors** — the `try/catch` returns `{ ok: true }` even
   when the insert fails.
7. **No tests** — there is no co-located test file for the new route.

The two reviewers will phrase these differently. That's part of the point —
disagreement and overlap between independent reviewer families is itself a
signal worth talking about on camera.

### Step 4 — Talking points while waiting

The bots take ~2–5 minutes to post. Useful things to narrate during the wait:

- **What the reviewers are doing right now.** Open the Actions tab and show the
  changed-files detection step, then the skill-loading step. Make the
  conditional context visible: this PR touches `src/app/api/`, so the
  restrictions skill loads; it doesn't touch `src/components/`, so the
  composition skill doesn't.
- **Why two reviewers, not one.** Different model families catch different
  categories of issue. Two independent loops are stronger than one.
- **What the JSON output is for.** The reviewers don't just leave inline
  comments — they emit a structured JSON block that the `/pr-loop`
  writer-agent reads on the next push. That's the loop closing.

### Step 5 — Close the loop on camera

After the comments land, from a local Claude Code session in the cloned repo:

```
/pr-loop
```

Narrate as the writer-agent reads the reviewer JSON, applies fixes (auth
check, input validation, redacted response, tests), commits, pushes, and
waits for the next round. Cut to the green checks. That's the loop.

## Cleanup after recording

```bash
git checkout main
git branch -D demo/broken-api-route
git push origin --delete demo/broken-api-route
```

Leave `demo/` itself in the repo so the next person recording the video has
the same starting point.

## When you adopt this boilerplate

Delete `demo/` entirely:

```bash
rm -rf demo/
```

Nothing in the agentic loop depends on it.
