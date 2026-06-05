<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Agentic engineering principles

This repo is the reference for the *agentic engineering* series. The
differentiator over a vanilla Next.js scaffold isn't the framework choices
— it's the **feedback-loop discipline** baked into how the codebase reviews
itself: two AI reviewers running on every PR, a `.agents/skills/` directory
that teaches those reviewers what your team cares about, and a writer-agent
(`/pr-loop`) that closes the loop by iterating against the reviewer signal
until the PR is clean. Read the post for the full thesis:
[Blog post — Post 1 of the agentic engineering series](TBD).

## The two-reviewer loop

Every PR opened against this repo is read by two independent reviewers
before a human looks at it:

- **`pr-review-codex`** — driven by `openai/codex-action`. Runs on the
  PR's diff, loads the skills that match the changed files, posts inline
  comments and a structured JSON summary.
- **`pr-review-claude`** — driven by `anthropics/claude-code-action`. Same
  shape, different model family. Two independent loops > one loop;
  disagreement is itself a signal.

Both workflows live under `.github/workflows/`. They share the same
"changed files → skill globs → load skills → review" structure. When the
two reviewers disagree, the author arbitrates, and that arbitration feeds
back into the skill files as a learning signal.

## The `.agents/skills/` directory

The reviewers don't review against generic LLM intuition — they review
against your team's conventions, encoded as Markdown files with
frontmatter. Each skill describes *when* it applies (in its
`description`); the workflows load only the skills whose globs match the
files changed in the PR.

Three classes of skill live here:

- **`vercel/`** — community baseline from
  [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills)
  (MIT, vendored with attribution). Covers React, composition, and web
  design guidelines that apply to almost any Next.js project.
- **`example-restrictions/`** and **`example-code-standards/`** — templates
  for the team-encoded layer. Edit these to spell out *your* team's hard
  rules and house style; the reviewers will then enforce them.
- **`pr-loop/`** — the writer-agent's loop-closing skill. After pushing
  to a PR, run `/pr-loop` from a Claude Code session and it will poll the
  reviewer signal and iterate fixes until the PR is clean.

When you add a new skill, also add its glob to both reviewer workflows so
they know to load it.

## Supply chain guard — do not modify

The repo's `pnpm-workspace.yaml` enforces a one-week quarantine on freshly
published npm packages plus a `no-downgrade` trust policy. The three
fields **`minimumReleaseAge`, `trustPolicy`, and `minimumReleaseAgeExclude`**
are a hard line: do **not** edit them, add packages to the exclude list,
or otherwise bypass the guard without explicit human approval. If `pnpm
install` blocks because a release is too new, revert the version bump and
wait it out. The guard isn't decoration — it's the layer of defense that
catches the Shai-Hulud-style malware that gets pulled from the registry
inside 24-48 hours.
