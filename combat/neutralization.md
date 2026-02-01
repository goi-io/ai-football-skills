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
When players engage in physical contact, one can neutralize the other:
- **Blocking:** Offensive players neutralize defenders
- **Tackling:** Defensive players neutralize ball carriers
- **Result:** Neutralized player cannot move for a duration

## Neutralization Factors

### Determining Outcomes
The game engine determines neutralization outcomes based on:
- **Speed:** Impacts engagement advantage and initial contact
- **Strength:** Key attribute for physical engagements
- **Football IQ:** Affects recovery speed
- **Position matchups:** Some roles have advantages

Higher attribute values generally improve outcomes, but exact calculations are proprietary.

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
- Higher Strength improves outcomes

## Engagement Decisions

### When to Engage
- Clear advantage (Strength matchup favorable)
- High-value target (ball carrier, QB)
- Protecting critical objective

### When to Avoid
- Unfavorable matchup (stronger opponent)
- Better positioning available
- Coverage responsibility more important

## Evasion Concepts

### For Ball Carriers
- Track nearby defenders
- Compare Strength attributes
- Route around stronger defenders
- Accept short neutralization from weak defenders if beneficial

### Path Planning
- Avoid adjacent cells with stronger defenders
- Use speed to maintain distance
- Cut back when pursuit commits

## Strategic Considerations

### Duration Impact
- Neutralization removes a player for time
- Short neutralization may be acceptable trade
- Long neutralization severely hurts
- Plan for recovery timing

### Attribute Awareness
- Know your players' Strengths
- Know opponents' Strengths
- Match favorable engagements

### Positional Sacrifice
- Blockers expect to be engaged
- Trading neutralization can open opportunities
- Accept blocking duties to protect key players
