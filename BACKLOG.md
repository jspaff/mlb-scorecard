# Backlog

Polish notes from poking around the app (captured 2026-11-10). Check an item off (`- [x]`) when it lands, and add the PR link after it. Function names are starting points, not prescriptions.

## Scorecard

- [x] **Base advancement tags: narrower, and one standard width.** They're a touch wide, and stacked tags of different widths look ragged — make every tag the same width so a stack lines up. (`.adv-tag` CSS, `renderCellInner`)
- [ ] **Rethink the "out on the bases" mark.** The idea — a runner's line cut off by something red — is right, but the current drawing doesn't land. Explore a better treatment. (`diamondSVG`, its `isOut`/`outEdge` handling)
- [ ] **Show batter substitutions on the scorecard.** Long-parked: find a clean, scorebook-true way to show pinch hitters / pinch runners / defensive subs in the batting grid. (`buildTeamRows`, `renderTeamTable`; substitution detection near the `looksLikeSub` pass)

## Roster popup (bullpen)

- [x] **Make IP / Pitch count / Yesterday / 2 Days Ago column headers** instead of repeating the labels in every row. (`renderBullpen…` rows with `.bp-recent-item` / `.bp-recent-label`)

## Game Summary popup

- [ ] **Restyle the Game Summary modal to match the Scorecard's newer format.** (`openGameSummaryModal`, `renderGameSummaryContent`, `renderGameSummaryGame`)

## Highlights tab

- [x] **Bigger logo / team name / inning on the video card thumbnails.** (`renderScorebookStill`)
- [x] **Rethink the logos on the score boxes.** They're small, and the more detailed or colorful logos look off at that size. Look for something more space-efficient (e.g. a wordmark/abbreviation treatment). (`renderDayGame`)

## Team tab

- [ ] **Overview feels empty — add content.** Video highlights for the team could be a fun fit. (`renderTeamOverview`)
- [x] **Games-over-.500 chart: replace the "Current … Best … Worst …" caption.** Reads as generic. Possibly drop it and mark the high and low points on the line itself. Applies everywhere the chart appears (Team Overview and the Standings popup). (`renderWinLossSvg`)

## Player profile

- [x] **Back button and profiles — verify.** Check that Back returns to the player you were looking at (e.g. profile → another player → Back; profile → scorecard → Back). Fix any path that doesn't push/restore a profile entry. (`navPush`, `jumpToPlayerProfile`, the `popstate` handler) — Verified: profile → profile, profile → game, popup → profile and Team → profile all return correctly. Fixed: Back now also restores the profile's tab (it used to reset to Overview).
- [x] **Game Log (desktop): tighten the last-5-games scorecards.** Cards leave room for ~6 diamonds and usually use 1–4; reduce the dead space. (`renderProfileGameLog`, `loadProfileRecent`)
- [x] **Spray chart: smoother field drawing.** The outline has a visible point/kink in the middle (likely the outfield arc meeting at center). (`renderBattedBallFieldSVG`)
- [x] **Pitcher Splits: invert the axis?** Lower OPS against is better for a pitcher, so the scale should probably read that way. (pitcher Splits tab chart)

## Player profile — Defense tab

- [x] **Outfielder catch board: Play button is scrunched to the right, and should be gold** like the app's other play buttons. (`renderCatchBoard`, `.pp-cb-play`)
- [x] **Outfielders on wide screens: use the empty space.** The Defense tab leaves a lot unused on a wide sheet.
- [x] **Infielders / catchers on wide screens: put Highlights to the right** of the double plays (or catcher boards) instead of below them. (`renderInfieldPlays`, `renderCatcherPlays`)
