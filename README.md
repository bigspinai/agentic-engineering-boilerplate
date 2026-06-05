# agentic-engineering-boilerplate

A Next.js boilerplate where two AI agents review every PR before you do. The base is a vanilla `create-next-app` scaffold; the differentiator is the **agentic engineering layer** — two independent reviewer workflows, a `.agents/skills/` directory that teaches them your team's conventions, and a `/pr-loop` writer-agent that closes the loop by iterating against reviewer feedback. v1 is intentionally minimal: the agentic layer is the point, not the feature count. The full thesis lives in the blog post (link TBD).

## What you get

- Modern Next.js 16+ baseline — TypeScript (strict), Tailwind 4, App Router, Turbopack, ESLint flat config, `src/` layout, `@/*` import alias
- Two AI PR reviewers running on every PR — **`pr-review-codex`** (via `openai/codex-action`) and **`pr-review-claude`** (via `anthropics/claude-code-action`)
- **`/pr-loop`** writer-agent that polls reviewer signal after each push and iterates fixes until the PR is clean
- **`.agents/skills/`** conditional-context system — reviewers load only the skills whose globs match the files changed in a PR
- **Vercel's official agent skills** pre-installed under `.agents/skills/vercel/` (`react-best-practices`, `composition-patterns`, `web-design-guidelines`) — MIT, vendored with attribution
- **Supply chain guard** — `pnpm-workspace.yaml` enforces a 1-week release-age quarantine and a `no-downgrade` trust policy
- pnpm-based; Node 20.9+ and pnpm 10.16+ required

## Setup

```bash
pnpm install
```

Then, in your GitHub repo settings, add two secrets:

| Secret | Used by |
|---|---|
| `OPENAI_API_KEY` | `pr-review-codex` workflow |
| `CLAUDE_CODE_OAUTH_TOKEN` | `pr-review-claude` workflow |

Push a branch and open a PR — both reviewers fire automatically and post inline comments plus a structured summary.

Then make it yours:

1. Edit `.agents/skills/example-restrictions/SKILL.md` to list your team's hard "never do this" rules.
2. Edit `.agents/skills/example-code-standards/SKILL.md` to encode your style guide and architectural patterns.
3. From a local Claude Code session, after pushing to a PR, run `/pr-loop` to close the loop — the writer-agent reads the reviewers' JSON output and iterates fixes until everything is clean.

## How it works

Every PR triggers both workflows in parallel:

- `.github/workflows/pr-review-codex.yml`
- `.github/workflows/pr-review-claude.yml`

Each workflow:

1. Detects the files changed in the PR.
2. Loads only the skills whose path globs match those files (e.g., a `src/components/` change loads the React + composition + design skills; a `src/app/api/` change loads the restrictions skill).
3. Runs the reviewer with the loaded skills as context.
4. Posts inline review comments and a structured summary (Strengths / Issues / Suggestions, plus a JSON block the writer-agent can parse).

The **loop closes** when you run `/pr-loop` locally: the writer-agent reads the reviewers' structured output, applies fixes, verifies locally, pushes, and waits for the next round. Two reviewer families (different model providers) means two independent loops — disagreement between them is itself a signal.

For the full conceptual map see [`AGENTS.md`](./AGENTS.md). For the skills directory layout and how to add new skills, see [`.agents/skills/README.md`](./.agents/skills/README.md).

## Customize for your team

The two `example-*` skills are templates — they ship with generic placeholder rules so the workflow runs out of the box, but the value comes from rewriting them to match what your team actually cares about. A reviewer without your conventions just gives generic LLM feedback. A reviewer with `example-restrictions/SKILL.md` rewritten to say "all API routes under `src/app/api/admin/` must call `requireAdmin()`" flags a missing call by name.

When you add a new skill folder, add its glob to both reviewer workflows so they know when to load it.

The vendored Vercel skills under `.agents/skills/vercel/` are frozen-in-repo (zero-setup on fresh clone). To swap to a live-update path, delete them and run `npx skills add vercel-labs/agent-skills` instead. Trade-off documented in `.agents/skills/README.md`.

## Supply chain guard

`pnpm-workspace.yaml` sets `minimumReleaseAge: 10080` (one week) and `trustPolicy: no-downgrade`. The one-week quarantine catches the class of npm supply-chain attack where a malicious version is published, detected by the community, and pulled within 24–48 hours — the delay lets that detection happen before the package lands in your `node_modules`.

These three fields — `minimumReleaseAge`, `trustPolicy`, `minimumReleaseAgeExclude` — are security-critical. **Do not edit them, and do not add packages to the exclude list, without explicit human approval.** If `pnpm install` blocks because a release is too new, revert the version bump and wait it out.

**Exception — critical CVEs:** if a fix is <7 days old but addresses an actively exploited or CVSS ≥9 vulnerability, that's the one case to consider adding to `minimumReleaseAgeExclude`. Present the CVE, publish date, and risk-of-waiting first; never make this call autonomously.
