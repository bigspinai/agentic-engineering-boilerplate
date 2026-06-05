---
name: example-restrictions
description: >
  Things AI agents must NEVER do in this Next.js codebase. Apply to EVERY change
  — security and architectural guarantees that don't depend on what files were
  touched. The reviewer should flag any change that violates one of these rules.
metadata:
  type: feedback
---

# Restrictions — Never Do These

This skill encodes the team's hard "no" list. The rules below apply to every
change, regardless of which files were touched. The reviewers should flag a
violation as a blocking issue, not a suggestion.

Edit this file to encode your team's actual restrictions. The list below is a
generic starting point for a Next.js app.

## Hard rules

1. **No secrets in the repo.** Never commit API keys, OAuth tokens, database
   URLs with credentials, service-account JSON, `.env` contents, or anything
   that looks like a secret. If a value belongs in `.env.local`, it belongs in
   GitHub Actions secrets — not in source. Flag any change that adds a string
   matching common secret patterns (`sk_`, `pk_`, `Bearer `, AWS access keys,
   long base64 blobs).

2. **No unauthenticated API routes.** Every handler under `app/api/` must
   either (a) verify the caller's identity (session cookie, JWT, API key) or
   (b) have an explicit comment on the route saying it is intentionally public
   *and* what that public surface returns. The reviewer should flag any new
   `route.ts` lacking either an auth check or that comment.

3. **Don't skip type checks.** Don't add `// @ts-ignore`, `// @ts-expect-error`
   without a comment explaining why, or `any` as a shortcut to make
   `tsc --noEmit` pass. If the types are wrong, fix the types or the code.

4. **Don't bypass the supply chain guard.** `pnpm-workspace.yaml` enforces a
   one-week quarantine on freshly published packages via `minimumReleaseAge`,
   plus a `trustPolicy: no-downgrade` check. Do not modify those three fields
   (`minimumReleaseAge`, `trustPolicy`, `minimumReleaseAgeExclude`) or add
   packages to the exclude list without explicit human approval. If a fresh
   release breaks your install, revert and wait — don't disable the guard.

5. **Don't write tests against mocks of your own code.** A test that mocks the
   function it is testing proves nothing. Mock external boundaries (network,
   database, filesystem, time); never mock the unit under test or the
   collaborators you actually want to exercise. Flag any new test file whose
   mocks shadow internal modules.

## How the reviewer should use this

Treat every rule above as a blocking issue when violated. If a change is
ambiguous, ask in a comment rather than approve. Suggestions for *better*
patterns belong in the other skills; this skill is the "never" list.
