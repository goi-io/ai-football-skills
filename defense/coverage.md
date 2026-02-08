# SKILL: Coverage

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Defense  
**Required By:** Defensive Coach Agent

## Purpose
Assign and execute defensive coverage to prevent offensive completions and create turnovers.

## Applicable Positions
- **CB1, CB2** - Cornerbacks (primary receiver coverage)
- **S** - Safety (deep coverage, help)
- **LB** - Linebacker (short coverage, RB responsibility)

## Coverage Types

### Man-to-Man Coverage
Each defender assigned a specific receiver:
- CB1 typically covers WR1
- CB2 typically covers WR2
- LB typically covers RB
- S provides help or covers extra receiver

**Strengths:** Clear assignments, accountability
**Weaknesses:** Beaten by speed mismatches

### Zone Coverage
Defenders cover areas of the field:
- Deep zones protect against long passes
- Underneath zones cover short/intermediate
- Players react to receivers entering their zone

**Strengths:** Eyes on QB, better for interceptions
**Weaknesses:** Soft spots between zones

### Mixed Coverage
Combination approach:
- Some players in man, others in zone
- Can disguise coverage pre-snap
- Adjust based on offensive formation

## Coverage Execution

### Maintaining Position
- Stay between receiver and QB
- Match receiver's depth (how far from line)
- Maintain leverage (inside or outside positioning)

### Reading and Reacting
- Watch for route breaks
- Anticipate throws
- Break on the ball when thrown
- Use the QB's `QbTargetPattern` from the AI state response each tick to anticipate likely throw windows.

### Communication
- Pass off receivers crossing zones (zone)
- Call out picks or rubs (man)
- Identify threats developing

## Strategic Considerations

### Matchup Awareness
- Speed vs. speed: Who has advantage?
- Hands: Who wins contested catches?
- Strength: Physical receivers need physical coverage

### Help Responsibility
- Safety helps over top on deep routes
- LB helps underneath on crossing routes
- Don't leave teammates isolated on tough matchups

### Risk Management
- Tight coverage risks giving up big plays
- Loose coverage gives up shorter completions
- Balance based on game situation
