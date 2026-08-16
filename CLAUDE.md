# Plump

Digital version of the card game **Plump**. Runs in the browser on phones and
tablets, served as a static site from GitHub Pages.

> **Status: greenfield.** As of this file's creation the repository is empty.
> The toolchain below is the intended setup, not the observed one. Once real
> code exists, correct anything here that drifted — the commands are the part
> that matters most, since agents run them verbatim.

## Platform constraints

These shape most technical decisions, so check a change against them before
reaching for a dependency:

- **Static only.** GitHub Pages serves files. No server, no backend process, no
  server-side secrets. Anything resembling a "server" has to run in the browser
  or in another player's browser.
- **Touch first.** Phones and tablets are the primary target, not desktop.
  Tap targets, viewport handling, and portrait/landscape both matter.
- **Offline-tolerant.** Mobile connections drop. Don't assume a live network
  round trip mid-hand.

## Commands

| Task | Command |
| --- | --- |
| Install | `npm ci` |
| Dev server | `npm run dev` |
| Unit tests | `npm test` |
| Unit tests (watch) | `npm run test:watch` |
| Coverage | `npm run test:coverage` |
| Browser / E2E tests | `npm run test:e2e` |
| Typecheck | `npm run typecheck` |
| Lint | `npm run lint` |
| Production build | `npm run build` |

Run `npm run typecheck && npm test` before committing. That pair catches most
regressions cheaply; the E2E suite is slower and can be left to CI.

## Stack

- **Vite + TypeScript** — builds to plain static assets, which is what Pages
  needs.
- **Vitest** — unit tests, sharing Vite's config and transform pipeline.
- **Playwright** — browser tests driven through mobile device emulation.
- **GitHub Actions** — builds and deploys to Pages on push to `main`.

### GitHub Pages base path

This is a project site, so it is served from `https://timpan8.github.io/Plump/`,
not from a domain root. Vite must be configured with `base: '/Plump/'` or every
asset URL breaks in production while working fine in dev. Use relative asset
paths and router basenames accordingly. A build that works locally and 404s on
Pages is almost always this.

## Architecture rule that matters most

**Keep game rules pure and separate from the DOM.**

Split the code so that rule logic — dealing, bidding, trick resolution, scoring,
turn order, legal-move validation — lives in plain TypeScript functions with no
DOM, no timers, and no framework imports. The UI layer reads that state and
renders it.

The reason is testability. Pure rule functions are exhaustively testable in
milliseconds, and the rules are where the real bugs live in a card game. If
scoring logic is tangled into a click handler, it can only be tested by driving
a browser, which is slow enough that the edge cases quietly stop being covered.

Practical consequences:

- Rule functions take state and return new state. No mutation of shared objects.
- **Randomness is injected, never ambient.** Shuffling takes a seeded RNG passed
  in as an argument. A test that cannot fix the deal cannot test a hand.
- The same applies to time and player input — pass them in.

## Testing expectations

- **Rule logic: exhaustive.** Every bidding path, trick-winner comparison, and
  scoring branch. This is the layer worth near-total coverage.
- **Edge cases deserve named tests**, not just happy paths: bidding zero,
  everyone missing their bid, following suit when void, ties, and the final-hand
  scoring transition.
- **UI: representative, not exhaustive.** A handful of Playwright flows on a
  mobile viewport — play a full hand, rotate the device, resume after reload.
- Prefer table-driven tests for rule branches; they stay readable as the rule
  count grows.

## Domain notes

Fill in the exact house rules as they get implemented — **variants differ
between families and regions, so the rules encoded here are the spec**, and
disagreements about intended behavior should be settled in this section rather
than in test fixtures:

- Bidding: _to document_
- Trick-taking and trump handling: _to document_
- Scoring, and what exactly earns a "plump": _to document_
- Number of players supported, and hand-size progression across rounds:
  _to document_

## Conventions

- Language of code, comments, identifiers, and commit messages: **English.**
  UI-facing strings are Swedish — keep them in a separate strings module rather
  than inline, so the two never get confused.
- Don't commit build output; `dist/` is deployed by CI, not from a developer
  machine.
