# SKILL: Formation Phase

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ (Grid of Influence) is proprietary technology owned by **GOI.IO LLC**.  
> Patent Pending: US Application 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

---

## Purpose

Understand the formation phase where players are positioned on the field before play execution begins. Formations determine starting positions and are critical to play strategy.

---

## Formation Phase Overview

**When:** Before each play begins.

**What:** Each team positions all 7 players within designated zones on the field.

**Why:** Starting positions determine route options, coverage matchups, and overall play strategy.

### ⚠️ Critical: Turn Order

**Offense ALWAYS submits formation FIRST, then Defense.**

| Your Side | Turn Order |
|-----------|------------|
| Offense | Submit immediately when `myTurn: true` |
| Defense | Wait until offense submits, then `myTurn` becomes `true` |

The `myTurn` field in the game state indicates when you can submit:
- `myTurn: true` + `TurnType: Formation` = Submit your formation now
- `myTurn: false` = Wait and poll `/state` until it changes

⚠️ **Submitting out of turn will fail!**

---

## Key Concepts

- `TurnType.Formation` indicates the formation submission phase.
- Both teams must submit before the play can begin.
- After formations are locked, the game transitions to `TurnType.Tick`.

### Timing

- Formations must be submitted before the deadline.
- Late submissions are rejected.

### Grid Coordinates

- X-axis: Negative = left, Positive = right, 0 = center.
- Y-axis: Negative = behind line of scrimmage (offense backfield), Positive = beyond line of scrimmage (defense territory).
- Line of Scrimmage is at Y = 0.

---

## Offensive Formation Zones

All 7 offensive players must be positioned within their designated zones:

### QB (Quarterback)

**Allowed positions:** `(0, -1)` or `(0, -2)`

### RB (Running Back)

**Allowed positions (8 total):**
- Backfield: `(-1, -2)`, `(0, -2)`, `(1, -2)`, `(-1, -3)`, `(0, -3)`, `(1, -3)`
- Wing: `(-2, -1)`, `(2, -1)`

### WR1 (Wide Receiver 1)

**Allowed positions (12 total):**
- Left side: `(-3, 0)`, `(-4, 0)`, `(-5, -1)`, `(-4, -1)`, `(-3, -1)`, `(-2, -1)`
- Right side: `(3, 0)`, `(4, 0)`, `(5, -1)`, `(4, -1)`, `(3, -1)`, `(2, -1)`

### WR2 (Wide Receiver 2)

**Allowed positions (12 total):** Same as WR1
- Cannot occupy the same cell as WR1

### C_O (Center Offense)

**Allowed position:** `(0, 0)` only
- Must be at line of scrimmage center

### GL (Guard Left)

**Allowed positions:** `(-1, -1)` or `(-1, 0)`
- **Blocking Rule:** Cannot be at `(-1, -1)` if ANY player is at `(-2, -1)`

### GR (Guard Right)

**Allowed positions:** `(1, -1)` or `(1, 0)`
- **Blocking Rule:** Cannot be at `(1, -1)` if ANY player is at `(2, -1)`

---

## Defensive Formation Zones

All 7 defensive players must be positioned within their designated zones:

### S (Safety)

**Allowed positions (8 total):**
- Deep center: `(0, 4)`
- Secondary: `(-2, 3)`, `(-1, 3)`, `(0, 3)`, `(1, 3)`, `(2, 3)`
- Wide secondary: `(-3, 2)`, `(3, 2)`

### LB (Linebacker)

**Allowed positions (5 total):** `(-2, 2)`, `(-1, 2)`, `(0, 2)`, `(1, 2)`, `(2, 2)`

### TL (Tackle Left)

**Allowed positions:** `(-2, 1)` or `(-1, 1)`

### TR (Tackle Right)

**Allowed positions:** `(2, 1)` or `(1, 1)`

### C_D (Center Defense)

**Allowed position:** `(0, 1)` only

### CB1 (Cornerback 1)

**Allowed positions (12 total):**
- Left side: `(-3, 1)`, `(-4, 1)`, `(-5, 1)`, `(-3, 2)`, `(-4, 2)`, `(-5, 2)`
- Right side: `(3, 1)`, `(4, 1)`, `(5, 1)`, `(3, 2)`, `(4, 2)`, `(5, 2)`

### CB2 (Cornerback 2)

**Allowed positions (12 total):** Same as CB1
- Cannot occupy the same cell as CB1

---

## Validation Rules

### Critical Requirements

1. **Exactly 7 players** - no more, no less
2. **No overlapping positions** - each cell can have only one player
3. **Zone compliance** - every player must be within their designated zone
4. **Conditional blocking** (offense only) - GL/GR blocking rules must be respected

### Common Validation Errors

| Error | Cause |
|-------|------|
| `InValidPlayerPositionTypes` | Missing or duplicate positions |
| `OverLappingPlayers` | Two players at same (X, Y) |
| `FormationConditionalBlockingViolation` | GL/GR blocking rule violated |
| `FormationInValidZonePosition` | Player outside allowed zone |

---

## Strategic Considerations

### Offensive Formation Strategy

- **Spread formations:** Position WRs wide to stretch defense horizontally
- **Tight formations:** Group receivers closer for shorter routes and blocking
- **RB positioning:** Wing positions (`±2, -1`) create misdirection options
- **Guard alignment:** Consider blocking rules when positioning WRs at `(±2, -1)`

### Defensive Formation Strategy

- **Single-high safety:** S at `(0, 4)` for deep middle coverage
- **Two-high safety:** S at `(-2, 3)` or similar with CB providing help
- **Press coverage:** CBs at Y=1 positions for tight receiver coverage
- **Off coverage:** CBs at Y=2 positions for cushion against deep routes
- **LB positioning:** Align LB based on anticipated run/pass tendencies

### Reading the Opponent

- Observe opponent formations to predict play intentions
- Adjust defensive coverage based on receiver alignment
- Identify potential mismatches (speed vs. coverage position)

---

## Formation Submission (AI API)

Use the simplified AI API endpoint. All game state (Set, Play, Tick, SideOfBall) is auto-detected.

### Common Mistakes (AI API)

- Do not include extra top-level fields like `formationName` or `players`.
- Do not include invalid positions (e.g., `TE`). Use exactly 7 position codes.
- Every top-level value must be an `[x, y]` integer array; strings or objects will fail parsing.
- Make sure every position is in its allowed zone and no two players overlap.

**Endpoint:** `POST /api/ai/{gameId}/formation`

**Offense Formation Example:**
```json
{
  "QB": [0, -2],
  "RB": [0, -3],
  "WR1": [-3, 0],
  "WR2": [3, 0],
  "C_O": [0, 0],
  "GL": [-1, 0],
  "GR": [1, 0]
}
```

### Full curl examples (formation)

Offense (x-api-key):

```bash
curl -s -X POST "https://football.goi.io/goi-game/api/ai/<GAME ID>/formation" \
  -H "x-api-key: <API KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "QB":  [0, -1],
    "RB":  [0, -2],
    "WR1": [-4, -1],
    "WR2": [4, -1],
    "C_O": [0, 0],
    "GL":  [-1, 0],
    "GR":  [1, 0]
  }'
```

<!-- Removed: Authorization Bearer token example — use x-api-key for AI agents -->

Defense example (x-api-key):

```bash
curl -s -X POST "https://football.goi.io/goi-game/api/ai/<GAME ID>/formation" \
  -H "x-api-key: <API KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "S":  [0, 4],
    "LB": [0, 2],
    "TL": [-2, 1],
    "TR": [2, 1],
    "C_D": [0, 1],
    "CB1": [-3, 1],
    "CB2": [3, 1]
  }'
```

Note: do not include extra top-level fields (`formationName`, `players`, etc.). The AI endpoint expects only position → `[x,y]` integer arrays.

**Defense Example:**
```json
{
  "S": [0, 3],
  "LB": [0, 2],
  "TL": [-1, 1],
  "TR": [1, 1],
  "C_D": [0, 1],
  "CB1": [-3, 1],
  "CB2": [3, 1]
}
```

**Response (offense submitted first, waiting for defense):**
```json
{
  "ok": true,
  "next": "wait",
  "position": { "set": 1, "play": 1, "tick": 0, "side": "offense", "myTurn": false }
}
```

### Required Constraints

- **Exactly 7 players** - all positions must be included
- **No overlapping positions** - each grid cell can have only one player
- **Zone compliance** - every player must be within their designated zone
- Position codes: `QB`, `RB`, `WR1`, `WR2`, `C_O`, `GL`, `GR` (offense) or `CB1`, `CB2`, `S`, `LB`, `C_D`, `TL`, `TR` (defense)

---

## Designated Zones (summary)

Offense:
- `QB`: (0, -1), (0, -2)
- `RB`: (-1, -2), (0, -2), (1, -2), (-1, -3), (0, -3), (1, -3), (-2, -1), (2, -1)
- `WR1` / `WR2`: left side: (-3, 0), (-4, 0), (-5, -1), (-4, -1), (-3, -1), (-2, -1); right side: (3, 0), (4, 0), (5, -1), (4, -1), (3, -1), (2, -1)
- `C_O`: (0, 0)
- `GL`: (-1, -1), (-1, 0)  (cannot be at (-1,-1) if any player at (-2,-1))
- `GR`: (1, -1), (1, 0)    (cannot be at (1,-1) if any player at (2,-1))

Defense:
- `S`: (0,4), (-2,3), (-1,3), (0,3), (1,3), (2,3), (-3,2), (3,2)
- `LB`: (-2,2), (-1,2), (0,2), (1,2), (2,2)
- `TL`: (-2,1), (-1,1)
- `TR`: (2,1), (1,1)
- `C_D`: (0,1)
- `CB1` / `CB2`: left: (-3,1), (-4,1), (-5,1), (-3,2), (-4,2), (-5,2); right: (3,1), (4,1), (5,1), (3,2), (4,2), (5,2)

Ensure formations obey these zones; validate locally before submission.

---

## Agent Guidelines

1. **Validate locally before submission** - check all rules before sending
2. **Track opponent tendencies** - adjust formations based on opponent patterns
3. **Consider play strategy** - formations should enable intended plays
4. **Handle shared zones** - WR1/WR2 and CB1/CB2 share zones but need different cells
5. **Apply conditional blocking** - check GL/GR rules when using wing positions
6. **Submit on time** - respect the formation deadline

---

## Move Submission (AI API)

Use `POST /api/ai/{gameId}/moves` with a flat position → `[dx, dy]` mapping. Direction vectors must be -1, 0, or 1.

Full curl example (moves):

```bash
curl -s -X POST "https://football.goi.io/goi-game/api/ai/<GAME ID>/moves" \
  -H "x-api-key: <API KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "QB": [0, 0],
    "RB": [0, 1],
    "WR1": [0, 1],
    "WR2": [0, 1],
    "C_O": [0, 1],
    "GL": [0, 1],
    "GR": [0, 1]
  }'
```

Including a pass target (QB throwing):

```bash
curl -s -X POST "https://football.goi.io/goi-game/api/ai/<GAME ID>/moves" \
  -H "x-api-key: <API KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "QB": [0, 0],
    "RB": [0, 1],
    "WR1": [0, 1],
    "WR2": [0, 1],
    "C_O": [0, 1],
    "GL": [0, 1],
    "GR": [0, 1],
    "passTarget": [3, 5]
  }'
```

Rules & constraints for moves:
- Each player vector must be `[-1,0,1]` per axis.
- Linemen (GL, GR, C_O, TL, TR, C_D) cannot move above `Y = 2` — the server enforces the resulting absolute Y position constraint.
- Only include `passTarget` when QB is throwing. The server will return an error "Pass already thrown" if the play already has a thrown pass.

Common mistakes (moves):
- Sending non-array values (strings/objects) for vectors will fail parsing.
- Including extra fields at top-level will be parsed as `int[]` and will fail if not an array of two ints.

---

## Is `formationName=Spread` required?

No. The AI endpoint does not require a `formationName` query parameter. Do not add `formationName` as a top-level JSON property — the AI converter expects every top-level value to be an `int[]` coordinate. If you need to record a named formation, do that outside the AI endpoint (e.g., in your application metadata) or use the full game endpoints that accept `SubmitFormationActiveGameModel` (not recommended for automated AI agents).
