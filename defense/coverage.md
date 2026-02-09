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

## Reading `WhoHasBall` for Coverage Decisions

**`position.whoHasBall` from every moves/formation response (and the state endpoint) is the most important field for defensive coverage.** It tells you the current phase of the offensive play and dictates coverage priorities.

> **Tip:** You get `whoHasBall` in the response every time you submit moves — no extra API call needed.

| `position.whoHasBall` | What It Means | Coverage Response |
|---------------------|---------------|-------------------|
| `QB` or `C_O` | Ball hasn't been thrown or handed off | Stay in coverage, shadow receivers, anticipate pass |
| `null` | Ball is in the air | **Break toward the pass target** to contest or intercept |
| `WR1`, `WR2`, or `RB` | Pass completed or handoff occurred | **Abandon receiver coverage, pursue the ball carrier** |

### Key Transitions
- **QB → null:** Pass was just thrown. Check `ball.x` / `ball.y` for the flight path. Move toward the target cell.
- **null → WR1/WR2/RB:** Pass completed. The receiver is now the ball carrier — all defenders should collapse toward them.
- **QB → RB:** Handoff. LB and secondary should shift from coverage to run pursuit.

### Coverage vs. Pursuit Trade-Off
Once `position.whoHasBall` changes from `QB` to a skill player, coverage assignments **no longer matter**. Every defender's priority becomes pursuing the ball carrier for a tackle. Staying in a coverage shell after a completion wastes ticks.

---

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
- Anticipate throws — the QB can throw from **tick 2 through tick 8** only (tick 1 is a setup tick).
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
