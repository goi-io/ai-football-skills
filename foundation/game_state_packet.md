# SKILL: Game State Packet

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ (Grid of Influence) is proprietary technology owned by **GOI.IO LLC**.  
> Patent Pending: US Application 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

---

## Purpose

Understand the GameStatePacket structure—the single source of truth for all game information. AI agents receive this packet each tick to make informed decisions.

---

## Game State Overview

The GameStatePacket contains everything an agent needs to understand the current game situation:
- Current position in the game (set, play, tick)
- All player positions and states
- Ball location and possession
- Pass information and results
- Scoring data
- Turn information

---

## Game Time Structure

### Sets and Plays
```
Set 1, Play 1 = Play 1 of game
Set 1, Play 2 = Play 2 of game
Set 2, Play 1 = Play 3 of game
Set 2, Play 2 = Play 4 of game
```

### Ticks
- Each play consists of **8 ticks** maximum
- Tick count starts at 0 or 1 (check current index)
- Play ends early if: tackle, blocked pass, time expired, score

---

## Core State Index

The `CurrentIndex` tells you exactly where you are in the game:

| Field | Description |
|-------|-------------|
| `setCount` | Current set (1-2) |
| `playCount` | Current play within set (1-2) |
| `tickCount` | Current tick within play (max 8) |

---

## Turn Information

### ⚠️ Critical: Turn Order

**Offense ALWAYS submits first, then Defense** - for BOTH formation AND tick phases.

This is enforced by the server. If you try to submit when `isMyTurn` is `false`, you'll get an error.

| Field | Description |
|-------|-------------|
| `turnType` | `Formation` or `Tick` - current phase |
| `isUserTurn` | **Check this FIRST** - whether it's your turn to submit |
| `userTeamType` | `Home` or `Away` - your team |
| `userSideOfBall` | `Offense` or `Defense` - your current role |
| `TeamOnOffense` | Which team is currently on offense |
| `TeamOnDefense` | Which team is currently on defense |

### Turn Order Workflow
```
1. Check isUserTurn
   - If false: WAIT (poll state until true)
   - If true: Continue to step 2

2. Check userSideOfBall
   - "Offense": You submit FIRST (before defense)
   - "Defense": You submit SECOND (after offense)

3. Check turnType
   - "Formation": Submit formation via /formation endpoint
   - "Tick": Submit moves via /moves endpoint
```

---

## Play Data

### Accessing Plays
| Method | Description |
|--------|-------------|
| `homePlays` | All plays for home team |
| `awayPlays` | All plays for away team |
| `getOffensePlay()` | Current play for offensive team |
| `getDefensePlay()` | Current play for defensive team |

### PlayModel Structure
| Field | Description |
|-------|-------------|
| `SetCount` | Set number for this play |
| `PlayCount` | Play number within set |
| `TickCount` | Current tick of play |
| `Players` | Array of PlayerPacketModel |
| `TeamType` | Home or Away |
| `PassTargets` | Array of pass target information |

---

## Player Information

### PlayerPacketModel
| Field | Description |
|-------|-------------|
| `Pos` | Position type (QB, RB, WR1, etc.) |
| `Path` | Array of Vector2 movements for entire play |
| `FormationPoint` | Starting position (formation) |
| `getLocationAtTickCount(tick)` | Get player position at specific tick |
| `NeutralizedData` | Array of neutralization states |

### NeutralizedState
| Field | Description |
|-------|-------------|
| `IsNeutralized` | Whether player is currently neutralized |
| `NeutralizedBy` | Position type that caused neutralization |
| `Tick` | Tick when neutralization occurred |
| `NeutralizedDuration` | How long player remains neutralized |

---

## Ball Carrier — `WhoHasBall` (Critical Concept)

`WhoHasBall` identifies **which position currently possesses the ball**. It is one of the most important fields in the game state because it drives decisions for **both** sides of the ball.

### Where to Find It

| Source | Field | Example Value |
|--------|-------|---------------|
| **Moves/Formation response** (`position` object) | `whoHasBall` | `"QB"`, `"WR1"`, `"RB"`, `null` |
| **Moves/Formation response** (`ball` object) | `ball.x` / `ball.y` | `0`, `-3` |
| **AI State** (`/api/ai/{gameId}/state`) | `position.whoHasBall` | `"QB"`, `"WR1"`, `"RB"`, `null` |
| **AI State** (`/api/ai/{gameId}/state`) | `ball.carrier` | Same as above (also in the `ball` object) |
| PlayTransactionModel (per-tick) | `WhoHasBall` | Position type enum (`QB`, `RB`, `WR1`, `WR2`, `C_O`) |

> **⚠️ `whoHasBall` is returned in EVERY response** — formation submissions, move submissions, and state queries. You never need a separate call to find the ball carrier.

### Ball Carrier Lifecycle

| Game Phase | Typical `WhoHasBall` | Notes |
|------------|---------------------|-------|
| Pre-snap / Tick 0 | `C_O` | Center holds the ball at the start of each play |
| Post-snap (ticks 1+) | `QB` | Ball snapped to quarterback |
| After handoff | `RB` | If QB hands off to running back |
| After completed pass | `WR1`, `WR2`, or `RB` | Receiver catches ball |
| Ball in flight | `null` | No one has the ball — it is airborne |
| After interception | *(defense player)* | Defender who intercepted now has ball |

### Why `WhoHasBall` Matters

**For Offense:**
- **Blocking assignments pivot on the ball carrier.** O-Line should position between defenders and whoever has the ball — not just the QB.
- After a completed pass, `WhoHasBall` changes from `QB` to the receiver. Blockers must immediately shift protection to the new carrier.
- The ball carrier is **excluded from neutralization collisions** — collisions with the carrier go to the tackle stage instead.
- Offensive teammates sharing the ball carrier's cell for 2+ consecutive ticks trigger a **stacking/riding penalty** (3-tick neutralization).

**For Defense (MOST CRITICAL):**
- **`WhoHasBall` tells you who to tackle.** Every defensive move decision should start with: *where is the ball carrier, and how do I get to them?*
- When `WhoHasBall` is `QB` or `C_O`, the ball has not been thrown or handed off — **rush the QB and block passing lanes.**
- When `WhoHasBall` changes to `WR1`, `WR2`, or `RB`, a pass was completed or handoff occurred — **all pursuit must redirect to the new carrier.**
- When `WhoHasBall` is `null`, the ball is in flight — **break toward the pass target location** for an interception or to contest the catch.
- Tackling the ball carrier ends the play. Neutralizing a non-carrier does not.

### Reading `WhoHasBall` Each Tick

`whoHasBall` is available directly in the `position` object of every moves/formation response:

```python
# After submitting moves, read whoHasBall from the response
result = submit_moves(game_id, moves)
carrier = result["position"]["whoHasBall"]  # e.g., "QB", "WR1", None

# Or from the state endpoint
state = get_game_state(game_id)
carrier = state["position"]["whoHasBall"]   # same field
# Also available at: state["ball"]["carrier"]

if carrier is None:
    # Ball is in flight — break toward pass target for interception
    pass_target = state["ball"]  # check ball x,y for flight location
elif carrier in ["WR1", "WR2", "RB"]:
    # Pass completed or handoff — pursue the new ball carrier
    carrier_pos = find_player_position(state, carrier)
elif carrier in ["QB", "C_O"]:
    # Ball still with QB/Center — rush QB, block passing lanes
    qb_pos = find_player_position(state, "QB")
```

---

## Ball and Pass Information

### PlayTransactionModel
| Field | Description |
|-------|-------------|
| `SetCount` | Set of transaction |
| `PlayCount` | Play of transaction |
| `TickCount` | Tick of transaction |
| `BallLocation` | Current ball position (Vector2) |
| `WhoHasBall` | Position type holding ball (see **Ball Carrier** section above) |
| `PassTargetLocation` | Where QB threw the ball |
| `PassTargetTick` | Tick when pass was thrown |
| `PassTargetingResult` | Result of pass attempt |
| `TackledStatus` | Tackle information if applicable |
| `EndOfPlay` | Whether play has ended |
| `EndOfPlayReasonTypes` | How play ended |
| `ScoreResults` | Scoring data if points scored |

### PassTargetingResultTypes
- `LiveBallExpired` - Ball expired without catch
- `InterceptedActive` - Defender caught ball, still active
- `InterceptedDead` - Defender caught ball, play ended
- `Completed` - Receiver caught ball
- `Incomplete` - Pass not caught
- `Blocked` - Pass blocked by defender

### EndOfPlayReasonTypes
- `Tackle` - Ball carrier was tackled
- `BlockedPass` - Pass was blocked
- `TimeExpired` - All 8 ticks used
- `Score` - Points were scored
- `Turnover` - Possession changed

---

## Tackle Information

### TackledStatus
| Field | Description |
|-------|-------------|
| `TackledBy` | Position type that made tackle |
| `TackledLocation` | Where tackle occurred (Vector2) |
| `TackledTick` | Tick when tackle was made |

---

## Scoring Data

### GameScoringData
| Field | Description |
|-------|-------------|
| `pointsScored` | Points scored (can be negative) |
| `sideOfBall` | Offense or Defense |
| `teamType` | Home or Away |
| `teamName` | Team name |
| `player` | Position type that scored |
| `playerId` | Player ID |
| `playerName` | Player name |

---

## Spectator View State

### UpDateSpectatorView
| Field | Description |
|-------|-------------|
| `UpdateView` | Whether to refresh display |
| `ShowDefense` | Whether to show defensive positions |
| `ShowOffense` | Whether to show offensive positions |
| `ShowPlayersInfo` | Whether to show player details |

---

## Using Game State for Decisions

### Pre-Play (Formation Phase)
```
if (gameState.turnType === 'Formation') {
  // Submit formation based on strategy
  // Consider opponent tendencies
  // Check userSideOfBall for offense/defense
}
```

### During Play (Tick Phase)
```
if (gameState.turnType === 'Tick') {
  // Get current positions
  let offensePlay = gameState.getOffensePlay();
  let defensePlay = gameState.getDefensePlay();
  
  // Check ball state
  let transaction = gameState.PlayTransactions[latest];
  let ballLocation = transaction.BallLocation;
  let ballCarrier = transaction.WhoHasBall;
  
  // Check for neutralized players
  for (player of play.Players) {
    if (player.NeutralizedData.some(n => n.IsNeutralized)) {
      // Player cannot move
    }
  }
  
  // Make move decisions
}
```

### Reading Player Positions
```
// Get player location at current tick
let currentTick = gameState.CurrentIndex.tickCount;
let player = play.Players.find(p => p.Pos === 'WR1');
let location = player.getLocationAtTickCount(currentTick);

// Get formation (starting) position
let startPosition = player.FormationPoint;

// Get full path for play
let fullPath = player.Path;
```

---

## Agent REST API (Lightweight Alternative)

For agents with limited context windows, use the **Agent REST API** instead of full MCP tools. It returns the same data in a much more compact format (~300-500 tokens vs ~4000-6000).

### How to Call the Agent API

**Base URL:** `https://football.goi.io`

**Authentication:** Include one of these headers:
- `X-API-KEY: your_api_key_here`
- `Authorization: Bearer your_jwt_token`

**Example: Get Turn Status**
```bash
curl -X GET "https://football.goi.io/api/agent/v1/game/627/turn" \
  -H "X-API-KEY: goi_your_api_key_here"
```

**Example: Get Game State with Players and Ball**
```bash
curl -X GET "https://football.goi.io/api/agent/v1/game/627/state?include=players,ball" \
  -H "X-API-KEY: goi_your_api_key_here"
```

**Example: Get Full State**
```bash
curl -X GET "https://football.goi.io/api/agent/v1/game/627/state?include=all" \
  -H "X-API-KEY: goi_your_api_key_here"
```

### Agent API Endpoints

| Endpoint | Purpose | ~Tokens |
|----------|---------|---------|
| `GET /api/agent/v1/game/{gameId}/state` | Full optimized state | 300-500 |
| `GET /api/agent/v1/game/{gameId}/turn` | Turn info only | ~50 |
| `GET /api/agent/v1/game/{gameId}/players` | Player positions | ~200 |
| `GET /api/agent/v1/game/{gameId}/ball` | Ball state | ~30 |

### Compact Response Format

**Agent API Turn Info:**
```json
{
  "set": 1, "play": 2, "tick": 3,
  "type": "tick", "action": "submit_moves",
  "isMyTurn": true, "mySide": "offense"
}
```

> **⚠️ IMPORTANT:** Always check `isMyTurn` before submitting. If `false`, poll until `true`.
> Offense submits first, then defense. Submitting out of turn will fail.
```

**Agent API Player:**
```json
{ "pos": "QB", "x": 5, "y": 2 }
{ "pos": "RB", "x": 4, "y": 1, "n": true, "nEnd": 4 }  // neutralized
```

**Field Selection:**
Use `?include=` to customize `/state` response:
- `players` (default) - Player positions
- `ball` (default) - Ball state
- `targets` - QB target locations
- `lastplay` - Previous play result
- `all` - Everything

---

## Agent Guidelines

1. **⚠️ Check isMyTurn FIRST** - never submit when `false`; offense goes first
2. **Always check turnType** before deciding action type
3. **⚠️ Check `position.whoHasBall` EVERY TICK** — this is the single most important tactical field (`null` means the ball is in flight):
   - **Offense:** Shift blocking to protect whoever has the ball; ball carrier should advance toward hotspots
   - **Defense:** Pursue the ball carrier for a tackle; when `carrier` is `null`, ball is airborne — contest the catch
4. **Monitor neutralized players** - they cannot move; submit `[0,0]` for them
5. **Use getLocationAtTickCount()** for precise positioning
6. **Check PassTargetingResult** to understand pass outcomes
7. **Review EndOfPlayReasonTypes** to learn from completed plays
8. **Track scoring patterns** via GameScoringData for strategy
9. **Use Agent REST API** (`/api/agent/v1/`) for minimal context consumption
