# SKILL: Neutralization

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Combat  
**Required By:** All agents (Offense & Defense)

## Purpose
Execute and anticipate player-vs-player physical engagements that disable opponents temporarily.

## What is Neutralization?
Neutralization is a temporary disable applied when players collide or commit certain penalties.
When neutralized, a player cannot move or interact for a set number of ticks.

Neutralization happens in two ways:
- **Collisions (movement stage):** Opposing players collide, one loses and is neutralized.
- **Penalties (penalty stage):** Stacking/riding or hotspot squatting neutralize the offender.

The engine stores neutralization in **both** the player tick vector and the player object to keep state consistent across the pipeline.

## Neutralization Factors

### Determining Outcomes
The game engine determines neutralization outcomes based on:
- **Speed:** First collision tiebreaker
- **Strength:** Second collision tiebreaker and primary duration driver
- **Football IQ:** Third collision tiebreaker, also reduces duration when high
- **Hands:** Fourth collision tiebreaker, also reduces duration when high
- **Position matchups:** Linemen vs. pass-responsible roles influence base duration

Higher attribute values generally improve outcomes, but exact calculations are proprietary.

## Engine Mechanics (Movement Stage)

### Neutralization Countdown
Before resolving new collisions each tick, the engine updates neutralized players:
- Decrements `NeutralizedDuration`
- Resets their movement vector to `(0, 0)` while neutralized
- Clears `NeutralizedBy` when the duration reaches 0

### Collision Detection
The engine compares offense vs. defense player locations:
- **Collision** = same location
- If **Influence** is enabled, collisions can also occur through influence overlap
- Influence overlap is only considered from tick 3 onward

### Collision Resolution Order
If a collision occurs (excluding the ball carrier; see below), the winner is decided by:
1. **Stationary beats moving**
2. **Speed**
3. **Strength**
4. **FootballIQ**
5. **Hands**
6. **Tie goes to defense** (defense is rewarded for reading the play)

The loser is neutralized for a duration computed by the neutralization formula.

### Ball Carrier Exception
The ball carrier is **excluded** from neutralization collision handling.
If the ball carrier collides with a defender, the play proceeds to the **tackle** stage
so the tackle can be recorded instead of a neutralization.

## Neutralization Duration (Engine Logic)

Duration is computed by `calculateNeutralizedTicksDuration` and results in **1-3 ticks**:

1. **Base duration** is driven by position matchups and strength:
	- Lineman vs. lineman, or lineman vs. non-lineman use **Strength** comparisons
2. **Reduction for high recovery stats:**
	- If the neutralized player's average of `FootballIQ` + `Hands` is >= 2.5, reduce by 1
	- If the average is >= 3.5, reduce to **1 tick**

### Holding-Style Neutralization
If a player continues to occupy the same cell as a player they previously neutralized,
the engine applies a **holding-style penalty neutralization** of **1 tick**
(via `HOLDING_PENALITY_TICK_COUNT = 1`).

This discourages camping on a neutralized opponent's space.

## Penalty-Based Neutralization (Penalty Stage)

Some neutralizations are penalties, not collisions:

- **Stacking/Riding:** A teammate sharing the **ball carrier's cell** for 2+ consecutive ticks
	(3+ ticks for QB at/below LOS overlapping RB/WR1/WR2) gets neutralized for **3 ticks**.
- **Hotspot Squatting:** Any player staying on a hotspot for 2+ consecutive ticks
	gets neutralized for **3 ticks**.

These penalties **do not** deduct points; neutralization is the only consequence.

## Offensive Blocking

### Blocker Assignments
| Blocker | Typical Target |
|---------|----------------|
| GL | TL (left side) |
| GR | TR (right side) |
| C_O | C_D (center) |

### Blocking Objectives
- Protect QB from pressure
- Open lanes for ball carriers
- Neutralize pass rushers

### Blocking Strategy
- Position between threat and QB
- Engage defenders before they reach target
- Maintain blocks as long as needed

## Defensive Tackling

### Tackle Priorities
1. **Ball carrier** - Highest priority
2. **QB with ball** - Stop scrambles, force throws
3. **Receivers** - Post-catch tackles

### Tackle Approach
- Close distance to target
- Engage when adjacent
- Higher Speed improves outcomes

## Engagement Decisions

### When to Engage
- Clear advantage (Speed matchup favorable)
- High-value target (ball carrier, QB)
- Protecting critical objective
- When defending passes, prioritize neutralizing receivers or defenders near likely throw cells using the QB's `QbTargetPattern` from the AI state response each tick. The QB can throw from **tick 2 through tick 8** only.

### When to Avoid
- Unfavorable matchup (faster opponent)
- Better positioning available
- Coverage responsibility more important

## Evasion Concepts

### For Ball Carriers
- Track nearby defenders
- Compare Speed attributes
- Route around faster defenders
- Accept short neutralization from defenders with weaker Football IQ and Hands if beneficial

### Path Planning
- Avoid adjacent cells with faster defenders
- Use speed to maintain distance
- Cut back when pursuit commits

## Strategic Considerations

### Duration Impact
- Neutralization removes a player for time
- Short neutralization may be acceptable trade
- Long neutralization severely hurts
- Plan for recovery timing

### Attribute Awareness
- Know your players' Speeds
- Know opponents' Speeds
- Match favorable engagements

### Positional Sacrifice
- Blockers expect to be engaged
- Trading neutralization can open opportunities
- Accept blocking duties to protect key players
