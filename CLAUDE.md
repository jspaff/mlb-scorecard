# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A self-contained HTML app (`index.html`, ~9,300 lines) implementing a live MLB scorecard widget styled after a traditional paper scorecard, installable as a PWA on mobile home screens. There is no backend, no build step, and no dependencies — all data comes from the public MLB Stats API (plus Baseball Savant for the profile's Defense tab), fetched directly from the browser at runtime.

The whole page is one file: `<style>` (lines ~18-2985), then the markup (`#mlb-root` and its tab panels/overlays), then a single `<script>` IIFE (from ~3153 to the end) containing all the JS. There is no test suite, linter, or package manager config in this repo.

Alongside `index.html` sit the PWA support files: `manifest.json` (name, theme colors, icons), `sw.js` (an app-shell service worker — same-origin GET requests only; it never touches `statsapi.mlb.com`, `baseballsavant.mlb.com` or Google Fonts, so it can't go stale on live data), and `icons/` (generated PNGs, see `sizes` in `manifest.json`). These are separate files by necessity (a manifest must be linked, a service worker must be its own script with its own scope) — the JS/CSS/markup itself stays single-file.

## Running it

There's nothing to install or build. Serve the directory locally (e.g. `python3 -m http.server`) and open `index.html` over `http://`/`https://` — the service worker only registers on http(s), not `file://`, so PWA install/offline behavior needs a server. All calls to `statsapi.mlb.com` and `baseballsavant.mlb.com` are made client-side with plain `fetch()`; there's no proxy or API key involved.

There's no automated way to verify a change — after editing, open the file in a browser and click through the affected tab/overlay to confirm it still works, including a live or in-progress game if the change touches polling or the scorecard grid. Bump `sw.js`'s `CACHE_NAME` on **any change to `index.html`, `manifest.json`, or the icons** (not just edits to `sw.js` itself) — page navigations are network-first (the cache is only the offline fallback), but the manifest and icons are served cache-first and only refreshed in the background for the *next* load, and old caches are only cleared when a new `CACHE_NAME` activates. `install` fetches the shell with `cache: 'reload'` so a new version can't be seeded from the browser's HTTP cache (GitHub Pages sends `max-age=600`); keep that if you touch it. Confirm via DevTools → Application → Service Workers that it updates/activates as expected.

Whenever a Claude Code session edits `index.html` (or `sw.js`/`manifest.json`), attach the updated file(s) to the response (e.g. via a "send file" tool if available) rather than waiting to be asked — the user's workflow is downloading it straight into a local server setup to test.

## Product and design context

`PRODUCT.md` records who the app is for, how it's used, and its binding commitments (the Green Monster green frame among them). `DESIGN.md` records the visual system — tokens in its YAML frontmatter, then the rules for applying them (gold is earned, blank zeros, square corners, slots vs. nameplates), with `.impeccable/design.json` as its machine-readable companion. Read both before any visual or UX change, and update `DESIGN.md` when a change establishes or retires a design rule.

## Structure

The page is scoped under `#mlb-root` with a set of CSS custom properties (`--paper`, `--ink`, `--gold`, `--brick`, `--frame`) defining the vintage scorecard palette in `:root`. The `PALETTE` JS constant near the top of the script (`const PALETTE = { ink, inkSoft, gold, brick, ball, paper, paperLine, gray, startMuted }`) mirrors those same colors for the handful of SVG strings that can't reference a CSS variable directly — if you change the palette, update both places.

### Tabs

Six tabs — Scorecard, Highlights, Standings, Schedule, Roster, Players — are plain markup (`data-tab-panel` / `data-tab-controls` attributes) driven by a small hand-rolled router near the bottom of the script (the `Tabs` IIFE: `register`/`switchTo`/`init`/`current`). Each tab is registered with an optional `init` (runs once, on first activation) and `onShow` (runs every time the tab becomes active). The active tab is also mirrored into the URL hash, and `Tabs.init('scorecard')` makes Scorecard the default on load. Adding a new tab means a `register()` call plus a button + panel in the HTML — there's no routing framework to wire up.

`.tab-header` is sticky, and its real height (it varies by tab, since each tab has its own controls row) is kept in the `--tab-header-h` custom property on `#mlb-root` by `syncSidebarOffsets` — on resize, on any change to the header's own size (a `ResizeObserver`, since web fonts landing or a controls row wrapping changes it without a resize), and on showing the Scorecard and Players tabs. Anything else that sticks below the header (the Scorecard sidebar, the Game Log's column heads and recent-games column) reads that property rather than hard-coding an offset.

### History and the Back button

The app takes part in browser history so the phone's Back button behaves like an app's: it undoes the last drill-in rather than leaving the app. The model, in the block right after the `Tabs` IIFE:

- **Drill-ins push an entry** describing the screen they open — `navPush({ tab, ... })` records the scroll position on the entry being left, then pushes the new one. Opening a player pushes `{ tab: 'players', profile, from, depth }` (`from` is `'list'` or `'other'`, `depth` how many players deep through search); opening a game from elsewhere (`jumpToScorecardGame`) pushes `{ tab: 'scorecard', game: { date, pk } }`; opening a popup pushes the current screen's state plus `modals` (see below).
- **Tab-bar switches replace** the current entry (`Tabs.switchTo` does a `replaceState`) rather than pushing, so Back doesn't walk through every tab that was tapped.
- **One `popstate` handler restores** whatever entry Back or Forward lands on: popups first (closing any the entry doesn't list), then the tab (`Tabs.switchTo(id, { history: false })` — the browser has already set the URL), the Players tab's list or profile, the Scorecard's game, and the scroll position.
- Navigating *from another tab* (`jumpToPlayerProfile`, `jumpToScorecardGame`) pushes its entry *before* switching tabs, so the tab being left keeps its own entry and Back returns to it.
- A profile's back link and the Batters/Pitchers toggle go "to the list" through `leaveProfileForList`: `history.go(-depth)` when the profile was opened from the list (so Back and the link agree), otherwise a `replaceState` to the list in place.

When adding a new drill-in, push an entry for it with `navPush` and teach the `popstate` handler to restore it; when adding a new way to *switch* screens within a tab (a date change, a filter), don't push one.

### Players tab

`playersBody` shows either the sortable list or one player's profile — `playersView` (`'list'`/`'profile'`) says which, and `rerenderPlayerProfile` is a no-op unless it's `'profile'`, so a slow profile fetch can't redraw over the list after the user has gone back. The list's filters (`playerListState`) apply client-side over cached rows except season and pool, which pick what's fetched (`playerListCacheKey`). The profile follows the list's season; its data arrives in stages (`showPlayerProfile` → person/career/awards, then `loadProfileSeason` → the season detail and the Overview board, then — only once the Game Log tab is opened — `loadProfileRecent` for the last five games' play-by-play), each stage guarded by `playerProfileRequestId` and each kept in a `lastProfile*` variable that `renderPlayerProfile` reads. Ranks and gold on the Overview board are against that season's qualified pool — the same rows, and so the same count, the list shows (`buildProfileBoard`). Outfielders and infielders (`primaryPosition.type`, see `DEFENSE_GROUPS`) also get a Defense tab, loaded only once it's opened (`ensureProfileDefense` → `loadProfileDefense`): OAA and Fielding Run Value per position plus the group as a whole, both ranked against Savant's OAA-qualified pool for that position (`buildProfileDefense`, `profileDefensePool` picks which). Outfielders also get the catch board from `lastProfileCatches`, whose clips open in the clip popup (`showClipModal`); infielders get a double-play ledger and MLB's defensive highlight clips of them (`renderInfieldPlays`), from `getInfieldSeasonPlays` — a sweep of every game he played the infield (`/v1/game/{gamePk}/playByPlay` for double plays, built into scorebook order by `buildDoublePlay`, and `/v1/game/{gamePk}/content` for highlights tagged `defense` plus his `player_id`), ~14 MB a season, so it runs only once the Defense tab opens, reports progress straight into its status line, and keeps only the plays it extracts for the session. Catch by catch only exists for the player's latest season.

The Highlights tab is `daily` internally (`data-tab="daily"`, `dailyBody`, `dailyPollTimer`, etc.) — it used to be called "Daily" and show a grid of the day's games before that moved to the Scorecard tab's own game strip (see below), leaving Highlights with just its Home Runs/Strikeouts views. The user-facing rename wasn't pushed through the internal identifiers; match that if you touch this code.

### Overlays / modals

Pitch detail, roster, spray chart, player, game summary, team win/loss chart, the Scorecard's game-strip list, and the clip player each have their own overlay `<div>`, a `show*Modal`/`hide*Modal` pair, and are dismissed via close button, backdrop click, the global `Escape` keydown handler at the bottom of the script — or the Back button.

Every overlay must open through `openOverlay(el)` and close through `closeOverlay(el)` (`hidePitchPopup` wraps it), never by toggling the `open` class directly — that's what keeps them in step with history. `openOverlay` pushes an entry listing the popups open over the screen (`state.modals`, mirrored in memory as `modalStack`), so Back closes the top one; `closeOverlay`, used by every other way of closing one, steps history back to match (batched, so `Escape` closing a stack is a single `history.go(-n)`), which leaves no dead entries behind. Navigation *out* of a popup (View Full Profile, View Full Scorecard, a Players of the Game name) has to wait for that step back to land before pushing its own entry, or the pending step would pop the new one — wrap it in `whenModalHistorySettled(fn)`, as `jumpToPlayerProfile` and `jumpToScorecardGame` do.

### Live polling

Live-game and daily-summary polling deliberately self-reschedule with `setTimeout` (see `dailyPollTimer`) or clear-then-`setInterval` (see `pollTimer` in `fetchAndRender`) rather than a bare `setInterval`, so that a fresh call — whether triggered by the user changing dates or by the poll tick itself — can cancel whatever's pending with one `clearTimeout`/`clearInterval` at the top. This avoids a stale date's poll cycle surviving a race against a newer one. Polling only runs while a game's `abstractGameState` is `Live`, at a 7s interval (`pollTimer`) for the Scorecard tab and a 15s interval for the Daily tab's summary (`dailyPollTimer`).

`clearTimeout`/`clearInterval` alone only stops *future* ticks, though — it can't cancel a call already in flight. Every async load path that can have more than one call outstanding at once (`fetchAndRender`, `loadDailySummary`, `openGameSummaryModal`'s venue/highlights loaders, player search, `showPlayerProfile`) also guards with a bumped `xRequestId` counter, checked right before that call writes to shared state or the DOM, so a slow, now-stale response can never overwrite what a newer call already rendered. Match this when adding a new polling or debounced-fetch path.

### Scorecard rendering (the core feature)

The Scorecard tab turns the live game feed's play-by-play (`allPlays`) into traditional scorecard notation:
- `getPlayCode(play)` derives shorthand like `6-3`, `F8`, `K`, `DP` from the event name and fielder sequence.
- `diamondSVG(...)` draws the small diamond glyph per plate appearance (base-runner lines, out ticks, HR/K shading, special-play labels like `CS`/`PO`).
- `buildTeamRows(...)` / `renderTeamTable(...)` assemble the per-inning batting grid from the boxscore and play list.

The spray charts (the Scorecard's per-team chart and a hitter's profile chart) share `renderBattedBallFieldSVG`, which draws the field in the API's own hit-coordinate space (`hitData.coordinates`): home plate at `SPRAY_HOME` (125.42, 198.27), `SPRAY_FT_PER_UNIT` ≈ 2.495 ft per unit — a convention checked against MLB's own `totalDistance` on fly balls and home runs. Keep the field and the dots in those same units, so a 400-ft home run lands just past a 400-ft fence.

### Data flow and caching

Functions split into three layers by name: `getX`/`fetchX` (async, hits the API, returns data — the `get`/`fetch` prefix itself carries no distinction, both are used for the same kind of function), `buildX` (sync, transforms raw API data into a structured intermediate like `buildTeamRows`/`buildBullpenList` — not HTML), and `renderX` (sync, returns an HTML string, assigned via `innerHTML` — there's no virtual DOM or DOM-diffing).

Every `getX`/`fetchX` function wraps its `fetch` in its own try/catch and degrades gracefully on failure rather than letting callers deal with a rejected promise — match this when adding a new one. In-memory, module-scoped caches/stores (`gameLogCache`, `standingsCache`, `venueCache`, `playerStore`, `rosterStore`, `sprayStore`, `pitchStore`, etc.) key off IDs or composite keys (e.g. `` `${personId}-${season}` ``) to avoid refetching when switching tabs or reopening modals within a session; nothing is persisted across page loads. Whether a *failure* gets cached depends on the data, and both cases exist on purpose:
- Static, low-stakes lookups (venue info, player bio/awards) cache `null` on failure — a bad request that stays bad for the rest of the session is a fair trade against hammering a flaky endpoint, and a blank field is a low-stakes degradation.
- Anything that can still change during the session (live game stats, in-progress schedules) does *not* cache a failure — see `fetchGameStats`, which only writes to its cache when `isFinal` and otherwise returns fresh empty results every call, so a transient error doesn't lock in stale/missing data for a game that's still moving.

When adding a new cached fetcher, pick the side of that line matching the data, don't default to "always cache null on error."

MLB Stats API endpoints in use: `/v1/schedule`, `/v1/standings`, `/v1/seasons/{year}`, `/v1/stats` (league-wide season leaderboards for the Players list), `/v1/teams/stats`, `/v1/teams/{id}/roster`, `/v1/people` and `/v1/people/{id}` (search, bio, awards), `/v1/people/{id}/stats` (season, yearByYear, gameLog, and statSplits for the player profile), `/v1/venues/{id}`, `/v1/transactions`, `/v1/game/{gamePk}/winProbability`, `/v1/game/{gamePk}/playByPlay` (the profile Game Log's last-five hitter cards and the Defense tab's double plays), `/v1/game/{gamePk}/content` (the Defense tab's highlight clips), and `/v1.1/game/{gamePk}/feed/live` (the live boxscore/play-by-play feed that drives the Scorecard and Daily tabs).

Baseball Savant (`baseballsavant.mlb.com`, which answers with `Access-Control-Allow-Origin: *`) supplies what the Stats API doesn't: the Defense tab's Outs Above Average, Fielding Run Value and (outfielders only) catch star ratings, and the in-app video clips. `getSavantOaa` reads the `/leaderboard/outs_above_average` CSV (per position `7`/`8`/`9` or `of`, qualified `min=q` or everyone `min=0`); `getSavantFrv` reads the `/leaderboard/fielding-run-value` CSV per position `3`–`9` (its `position` filter splits a player's runs by position, its qualified filter does nothing, and a whole group is the sum of its positions); Savant's `/leaderboard/arm-strength` CSV (average mph per position, 2020 on) also works from the browser but isn't used; `getSavantCatches` reads the `rangeLine: [...]` JSON embedded in the `/savant-player/{id}` page (every catch opportunity with its stars and `play_id`; there's no API for it, it always covers the player's latest season, and it's ~190 KB, so it's fetched once per player, only when the Defense tab opens); `getSavantClip` reads `/sporty-videos?playId=` for a play's mp4, description, date and gamePk, which `showClipModal` plays in `#clipOverlay` (above every other popup) — for the Defense tab's catches and every Watch link (the Scorecard's at-bats, the Highlights tab's home runs, the game summary's Play of the Game). Build a new Watch link with `clipWatchAttrs`: it keeps the Savant `href` for open-in-new-tab, and one delegated click handler on `#mlb-root` plays any `a[data-clip-play-id]` in the app on a plain click. A Savant `play_id` is the same id as `playEvents[].playId` in the Stats API's game feed. Leaderboard-wide work (ranks, list columns) belongs on the CSVs, never on per-player pages. The Stats API's own `stats=outsAboveAverage` exists but is wrong (no player is ever below zero) — don't use it.

### HTML building and escaping

There is no `escapeHtml`/sanitization helper anywhere in the file — every `renderX` function interpolates data straight into template strings assigned via `innerHTML`. This is safe only because every value ever interpolated originates from the MLB Stats API or Baseball Savant, never from anything the user types or pastes. The two `localStorage` values (below) don't break this either, since both are team IDs chosen from a fixed `<select>`, not free text. If a future change ever interpolates user-typed input (a search box, a note field, anything) into an `innerHTML` template, it must be escaped explicitly at that point — don't assume the rest of the file's unescaped interpolation makes that safe too.

### Persisted preferences

`localStorage` is used sparingly and only for trivial, session-spanning UI preferences — currently `mlb-schedule-team` and `mlb-roster-tab-team`, each just a team ID string. Keys are prefixed `mlb-`, and every read/write is wrapped in its own try/catch (private browsing / storage-disabled safety). Keep any new persisted preference this narrow — small values, `mlb-`-prefixed key, try/catch-wrapped — rather than growing an ad hoc storage layer.

## Conventions observed in this codebase

- 2-space indentation, no semicolon-less style — match what's already there.
- Comments are sparse and used specifically to explain non-obvious rationale (a race condition avoided, why a cache is keyed a certain way, why two values must be kept in sync) — not to restate what the code does. Follow that pattern rather than adding narrative comments.
- Everything lives in the one script-level IIFE; there are no ES modules and nothing is attached to `window` except through normal DOM event wiring.
