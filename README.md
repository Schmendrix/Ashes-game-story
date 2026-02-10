# Ashteki Replay Visualizer

A single-page tool to visualize [Ashteki](https://github.com/Ashteki/ashteki) game replays: phoenixborn, dice advantage, health over time, and attacks.

## Features

- **Two input modes**
  - **Replay file** – Upload an **.ashteki** replay (ZIP containing a `.replay` JSON file, as downloaded from Ashteki) or a raw **.json** (API response from `/api/game/:id/replay` or a single state object).
  - **Game chat log** – Paste or upload the plain-text game chat (e.g. copied from Ashteki). The parser reconstructs the timeline from:
    - “X brings Y to battle” (player and Phoenixborn)
    - “Round N”
    - “Player attacks … with K units”
    - “PhoenixbornName receives N damage”
- **Phoenixborn** – Shows which Phoenixborn each player used. **Starting life** for each Phoenixborn uses the standard card values (health = middle stat; unknown Phoenixborn default to 20).
- **One timeline, three views** (selector at top of chart):
  - **Dice advantage** – Unspent dice lead (JSON replay only; not available when using a chat log)
  - **Health remaining** – Both Phoenixborn’s health over the game (from replay snapshots or from damage lines in chat)
  - **Attacks** – When each player attacked and how many units they attacked with

## Replay format

The page accepts:

1. **API response** from `GET /api/game/:id/replay` on an Ashteki server (e.g. `https://ashteki.com/api/game/<gameId>/replay`):  
   `{ "success": true, "replay": [ { "gameId", "state", "tag", "username", "timeStamp" }, ... ] }`
2. **Single state object** – A game state object with `players`, `round`, and optionally `attack` (for attack snapshots).

Replays with multiple snapshots (e.g. `tag: "defenders-declared"` during attacks and `tag: "end"` at game end) give a full timeline. With only the final state you get one point per view. See [docs/replay-format.md](docs/replay-format.md) for the full replay structure.

## Hosting as a GitHub Page

1. Push this repo to GitHub (or add `index.html` and optional `docs/` to a repo).
2. In the repo: **Settings → Pages**.
3. Under “Build and deployment”, choose **Deploy from a branch**.
4. Select the branch (e.g. `main`) and folder **/ (root)** (so `index.html` is at the root of the site).
5. Save. The page will be at `https://<username>.github.io/<repo>/`.

No build step is required; the page is a single HTML file and loads Chart.js from a CDN.

## Local use

Open `index.html` in a browser (file protocol is fine). To use replays from ashteki.com you must first download the JSON (e.g. open `https://ashteki.com/api/game/<gameId>/replay` in the browser and save as `.json`), then load that file in the visualizer.

## License

Compatible with the Ashteki project (AGPL-3.0). See [Ashteki repository](https://github.com/Ashteki/ashteki) for game and replay format details.
