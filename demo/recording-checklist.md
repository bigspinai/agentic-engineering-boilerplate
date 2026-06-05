# Recording checklist — before you hit record

Quick pre-flight so the take doesn't get blown by a missing secret or a
zoomed-out terminal. Run through this in order.

## Environment

- [ ] **GitHub secrets set** on the demo repo:
  - [ ] `OPENAI_API_KEY` (for `pr-review-codex`)
  - [ ] `CLAUDE_CODE_OAUTH_TOKEN` (for `pr-review-claude`)
- [ ] Confirm both reviewer workflows are enabled in the repo's Actions tab
- [ ] Default branch is `main` and the demo branch isn't already pushed
- [ ] Local clone is on `main` and up to date (`git pull`)
- [ ] `pnpm install` has been run at least once (don't catch a fresh install on camera)
- [ ] A Claude Code session is open in the repo (for the `/pr-loop` segment)
- [ ] `gh` CLI is authenticated (`gh auth status` returns green)

## Screen setup

- [ ] Screen resolution at **1920×1080** (1080p) or higher — anything lower and
      the inline PR comments are hard to read on playback
- [ ] Browser zoom set so a PR page shows the diff + comment in a single
      viewport (typically 100% on 1080p, 90% on 1440p)
- [ ] Terminal font **bumped one size up** from your default — viewers should
      be able to read commands without squinting
- [ ] Dark / light mode locked — pick one before the take so it doesn't flip
      mid-screen-share
- [ ] Notifications silenced: Slack, Mail, iMessage, calendar reminders, Linear,
      Discord. macOS: Focus → Do Not Disturb
- [ ] Browser tabs: close anything personal or NDA'd before screen-sharing

## Audio

- [ ] Mic level test (one short sentence, listen back)
- [ ] Room noise check (AC, fan, traffic) — record 5 sec of silence and listen
- [ ] Phone on silent and face-down

## On-camera (if doing talking head)

- [ ] Lighting front-facing, not from behind
- [ ] Camera at eye level
- [ ] Background isn't distracting (or use a clean Zoom/Meet style backdrop)

## Repo state

- [ ] Working tree clean (`git status` shows nothing)
- [ ] Demo branch `demo/broken-api-route` does **not** exist yet locally or on
      origin — start the take from a clean slate so viewers see the branch
      being created
- [ ] No lingering open PRs from previous takes — close or delete them so the
      Actions tab is uncluttered

## Have ready in another tab

- [ ] The `demo/README.md` file open (for talking-point reference)
- [ ] The repo's Actions tab (you'll cut here to show the workflow running)
- [ ] The `.github/workflows/pr-review-codex.yml` file (for the brief
      "look at how skills load conditionally" beat)

## After the take

- [ ] Delete the demo branch locally and on origin (see `demo/README.md`)
- [ ] Close the demo PR
- [ ] Reset any throwaway secrets if you used scratch keys
