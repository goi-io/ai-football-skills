# SKILL: Ball Carrier Movement

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Offense  
**Required By:** Offensive Coach Agent

## Purpose
Guide ball carriers to maximize yards gained while avoiding defenders.

## Identifying the Ball Carrier via `WhoHasBall`

**Before planning ball carrier movement, confirm WHO actually has the ball.** The `position.whoHasBall` field — returned in every moves/formation response and the state endpoint — tells you which position is the current ball carrier.

> **Tip:** After each move submission, the response includes `whoHasBall` in `position` — use it immediately for your next tick's planning.

| `position.whoHasBall` | Meaning |
|---------------------|----------|
| `C_O` | Pre-snap; center holds the ball |
| `QB` | Post-snap; QB has the ball (can throw or hand off) |
| `RB` | Handoff or reception; RB is running |
| `WR1` / `WR2` | Pass completed to a receiver |
| `null` | Ball is in flight (no carrier) |

### Why This Matters for Offense
- **Only the actual ball carrier should be moved toward scoring zones.** Moving the wrong player toward a hotspot is wasted motion.
- **Blockers must protect the current carrier, not necessarily the QB.** After a completed pass, `WhoHasBall` changes from `QB` to `WR1`/`WR2`/`RB`. Blocking assignments should immediately shift to shield the new carrier from defenders.
- The ball carrier is **excluded from neutralization collisions** — instead, defender contact triggers a tackle. This means the carrier can take riskier paths through congestion since the worst outcome is a tackle (ending the play), not a multi-tick disable.
- Teammates sharing the ball carrier's cell for 2+ consecutive ticks get penalized (stacking/riding), so blockers should stay **adjacent** to the carrier, never on the same cell.

---

## Applicable Players
- **RB** - Primary ball carrier
- **WR1/WR2** - After catching passes
- **QB** - When scrambling

## Core Objectives

### Primary Goals (Priority Order)
1. **Advance toward scoring zones** - More points for deeper advancement
2. **Target high-value areas** - Bonus points at hotspot locations
3. **Avoid neutralization** - Lost time hurts scoring potential
4. **Protect the ball** - Don't risk turnovers unnecessarily

## Movement Strategy

### Threat Assessment
When planning ball carrier movement:
- Identify nearest defenders
- Compare Speed attributes (can you outrun them?)
- Compare Strength attributes (can you break tackles?)
- Find paths with least resistance

### Path Selection
- **Direct path:** Fastest but may have more defenders
- **Perimeter path:** Wider but may have open lanes
- **Cutback:** Change direction when defenders overcommit

## Evasion Techniques

### Speed-Based Evasion
When carrier has speed advantage:
- Take angles that maximize distance from defenders
- Use diagonal movement to gain while evading
- Outrun pursuit to open field

### Agility-Based Evasion
When facing faster defenders:
- Sharp direction changes
- Cut opposite defender's momentum
- Find gaps between defenders

### Power Running
When carrier has strength advantage:
- Can accept brief contact
- Push through weaker defenders
- Short neutralization may be worth the advance

## Situational Strategies

### Open Field
- Maximize yards per movement
- Target scoring zones if reachable
- Avoid unnecessary risks

### Congested Area
- Pick through gaps
- Be patient for openings
- Accept shorter gains

### Goal Line
- Focus on reaching high-value hotspots
- Accept contact if scoring is likely
- Don't go backward trying to find better angles

### Scramble (QB)
- Weigh running gain vs. passing options
- Keep passing available if receivers are open — the QB can throw from **tick 2 through tick 8** only.
- Protect the ball above all
- If considering a scramble-and-throw, use the QB's `QbTargetPattern` from the AI state response each tick to evaluate reachable pass targets after the move.

## Strategic Considerations

### Risk Tolerance
- **Conservative:** Protect ball, accept shorter gains
- **Balanced:** Take calculated risks for better positions
- **Aggressive:** Pursue hotspots despite higher tackle risk

### Time Awareness
- More time = more options
- Late play = take what you can get
- Don't waste ticks on inefficient movement
