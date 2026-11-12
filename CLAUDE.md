# Coalition of Couch Coaches – Keeper Tracker

Keeper-eligibility tracker for "The Coalition of Couch Coaches," a 12-team half-PPR keeper league. The site is a single self-contained `index.html` served by GitHub Pages at https://mcossa904.github.io/keeper-tracker/. Michael (Sleeper: mikecossa, team Eastwood Eagles) maintains it; the rest of the league only views it.

## Workflow (every time)
1. `git pull` before changing anything.
2. Make the change in `index.html`.
3. Test it (see Testing).
4. Commit with a short message and `git push`. GitHub Pages rebuilds in a minute or two.

## League rules (encoded in the page)
- 2 keepers per team per year. A player can be counted for at most 3 seasons; the first season a player is owned counts.
- Keep cost = the round he was drafted in that season. Undrafted, or dropped and picked back up, = "2027 ADP minus one round."
- Round 1 picks can't be kept. Can't keep two players at the same position.
- A traded player keeps his draft round and eligibility. A dropped player's eligibility resets.
- You can't keep a player if you don't own that round's 2027 pick (assumed rule; see `state.picks`).
- Kickers were dropped from the league in 2026.

## History
- 2024: Yahoo league 109937, draft Sep 1, 2024
- 2025: Yahoo league 146427, draft Aug 20, 2025
- 2026: Sleeper league 1370092323513319424, draft Aug 22, 2026

## Managers and Sleeper roster IDs
| roster_id | Manager | Sleeper handle |
|---|---|---|
| 1 | Christian | christianmpowell83 |
| 2 | Thomas | DingleberryFarmer |
| 3 | Jordan | gridironbelle |
| 4 | Anthony | OGTone5 |
| 5 | Frank | Dudemanfrank |
| 6 | Mike | mikecossa |
| 7 | Chris | Critzert90 |
| 8 | Matt | JimSocks35 |
| 9 | Dustin | Alledr3 |
| 10 | Nick | nicksfriends |
| 11 | Trey | mynameistrey |
| 12 | Alex | alexehayes15 |

## Data model
All data lives in one JSON block in `index.html`: `<script type="application/json" id="state">`. Edit the JSON, not the rendered HTML. Escape `<` as `\u003c` when writing it back.

- `updated`: "Mon D, YYYY" shown as "Last updated."
- `info[mgr]`: `{h: handle, n: current team name}`.
- `teamNames[year][mgr]`: team names at the 2024 and 2025 drafts.
- `nameHistory[mgr]`: 2024 and 2025 entries (`"Name (Yahoo)"`) plus one 2026 entry with the current name, dated the day of the latest update (`"Name (Sleeper)"`).
- `players[]`: rostered players shown on the main list: `{id, sid, name, pos, team, kept:[years], round, adp, nfl, path:[...]}`. `sid` is the Sleeper player ID. `adp: true` means the cost is ADP minus one round.
- `others{key}`: everyone else with history (free agents, defenses, kickers). Key is the lowercase name with punctuation and Jr./Sr./II/III removed. Defenses live here with `team` set to their current manager; they never appear on the main list.
- `path[]` entries: `{d: "Mon D, YYYY", t: text, tr?: tradeKey, dr?: draftYear}`, in date order. Text formats:
  - `Drafted by X with pick R.PP`, `Kept by X with pick R.PP`
  - `X did not keep - released to the draft pool`
  - `Went undrafted` (or `No longer playing with kickers` for kickers in 2026)
  - `Picked up by X`, `Dropped by X`, `Traded from X to Y`
- `trades{key}`: key is the date, with " (2)", " (3)" for extra trades that day. `{d, sides:[{from, to, players:[{n,p,t} or "2027 5th round pick (Anthony’s)"]}]}`.
- `drafts[year][mgr]`: `[{n, p, t, pk:"R.PP", k?:true}]`.
- `picks`: `{year: 2027, rounds: 15, traded:[{r, orig, owner, d, tr}]}`.
- `weekly`: the collapsed "This week" box: `{range, trades:[tradeKeys], adds:[[mgr,[names]]], drops:[[mgr,[names]]], names:[[mgr, old, new]]}`.

## Weekly update (when Michael says "update")
1. Pull transactions from `https://api.sleeper.app/v1/league/1370092323513319424/transactions/{week}` for the recent weeks. Include everything newer than the last update. Waiver claims can carry an earlier timestamp than when they processed, so also check recent weeks for completed moves you haven't recorded.
2. Pull `/rosters` and treat it as the source of truth for who is on which team. Reconcile every player in `players` against it.
3. Pull `/users` and update each manager's team name (`metadata.team_name`; blank means the default "Team <handle>"). Update `info`, the 2026 `nameHistory` entry and its date.
4. Get names for new Sleeper IDs from `https://api.sleeper.app/v1/players/nfl` (large file; look up only what's needed).
5. For each move, append to the player's `path`. Player leaves every roster: move him from `players` to `others`. Player joins via waivers/free agency: `kept:[2026]`, `round:null`, `adp:true`. Player joins via trade: change `team` only, eligibility stays.
6. Add new trades to `trades` (and any traded picks to `picks.traded`) and add a "Traded from X to Y" entry with `tr` to each player.
7. Rebuild `weekly` with net changes since the last update (ignore players added and dropped within the window) and set `updated` to today.

## Style decisions already made (keep them)
- Season colors: 2024 blue, 2025 orange, 2026 purple (darker shades for filled boxes in dark mode). Links and "2 years left" are green; "1 year left" is teal (yellow in dark mode); ineligible is red.
- Teams start collapsed. Pop-ups lock background scrolling, have Back and × in the top right, and no Close button at the bottom.
- Draft and window titles capitalize "Draft" and "Keepers." Text says "with pick," "did not keep," "season still eligible."
- Mobile first. Most of the league uses phones; tables and rows must fit about 390px wide without sideways scrolling.

## Testing
Open the page at about 390px wide and at desktop width, check the browser console for errors, and click a player, a draft, a trade and the Trade Log. Keep `index.html` self-contained: no external scripts or images.
