# SKILL: Penalties

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Combat  
**Required By:** All agents (Offense & Defense)

## Purpose
Understand and avoid move-validation rejections and in-game penalty conditions that neutralise players.

## Layer 1 — Move Validation (Submission Rejection)

These checks run **before** moves are applied. If any fails your entire tick submission is rejected and you must resubmit.

### 1. Overlapping Players
**Trigger:** Two or more same-team players would occupy the same grid cell after moves resolve.

**Result code:** `OverLappingPlayers`

**How to Avoid:**
- Before submitting, compute every player's resulting position: `formationPos + sum(priorTickMoves) + proposedMove`
- Confirm no two teammates share the same (X, Y)
- Formation positions must also be unique

### 2. Out-of-Bounds Movement
**Trigger:** A player's resulting position falls outside the 11×11 grid (coordinates must stay within −5 to +5 on both axes).

**Result code:** `OutOfBoundsMovement`

**How to Avoid:**
- Calculate each player's absolute position before submitting
- Players near edges (|X| ≥ 4 or |Y| ≥ 4) need extra care

### 3. Lineman Movement Constraint Violation
**Trigger:** A lineman's resulting Y-coordinate exceeds 2 (Y > 2).

**Result code:** `LinemanMovementConstraintViolation`

**Applies to:** Offensive linemen (GL, GR, C_O) and defensive linemen (TL, TR, C_D).

**How to Avoid:**
- Track linemen absolute Y positions each tick
- Plan blocking routes that stay at Y ≤ 2

## Layer 2 — In-Game Penalties (Neutralisation)

These penalties are evaluated **after** moves are applied during the game-engine pipeline. They do **not** reject the submission — instead they **neutralise the offending player for 3 ticks** (their moves are zeroed out). No points are deducted.

### 4. Stacking / Riding Penalty
**Trigger:** A teammate occupies the **same cell as the ball carrier** for **2 or more consecutive ticks**.

**Special case:** If the offense QB is at or below the line of scrimmage and overlaps with RB, WR1, or WR2, the threshold is raised to **3 consecutive ticks**.

**Impact:**
- The **overlapping teammate** (not the ball carrier) is neutralised for 3 ticks
- The engine tracks consecutive overlap ticks per player and resets the counter when they separate
- No point deductions — neutralisation only

**How to Avoid:**
- After the ball is received, move teammates **away** from the ball carrier's cell within 1 tick
- Brief overlap (1 tick) is allowed — it's sustained overlap that triggers the penalty
- QB handoff scenarios (RB near QB at LOS) get extra grace (3-tick threshold) but don't abuse it
- Plan routes so escorts/blockers run *adjacent* to the ball carrier, not on top of them

### 5. Hotspot Squatting Penalty
**Trigger:** **Any player** (offense or defense) stays on a **hotspot** for **2 or more consecutive ticks**.

Hotspot positions: corners (±5, ±5) = 10 pts, primes (0, ±5) = 20 pts.

**Impact:**
- The **squatting player** is neutralised for 3 ticks
- The engine tracks consecutive hotspot-occupation ticks per player and resets when they leave
- No point deductions — neutralisation only

**How to Avoid:**
- Score the hotspot and **immediately move off** the next tick
- Plan an exit vector before arriving at the hotspot
- On defense, don't park a defender on a hotspot to block it — they'll get neutralised too
- Even a 1-cell move away resets the counter

## Penalty Awareness Strategy

### Pre-Submission Checklist
Before submitting any tick moves, verify:

1. **No overlapping teammates** — compute all resulting positions, confirm uniqueness (Layer 1)
2. **All positions in bounds** — every player's resulting (X, Y) stays within [−5, +5] (Layer 1)
3. **Linemen below ceiling** — GL, GR, C_O / TL, TR, C_D stay at Y ≤ 2 (Layer 1)
4. **Ball carrier not sharing cell with teammate for 2+ ticks** — move escorts aside (Layer 2)
5. **No player camping a hotspot** — score and move off within 1 tick (Layer 2)

### Recovery from Rejected Submissions (Layer 1 only)
If your tick is rejected:
- Read the `result` field in the API response to identify which validation failed
- Adjust the offending player's move vector
- Resubmit immediately — the game is waiting for your valid moves

### Recovery from Neutralisation (Layer 2)
If a player is neutralised:
- Their moves are zeroed for the next 3 ticks — you cannot control them
- Plan around the missing player: adjust routes, reassign blocking
- The neutralised player stays at their current position until the penalty expires

### Engine Validation Order
The engine checks Layer 1 validations in this order:
1. Overlapping players
2. Invalid move vectors (magnitude/direction)
3. Lineman movement constraint
4. Out-of-bounds movement
5. Invalid pass target (offense only)

If multiple violations exist, only the **first detected** error is returned. Fix it and resubmit — additional errors may surface on the next attempt.

Layer 2 penalties are evaluated after all moves have been applied and do not produce rejection errors.

## Integration with Other Skills

| Skill | Penalty Relevance |
|-------|-------------------|
| **Movement Planning** | Calculate resulting positions to avoid stacking & out-of-bounds |
| **Ball Carrier Movement** | Keep the ball carrier moving to avoid squatting penalty |
| **Route Running** | Ensure routes don't push receivers out of bounds |
| **Pass Rush** | Keep defensive linemen below Y = 2 |
| **Neutralization** | After engagement, check that repositioned players don't stack |
