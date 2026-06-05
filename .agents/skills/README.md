# `.agents/skills/`

This directory holds the **conditional context** the two AI PR reviewers (and the
writer-agent that closes the loop via `/pr-loop`) load when they evaluate a
change. A skill is just a folder with a `SKILL.md` file: frontmatter that
describes when the skill applies, and a body that spells out the conventions an
agent should follow.

## Why "conditional"?

Loading every skill into every review burns context and dilutes the signal.
Instead, the GitHub Actions workflows look at the **files changed in the PR**
and load only the skills whose path globs match. A change under `src/components/`
loads the React + composition + design skills. A change under `app/api/` loads
the API-safety skills. A typo fix in a `README.md` loads neither.

The mapping from changed-files-glob to skill lives in
`.github/workflows/pr-review-codex.yml` and `.github/workflows/pr-review-claude.yml`.
When you add a new skill, wire it into that mapping so the reviewers know to
pick it up.

## What's in here

```
.agents/skills/
├── vercel/                       # Community baseline (MIT, vendored from
│   ├── react-best-practices/     # vercel-labs/agent-skills). General-purpose
│   ├── composition-patterns/     # React/Next.js conventions every reviewer
│   └── web-design-guidelines/    # should know.
│
├── example-restrictions/         # Team-encoded templates. Edit these to teach
├── example-code-standards/       # the reviewers YOUR team's conventions.
│
└── pr-loop/                      # The writer-agent's loop-closing skill.
                                  # Invoke as `/pr-loop` from Claude Code after
                                  # pushing to a PR — it polls reviewer signal
                                  # and iterates until the PR is clean.
```

## How to add a skill

1. Create a new folder under `.agents/skills/your-skill-name/`.
2. Write a `SKILL.md` with frontmatter: `name`, `description` (the description
   is what the reviewer agents read to decide whether to load it — be
   specific about *when* it applies), and an optional `metadata.type`.
3. Add a path-glob entry in both reviewer workflows so the skill loads on the
   right changes.
4. Open a PR. The reviewers will load your new skill on any change matching
   the glob.

## Updating the vendored Vercel skills

The Vercel skills under `vercel/` were copied in via `curl` at a point in time
(see the attribution comment at the top of each `SKILL.md`). To resync to the
latest upstream version:

```bash
npx skills add vercel-labs/agent-skills
```

…then delete any skills you don't want. The trade-off: frozen-in-repo (current
default) works on a fresh clone with zero setup; the `skills add` path
auto-updates but adds a setup step. Pick whichever fits your team.
