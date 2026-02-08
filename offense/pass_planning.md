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
- The pattern is derived from the QB's player attributes.
- Pattern position is **relative to QB's current location** — it defines offsets, not absolute cells.
- As QB moves, the absolute target positions **shift with the QB**. The engine recalculates: `absolute_target = base_pattern_offset + qb_current_position`.
- Agents receive the computed absolute target points in the game state (via the QB's `QbTargetPattern` in the AI state endpoint).
- `passTarget` submitted in the moves request must be one of these absolute target points (or very near one).

---

## Move-and-Pass: Throwing While the QB Moves (Critical)

The QB can **move and pass in the same tick** by submitting both a non-zero QB move vector and a `passTarget` in the same request. However, because the target pattern moves with the QB, the agent **must recalculate targets based on where the QB will be after the move** — not where the QB currently is.

### How It Works

1. The QB's base target pattern is a fixed set of **relative offsets** derived from the QB's attributes (e.g., `[(0,2), (1,2), (-1,2), (0,3), ...]`).
2. These offsets are anchored to the QB's position. When the game engine processes a tick, it applies the QB's move first, then evaluates the pass from the **new** position.
3. Therefore: `valid_targets = [offset + new_qb_position for offset in base_pattern]`

### Recalculation Formula

```
new_qb_position = current_qb_position + qb_move_vector
valid_targets   = [base_offset + new_qb_position  for base_offset in qb_base_pattern]
```

Where:
- `current_qb_position` — QB's absolute `[x, y]` from the game state at the start of this tick.
- `qb_move_vector` — the `[dx, dy]` direction vector you are submitting for QB (e.g., `[1, 0]`).
- `qb_base_pattern` — the set of relative offsets provided via `QbTargetPattern` in the AI state. These are **constant for a given QB** throughout the game.
- `valid_targets` — the absolute field coordinates the QB can throw to **after** moving.

### Example: QB Moves Right and Throws

**Setup:** QB is at `(0, -2)`. Base pattern includes offsets `(0,2), (1,2), (-1,2), (0,3)`.

**Case A — QB stays still `[0, 0]`:**
```
new_qb_position = (0, -2) + (0, 0) = (0, -2)
valid_targets   = [(0,0), (1,0), (-1,0), (0,1)]
```

**Case B — QB moves right `[1, 0]`:**
```
new_qb_position = (0, -2) + (1, 0) = (1, -2)
valid_targets   = [(1,0), (2,0), (0,0), (1,1)]
```

Notice the entire target pattern shifted right by 1. If WR1 is at `(2, 0)`, the `passTarget` must be `[2, 0]` — a target that is only valid when the QB moves right this tick.

### Request Body Example (Move + Pass)

```json
{
  "QB": [1, 0],
  "RB": [0, 1],
  "WR1": [0, 1],
  "WR2": [0, 1],
  "GL": [0, 1],
  "GR": [0, 1],
  "C_O": [0, 0],
  "passTarget": [2, 0]
}
```

Here the QB moves `[1, 0]` (east) and throws to `[2, 0]`. The `passTarget` aligns with the **shifted** target pattern, not the pattern at the QB's pre-move position.

### Step-by-Step Agent Algorithm

```python
def plan_qb_move_and_pass(state, qb_base_pattern):
    """
    Decide QB's move vector and, if throwing, compute valid passTarget
    from the POST-MOVE position.
    """
    qb_pos = get_player_position(state, "QB")  # e.g., [0, -2]
    
    # 1. Decide the best move for QB (scramble, pocket step, hold)
    qb_move = decide_qb_move(state)  # e.g., [1, 0]
    
    # 2. Calculate new QB position after this move
    new_qb_pos = [qb_pos[0] + qb_move[0], qb_pos[1] + qb_move[1]]
    
    # 3. Recalculate absolute targets from the NEW position
    valid_targets = [
        [offset[0] + new_qb_pos[0], offset[1] + new_qb_pos[1]]
        for offset in qb_base_pattern
    ]
    
    # 4. Find the best target (closest to an open receiver)
    receivers = get_receiver_positions(state)  # WR1, WR2, RB
    best_target = select_best_target(valid_targets, receivers, state)
    
    # 5. Build the moves dict
    moves = {"QB": qb_move, ...other_positions...}
    if best_target and should_throw(state):
        moves["passTarget"] = best_target  # Absolute [x, y]
    
    return moves
```

### Common Mistakes (Move + Pass)

| Mistake | Result | Solution |
|---------|--------|----------|
| Using targets from the **pre-move** QB position | `passTarget` is outside the shifted pattern — pass may miss or be invalid | Always recalculate targets from `current_pos + move_vector` |
| Assuming the target pattern is absolute | Targets are wrong after any QB movement | Treat pattern as **relative offsets**; anchor to QB's new position each tick |
| Not accounting for QB movement when planning receiver routes | Receiver runs to a cell the QB can no longer reach | Plan receiver routes relative to the QB's projected position |
| Moving QB out of bounds then passing | Move rejected entirely | Validate `new_qb_pos` is within grid bounds (`-5 ≤ x ≤ 5`, `-5 ≤ y ≤ 5`) before planning pass |

### Strategic Uses of Move-and-Pass

- **Scramble throw:** QB steps away from pressure (`[-1, 0]`) and throws from a new angle, opening lanes that were blocked from the original position.
- **Roll-out pass:** QB moves laterally (`[1, 0]` or `[-1, 0]`) to shift the entire target window toward one side of the field where a receiver has separation.
- **Step-up throw:** QB moves forward (`[0, 1]`) to shorten the distance to a receiver, bringing shorter targets into range that were previously too far.
- **Escape and throw:** When defenders converge on the pocket, the QB moves diagonally (`[1, 1]`) and throws to a target that is now reachable from the new position.

> **Key Principle:** The `passTarget` coordinate must always be valid for where the QB **will be**, not where the QB **is**. Think of it as: "move first, then throw from the destination."

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
