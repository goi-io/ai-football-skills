# SKILL: Scoring — Hotspot Pursuit & Risk Management

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Offense  
**Required By:** All offensive agents; defensive agents pursuing interception returns

## Purpose
Guide agents toward maximising points by reaching hotspots while understanding the risks that can erase those points entirely.

## Hotspot Map

The 11×11 field (coordinates −5 to +5 on both axes) has **6 hotspots** arranged symmetrically:

```
  (-5,5)──────(0,5)──────(5,5)      ← UPPER hotspots (OFFENSE goal side)
    10 pts    20 pts     10 pts

              FIELD AREA
         Y = 0  (Line of Scrimmage)
              FIELD AREA

  (-5,-5)────(0,-5)─────(5,-5)      ← LOWER hotspots (DEFENSE goal side)
    10 pts    20 pts     10 pts
```

### Point Values by Side of Ball

| Hotspot | Offense | Defense (after interception) |
|---|---|---|
| Upper corners (±5, 5) | **+10** | **+10** |
| Upper prime (0, 5) | **+20** | **+20** |
| Lower corners (±5, −5) | **−10** (safety!) | **+10** |
| Lower prime (0, −5) | **−20** (safety!) | **+20** |

**Key insight for offense:** Upper hotspots are always positive. Lower hotspots are **negative** — reaching them is worse than being tackled at midfield. Never route the ball carrier toward Y = −5.

**Key insight for defense:** All hotspots are positive after an interception. The absolute value is used, so lower hotspots are just as valuable as upper ones.

## Non-Hotspot Scoring (Fallback)

If a play ends without the ball carrier on a hotspot (tackle, time expired, etc.):

| Side | Score Formula |
|---|---|
| **Offense** | Ball carrier's Y-coordinate (can be negative if behind LOS) |
| **Defense** | `abs(Y)` of ball carrier's position (always ≥ 0) |

This means every cell of Y-advancement matters for offense, even if you don't reach a hotspot.

### Zero-Score Outcomes
These end-of-play scenarios always award **0 points** regardless of position:
- **Blocked pass** — D-Line blocks the throw
- **Live ball expired** — pass in flight when tick 8 hits with no one at the target

## The "DENIED!!" Rule — Tackle/Neutralisation at Hotspot

This is the critical risk every agent must account for.

**If the ball carrier is tackled OR neutralised at a hotspot, the score is 0 — not the hotspot value.**

The engine performs a multi-source neutralisation check:
1. Direct check on the ball carrier's `NeutralizedBy` / `NeutralizedDuration`
2. Cross-reference against the player's vector data for latest-tick neutralisation status
3. Check for tackle status (`TackledStatus`) at a hotspot location

If **any** of these indicate the ball carrier is down at a hotspot, the play ends as `EndOfPlayReasonTypes.Tackle` (not `HotSpotReached`) and scores **0**.



```
penalty → movement → tackle → targeting → passReception → hasHotSpotBeenReached → scoring
```

**Tackle (stage 4) runs before hotspot check (stage 7).** This means a defender who arrives at the same cell as the ball carrier on the same tick the ball carrier reaches a hotspot will tackle them — erasing the score. The hotspot check then sees the tackle status and downgrades the play from `HotSpotReached` to `Tackle` with 0 points.

### Scenarios That Trigger DENIED!!

| Scenario | What Happens | Points |
|---|---|---|
| Defender at hotspot when ball carrier arrives | Tackle at hotspot | **0** |
| Ball carrier collides with defender en route, gets neutralised, slides onto hotspot | Neutralised at hotspot | **0** |
| Ball carrier reaches hotspot but was just neutralised by collision this tick | Neutralised at hotspot | **0** |
| Ball carrier reaches hotspot with no defenders nearby | Clean hotspot score | **10 or 20** |

## Hotspot Squatting — Don't Linger

Even if you score a hotspot cleanly, **do not stay on it**. The penalty engine tracks consecutive ticks at a hotspot:

- **2+ consecutive ticks** at any hotspot → player neutralised for 3 ticks
- This applies to **any player** (offense or defense), not just the ball carrier
- The counter resets when the player moves off the hotspot

In practice, a ball carrier who reaches a hotspot ends the play immediately (the hotspot check triggers `EndOfPlay`), so squatting is more relevant for **non-ball-carrier players** who might be positioned on or near hotspots for strategic reasons (e.g., a blocker camped at a corner). But if the ball is intercepted and the interceptor is already on a hotspot from a prior tick, they could be penalised.

## Offensive Hotspot Strategy

### Route Planning to Hotspots
1. **Identify reachable hotspots** — from the ball carrier's current position, calculate how many ticks remain and whether a hotspot is within reach
2. **Prefer prime hotspots (20 pts)** over corners (10 pts) when both are reachable
3. **Avoid lower hotspots entirely** — they cost you 10 or 20 points on offense
4. **Factor in Y-advancement** — if no hotspot is reachable, maximise the ball carrier's Y-coordinate for fallback scoring

### Calculating Reachability
- Ball carrier moves 1 cell per tick in any cardinal or diagonal direction
- From position (X, Y), the upper prime (0, 5) is reachable in `max(|X|, |5 − Y|)` ticks
- From position (X, Y), upper corner (5, 5) is reachable in `max(|5 − X|, |5 − Y|)` ticks
- Compare remaining ticks (8 − currentTick) against distance to decide if a hotspot run is viable

### Escort Without Stacking
Teammates can run alongside the ball carrier to create a screen, but remember the **stacking/riding penalty**: a teammate sharing the ball carrier's exact cell for 2+ consecutive ticks gets neutralised for 3 ticks. Run escorts on **adjacent cells**, not on top of the ball carrier.

## Defensive Hotspot Risk — Clearing the Path

As a defender, your primary job is to **deny hotspot access**:

1. **Position defenders between the ball carrier and the nearest hotspot** — the tackle-at-hotspot rule means even one defender at the hotspot cell erases the score
2. **Converge on the ball carrier's lane** — force them to take longer routes, burning ticks
3. **Don't camp on hotspots yourself** — the squatting penalty neutralises defenders too. Patrol *near* the hotspot, moving onto it only when the ball carrier is approaching
4. **Interception return scoring** — after an interception, all hotspots are positive for defense. The interceptor should sprint for the nearest hotspot using the same reachability math above

## Scoring Decision Tree

```
Ball carrier has ball → Is a hotspot reachable within remaining ticks?
├── YES → Is the hotspot path clear of defenders?
│   ├── YES → Route ball carrier to hotspot (prefer 20-pt prime)
│   └── NO → Can escorts neutralise/distract defenders?
│       ├── YES → Send escorts ahead, route ball carrier behind them
│       └── NO → Is a different hotspot reachable and clear?
│           ├── YES → Reroute to alternative hotspot
│           └── NO → Maximise Y-advancement for fallback scoring
└── NO → Maximise Y-advancement for fallback scoring
```

## Quick Reference

| Concept | Detail |
|---|---|
| **Best case** | Ball carrier reaches upper prime (0, 5) = **+20 pts** |
| **Good case** | Ball carrier reaches upper corner (±5, 5) = **+10 pts** |
| **Fallback** | No hotspot, play ends = ball carrier's **Y value** as score |
| **Worst case** | Ball carrier reaches lower prime (0, −5) = **−20 pts** |
| **DENIED!!** | Tackled or neutralised at hotspot = **0 pts** |
| **Squatting trap** | Stay on hotspot 2+ ticks = **3-tick neutralisation** |
| **Play length** | 8 ticks maximum; most plays end earlier via tackle or hotspot |
| **Stacking risk** | Escort on ball carrier's cell 2+ ticks = escort neutralised 3 ticks |
