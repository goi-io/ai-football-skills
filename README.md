# GOI™ AI Agent Skills Index


This directory contains modular skills that AI agents can load to coach GOI teams.

## Directory Structure

```
agents/
├── skills/
│   ├── skill.md                     # Master reference document
│   │
│   ├── foundation/                   # Core knowledge (required by all agents)
│   │   ├── field_awareness.md        # Grid, territories, hotspots
│   │   ├── player_knowledge.md       # Positions, roles, attributes
│   │   ├── movement_planning.md      # Valid moves, pathfinding
│   │   ├── time_management.md        # Tick timing, play phases
│   │   └── game_lifecycle.md         # Turn polling, game loop, never quit
│   │
│   ├── offense/                      # Offensive-specific skills
│   │   ├── pass_planning.md          # QB targeting, throw timing
│   │   ├── route_running.md          # Receiver routes, separation
│   │   ├── ball_carrier_movement.md  # Running, evasion, YAC
│   │   └── scoring.md                # Hotspot pursuit, DENIED!! risk, point values
│   │
│   ├── defense/                      # Defensive-specific skills
│   │   ├── pass_rush.md              # Pressure, lane blocking
│   │   ├── coverage.md               # Man/zone assignments
│   │   └── interception.md           # Ball-hawking, route jumping
│   │
│   ├── combat/                       # Player-vs-player interactions
│   │   ├── neutralization.md         # Blocking, tackling, evasion
│   │   └── penalties.md              # Stacking, squatting, out-of-bounds
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
Players have five key attributes (rated 1–5) that drive collisions and neutralization:
- **Speed (SPD)** — Primary collision attribute. The faster player neutralizes the slower one when they land on the same square.
- **Strength (STR)** — Determines neutralization duration. Stronger neutralizer → opponent stays out longer.
- **Football IQ (FbIQ)** — 3rd collision tiebreaker. High IQ helps players recover from neutralization faster.
- **Hands (HND)** — Catching/interception ability; 4th collision tiebreaker. High Hands also reduces neutralization duration.
- **Accuracy (Acc)** — **QB only.** Determines the QB's pass target pattern (where the QB can throw). Pattern moves with QB position.

### Neutralization Mechanics
- When opposing players collide on the same square, attributes are compared: Speed → Strength → Football IQ → Hands → Defense wins ties.
- **Winner** stays active; **loser** is neutralized (frozen in place) for 1–3 ticks.
- **Duration** is based on Strength comparison; reduced by loser's avg(Football IQ + Hands).
- Neutralized players cannot move until they recover.

### Ball State
- Ball location is tracked each tick
- QB pass pattern shows valid throw targets
- Interception opportunities arise when defenders can reach the ball

---

*For detailed implementation guidance, refer to individual skill files.*
