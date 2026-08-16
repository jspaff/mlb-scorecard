# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A self-contained HTML app (`index.html`, ~4,700 lines) implementing a live MLB scorecard widget styled after a traditional paper scorecard, installable as a PWA on mobile home screens. There is no backend, no build step, and no dependencies — all data comes from the public MLB Stats API, fetched directly from the browser at runtime.

The whole page is one file: `<style>` (lines ~9-1471), then the markup (`#mlb-root` and its tab panels/overlays), then a single `<script>` IIFE (from ~1592 to the end) containing all the JS. There is no test suite, linter, or package manager config in this repo.

Alongside `index.html` sit the PWA support files: `manifest.json` (name, theme colors, icons), `sw.js` (an app-shell service worker — same-origin GET requests only; it never touches `statsapi.mlb.com` or Google Fonts, so it can't go stale on live data), and `icons/` (generated PNGs, see `sizes` in `manifest.json`). These are separate files by necessity (a manifest must be linked, a service worker must be its own script with its own scope) — the JS/CSS/markup itself stays single-file.

## Running it

There's nothing to install or build. Serve the directory locally (e.g. `python3 -m http.server`) and open `index.html` over `http://`/`https://` — the service worker only registers on http(s), not `file://`, so PWA install/offline behavior needs a server. All calls to `statsapi.mlb.com` are made client-side with plain `fetch()`; there's no proxy or API key involved.

There's no automated way to verify a change — after editing, open the file in a browser and click through the affected tab/overlay to confirm it still works, including a live or in-progress game if the change touches polling or the scorecard grid. If you touch `sw.js`, bump `CACHE_NAME` so installed clients pick up the new app shell instead of serving a stale cached copy, and confirm via DevTools → Application → Service Workers that it updates/activates as expected.

## Structure

The page is scoped under `#mlb-root` with a set of CSS custom properties (`--paper`, `--ink`, `--gold`, `--brick`, `--frame`) defining the vintage scorecard palette in `:root`. The `PALETTE` JS constant near the top of the script (`const PALETTE = { ink, gold, brick, ball, paper, gray }`) mirrors those same colors for the handful of SVG strings that can't reference a CSS variable directly — if you change the palette, update both places.

### Tabs

Five tabs — Daily, Scorecard, Standings, Schedule, Roster — are plain markup (`data-tab-panel` / `data-tab-controls` attributes) driven by a small hand-rolled router near the bottom of the script (the `Tabs` IIFE: `register`/`switchTo`/`init`). Each tab is registered with an optional `init` (runs once, on first activation) and `onShow` (runs every time the tab becomes active). The active tab is also mirrored into the URL hash. Adding a new tab means a `register()` call plus a button + panel in the HTML — there's no routing framework to wire up.

### Overlays / modals

Pitch detail, roster, spray chart, player, game summary, and team win/loss chart each have their own overlay `<div>`, a `show*Modal`/`hide*Modal` pair, and are dismissed via close button, backdrop click, or the global `Escape` keydown handler at the bottom of the script.

### Live polling

Live-game and daily-summary polling deliberately self-reschedule with `setTimeout` (see `dailyPollTimer`) or clear-then-`setInterval` (see `pollTimer` in `fetchAndRender`) rather than a bare `setInterval`, so that a fresh call — whether triggered by the user changing dates or by the poll tick itself — can cancel whatever's pending with one `clearTimeout`/`clearInterval` at the top. This avoids a stale date's poll cycle surviving a race against a newer one. Polling only runs while a game's `abstractGameState` is `Live`, at a 15s interval.

### Scorecard rendering (the core feature)

The Scorecard tab turns the live game feed's play-by-play (`allPlays`) into traditional scorecard notation:
- `getPlayCode(play)` derives shorthand like `6-3`, `F8`, `K`, `DP` from the event name and fielder sequence.
- `diamondSVG(...)` draws the small diamond glyph per plate appearance (base-runner lines, out ticks, HR/K shading, special-play labels like `CS`/`PO`).
- `buildTeamRows(...)` / `renderTeamTable(...)` assemble the per-inning batting grid from the boxscore and play list.

### Data flow and caching

All rendering functions follow a `renderX(...) -> HTML string` pattern, assigned via `innerHTML` — there's no virtual DOM or DOM-diffing. In-memory, module-scoped caches/stores (`gameLogCache`, `standingsCache`, `venueCache`, `playerStore`, `rosterStore`, `sprayStore`, `pitchStore`) key off IDs (game, player, team, venue) to avoid refetching when switching tabs or reopening modals within a session; nothing is persisted across page loads.

MLB Stats API endpoints in use: `/v1/schedule`, `/v1/standings`, `/v1/teams/{id}/roster`, `/v1/people/{id}/stats`, `/v1/venues/{id}`, `/v1/transactions`, `/v1/game/{gamePk}/winProbability`, and `/v1.1/game/{gamePk}/feed/live` (the live boxscore/play-by-play feed that drives the Scorecard and Daily tabs).

## Conventions observed in this codebase

- 2-space indentation, no semicolon-less style — match what's already there.
- Comments are sparse and used specifically to explain non-obvious rationale (a race condition avoided, why a cache is keyed a certain way, why two values must be kept in sync) — not to restate what the code does. Follow that pattern rather than adding narrative comments.
- Everything lives in the one script-level IIFE; there are no ES modules and nothing is attached to `window` except through normal DOM event wiring.
