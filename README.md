# GOI™ AI Agent Skills Index


This directory contains modular skills that AI agents can load to coach GOI teams.

## Directory Structure

```
agents/
├── skills/
│   ├── GOI_AGENT_SKILLS.md          # Master reference document
│   │
│   ├── foundation/                   # Core knowledge (required by all agents)
│   │   ├── field_awareness.md        # Grid, territories, hotspots
│   │   ├── player_knowledge.md       # Positions, roles, attributes
│   │   ├── movement_planning.md      # Valid moves, pathfinding
│   │   └── time_management.md        # Tick timing, play phases
│   │
│   ├── offense/                      # Offensive-specific skills
│   │   ├── pass_planning.md          # QB targeting, throw timing
│   │   ├── route_running.md          # Receiver routes, separation
│   │   └── ball_carrier_movement.md  # Running, evasion, YAC
│   │
│   ├── defense/                      # Defensive-specific skills
│   │   ├── pass_rush.md              # Pressure, lane blocking
│   │   ├── coverage.md               # Man/zone assignments
│   │   └── interception.md           # Ball-hawking, route jumping
│   │
│   ├── combat/                       # Player-vs-player interactions
│   │   └── neutralization.md         # Blocking, tackling, evasion
│   │
│   └── strategy/                     # High-level decision making
│       ├── scoring_optimization.md   # Maximizing points
│       └── risk_assessment.md        # Risk/reward analysis
│
└── prompts/                          # Complete agent prompts
    ├── offensive_coach_agent.md      # Full offensive coach prompt
    └── defensive_coach_agent.md      # Full defensive coach prompt
```

## Skill Loading Guide

### For Offensive Coach Agent
Load these skills in order:
1. **Foundation (all 4)** - Core game understanding
2. **Offense (all 3)** - Play execution
3. **Combat (neutralization)** - Blocking, evasion
4. **Strategy (both)** - Decision making

### For Defensive Coach Agent
Load these skills in order:
1. **Foundation (all 4)** - Core game understanding
2. **Defense (all 3)** - Coverage & pressure
3. **Combat (neutralization)** - Tackling, engagement
4. **Strategy (both)** - Decision making

## Quick Reference

### Skill Dependencies
```
┌─────────────────────────────────────────────┐
│              STRATEGY LAYER                  │
│  ┌─────────────────┐  ┌─────────────────┐   │
│  │ Scoring         │  │ Risk            │   │
│  │ Optimization    │  │ Assessment      │   │
│  └────────┬────────┘  └────────┬────────┘   │
│           │                    │            │
├───────────┼────────────────────┼────────────┤
│           │    COMBAT LAYER    │            │
│           │  ┌──────────────┐  │            │
│           └──┤Neutralization├──┘            │
│              └──────┬───────┘               │
├─────────────────────┼───────────────────────┤
│    TACTICAL LAYER   │                       │
│  ┌───────────┐  ┌───┴───────┐  ┌─────────┐ │
│  │Pass       │  │Ball Carrier│ │Coverage │  │
│  │Planning   │  │Movement    │ │         │  │
│  ├───────────┤  └───────────┘  ├─────────┤ │
│  │Route      │                 │Pass Rush│  │
│  │Running    │                 ├─────────┤ │
│  └───────────┘                 │Intercept│  │
│     OFFENSE                    │         │  │
│                                └─────────┘  │
│                                  DEFENSE    │
├─────────────────────────────────────────────┤
│              FOUNDATION LAYER               │
│  ┌──────────┐┌──────────┐┌────────┐┌─────┐ │
│  │Field     ││Player    ││Movement││Time │  │
│  │Awareness ││Knowledge ││Planning││Mgmt │  │
│  └──────────┘└──────────┘└────────┘└─────┘ │
└─────────────────────────────────────────────┘
```

## Response Format Standard

All coach agents must respond with JSON in this format:

```json
{
  "tick": <integer>,
  "side": "offense" | "defense",
  "moves": {
    "<position>": {
      "from": [x, y],
      "to": [x, y],
      "action": "<action_type>"
    }
  },
  "reasoning": {
    "primary_objective": "<string>",
    "key_info": "<string>"
  }
}
```

## Usage Notes

### Attribute System
Players have five key attributes that affect gameplay:
- **Speed (Sp)** - Movement and pursuit capability
- **Strength (St)** - Physical engagements
- **Intelligence (Iq)** - Decision-making situations
- **Hands (H)** - Ball-catching ability
- **Accuracy (Acc)** - Throwing precision (QB only)

### Neutralization State
- Players can become neutralized through combat interactions
- Neutralized players cannot move until they recover
- Recovery timing is determined by game engine rules
- **Neutralization outcomes consider Speed, Strength, Football IQ, and position matchups**

### Ball State
- Ball location is tracked each tick
- QB pass pattern shows valid throw targets
- Interception opportunities arise when defenders can reach the ball

---

*For detailed implementation guidance, refer to individual skill files.*
