# Ashes Reborn Game Visualizer

A single-page tool to visualize [Ashteki](https://github.com/Ashteki/ashteki) game chat logs for [Ashes: Reborn](https://www.plaidhatgames.com/board-games/ashes-reborn/). Paste a game's chat log and get an interactive chart showing health, dice advantage, battlefield value, and attacks over the course of the game.

## Features

- **Chat log input** – Paste the plain-text game chat copied from Ashteki. The parser reconstructs the full game timeline from chat events including:
  - Player and Phoenixborn identification (`"X brings Y to battle"`)
  - Round and turn structure (`"Round N"`, `"Turn N - Player"`)
  - Attacks and defending units (`"Player attacks ... with K units: UnitName"`)
  - Phoenixborn damage (`"PhoenixbornName receives/takes N damage"`)
  - Dice spending (cost prefixes like `"natural class die : Player plays ..."` and inline die use)
  - Unit summoning and destruction for battlefield tracking
  - Manual mode corrections (units moved to/from play area)
  - Game result (concession or defeat)

- **Phoenixborn cards** – Displays each player's Phoenixborn with starting life values. All 32 standard Phoenixborn are supported with hardcoded life totals.

- **Game result banner** – Shows the winner, their Phoenixborn, the round/turn, and method (concession vs defeat).

- **Layered chart views** – Toggle any combination of layers using checkboxes:
  - **Health** – Both Phoenixborn's remaining health over time (line chart)
  - **Dice advantage** – Unspent dice difference per turn, estimated from chat (bar chart, green/red). Each player starts with 10 dice per round.
  - **Attacks** – When each player attacked and with how many units, with tooltips showing unit names (narrow bar chart)
  - **Battlefield value** – Total dice cost of units on the battlefield for each player (dashed lines), plus the battlefield advantage as bars (orange/cyan). Tracks units entering play, being destroyed, removed from the game, and manual mode corrections.

  Layers can be freely combined — e.g. overlay battlefield advantage with dice advantage to compare resource investment vs tempo, or attacks with health to see the impact of each swing.

## How it works

The x-axis represents **turns**, labeled as `R{round}.{turn} {player}` (e.g. `R1.4 Schmendrix`). Each data point is a snapshot of the game state at the end of that turn.

**Dice tracking**: Every round, each player starts with 10 dice. The parser detects dice spending from two chat patterns — cost prefixes before actions and inline die use — and estimates unspent dice per turn.

**Battlefield tracking**: When a unit enters play (`"puts UnitName (id) into play"`), the parser associates it with the dice cost from the preceding cost line (or 0 for free summons). When a unit is destroyed, removed from the game, or manually moved out of play, its dice value is subtracted.

## Hosting as a GitHub Page

1. Push this repo to GitHub.
2. In the repo: **Settings > Pages**.
3. Under "Build and deployment", choose **Deploy from a branch**.
4. Select the branch (e.g. `main`) and folder **/ (root)**.
5. Save. The page will be at `https://<username>.github.io/<repo>/`.

No build step is required — the page is a single HTML file that loads Chart.js from a CDN.

## Local use

Open `index.html` in any browser (file protocol works fine).

## License

Compatible with the Ashteki project (AGPL-3.0). See [Ashteki repository](https://github.com/Ashteki/ashteki) for game and replay format details.
