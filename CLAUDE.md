# agentic-engineering-boilerplate

A Next.js boilerplate with the **agentic engineering** layer baked in: two AI PR reviewers, a `.agents/skills/` directory that encodes team conventions, and a `/pr-loop` writer-agent that closes the loop against reviewer feedback. v1 is intentionally minimal — the agentic layer is the differentiator, not the feature count.

## Source

This is the source for the (eventually) public repo `bigspinai/agentic-engineering-boilerplate`. It is auto-synced from `public-repos/agentic-engineering-boilerplate/` in the `bigspinai/bigspin-app` monorepo.

**The sync workflow has not been created yet.** v1 is internal-only. The public mirror at `bigspinai/agentic-engineering-boilerplate` and the corresponding `.github/workflows/sync-agentic-boilerplate.yml` will be created after team approval for the public launch.

## Development

To iterate locally, consumer projects can reference this boilerplate by a local path during development:

```bash
# Copy or symlink the boilerplate into a sibling directory and develop against it
cp -R public-repos/agentic-engineering-boilerplate ../my-test-project
```

This avoids needing to push to the public repo during development. The two reviewer workflows under `.github/workflows/` can also be copied into any existing Next.js project to add the agentic layer incrementally.

See [`AGENTS.md`](./AGENTS.md) for the agentic-engineering principles and Next.js conventions.
