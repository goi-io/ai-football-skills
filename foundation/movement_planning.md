# SKILL: Movement Planning

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Foundation  
**Required By:** All agents (Offense & Defense)

## Purpose
Plan player movements and understand mobility constraints.

## Movement Fundamentals

### Movement Rate
- Players can move **one grid cell per tick** (in any direction)
- Players may stay in place
- Movement is constrained by grid boundaries

### Movement Directions
Players can move in cardinal and diagonal directions:
- **Cardinal:** North, South, East, West
- **Diagonal:** NE, NW, SE, SW
- **Stay:** Remain in current position

### Movement Constraints

#### Grid Boundaries
- Players cannot move outside the field boundaries
- The game state provides boundary information

#### Position-Based Constraints
- Certain positions (linemen) have restricted movement zones
- These players cannot advance beyond designated areas
- Agents receive constraint violations from the game engine

#### ⚠️ Lineman Movement Constraint (CRITICAL)

**Linemen cannot move above Y = 2 on the field.**

This applies to **all linemen** on both sides:
- **Offensive linemen:** `GL` (Guard Left), `GR` (Guard Right), `C_O` (Center Offense)
- **Defensive linemen:** `TL` (Tackle Left), `TR` (Tackle Right), `C_D` (Center Defense)

The constraint is checked against the **resulting absolute Y position**:
- `resulting_Y = formation_Y + sum(previous_tick_movements_Y) + submitted_move_Y`
- If `resulting_Y > 2`, the move submission is **rejected**

**Error Example:**
```json
{
  "ok": false,
  "error": "Lineman movement constraint violated. Linemen (GL, GR, C_O, TL, TR, C_D) cannot move above Y = 2."
}
```

**Safe Strategy for Linemen:**
- Track each lineman's current Y position from game state
- Before submitting a move with `dy = 1` (north), verify `current_Y + 1 <= 2`
- If a lineman is already at Y = 2, only submit `dy = 0` or `dy = -1`
- When in doubt, submit `[0, 0]` for linemen — staying in place is always valid

#### ⚠️ Neutralization Constraint (CRITICAL)

**Neutralized players MUST submit `[0, 0]` movement.**

When a player is neutralized (blocked by an opponent):
- They **cannot move** for the duration of neutralization
- You **MUST** submit `[0, 0]` for that player
- Submitting any other vector will **fail validation**

**Error Example:**
```json
{
  "ok": false,
  "error": "Neutralized players must have (0,0) movement. Invalid moves: WR1 (0,1)"
}
```

**Safe Strategy:**
- If unsure which players are neutralized, submit `[0, 0]` for all players
- This is always valid (staying in place is allowed for all players)
- Check `neutralizedData` in the game state for player neutralization status

---

## Turn Order for Tick Submission

**⚠️ CRITICAL: Offense submits FIRST, then Defense responds.**

| Your Side | When to Submit |
|-----------|----------------|
| Offense | Submit when `myTurn: true` |
| Defense | Wait until offense submits, then `myTurn` becomes `true` |

**Workflow:**
1. Check `/api/ai/{gameId}/state`
2. If `myTurn: false`, wait and poll again
3. If `myTurn: true` AND `next: "submit_moves"`, submit your moves
4. After you submit, poll `/state` for next tick

---

## AI API: Submit Moves

**Endpoint:** `POST /api/ai/{gameId}/moves`

**Request Body:** Position codes mapped to `[dx, dy]` direction vectors.
- Values: `-1` (left/down), `0` (stay), `1` (right/up)

**Offense Example:**
```json
{
  "QB": [0, 0],
  "RB": [0, 1],
  "WR1": [0, 1],
  "WR2": [0, 1],
  "C_O": [0, 0],
  "GL": [0, 0],
  "GR": [0, 0]
}
```

**Defense Example:**
```json
{
  "S": [0, -1],
  "LB": [0, -1],
  "TL": [0, 0],
  "TR": [0, 0],
  "C_D": [0, 0],
  "CB1": [0, -1],
  "CB2": [0, -1]
}
```

---

### Offensive Movement Goals
| Situation | Strategy |
|-----------|----------|
| Ball carrier (`WhoHasBall` = you) | Advance toward scoring zones / hotspots |
| Route running (pre-pass) | Create separation, reach targets |
| Blocking (when `WhoHasBall` = QB) | Position between defenders and QB (pass protection) |
| Blocking (when `WhoHasBall` = WR/RB) | Shift protection to the new ball carrier's path |

### Defensive Movement Goals
| Situation | Strategy |
|-----------|----------|
| Coverage (when `WhoHasBall` = QB) | Shadow receivers, anticipate throw |
| Coverage (when `WhoHasBall` = null) | Break toward pass target for contest/interception |
| Pursuit (when `WhoHasBall` = WR/RB) | Angle toward ball carrier for tackle |
| Pass rush (when `WhoHasBall` = QB) | Move toward QB or blocking lanes |

## Path Planning Concepts

### Reachability
- Determine if a player can reach a target position
- Consider time remaining and distance
- Account for movement constraints

### Optimal Paths
- Direct paths are fastest when unobstructed
- Diagonal movement can be efficient
- Obstacles may require alternate routing

### Pursuit Angles
- Defenders should cut off ball carrier paths
- Anticipate opponent's likely movement
- Position to intercept rather than chase

## Agent Decision Making

When planning movement, consider:
1. **Target position:** Where does this player need to go?
2. **Time available:** Can they reach it in time?
3. **Obstacles:** Are there blocking players in the way?
4. **Constraints:** Does this position have movement limits?
5. **Priority:** Is this move more important than alternatives?
