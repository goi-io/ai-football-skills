# SKILL: Pass Planning

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Offense  
**Required By:** Offensive Coach Agent

## Purpose
Determine optimal pass targets and timing strategy for the quarterback.

## Core Concepts

### QB Pass-Target Pattern
- QB has a defined pattern of cells where passes can be thrown
- Pattern position is relative to QB's current location
- As QB moves, the absolute target positions shift accordingly
- Agents receive pattern information in game state

### Pass Types

#### Direct Pass
- Target cell is occupied by a friendly receiver
- Ball goes directly to receiver
- Lower risk approach

#### Live Pass
- Target cell is empty when throw is made
- Ball arrives at cell and can be contested
- Higher risk, but can create opportunities

## Strategic Pass Planning

### Target Evaluation Factors
Consider these when choosing pass targets:
- **Receiver proximity:** Can receiver reach target in time?
- **Defender proximity:** Are defenders close to the target?
- **Field position:** Higher targets offer more scoring potential
- **Time remaining:** Deep routes need more time to develop

### Interception Risk Assessment
Higher risk when:
- Defenders are closer to target than receivers
- Multiple defenders near target area
- Pass is "live" (empty target cell)

Lower risk when:
- Receiver has clear advantage to reach target
- Direct pass to receiver already at target
- Throwing lanes are clear

### Throwing Lane Awareness
- Defenders positioned between QB and target can disrupt passes
- Consider alternate targets if primary lane is blocked
- QB movement can open new lanes

## Pass Timing Guidelines

### Route Development Phases
| Phase | Timing | Route Types |
|-------|--------|-------------|
| Quick | Early play | Screens, flats, quick outs |
| Medium | Mid play | Slants, curls, crossing |
| Deep | Later play | Posts, streaks, corners |
| Desperation | End of play | Any open receiver |

### When to Throw
Consider throwing when:
- Receiver reaches target with separation
- Pressure is increasing on QB
- Best window may be closing

Consider waiting when:
- Better options are developing
- Current risk is too high
- QB has time and protection

## Strategic Considerations

### Risk Tolerance
- **Conservative:** Prioritize low-risk, shorter throws
- **Balanced:** Trade off risk vs. reward
- **Aggressive:** Accept higher risk for bigger plays

### Situational Adjustments
- **Behind in score:** May need to take more risks
- **Ahead in score:** Protect the ball, safer throws
- **Time pressure:** Must execute quickly
