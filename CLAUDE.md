# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A self-contained HTML app (`index.html`, ~4,700 lines) implementing a live MLB scorecard widget styled after a traditional paper scorecard, installable as a PWA on mobile home screens. There is no backend, no build step, and no dependencies — all data comes from the public MLB Stats API, fetched directly from the browser at runtime.

The whole page is one file: `<style>` (lines ~9-1471), then the markup (`#mlb-root` and its tab panels/overlays), then a single `<script>` IIFE (from ~1592 to the end) containing all the JS. There is no test suite, linter, or package manager config in this repo.

Alongside `index.html` sit the PWA support files: `manifest.json` (name, theme colors, icons), `sw.js` (an app-shell service worker — same-origin GET requests only; it never touches `statsapi.mlb.com` or Google Fonts, so it can't go stale on live data), and `icons/` (generated PNGs, see `sizes` in `manifest.json`). These are separate files by necessity (a manifest must be linked, a service worker must be its own script with its own scope) — the JS/CSS/markup itself stays single-file.

## Running it

There's nothing to install or build. Serve the directory locally (e.g. `python3 -m http.server`) and open `index.html` over `http://`/`https://` — the service worker only registers on http(s), not `file://`, so PWA install/offline behavior needs a server. All calls to `statsapi.mlb.com` are made client-side with plain `fetch()`; there's no proxy or API key involved.

There's no automated way to verify a change — after editing, open the file in a browser and click through the affected tab/overlay to confirm it still works, including a live or in-progress game if the change touches polling or the scorecard grid. Bump `sw.js`'s `CACHE_NAME` on **any change to `index.html`, `manifest.json`, or the icons** (not just edits to `sw.js` itself) — the service worker serves the cached app shell immediately and only refreshes it in the background for the *next* load, so without a bump, an installed client can sit a full version behind indefinitely, with each reload only fetching the update that shows up on the load after it. Confirm via DevTools → Application → Service Workers that it updates/activates as expected.

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

Functions split into three layers by name: `getX`/`fetchX` (async, hits the API, returns data — the `get`/`fetch` prefix itself carries no distinction, both are used for the same kind of function), `buildX` (sync, transforms raw API data into a structured intermediate like `buildTeamRows`/`buildBullpenList` — not HTML), and `renderX` (sync, returns an HTML string, assigned via `innerHTML` — there's no virtual DOM or DOM-diffing).

Every `getX`/`fetchX` function wraps its `fetch` in its own try/catch and degrades gracefully on failure rather than letting callers deal with a rejected promise — match this when adding a new one. In-memory, module-scoped caches/stores (`gameLogCache`, `standingsCache`, `venueCache`, `playerStore`, `rosterStore`, `sprayStore`, `pitchStore`, etc.) key off IDs or composite keys (e.g. `` `${personId}-${season}` ``) to avoid refetching when switching tabs or reopening modals within a session; nothing is persisted across page loads. Whether a *failure* gets cached depends on the data, and both cases exist on purpose:
- Static, low-stakes lookups (venue info, player bio/awards) cache `null` on failure — a bad request that stays bad for the rest of the session is a fair trade against hammering a flaky endpoint, and a blank field is a low-stakes degradation.
- Anything that can still change during the session (live game stats, in-progress schedules) does *not* cache a failure — see `fetchGameStats`, which only writes to its cache when `isFinal` and otherwise returns fresh empty results every call, so a transient error doesn't lock in stale/missing data for a game that's still moving.

When adding a new cached fetcher, pick the side of that line matching the data, don't default to "always cache null on error."

MLB Stats API endpoints in use: `/v1/schedule`, `/v1/standings`, `/v1/teams/{id}/roster`, `/v1/people/{id}/stats`, `/v1/venues/{id}`, `/v1/transactions`, `/v1/game/{gamePk}/winProbability`, and `/v1.1/game/{gamePk}/feed/live` (the live boxscore/play-by-play feed that drives the Scorecard and Daily tabs).

### HTML building and escaping

There is no `escapeHtml`/sanitization helper anywhere in the file — every `renderX` function interpolates data straight into template strings assigned via `innerHTML`. This is safe only because every value ever interpolated originates from the MLB Stats API, never from anything the user types or pastes. The two `localStorage` values (below) don't break this either, since both are team IDs chosen from a fixed `<select>`, not free text. If a future change ever interpolates user-typed input (a search box, a note field, anything) into an `innerHTML` template, it must be escaped explicitly at that point — don't assume the rest of the file's unescaped interpolation makes that safe too.

### Persisted preferences

`localStorage` is used sparingly and only for trivial, session-spanning UI preferences — currently `mlb-schedule-team` and `mlb-roster-tab-team`, each just a team ID string. Keys are prefixed `mlb-`, and every read/write is wrapped in its own try/catch (private browsing / storage-disabled safety). Keep any new persisted preference this narrow — small values, `mlb-`-prefixed key, try/catch-wrapped — rather than growing an ad hoc storage layer.

## Conventions observed in this codebase

- 2-space indentation, no semicolon-less style — match what's already there.
- Comments are sparse and used specifically to explain non-obvious rationale (a race condition avoided, why a cache is keyed a certain way, why two values must be kept in sync) — not to restate what the code does. Follow that pattern rather than adding narrative comments.
- Everything lives in the one script-level IIFE; there are no ES modules and nothing is attached to `window` except through normal DOM event wiring.
