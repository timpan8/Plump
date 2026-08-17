# Plump

**Scoreboard** for the card game Plump. The cards are played with a real deck at
the table; this app only tracks bids, tricks won, and scores across rounds. Runs
in the browser on phones and tablets, served as a static site from GitHub Pages.

> **Status: greenfield.** As of this file's creation the repository is empty.
> The toolchain below is the intended setup, not the observed one. Once real
> code exists, correct anything here that drifted — the commands are the part
> that matters most, since agents run them verbatim.

## Scope

This is a scoreboard, not a game engine. It does **not** deal cards, shuffle,
decide who wins a trick, or validate card play. It records what humans report
after each hand and does the arithmetic. Keep proposed features on that side of
the line unless the scope is deliberately changed.

## Platform constraints

These shape most technical decisions, so check a change against them before
reaching for a dependency:

- **Static only.** GitHub Pages serves files. No server, no backend process, no
  server-side secrets.
- **Touch first.** Phones and tablets are the primary target, not desktop. The
  device gets passed around a table, so tap targets and number entry matter more
  than keyboard shortcuts.
- **Survives interruption.** A game runs over a whole evening on a phone that
  will lock, background the tab, and get reloaded. Game state must persist
  locally and restore exactly. Losing a game in progress is the worst bug this
  app can have.

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
- **GitHub Actions** — builds and deploys to Pages on push to the default
  branch.

### GitHub Pages base path

This is a project site, so it is served from `https://timpan8.github.io/Plump/`,
not from a domain root. Vite must be configured with `base: '/Plump/'` or every
asset URL breaks in production while working fine in dev. A build that works
locally and 404s on Pages is almost always this.

## Architecture rule that matters most

**Keep scoring logic pure and separate from the DOM.**

Rule logic — score calculation, bid validation, round progression, running
totals, end-of-game detection — belongs in plain TypeScript functions with no
DOM access, no timers, and no framework imports. The UI reads that state and
renders it.

The reason is testability. Pure functions are exhaustively testable in
milliseconds, and the scoring rules are where the real bugs live. If score
calculation is tangled into a click handler it can only be tested by driving a
browser, which is slow enough that the edge cases quietly stop being covered.

Practical consequences:

- Rule functions take state and return new state. No mutation of shared objects.
- **Persistence and time are injected, never ambient.** Pass storage and clock
  in as arguments rather than reaching for `localStorage` or `Date.now()` inside
  rule code. Tests that cannot control those cannot test restore behavior.
- The stored game state is effectively a file format. Version it from the first
  commit, so a later rule change doesn't corrupt games saved by an older build.

## Testing expectations

- **Scoring: exhaustive.** Every branch of making, missing, and bidding zero.
  This layer is worth near-total coverage — it is small, pure, and the entire
  point of the app.
- **Edge cases deserve named tests**, not just happy paths: bid zero and take
  zero, bid zero and take one, everyone missing, the bid-sum restriction on the
  last bidder, hand-size progression at the turn, and final-round totals with
  ties.
- **Persistence: round-trip tested.** Save mid-game, restore, and assert the
  state is identical. Add a test per stored-format version once one exists.
- **UI: representative, not exhaustive.** A handful of Playwright flows on a
  mobile viewport — score a full round, reload mid-game and confirm nothing was
  lost, rotate the device.
- Prefer table-driven tests for scoring branches; they stay readable as the rule
  count grows.

## Domain notes

**Variants of Plump differ between families and regions, so the rules recorded
here are the spec.** Disagreements about intended behavior get settled in this
section, not in test fixtures. Fill each in as it is implemented:

- Points for making a bid exactly: _to document_
- What a missed bid scores, and what earns a "plump": _to document_
- Whether the bid total is forbidden from equalling the number of tricks, and
  who is constrained by it: _to document_
- Hand-size progression across rounds, and how many rounds a game runs:
  _to document_
- Player count supported: _to document_

## Conventions

- Language of code, comments, identifiers, and commit messages: **English.**
  UI-facing strings are Swedish — keep them in a separate strings module rather
  than inline, so the two never get confused.
- Don't commit build output; `dist/` is deployed by CI, not from a developer
  machine.
