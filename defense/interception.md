# SKILL: Interception

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Defense  
**Required By:** Defensive Coach Agent

## Purpose
Position defenders to intercept passes and create turnovers for defensive scoring.

## Applicable Positions
- **S** - Safety (primary ball hawk, highest priority)
- **CB1, CB2** - Cornerbacks (coverage interceptions)
- **LB** - Linebacker (underneath route interceptions)

## Reading `WhoHasBall` for Interception Opportunities

**`position.whoHasBall` in the moves response is the trigger signal for interception attempts.** Interceptions are only possible when the ball is in the air or at the target cell.

> **Tip:** When `whoHasBall` transitions to `null` in a response, the ball was just thrown — act immediately on your next tick.

| `position.whoHasBall` | Interception Relevance |
|---------------------|------------------------|
| `QB` or `C_O` | Ball hasn't been thrown yet — prepare by reading routes and anticipating targets |
| `null` | **Ball is in flight — BREAK NOW.** Move toward `ball.x`/`ball.y` (pass target) to contest |
| `WR1`, `WR2`, `RB` | Pass completed — interception window is closed; shift to pursuit/tackle |

### Critical: The `null` Window
When `position.whoHasBall` transitions from `QB` to `null`, you have a **limited tick window** to reach the target cell before the receiver. This is the interception opportunity:
1. Read `ball.x`, `ball.y` — this is where the ball is heading.
2. Compare your defender's position to the target cell.
3. If reachable, move directly toward the target. Highest `avg` attributes win the contest.
4. If the ball becomes LiveBall (no one catches it at the target), it remains contestable for **2 ticks** — a second chance.

---

## Interception Value
Interceptions are highly valuable:
- Defense gains possession
- Points awarded based on interception location
- Interceptions at scoring zones award bonus points

## Interception Opportunities

### Live Ball Situations
When QB throws to an empty target cell:
- Ball becomes contestable
- Defender reaching target first can intercept
- **Hands attribute does NOT affect interception success**
- Interception chance increases when **receivers are neutralized** at the live ball

Engine notes for live ball:
- Only **non-neutralized** players can contest
- Eligible contesters are **any defense player** plus **WR1/WR2/RB**
- The highest `avg` attributes win the contest at the ball location

### Anticipated Throws
- Read QB's eyes/body language
- Jump routes based on timing
- Higher risk but higher reward

### Tipped Balls
- Contact with ball at line creates opportunities
- Alertness to deflected passes
- Position to recover loose balls

## Engine Interception Rules (Pass Reception Stage)

### Ordinal Lane Intercepts
If the QB throws along a direct ordinal line (N, NE, E, SE, S, SW, W, NW):
- **D-Line in the lane** blocks the pass (nearest + strongest to the QB)
- If no D-Line block, **pass defenders** in the lane intercept instead
	(CB1, CB2, S, and LB are considered pass defenders here)

### Target-Cell Priority (No Ordinal Block)
When the ball reaches its target cell, the engine checks players **on that cell** in this order:
1. **Offensive receivers** (WR1, WR2, RB) catch
2. **Interceptors** (CB1, CB2, S) intercept
3. **Blockers** (D-Line and LB) block
4. **O-Line** has no effect

This means an interceptor only wins if **no receiver is already on the target cell**.

### Timing Outcomes
- **Tick < 8:** Interception is **InterceptedActive** (play continues)
- **Tick >= 8:** Interception is **InterceptedDead** (play ends)

## Interception Strategy

### Positioning
- Be close enough to reach ball
- Understand QB's pass pattern — passes can only be thrown from **tick 2 through tick 8** (tick 1 is a setup tick).
- Anticipate likely targets
- Use the QB's `QbTargetPattern` from the AI state response each tick to identify reachable target cells.

### Timing
- Break on the ball when thrown
- Arrive at or before receiver
- Don't break too early (reveals coverage)

### Risk Assessment
- Gamble on interception vs. give up completion
- Consider game situation
- High-risk plays in desperate situations

## Key Factors

### Neutralization Advantage (Critical)
- Neutralizing a receiver **at a live ball** increases interception chance
- Neutralizing a receiver **at a direct pass target** increases interception chance
- Blocking the **QB directional throwing lane** increases interception chance

### Position Advantage
- Being closer to target helps
- Facing the ball helps
- Inside position often preferable

### Speed
- Ability to close on ball quickly
- Recover if initial position is off
- Chase down underthrown balls

## Strategic Considerations

### Ball-Hawking vs. Coverage
- Aggressive: Jump routes for interceptions
- Conservative: Stay in coverage, react to throw
- Balance based on score and situation

### Interception Scoring
- Interceptions in scoring zones give more points
- Consider positioning near high-value areas
- Don't sacrifice coverage for position

### Team Coordination
- Communicate ball location when thrown
- Support teammates going for ball
- Don't let receiver get free while hunting interception
