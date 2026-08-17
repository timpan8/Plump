# Plump

**Scoreboard** for the card game Plump. The cards are played with a real deck at
the table; this app only tracks bids, tricks won, and scores across rounds. Runs
in the browser on phones and tablets, served as a static site from GitHub Pages.

> **Status: shipped, no toolchain.** The app is a single hand-written
> `index.html` with inline CSS and JS — no build step, no package manager, no
> dependencies. That is a deliberate choice, not an unfinished setup: the whole
> app is small enough to read in one file, and it stays deployable by copying
> files. Do not introduce npm, a bundler, or a framework without agreeing on it
> first; several sections below used to describe a Vite/TypeScript stack that
> was never built.

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

## Files

| File | Contents |
| --- | --- |
| `index.html` | The entire app — markup, CSS, and JS in one file |
| `sw.js` | Service worker, network-first with a cache fallback |
| `manifest.webmanifest` | Home-screen install metadata |
| `icon.svg`, `apple-touch-icon.png` | Icons |
| `README.md` | User-facing description, in Swedish |

## Commands

There are none. Open `index.html` in a browser — that is the dev loop, and it is
the same file that gets deployed.

Two consequences worth knowing:

- **Relative URLs only.** The site is served from
  `https://timpan8.github.io/Plump/`, not a domain root. Every reference in the
  HTML, manifest, and service worker uses `./`-relative paths so the same files
  work locally over `file://` and under the `/Plump/` prefix. An absolute `/…`
  path 404s in production while working fine locally.
- **The service worker is off over `file://`.** It only registers on `https:` or
  `localhost`, so offline behavior can only be verified on a real deployment.

## Testing

There is no committed test suite. Changes are verified by driving the page with
Playwright from a scratch script (mobile viewport, `chromium` from
`/opt/pw-browsers`) and by reading screenshots. If a suite is ever added, these
are the cases worth encoding, in roughly this order of value:

- **Scoring: exhaustive.** Every branch of plump, made zero, and made bid.
- **Named edge cases**, not just happy paths: everyone plumping, ties in the
  final totals, a round where the entered tricks cannot fit the hand size.
- **Persistence: round-trip.** Save mid-game, reload, assert identical state.
  Add a case per stored-format version.
- **UI: representative, not exhaustive.** Score a full round, reload mid-game
  and confirm nothing was lost, check that the sticky header and the frozen
  left column stay put while scrolling.

## Architecture notes

The file is organized as: state and persistence, pure scoring helpers, the setup
view, the game view, the bid/result picker. Keep that separation — the helpers
(`pointsOf`, `totalAfter`, `buildSequence`, `bidOrder`, `forbiddenBid`,
`knownTricks`) touch no DOM and are the part where real bugs live.

- **A cell is `{b, m}`** — the bid, and whether it was made. `m: null` means the
  bid is down but the hand has not been played yet, and that is what drives the
  two-phase round: `nextBid(r)` returns a player while bids are missing, then
  `nextResult(r)` takes over. `currentRound()` is the first round that isn't
  finished, and everything else is locked unless the user turns on fix mode.
- **Turn order is guarded, not enforced.** Entering a player out of order was
  the mistake that actually happened at the table, so `tapCell` refuses to open
  the wrong player's picker and shows the bid order with a button to the right
  player instead. The override is one deliberate extra tap — keep it reachable;
  the goal is making the mistake harder, not impossible. Fix mode skips the
  guard entirely, since correcting is the whole point there.
- **`buildBoard(g, opts)` renders any match**, live or archived. Every helper
  takes the match (`g`: players, rounds, cells, firstDealer) as its first
  argument rather than reading the global — that is what lets the history view
  reuse the same table read-only (no `opts.onCell` → `<div>` cells, no
  handlers).
- **Finished matches are archived** into `state.archive`, keyed by `matchId` so
  re-archiving after a late correction updates the entry instead of duplicating
  it. Archiving happens when the last cell is filled and again on new
  match/restart/clear. The list is reachable only from the setup view — never
  mid-match, by request.
- **The stored state is a file format.** It carries a version (`v`) and the key
  is versioned too (`plump-state-v4`). `load()` refuses anything with a
  different version rather than trying to migrate, so an old save can never
  corrupt a new build. Bump both when the shape changes.
- **Storage is wrapped, never assumed.** `localStorage` throws in private mode
  and when cookies are blocked; the `store` shim falls back to memory so the app
  degrades instead of dying.
- **Render is a full redraw** from state on every change. It is fast enough at
  this size, and it keeps the display impossible to desync from the data.
- **Cells are real `<button>` elements**, not clickable `<td>`s, so the board
  works with a keyboard and a screen reader.

## Domain rules

**Variants of Plump differ between families and regions, so the rules recorded
here are the spec.** Disagreements about intended behavior get settled in this
section, not in test fixtures.

- **Making a bid of 1 or more:** 10 + the number of tricks bid (1 → 11, 8 → 18).
- **Making a bid of zero:** 5 points. Written `05` on the paper sheet.
- **Missing a bid ("plump"):** 0 points, regardless of how far off. Shown as ●.
- **Hand-size progression:** starts at a chosen maximum (default 8), one card
  fewer each round down to a chosen turning point (default 2), then back up.
  Both ends and the "and up again" half are configurable at setup.
- **Deal rotation:** chosen first dealer, then clockwise by round index.
- **Bidding order:** the player after the dealer bids first; **the dealer bids
  last.** This is what makes the restriction below land on the dealer.
- **Bid-total restriction:** the bids may not sum to exactly the hand size, and
  the dealer carries it. When every other player has bid, the value that would
  make the total equal the hand size is disabled in the dealer's picker (5-card
  round, others bid 2 and 2 → the dealer cannot bid 1). Bids summing to *more*
  than the hand size are fine; only the exact total is forbidden.
- **Players:** 2–10, capped further by what a 52-card deck can deal.

## Conventions

- **Swedish UI, Swedish comments, English identifiers.** The file is small and
  self-contained, so strings live inline where they are used rather than in a
  separate module.
- Commit messages: Swedish, matching the existing history.
- Don't commit scratch scripts, screenshots, or `node_modules` from local
  Playwright runs; keep them outside the repo.
