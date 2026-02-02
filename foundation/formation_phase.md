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

**When:** Before each play begins, both teams submit formations simultaneously.

**What:** Each team positions all 7 players within designated zones on the field.

**Why:** Starting positions determine route options, coverage matchups, and overall play strategy.

---

## Key Concepts

### Turn Type
- `TurnType.Formation` indicates the formation submission phase
- Both offense and defense submit formations before play execution begins
- After formations are locked, the game transitions to `TurnType.Tick`

### Timing
- Formations must be submitted before the deadline
- Late submissions are rejected
- Both teams must submit before play can begin

### Grid Coordinates
- X-axis: Negative = left, Positive = right, 0 = center
- Y-axis: Negative = behind line of scrimmage (offense backfield), Positive = beyond line of scrimmage (defense territory)
- Line of Scrimmage is at Y = 0

---

## Offensive Formation Zones

All 7 offensive players must be positioned within their designated zones:

### QB (Quarterback)
**Allowed positions:** `(0, -1)` or `(0, -2)`
- Behind center, primary ball handler

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
|-------|-------|
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

## Formation Submission Format

> **Important:** The API expects **PascalCase** property names and a wrapped `Formation` object.
> Lower-case fields or `playerLocations` will fail validation.

```json
{
  "GameId": <game_id>,
  "Sob": "Offense" | "Defense",
  "Formation": {
    "Set": <set_number>,
    "Play": <play_number>,
    "TeamId": <team_id>,
    "UserId": <user_id>,
    "Sob": "Offense" | "Defense",
    "Players": [
      { "PlayerPosition": "QB", "X": 0, "Y": -1 },
      { "PlayerPosition": "RB", "X": 1, "Y": -2 },
      { "PlayerPosition": "WR1", "X": -4, "Y": 0 },
      { "PlayerPosition": "WR2", "X": 4, "Y": 0 },
      { "PlayerPosition": "C_O", "X": 0, "Y": 0 },
      { "PlayerPosition": "GL", "X": -1, "Y": 0 },
      { "PlayerPosition": "GR", "X": 1, "Y": 0 }
    ]
  }
}
```

### Required Fields & Constraints
- **Exactly 7 players** in `Formation.Players`
- **No duplicate `PlayerPosition` values**
- **No overlapping (X, Y) grid cells**
- `PlayerPosition` must match: `QB`, `RB`, `WR1`, `WR2`, `C_O`, `GL`, `GR` (offense) or `CB1`, `CB2`, `S`, `LB`, `C_D`, `TL`, `TR` (defense)

---

## Agent Guidelines

1. **Validate locally before submission** - check all rules before sending
2. **Track opponent tendencies** - adjust formations based on opponent patterns
3. **Consider play strategy** - formations should enable intended plays
4. **Handle shared zones** - WR1/WR2 and CB1/CB2 share zones but need different cells
5. **Apply conditional blocking** - check GL/GR rules when using wing positions
6. **Submit on time** - respect the formation deadline
