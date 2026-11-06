# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Users

The owner and a small circle of friends and family they share it with, all of whom know baseball. They don't need the game or its stats explained; they want them presented the way they like to read them. The owner's own taste and habits are the main brief.

## Product Purpose

MLB Scorecard (installed as "Scorecard") is a personal MLB companion. It tells a game the way a paper scorecard does and puts the season's stats a tap away. It is used in four moments:

- **Following a live game** on a phone, with the scorecard updating play by play.
- **Catching up after** a finished game: its scorecard, highlights and box score.
- **Digging into players:** leaderboards, profiles, game logs and splits.
- **Tracking a team** through the season: schedule, standings and roster.

Success is that it stays the app they reach for in those moments instead of the official MLB app or a stats site.

## Positioning

- **Scorebook notation.** The game is told as a traditional paper scorecard tells it: a diamond per plate appearance, play codes like 6-3, F8 and K, and fielder sequences. It isn't a feed of text.
- **The look and feel.** A vintage-scorecard world anchored on Fenway Park's Green Monster scoreboard, not a generic sports-app UI.
- **Built the owner's way.** It shows the stats and views the owner cares about, such as wOBA+, K%-BB% and ranks against qualified players, rather than a one-size-fits-all set.

## Operating Context

- Used mostly on a phone as an installed home-screen app, often during a game. It is also used on desktop and tablet, and both must work well.
- Six tabs: Scorecard, Highlights, Standings, Schedule, Roster, Players. Popups provide pitch detail, player quick-looks, game summaries, rosters, spray charts and team win/loss charts.
- The phone's Back button behaves like an app's. It undoes the last drill-in (a player, a game opened from elsewhere, a popup) rather than leaving the app.

## Capabilities and Constraints

- **Single-file app.** One `index.html` with no build step, no dependencies and no backend, plus the PWA files (`manifest.json`, `sw.js`, `icons/`). It is hosted as static files on GitHub Pages.
- **Data source.** All data comes from MLB's public Stats API, fetched directly from the browser. Headshots and team logos come from MLB's image hosts. There are no accounts and no server-side storage. Persisted state is limited to a couple of small UI preferences in `localStorage`.
- **Live data.** Live games poll every few seconds. Anything that can still change during a session must not cache a failure.
- **Teams.** In function, every team is treated the same today. Whether defaults or touches should ever lean toward a particular team (the Fenway look notwithstanding) is undecided.
- **Terminology.** Standard baseball terms throughout (AVG/OBP/SLG/OPS, wOBA+, IP, WHIP, K%-BB%, QS, H-AB). Users know them, so they aren't spelled out.

## Brand Commitments

- **Name:** "MLB Scorecard", short name "Scorecard".
- **The Green Monster green frame is binding.** It is Fenway's Green Monster and stays as it is. Contrast problems are solved by changing what sits on it, never by changing the green.
- **The paper-scorecard concept is the product's identity.** Scorebook notation and the scoreboard world are what the app is, not a theme laid on top.

## Evidence on Hand

- All data is real and live from the MLB Stats API. There are no testimonials, user counts or press, and none should be invented.
- Team logos and player headshots are MLB's, used as a fan app does. They are not the app's own brand assets.

## Product Principles

1. **Tell it like a scorebook.** When there's a choice, prefer the notation and conventions a fan keeps a paper scorecard with over the conventions of generic sports apps.
2. **The owner's way over the standard way.** Pick the stats, ranks and views that are actually useful to these users, not the default set every stats site shows.
3. **Fans don't need explaining.** Skip helper captions and tutorials. The numbers and the notation carry the meaning.
4. **Live and phone first, everywhere else too.** Design for a phone during a game, then make sure desktop and tablet use their room well.
5. **Behave like an app.** Navigation, Back, scroll position and state should feel native, even though it is a web page.
