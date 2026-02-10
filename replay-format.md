# Ashteki Replay File Format

This document describes how replay data is stored and served by [Ashteki](https://github.com/Ashteki/ashteki) so that external tools (e.g. replay visualizers) can parse it.

## Downloaded replay files (.ashteki)

When you download a replay from Ashteki, you get an **.ashteki** file. This is a **ZIP archive** containing a single entry named `<gameId>.replay` (e.g. `0c986ec0-04ee-11f1-bf53-2f3928d3137c.replay`). The contents of that file are JSON in the same shape as the API response: `{"success":true,"replay":[{...},{...},...]}`. So to use it: unzip the .ashteki, read the .replay file as JSON, then use the `replay` array as described below.

## How replays are saved (server)

- When a game ends, the server saves a **replay state** via `ReplayService.save(username, state, tag)`.
- Replays are also saved at **defenders-declared** during attacks (when the "save replay" option is enabled for the game).
- Stored in MongoDB in a `replays` collection. Each document has:
  - `gameId` – game UUID
  - `username` – user who triggered the save (e.g. game owner)
  - `timeStamp` – when it was saved
  - `tag` – `'end'` (game over) or `'defenders-declared'` (mid-game attack)
  - `state` – full game state (see below)

## API

- **GET** `/api/game/:id/replay`  
  Returns: `{ success: true, replay: replay }`  
  `replay` is an **array** of replay documents (all replays for that game).  
  No auth required for this endpoint.

## Replay document shape

Each element of the `replay` array looks like:

```json
{
  "gameId": "uuid",
  "username": "string",
  "timeStamp": "ISO date",
  "tag": "end" | "defenders-declared",
  "state": { ... }
}
```

## State object (from GameStateWriter)

The `state` object is the same structure sent to the client for game/replay view:

| Field          | Type     | Description |
|----------------|----------|-------------|
| `id`           | string   | Game UUID |
| `round`        | number   | Current round (1-based) |
| `currentPhase` | string   | e.g. `'main'`, `'recovery'` |
| `players`      | object   | `{ "playerName": PlayerState }` |
| `cardLog`      | array    | `[{ type, obj: { id, name, type }, p: playerName }]` |
| `messages`     | array    | Game chat/alert messages |
| `attack`       | object \| null | Present during attack; see Attack state |
| `winner`       | string   | Winner’s player name (when game is over) |
| `started`      | boolean  | |
| `finishedAt`   | ISO date | When the game ended |

### Player state (`state.players[name]`)

| Field         | Type   | Description |
|---------------|--------|-------------|
| `name`        | string | Username |
| `id`          | number | Socket/player id (used in `attack.attackingPlayer`) |
| `phoenixborn` | object | Phoenixborn card: `name`, `life`, `damage`, `id`, `type`, … |
| `dice`        | array  | Dice objects: `{ uuid, magic, level, location, exhausted }` |
| `diceCounts`  | object | Dice counts (if present) |
| `stats`       | object | From `player.getStats()` |

- **Health**: Phoenixborn health = `phoenixborn.life - phoenixborn.damage` (and optionally `drowningLevel`).
- **Unspent dice**: Count dice in `dice` where `exhausted === false` (and location is the player’s pool).

### Attack state (`state.attack`)

Only set during an attack (e.g. in `defenders-declared` snapshots):

| Field             | Type   | Description |
|-------------------|--------|-------------|
| `attackingPlayer` | number | Player **id** (match to `players[name].id`) |
| `isPBAttack`      | boolean| Direct Phoenixborn attack |
| `battles`         | array  | One entry per attacker: `{ attacker, target, guard }` (uuids) |

- **Number of attackers** = `state.attack.battles.length`.
- **Attacker’s name**: find the player with `players[name].id === state.attack.attackingPlayer`.

## Multiple snapshots

- **Tag `end`**: One replay document with the final game state (round, health, dice, winner).
- **Tag `defenders-declared`**: One document per attack (when save replay is on), giving a snapshot at that moment (round, health, dice, and `attack` with attacker id and number of battles).

Sorting replay documents by `state.round` and then `timeStamp` gives a time-ordered list of snapshots for building turn-by-turn or attack-by-attack visualizations (dice advantage, health over time, attack timeline).

## References

- Replay save: `server/gamerouter.js` (`REPLAY_STATE`), `server/services/AshesReplayService.js`
- State shape: `server/gamenode/GameStateWriter.js`, `PlayerStateWriter.js`, `CardStateWriter.js`, `DieStateWriter.js`
- Attack summary: `server/game/gamesteps/AttackFlow.js`, `AttackState.js`
- Game log: `server/game/game.js` (`gameLog`, `cardLog` in state)
