# Vermillion: Ashes Game Report

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

- **Game result banner** – Shows the winner (highlighted in gold), their Phoenixborn, the round/turn, and method (concession vs defeat).

- **Layered chart views** – Toggle any combination of layers using checkboxes:
  - **Health** – Both Phoenixborn's remaining health over time (line chart)
  - **Total resource advantage** – Unspent dice + battlefield value, Chess.com-style (stacked bars on right axis)
  - **Dice advantage** – Unspent dice difference per turn pair, estimated from chat (bar chart). Each player starts with 10 dice per round.
  - **Battlefield value** – Total dice cost of units on the battlefield for each player (dashed lines), plus the battlefield advantage as bars. Tracks units entering play, being destroyed, removed from the game, and manual mode corrections.

  Layers can be freely combined — e.g. overlay total resource advantage with health to see the impact of each swing.

- **Strategic game state analysis** – Each turn pair is assessed as one of 6 strategic situations based on dice advantage, battlefield advantage, and health advantage:
  - **Dominant Position** – One player leads both dice and battlefield
  - **Committed Advantage** – Battlefield leader with dice parity
  - **Unrealized Advantage** – Dice leader with battlefield parity
  - **Comfortable Parity** – Parity on resources, but one player has a health lead
  - **Parity** – No advantage either way
  - **Split (Proactive / Reactive)** – Different players lead dice vs battlefield

  When the advantaged player is 6+ life behind, the state adds "With Compensation". The game state is shown as a colored ribbon below the charts.

- **Game summary statistics** – A table comparing both players across:
  - Starting and final health
  - Damage dealt to opposing Phoenixborn
  - Total attacks and units attacking
  - Cards played and meditated
  - Dice spent vs dice available
  - Turns in control vs turns at parity/contested

- **Accessible game narrative** – A text-based turn-by-turn account of the game for screen readers. Includes damage, attacks, dice/battlefield advantage, and strategic state per turn pair. **Pivot turns** (where resource advantage swings by ≥2) are highlighted in gold with the swing amount and direction, plus chat lines from that turn.

- **Copy for Discord** – One-click copy of a formatted summary including game result, stats table (Final health, Damage dealt, Attacks, Dice spent, Cards played, Turns in control), and the **hypest turn** — the turn with the biggest resource swing. Ready to paste into Discord chat.

- **Export report** – Download a standalone HTML file containing all charts (Health, Total Advantage, Dice Advantage, Battlefield Value, Attacks), the game state strip, summary statistics, and the full narrative. Self-contained (loads Chart.js from CDN, all data and styles inline) and can be shared or viewed offline.

## How it works

The x-axis represents **turn pairs**, labeled as `R{round}.{turn}` (e.g. `R1.4`). Each data point is a snapshot of the game state at the end of that turn pair (after both players have acted).

**Turn-pair model**: In Ashes, each turn number has two players acting. The parser groups both players' actions within the same turn number into a single data point, giving a cleaner view of the game flow.

**Dice tracking**: Every round, each player starts with 10 dice. The parser detects dice spending from two chat patterns — cost prefixes before actions and inline die use — and estimates unspent dice per turn pair. A heuristic treats consecutive same-player, same-die-type inline uses (e.g. one die used twice for place + remove status token) as a single die.

**Battlefield tracking**: When a unit enters play (`"puts UnitName (id) into play"`), the parser associates it with the dice cost from the preceding cost line (or uses a custom override for units like Raptor Herder, Pack Wolf, etc.). When a unit is destroyed, removed from the game, or manually moved out of play, its dice value is subtracted.

**Damage tracking**: Three layers of detection ensure accurate health tracking. Standard `"takes/receives N damage"` lines handle most cases. Manual mode `"adds a damage"` lines catch corrections made during manual play. As a safety net, `"PBName is destroyed"` forces health to 0 if any damage was unaccounted for in the chat log.

## Visual design

- Off-white background with teal (P1) and amber (P2) accents for charts and Phoenixborn cards
- Vermillion accent color for the Parse button and title
- Lora for the app title; Poppins for body text
- Alternating round background bands on charts for visual grouping
- Custom HTML legend with Phoenixborn names and correct color coding
- Semi-transparent game state ribbon with colored segments

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
