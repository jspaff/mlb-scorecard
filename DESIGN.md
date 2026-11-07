---
name: MLB Scorecard
description: A live MLB scorebook kept under Fenway's Green Monster.
colors:
  monster-green: "#54796d"
  scorebook-board: "#3b4336"
  board-rule: "#545e4d"
  scorekeeper-cream: "#f3ecdb"
  faded-cream: "#c3c8bb"
  pennant-gold: "#d9a91a"
  brick-red: "#e0604a"
  brick-red-text: "#f59a87"
  brick-shadow: "#5c4038"
  scoreboard-lamp-green: "#3fae56"
  scoreboard-panel: "#33422e"
  weathered-sage: "#7d8577"
typography:
  display:
    fontFamily: "Oswald, sans-serif"
    fontSize: "42px"
    fontWeight: 600
    lineHeight: 1.05
  headline:
    fontFamily: "Oswald, sans-serif"
    fontSize: "20px"
    fontWeight: 500
    lineHeight: 1.3
  stat:
    fontFamily: "Oswald, sans-serif"
    fontSize: "46px"
    fontWeight: 600
    lineHeight: 1.45
    fontFeature: "tnum"
  title:
    fontFamily: "Oswald, sans-serif"
    fontSize: "15px"
    fontWeight: 500
    lineHeight: 1.3
  body:
    fontFamily: "Inter, sans-serif"
    fontSize: "13px"
    fontWeight: 400
    lineHeight: 1.5
  data:
    fontFamily: "Overpass, sans-serif"
    fontSize: "12px"
    fontWeight: 700
    fontFeature: "tnum"
  label:
    fontFamily: "Overpass, sans-serif"
    fontSize: "10px"
    fontWeight: 700
    letterSpacing: "0.08em"
  notation:
    fontFamily: "Special Elite, monospace"
    fontSize: "14px"
    fontWeight: 400
rounded:
  none: "0px"
  slot: "2px"
  dot: "50%"
spacing:
  hairline: "4px"
  tight: "10px"
  gutter: "18px"
  section: "22px"
components:
  sheet:
    backgroundColor: "{colors.scorebook-board}"
    textColor: "{colors.scorekeeper-cream}"
    rounded: "{rounded.none}"
    padding: "18px"
  nameplate:
    backgroundColor: "rgba(243, 236, 219, 0.08)"
    textColor: "{colors.scorekeeper-cream}"
    typography: "{typography.data}"
    rounded: "{rounded.none}"
    padding: "2px 7px"
  nameplate-hover:
    backgroundColor: "{colors.pennant-gold}"
    textColor: "{colors.scorebook-board}"
  stat-slot:
    backgroundColor: "rgba(0, 0, 0, 0.22)"
    textColor: "{colors.scorekeeper-cream}"
    typography: "{typography.stat}"
    rounded: "{rounded.slot}"
  stat-slot-elite:
    textColor: "{colors.pennant-gold}"
  award-plate:
    textColor: "{colors.pennant-gold}"
    typography: "{typography.label}"
    rounded: "{rounded.none}"
    padding: "4px 7px 3px"
  award-count:
    backgroundColor: "{colors.pennant-gold}"
    textColor: "{colors.scorebook-board}"
    padding: "1px 5px 0"
  tab:
    textColor: "{colors.faded-cream}"
    typography: "{typography.label}"
    padding: "9px 10px 8px"
  tab-active:
    textColor: "{colors.scorekeeper-cream}"
  segmented-toggle:
    backgroundColor: "{colors.scorebook-board}"
    textColor: "{colors.faded-cream}"
    typography: "{typography.label}"
    rounded: "{rounded.none}"
    padding: "7px 12px"
  segmented-toggle-active:
    backgroundColor: "rgba(217, 169, 26, 0.15)"
    textColor: "{colors.scorekeeper-cream}"
  field-select:
    backgroundColor: "{colors.scorebook-board}"
    textColor: "{colors.scorekeeper-cream}"
    typography: "{typography.title}"
    rounded: "{rounded.none}"
    padding: "7px 26px 7px 10px"
  status-plate:
    backgroundColor: "{colors.scorebook-board}"
    textColor: "{colors.scorekeeper-cream}"
    rounded: "{rounded.none}"
    padding: "16px 18px"
---

# Design System: MLB Scorecard

## Overview

**Creative North Star: "The Fenway Scorebook"**

A fan's paper scorebook, kept under the Green Monster. Every surface is either the Monster-green wall (the frame around everything), the dark board pages laid on it (sheets), or the cream ink a scorekeeper writes with. The game is told in scorebook notation: diamonds per plate appearance, play codes like 6-3 and F8, and fielder sequences. The rest of the app, from leaderboards to profiles and game logs, is kept with the same pencil.

The mood is precise, spare and scorekeeper-like. The old-ballpark character lives in the type, the notation and a few physical scoreboard parts, not in ornament. Numbers carry the page; captions, legends and helper text are left out because these readers know baseball. Color is rationed: cream for everything that is simply true, gold for the scoreboard's trim and for what is excellent, and brick red only for outs and runs against.

Components split by role. Data and identity get physical scoreboard parts: recessed number slots, pegged nameplates and award plates. Controls and navigation stay quiet: thin rules, soft labels and square segments that recede until touched.

**Key Characteristics:**
- Monster-green frame, dark board sheets, cream ink. Three layers, always in that order.
- Square corners everywhere except the 2px recessed slot and round data dots.
- Condensed Oswald for names and numbers; small-caps Overpass for every label.
- Gold does two jobs: trim in the scheme (section titles, active and focus rules), and earned in the data (elite rank, quality start, award).
- Zeros left blank in tables, so the cells that carry ink are the ones that happened.

## Colors

A three-layer palette of wall, board and ink, with two spare signal colors.

### Primary
- **Monster Green**: the frame. It's the page background behind every sheet and the PWA theme color, matching Fenway's Green Monster. It is binding and never changes. Contrast is solved by what sits on it.

### Secondary
- **Pennant Gold**: the scheme's accent and its one positive signal. As trim it paints section titles, the active tab and segment rules, focus rings and the jersey number. In the data it marks award plates, a stat in the top 10% of qualified players, a made 5-star catch, quality starts, RBI dots and hits on the spray chart.

### Tertiary
- **Brick Red**: the out and the run against. It fills strikeout diamonds and out ticks. Its text-only tint, **Brick Red Text**, is lighter for small type on the board, where the fill color is too dark to read (runs allowed, strikes).

### Neutral
- **Scorebook Board**: sheets, modals, plates and inputs; the dark page the scorebook is written on.
- **Board Rule**: hairlines, table rules, input borders and slot edges.
- **Scorekeeper Cream**: primary text and ink on both the board and the frame.
- **Faded Cream**: secondary text on the board (labels, column heads, inactive controls, PA counts). It is never used on the frame, where it fails contrast.
- **Weathered Sage**: muted data fills, such as a non-quality start on the Starts chart.
- **Scoreboard Panel** and **Scoreboard Lamp Green**: the live game's scoreboard panel and its lit count lamps.

### Named Rules
**The Monster Stays Rule.** The frame is Monster Green and never darkens or shifts. Text on the frame is full Scorekeeper Cream. Small or secondary text belongs on a board plate, not directly on the wall.

**The Two Golds Rule.** Gold has two jobs, and they don't mix. As trim it is simply part of the scheme, like the paint on a scoreboard: section titles, the active tab and segment rules, focus rings, the jersey number, the wash on the user's own team. In the data it is earned: a number, dot, lamp or mark turns gold only for excellence or a positive event (top 10% of that season's qualified pool, a quality start, an award, a hit, an RBI, a made 5-star catch). Trim never reaches into the data: a value is never gold just to lead the eye, so a "lead stat" stays cream.

**The Blank Zero Rule.** In game logs and tables, a zero count is left blank, never written as 0. The exceptions are pairs read aloud ("0-4" H-AB, an inning pitched) and a linescore or big-number summary, where a blank would read as missing.

## Typography

**Display Font:** Oswald (with sans-serif)
**Label/Data Font:** Overpass (with sans-serif)
**Body Font:** Inter (with sans-serif)
**Notation Font:** Special Elite (with monospace)

**Character:** Oswald is the condensed scoreboard numeral and the player's name on their plate. Overpass is the stenciled label and the box-score figure. Special Elite is the scorekeeper's typewritten notation inside each diamond. Inter appears only for running text, which is rare by design.

### Hierarchy
- **Display** (Oswald 600, 42px on wide sheets, 26px on narrow, 1.05): player names on the profile.
- **Stat** (Oswald 600, 46px / 34px narrow, tabular figures): numbers in recessed slots on the Overview board.
- **Headline** (Oswald 500, 20px / 16px narrow): team and position line under a name. The position is a step softer than the team.
- **Title** (Oswald 500, 15px): vitals values, select values, card summaries.
- **Data** (Overpass 700, 12px, tabular figures): table cells, game-log rows, splits.
- **Label** (Overpass 700–800, 9–11px, 0.05–0.12em tracking, uppercase): tabs, column heads, slot labels ("painted" on the board), section titles, award plates.
- **Body** (Inter 400, 12.5–13px, 1.5): rare running text such as tooltips and footnotes.
- **Notation** (Special Elite, 10–18px inside a diamond): play codes K, HR, 6-3, F8. Used in the diamond glyph, and wherever a play is named by its scorebook notation outside one (the Defense tab's double-play ledger, a double play's clip popup).

### Named Rules
**The Tabular Figures Rule.** Every column of numbers uses tabular figures, so ranks, averages and scores line up digit for digit.

**The Painted Label Rule.** Labels sit above what they name, in small uppercase Overpass with wide tracking, like paint on a scoreboard. A value is never prefixed inline with its abbreviation ("Ht / Wt …") when a labelled pair fits.

## Layout

The page is a Monster-green frame with an 18px gutter. A sticky tab header holds the tab bar and that tab's controls row; its live height is published as `--tab-header-h`, and anything that sticks below it (sidebars, table heads, the recent-games column) offsets by that value. Content sits in sheets, and each sheet is a container that responds to its own width (container queries), not the viewport's.

Breakpoints are content-driven:
- **Below 520px (phone):** single column. Stat boards become 2×4, recent-game cards become a swipeable row (each card capped near its content width), and tables abbreviate opponents and tighten cells so they fit without sideways scrolling down to 320px.
- **Around 700px:** wide variants switch on, such as the larger name and numerals, the extra hit columns, and labels inline instead of stacked.
- **Around 900px:** recent-game cards line up five across.
- **Around 1150px:** they stand in a sticky column beside the season log.

Tables size their identity columns (date, opponent, result) to content, and give stat columns a fixed compact width on wide sheets, so stats read as one block rather than spreading across the sheet.

### Named Rules
**The Fixed Slot Rule.** Any control whose content changes, such as a filter select, a stat column or a swipe card, gets a fixed slot sized for its longest real value. Changing a filter must never shift the row around it.

## Elevation & Depth

The system is flat paper on a wall with a few physical exceptions. Sheets lift off the frame with an inset top line and a soft drop. Pegged parts (nameplates, the player photo print) cast short, tight shadows as if hung. Number slots are pressed in, with an inset shadow. Nothing floats, glows decoratively, or blurs.

### Shadow Vocabulary
- **Sheet** (`box-shadow: 0 2px 0 rgba(0,0,0,0.25) inset, 0 4px 14px rgba(0,0,0,0.35)`): a board page laid on the wall.
- **Hung plate** (`box-shadow: 0 2px 3px rgba(0,0,0,0.35)`, rising to `0 3px 4px rgba(0,0,0,0.45)` on hover): nameplates.
- **Print** (`box-shadow: 0 6px 14px rgba(0,0,0,0.45)`): the player headshot, pinned with a 3px cream border.
- **Recessed slot** (`box-shadow: inset 0 2px 3px rgba(0,0,0,0.5), inset 0 -1px 1px rgba(255,255,255,0.05)` on `rgba(0,0,0,0.22)`): stat slots, live-scoreboard cells, card number slots.
- **Header lip** (`box-shadow: 0 6px 8px -6px rgba(11,15,12,0.35)`): the sticky tab header's edge over scrolling content.

### Named Rules
**The Pressed, Not Raised Rule.** Numbers sit in slots pressed into the board; names hang on plates in front of it. Data is never raised on a card.

## Shapes

Square by default (0px): sheets, plates, selects, toggles, buttons, award plates, cards and the photo print. The only rounding is the 2px corner of a recessed slot and the top corners of chart bars (3px, data ends only), plus full circles for data dots. Nameplates carry two tiny peg dots in their top corners, as if hung on a rail.

## Components

### Nameplates (signature)
Clickable names (players and teams) hang like scoreboard plates.
- **Shape:** square, 1px Board Rule border, two 2px peg dots at the top corners.
- **Default:** faint cream wash, cream Overpass 700 uppercase at 10.5–11px, padding 2px 7px, hung-plate shadow.
- **Hover / Focus:** fills Pennant Gold with Scorebook Board text, and the shadow deepens. Long names ellipsize inside the plate.
- **Use:** only for things you can click into. Identity that isn't a link (a profile's own team and position) is plain type.

### Stat Slots (signature)
Numbers in recessed scoreboard slots, under a painted label.
- **Shape:** 2px corners, recessed slot shadow, centered Oswald numerals.
- **Elite:** the number turns Pennant Gold when the player ranks in the top 10% of the season's qualified pool.
- **Rank:** a small Faded Cream rank in the bottom-right corner ("T12/134"), shrinking to 8px on phones so the number keeps its full slot. Playing-time counts (G, PA, GS) are never ranked.

### Catch Lamps
The Defense tab's catch board: one row per Statcast star rating (5 down to 1), every catch chance a lamp in that row, like the live scoreboard's count lamps. A catcher's boards use the same lamps: steals against him by base (lit for a runner he threw out) and his ABS challenges, those with two strikes apart from the rest (lit for an overturned call). Their rows are labelled in painted Overpass instead of stars, and none of their lamps light gold.
- **Lamps:** 11px circles. A catch is lit cream; a miss is a dark recessed socket. Made catches come first, then misses, so the lit run reads as the count.
- **Gold:** only a made 5-star catch lights gold, the catch counterpart of a top-10% stat. Other rows stay cream.
- **Row:** the stars and the made/chances count sit in fixed columns, so every row's lamps start in line. Routine (0-star) chances get a count line, not lamps.
- **Drill-in:** a row opens its catches (made first, hardest first), and each opens its clip in a popup.

### Double-Play Ledger
An infielder's double plays on the Defense tab, kept the way a scorebook would.
- **Rows:** one per notation and role (6-4-3 · Started, 4-6-3 · Turned), the notation in Special Elite, the count in a fixed column, most frequent first. A role line above totals Started / Turned / Finished.
- **Drill-in:** a row opens its plays (date, opponent, batter), each opening its clip.
- **Highlights** follow it: MLB's own defensive clips of the player, newest first, the latest six until asked for all.

### Award Plates
- **Shape:** square, 1px Pennant Gold border, gold uppercase label.
- **Count:** a repeat award gets a solid gold cell with the count ("9×") before the label.
- **Order:** by standing: MVP, Cy Young, Rookie of the Year, Reliever of the Year, All-Star, Silver Slugger, Gold Glove. The seasons won appear on hover.

### Diamond Glyph (signature)
The scorebook cell: a 40-unit diamond per plate appearance, drawn in SVG.
- **Paths:** the bases the batter reached are drawn in gold; unreached paths are faint cream.
- **Fills:** a run scored fills the diamond gold; a strikeout fills it brick red, with the K mirrored for a called third strike.
- **Code:** the play code is typed in Special Elite at the center, in brick red for an out.
- **Marks:** outs on the bases get a brick tick; steals, caught stealing and pickoffs get small gold-framed tags. RBIs are gold dots in the corner, one per run.

### Tabs
- **Style:** Overpass 700 uppercase labels, full cream on the frame (the tab bar) and Faded Cream inside a sheet (profile tabs). The active tab has a 2px Pennant Gold underline. Tab rows scroll sideways on a phone rather than wrapping.

### Segmented Toggle
- **Style:** square segments on a Scorebook Board backing with Board Rule dividers. The active segment gets a gold wash and a 2px gold underline.
- **State:** on a screen the toggle doesn't describe (a player profile), no segment is lit.

### Selects and Inputs
- **On the frame (controls row):** boxed. Scorebook Board fill, 2px Board Rule border, Oswald 500 uppercase, small cream triangle, gold focus outline.
- **Inside a sheet (filters):** a written-in field. No box, a 1px rule underneath, sentence-case Oswald, a thin chevron, and a gold rule on focus. A filter still at its default recedes to Faded Cream.

### Cards and Sheets
- **Sheets:** square, Scorebook Board, 2px cream border, 18px padding, sheet shadow.
- **Recent-game cards:** square, darker board wash, 1px rule. The header row carries date · opponent · result · stat line, and the body is diamonds or a row of number slots. The whole card links to the game.
- **Status plates:** loading, empty and error messages sit on a small board plate when on the frame, and as plain Faded Cream text inside a sheet.

### Postseason Bracket
The Standings tab's Postseason view, drawn like a printed bracket.
- **Layout:** the World Series on top, then each league as three columns (Wild Card, Division, LCS) joined by Board Rule hairlines. A Division Series sits beside the Wild Card that feeds it, and the LCS sits centered across both. Leagues sit side by side on wide sheets and stack on a phone, where team names become abbreviations.
- **Series:** a square card on a darker board wash, two team rows (higher seed on top) with a cap, the name, and series wins in a recessed slot. The whole card opens that series' games. The names aren't nameplates, because the card is what you tap.
- **Outcome:** a team that's out fades to Faded Cream. Only the World Series winner turns gold, with an award plate beneath ("2026 World Series Champions"). A series not yet set shows TBD rows.
- **Older formats:** before 2022 the series are listed by round instead.

### Your Team
The user's team is a cap in a fixed slot at the end of the tab row (a stitched ball when none is picked). Its rows in standings and the bracket get the chosen-segment gold wash (`rgba(217,169,26,0.13)`): trim marking the user's own focus, never a gold value.

### Charts
Single-series and quiet: Board Rule gridlines, Faded Cream axis labels, thin marks with 3px data-end corners, gold for the meaningful category (a quality start), and Weathered Sage or Faded Cream for the rest. Axis titles are labelled, a bar scale that doesn't start at zero has both ends labelled, and readouts appear on hover or tap. There are no legends where the page already explains the color.

## Do's and Don'ts

### Do:
- **Do** keep the Monster Green frame (#54796d) exactly. Fix contrast by changing what sits on it.
- **Do** put numbers in recessed slots and clickable names on pegged nameplates.
- **Do** leave zero counts blank in logs and tables.
- **Do** use Pennant Gold as trim (section titles, active and focus rules) and, within the data, only for top-10% stats, made 5-star catches, quality starts, awards, RBIs and hits.
- **Do** use tabular figures for every column of numbers.
- **Do** size changeable controls to fixed slots so filters never shift the layout.
- **Do** draw spray charts in the API's own coordinate space (home 125.42, 198.27; 2.495 ft per unit) so fences and batted balls agree.

### Don't:
- **Don't** round corners on plates, cards, buttons or selects. Only slots (2px), bar ends (3px) and dots are rounded.
- **Don't** use Faded Cream for text on the green frame. It fails contrast there.
- **Don't** add helper captions, legends or tutorial text. These users know baseball.
- **Don't** make a value gold that hasn't earned it, such as a "lead stat"; trim stays out of the data.
- **Don't** use nameplates for identity that isn't a link, such as a profile's own team and position.
- **Don't** let a zero-based scale waste half a bar. Start the axis near the data and label both ends.
