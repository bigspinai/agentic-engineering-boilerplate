---
name: example-code-standards
description: >
  Code style and structure conventions for this Next.js codebase. Apply to all
  changes that touch source code (TypeScript, TSX, CSS, config). The reviewer
  should flag clear violations and suggest fixes; nits should be noted but not
  block.
metadata:
  type: feedback
---

# Code Standards

Generic starting point for a Next.js + TypeScript project. Edit this file to
encode the conventions your team actually enforces — the more specific you make
the rules, the more useful the reviewer's feedback becomes.

## TypeScript

- **Strict mode is on.** Don't widen the type to make an error go away. If
  `tsc --noEmit` is unhappy, fix the underlying type or the code.
- **No implicit `any`.** Every parameter, return value, and exported symbol has
  an explicit type. `unknown` is fine when the shape is genuinely unknown at
  the call site; narrow it before use.
- **Prefer `type` for unions and aliases; `interface` for object shapes that
  may be extended.** Don't mix the two for the same concept.

## Components

- **Functional components only.** No class components.
- **Props are explicit.** Declare a `Props` type or interface for every
  component, even single-prop ones. No `React.FC` — it adds an implicit
  `children` you usually don't want.
- **Server vs. client.** Default to server components. Add `'use client'`
  only when the component needs state, effects, or browser APIs.

## Imports

- **Use the `@/*` path alias** for anything under `src/` (configured in
  `tsconfig.json`). Relative imports (`../../foo`) are a smell beyond one level.
- **No barrel files** (`index.ts` that re-exports a directory). They wreck
  tree-shaking and slow type-checking. Import from the source file.

## Style

- **Prettier handles formatting.** Don't argue about spaces, semicolons, or
  quote style — let the formatter decide.
- **ESLint is the source of truth for lint.** The flat config in
  `eslint.config.mjs` extends `eslint-config-next`. Fix lint errors before
  pushing; don't add `// eslint-disable-next-line` without a comment that
  explains why.

## Commits and PRs

- **Conventional Commits.** `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`,
  etc. Optional scope in parentheses (`feat(auth): ...`). Body explains the
  *why*; the diff explains the *what*.
- **PRs should be small.** If your PR touches more than ~15 files, consider
  splitting it. Easier to review = faster to merge = tighter feedback loop.

## How the reviewer should use this

Treat clear violations of TypeScript strict / explicit-Props / `@/*` alias as
issues. Style preferences (e.g., naming, file structure) are suggestions
unless the team has decided otherwise — note them but don't block.
