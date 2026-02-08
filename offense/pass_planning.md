# SKILL: Pass Planning

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Offense  
**Required By:** Offensive Coach Agent

---

## Purpose

Determine optimal pass targets and timing strategy for the quarterback. This document covers the mechanics of `passTarget` submission, eligible receivers, pass types, and strategic timing — everything an AI agent needs to execute passes correctly.

---

## Pass Targeting API (How to Throw)

Passes are thrown by including an optional `passTarget` field in the moves submission body during a tick.

**Endpoint:** `POST /api/ai/{gameId}/moves`

**Key rules:**
- `passTarget` is an **absolute field coordinate** `[x, y]` — NOT a direction vector.
- `passTarget` is included **alongside** the normal 7 position move vectors.
- You may only throw **once per play**. A second `passTarget` in a later tick returns an error.
- Omit `passTarget` entirely on ticks where the QB is not throwing.

### Request Body (with pass)

```json
{
  "QB": [0, 0],
  "RB": [0, 1],
  "WR1": [0, 1],
  "WR2": [0, 1],
  "GL": [0, 1],
  "GR": [0, 1],
  "C_O": [0, 0],
  "passTarget": [3, 5]
}
```

- `"QB": [0, 0]` — QB holds position while throwing.
- `"passTarget": [3, 5]` — Absolute field coordinates where the ball is thrown.

### Request Body (no pass)

```json
{
  "QB": [0, -1],
  "RB": [0, 1],
  "WR1": [1, 1],
  "WR2": [-1, 1],
  "GL": [0, 1],
  "GR": [0, 1],
  "C_O": [0, 0]
}
```

Simply omit the `passTarget` key when not throwing.

### Validation & Errors

| Error | Cause |
|-------|-------|
| `passTarget must have [x, y] coordinates. Got: ...` | `passTarget` is missing a coordinate or has wrong length |
| `PassAlreadyThrown` | A pass was already thrown earlier in this play |

### curl Example (Offense, with pass)

```bash
curl -s -X POST "https://football.goi.io/goi-game/api/ai/<GAME_ID>/moves" \
  -H "x-api-key: <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "QB": [0, 0],
    "RB": [0, 1],
    "WR1": [0, 1],
    "WR2": [0, 1],
    "GL": [0, 1],
    "GR": [0, 1],
    "C_O": [0, 0],
    "passTarget": [3, 5]
  }'
```

---

## Core Concepts

### Eligible Pass Receivers

Only these positions can be targeted by a pass:
- **WR1** — Wide Receiver 1 (primary deep threat)
- **WR2** — Wide Receiver 2 (secondary receiver)
- **RB** — Running Back (short routes, screens)

The `passTarget` coordinate should be the cell where one of these receivers will be (or will arrive).

### Pass Types

#### Direct Pass
- Target cell is **occupied** by a friendly receiver when the ball arrives.
- Ball goes directly to receiver — lower risk, higher completion rate.

#### Live Pass
- Target cell is **empty** when the throw is made.
- Ball arrives at the cell and can be contested by nearby players.
- Higher risk, but can create opportunities if receiver is en route.

### QB Pass-Target Pattern
- QB has a defined pattern of cells where passes can be thrown.
- Pattern position is relative to QB's current location.
- As QB moves, the absolute target positions shift accordingly.
- Agents receive pattern information in game state.

---

## Reception Advantage Rule (Critical)

> **Hands attribute does NOT affect receptions.**

Reception chance increases **only** when defenders are **neutralized** at or near:
- **Live ball targets** (empty cell throws), or
- **Direct pass targets** (receiver already on target cell)

**Strategy:** Use blockers (GL, GR, C_O) to neutralize defenders in or near the target lane just before the ball arrives. This is the primary way to improve completion probability.

---

## Pass Timing Guidelines

### Route Development Phases

| Phase | Timing | Route Types |
|-------|--------|-------------|
| Quick | Ticks 1–2 | Screens, flats, quick outs |
| Medium | Ticks 3–5 | Slants, curls, crossing |
| Deep | Ticks 5–7+ | Posts, streaks, corners |
| Desperation | Final ticks | Any open receiver |

### When to Throw

**Consider throwing when:**
- Receiver reaches (or is about to reach) the target cell with separation from defenders.
- Pressure is increasing on QB (defenders converging).
- Best throwing window may be closing.

**Consider waiting when:**
- Better options are developing downfield.
- Current interception risk is too high.
- QB has time and protection from blockers.

---

## Interception Risk Assessment

**Higher risk when:**
- Defenders are closer to target than receivers.
- Multiple defenders near target area.
- Pass is "live" (empty target cell).

**Lower risk when:**
- Receiver has clear positional advantage on target cell.
- Direct pass to receiver already at target.
- Defenders in the lane are neutralized.
- Throwing lanes are clear of unblocked defenders.

### Throwing Lane Awareness
- Defenders positioned between QB and target can disrupt passes.
- Consider alternate targets if primary lane is blocked.
- QB movement (`[1, 0]`, `[-1, 0]`, etc.) can open new lanes.

---

## Strategic Considerations

### Risk Tolerance
- **Conservative:** Prioritize low-risk, shorter throws (RB screens, flat routes).
- **Balanced:** Trade off risk vs. reward based on game score and field position.
- **Aggressive:** Accept higher risk for bigger yardage (deep WR routes, live passes).

### Situational Adjustments
- **Behind in score:** May need to take more risks with deep passes.
- **Ahead in score:** Protect the ball, prefer safer Direct Passes.
- **Time pressure (late play ticks):** Must execute quickly — short routes only.

---

## Common Mistakes (Pass Targeting)

| Mistake | Error / Result | Solution |
|---------|----------------|----------|
| Using direction vector for `passTarget` | Pass lands at wrong cell | `passTarget` is **absolute** `[x, y]`, not `[dx, dy]` |
| Sending `passTarget` on multiple ticks | `PassAlreadyThrown` error | Only include `passTarget` once per play |
| Targeting a cell no receiver can reach | Ball arrives but no one catches it | Coordinate throw timing with receiver routes |
| Forgetting to omit `passTarget` on non-throw ticks | Unintended pass attempt | Only include the key when actually throwing |
| Throwing while QB is under pressure | High interception risk | Check defender positions relative to QB |

---

## Pass Decision Pseudocode

```python
def decide_pass(state, moves):
    """Decide whether to include passTarget this tick."""
    
    # Don't throw if pass already made this play
    if state.get("passAlreadyThrown"):
        return moves  # no passTarget
    
    # Find receiver positions and defender positions
    receivers = get_receiver_positions(state)  # WR1, WR2, RB
    defenders = get_defender_positions(state)
    qb_pos = get_player_position(state, "QB")
    
    for receiver_name, receiver_pos in receivers.items():
        # Check if receiver has separation from defenders
        closest_defender_dist = min_distance(receiver_pos, defenders)
        if closest_defender_dist >= 2:
            # Good separation — throw to receiver's current or projected cell
            moves["passTarget"] = [receiver_pos["x"], receiver_pos["y"]]
            moves["QB"] = [0, 0]  # Hold position while throwing
            return moves
    
    return moves  # No good target — keep developing routes
```

---

## See Also

- [foundation/moves_submission.md](../foundation/moves_submission.md) — Full API specification for move/pass submission.
- [offense/route_running.md](route_running.md) — Route concepts and receiver separation strategy.
- [foundation/movement_planning.md](../foundation/movement_planning.md) — Movement vectors and constraints.
