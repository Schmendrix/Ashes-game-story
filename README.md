# Ashes Reborn Game Visualizer

A single-page tool to visualize [Ashteki](https://github.com/Ashteki/ashteki) game chat logs for [Ashes: Reborn](https://www.plaidhatgames.com/board-games/ashes-reborn/). Paste a game's chat log and get interactive charts, strategic analysis, game statistics, and an accessible narrative — all in the browser.

## Features

- **Chat log input** – Paste the plain-text game chat copied from Ashteki. The parser reconstructs the full game timeline from chat events including:
  - Player and Phoenixborn identification (`"X brings Y to battle"`)
  - Round and turn-pair structure (`"Round N"`, `"Turn N - Player"`)
  - Attacks and attacking units (`"Player attacks ... with K units: UnitName"`)
  - Phoenixborn damage — three detection patterns:
    - Standard: `"PhoenixbornName receives/takes N damage"`
    - Manual mode: `"Player adds a damage to PhoenixbornName"`
    - Safety net: `"PhoenixbornName is destroyed"` forces health to 0 if any damage was missed
  - Dice spending (cost prefixes like `"natural class die : Player plays ..."` and inline die use)
  - Cards played (`"Player plays CardName"`) and meditated (`"Player meditates CardName ..."`)
  - Unit summoning and destruction for battlefield tracking
  - Manual mode corrections (units moved to/from play area)
  - Game result (concession or defeat)

- **Phoenixborn cards** – Displays each player's Phoenixborn with starting life values. All 32 standard Phoenixborn are supported with hardcoded life totals.

- **Game result banner** – Shows the winner, their Phoenixborn, the round/turn, and method (concession vs defeat).

- **Layered chart views** – Toggle any combination of layers using checkboxes:
  - **Health** – Both Phoenixborn's remaining health over time (line chart)
  - **Dice advantage** – Unspent dice difference per turn pair, estimated from chat (bar chart, green/red). Each player starts with 10 dice per round.
  - **Attacks** – When each player attacked and with how many units, with tooltips showing unit names (narrow bar chart)
  - **Battlefield value** – Total dice cost of units on the battlefield for each player (dashed lines), plus the battlefield advantage as bars (orange/cyan). Tracks units entering play, being destroyed, removed from the game, and manual mode corrections.

  Layers can be freely combined — e.g. overlay battlefield advantage with dice advantage to compare resource investment vs tempo, or attacks with health to see the impact of each swing.

- **Strategic game state analysis** – Each turn pair is assessed as one of 9 strategic situations based on dice advantage, battlefield advantage, and damage dealt:
  - **Active dominance** – One player leads both resources and is dealing damage
  - **Resource dominance** – One player leads both resources but not converting to damage
  - **Aggro control** – Battlefield leader is dealing damage with dice parity
  - **Board control** – Battlefield leader holding position with dice parity
  - **Direct aggression** – Dice leader is dealing damage with battlefield parity
  - **Holding back** – Dice leader saving resources with battlefield parity
  - **Contested** – Different players lead dice vs battlefield (compound description)
  - **Full parity** – No advantage either way, no damage
  - **Trading blows** – No advantage either way, but damage is being dealt

  The game state is shown as a colored strip below the charts with the controlling Phoenixborn's name on each segment. Parity states are left unlabeled; contested states show both players' initials.

- **Game summary statistics** – A table comparing both players across:
  - Starting and final health
  - Damage dealt to opposing Phoenixborn
  - Total attacks and units attacking
  - Cards played and meditated
  - Dice spent vs dice available
  - Turns in control vs turns at parity/contested

- **Accessible game narrative** – A text-based turn-by-turn account of the game for screen readers, including damage, attacks, dice/battlefield advantage, and strategic state per turn pair. Uses ARIA attributes for accessibility.

- **Export report** – Download a standalone HTML file containing all four charts (Health, Dice Advantage, Battlefield Value, Attacks), the game state strip, summary statistics, and the full narrative. The exported file is self-contained (loads Chart.js from CDN, all data and styles inline) and can be shared or viewed offline.

## How it works

The x-axis represents **turn pairs**, labeled as `R{round}.{turn}` (e.g. `R1.4`). Each data point is a snapshot of the game state at the end of that turn pair (after both players have acted).

**Turn-pair model**: In Ashes, each turn number has two players acting. The parser groups both players' actions within the same turn number into a single data point, giving a cleaner view of the game flow.

**Dice tracking**: Every round, each player starts with 10 dice. The parser detects dice spending from two chat patterns — cost prefixes before actions and inline die use — and estimates unspent dice per turn pair.

**Battlefield tracking**: When a unit enters play (`"puts UnitName (id) into play"`), the parser associates it with the dice cost from the preceding cost line (or 0 for free summons). When a unit is destroyed, removed from the game, or manually moved out of play, its dice value is subtracted.

**Damage tracking**: Three layers of detection ensure accurate health tracking. Standard `"takes/receives N damage"` lines handle most cases. Manual mode `"adds a damage"` lines catch corrections made during manual play. As a safety net, `"PBName is destroyed"` forces health to 0 if any damage was unaccounted for in the chat log.

## Visual design

- Dark theme inspired by Tokyonight
- Alternating round background bands on charts for visual grouping
- Custom HTML legend with Phoenixborn names and correct color coding
- Strategy strip with colored segments and Phoenixborn names (progressive abbreviation for narrow segments)

## Hosting as a GitHub Page

1. Push this repo to GitHub.
2. In the repo: **Settings > Pages**.
3. Under "Build and deployment", choose **Deploy from a branch**.
4. Select the branch (e.g. `main`) and folder **/ (root)**.
5. Save. The page will be at `https://<username>.github.io/<repo>/`.

No build step is required — the page is a single HTML file that loads Chart.js from a CDN.

## Local use

Open `index.html` in any browser (file protocol works fine). Append `?demo=1` to the URL to auto-load `demo-chat.txt` (requires serving over HTTP).

## License

Compatible with the Ashteki project (AGPL-3.0). See [Ashteki repository](https://github.com/Ashteki/ashteki) for game and replay format details.
