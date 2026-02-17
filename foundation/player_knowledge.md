# SKILL: Player Knowledge

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Foundation  
**Required By:** All agents (Offense & Defense)

## Purpose
Understand player positions, roles, attributes, and strategic deployment.

## Roster Overview

**Total Players:** 14 (7 offense + 7 defense)

### Offensive Positions

| Position | Full Name | Primary Role |
|----------|-----------|--------------|
| **QB** | Quarterback | Pass thrower, play initiator |
| **RB** | Running Back | Ball carrier, short receiver |
| **WR1** | Wide Receiver 1 | Primary deep target |
| **WR2** | Wide Receiver 2 | Secondary target |
| **GL** | Guard Left | Left side blocker |
| **GR** | Guard Right | Right side blocker |
| **C_O** | Center (Offense) | Central blocker |

**Note:** Linemen (GL, GR, C_O) have restricted movement zones.

### Defensive Positions

| Position | Full Name | Primary Role |
|----------|-----------|--------------|
| **LB** | Linebacker | Versatile defender, run stop |
| **S** | Safety | Deep coverage, interceptions |
| **CB1** | Cornerback 1 | Primary receiver coverage |
| **CB2** | Cornerback 2 | Secondary receiver coverage |
| **TL** | Tackle Left | Left side pass rush |
| **TR** | Tackle Right | Right side pass rush |
| **C_D** | Center (Defense) | Central pass rush |

**Note:** Linemen (TL, TR, C_D) have restricted movement zones.

## Player Attributes

Each player has five attributes (rated 1–5) that drive on-field outcomes, especially collisions and neutralization:

| Attribute | Role in Gameplay |
|-----------|-----------------|
| **Speed** | **Primary collision attribute.** When two opposing players land on the same square, the faster player neutralizes the slower one (removing them for 1–3 ticks). Also the 1st tiebreaker in collision resolution. Offense sends faster blockers/receivers into key defenders to clear lanes; defense sends faster defenders onto threats. |
| **Strength** | **Neutralization duration.** A stronger neutralizer imposes a longer neutralization on the loser. 2nd tiebreaker in collision resolution. |
| **Football IQ** | **Recovery & awareness.** High Football IQ helps a neutralized player recover faster (reduces neutralization duration). 3rd tiebreaker in collision resolution. |
| **Hands** | **Catching, interception & recovery.** Affects catch success and interception ability. High Hands also reduces neutralization duration. 4th tiebreaker in collision resolution. |
| **Accuracy** | **QB only — pass target pattern.** Determines the specific field locations (target dots) where the QB can throw. The target pattern moves with the QB's position, so moving the QB repositions all passing options. |

### Collision Resolution Order

When opposing players collide, the winner is determined by cascading attribute comparison:

```
1. Speed       → Higher wins
2. Strength    → Tiebreaker #2
3. Football IQ → Tiebreaker #3
4. Hands       → Tiebreaker #4
5. Defense     → Defensive player wins (reward for reading the play)
```

**Winner:** Maintains position/movement. **Loser:** Neutralized at current location.

### Neutralization Duration

- **Base duration** determined by Strength comparison (winner STR vs loser STR): stronger neutralizer → 3 ticks; weaker → 2 ticks; equal → 1 tick.
- **Duration reduction** based on the neutralized player's avg(Football IQ + Hands): ≥ 3.5 → minimum 1 tick; ≥ 2.5 → base − 1; < 2.5 → no reduction.

## Position Role Summaries

### Quarterback (QB)
- Field general who initiates plays
- Has a pass-target pattern defining throwable locations; pattern moves with QB position
- Key attributes: Accuracy (shapes passing options), Football IQ

### Running Back (RB)
- Versatile offensive weapon
- Receives short passes, carries ball
- Key attributes: Speed (collision winner), Strength, Hands (catching)

### Wide Receivers (WR1, WR2)
- Deep threats who run routes
- Create separation from defenders; must reach pass target dots to catch
- Key attributes: Speed (neutralize defenders, win collisions), Hands (catch success)

### Offensive Linemen (GL, GR, C_O)
- Protect QB through blocking; neutralize defenders to clear lanes
- Movement restricted to near line of scrimmage (Y ≤ 2)
- Key attributes: Speed (collision winner), Strength (longer neutralization on opponents)

### Linebacker (LB)
- Most versatile defensive player
- Covers short routes, rushes QB, blocks passing lanes
- Key attributes: Speed (neutralize receivers), Strength, Football IQ

### Safety (S)
- Last line of defense, deep coverage
- Primary interception threat; can reach pass target dots to intercept
- Key attributes: Speed, Hands (interception), Football IQ

### Cornerbacks (CB1, CB2)
- Shadow receivers in coverage
- Match up against WR1/WR2 respectively; intercept by reaching pass targets
- Key attributes: Speed (outrun receivers), Hands (interception)

### Defensive Linemen (TL, TR, C_D)
- Disrupt offensive plays at the line
- Rush QB, block passing lanes by occupying the QB's directional lanes
- Movement restricted to near line of scrimmage (Y ≤ 2)
- Key attributes: Speed (collision winner), Strength (longer neutralization)

## Strategic Considerations

### Matchup Analysis
- **Speed-first**: Compare Speed to determine who wins collisions — always target opponents with lower Speed
- **Strength-second**: When Speeds are close, Strength determines neutralization duration
- **IQ & Hands**: Players with high Football IQ and Hands recover faster from neutralization — they are more resilient targets
- **QB Accuracy**: Higher Accuracy = more pass target options on the field; position receivers near targets

### Attribute-Based Deployment
- High-Speed blockers/receivers: Send into slower defenders to neutralize them and clear lanes
- High-Speed defenders: Position onto the ball carrier or key receivers to erase threats
- High-Hands receivers: Target for catches at pass target locations
- High-Hands/IQ players: More resilient against neutralization — deploy in contested areas
- High-Accuracy QB: Wider pass target pattern — exploit more of the field
