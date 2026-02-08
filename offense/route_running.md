# SKILL: Route Running

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Offense  
**Required By:** Offensive Coach Agent

## Purpose
Plan receiver routes to reach pass targets while creating separation from defenders.

## Eligible Positions
- **WR1** - Wide Receiver 1 (primary deep threat)
- **WR2** - Wide Receiver 2 (secondary receiver)
- **RB** - Running Back (short routes, screens)

## Core Route Concepts

### Vertical Routes
- **Streak (Go):** Straight upfield movement
- Best for fast receivers against slower coverage
- Requires time to develop

### Breaking Routes
- **Out:** Move upfield then break toward sideline
- **Slant:** Diagonal movement toward center
- **Post:** Upfield then diagonal toward far side
- **Curl/Hook:** Move upfield then stop or come back

### Short Routes
- **Flat:** Move toward sideline, staying shallow
- **Screen:** Stay behind line, let blockers lead

## Route Selection Strategy

### Based on Coverage
- **Man coverage:** Use routes that create separation through direction changes
- **Zone coverage:** Find soft spots between defenders
- **Press coverage:** Consider quick releases or speed routes

### Based on Receiver Strengths
- **Fast receivers:** Vertical routes, streaks, posts
- **Quick receivers:** Slants, outs, curls
- **Physical receivers:** Contested catch situations

### Based on Time Available
- **Early play:** Any route viable
- **Mid play:** Medium-depth routes optimal
- **Late play:** Only short routes can complete

## Creating Separation

### Speed Advantage
When receiver is faster than covering defender:
- Vertical routes gain separation over time
- Push defender deep then break

### Quickness Advantage
When receiver needs to beat defender with moves:
- Sharp breaking routes (slants, outs)
- Direction changes create brief windows

### Route Combinations
Receivers can work together:
- One receiver clears an area
- Another receiver attacks vacated space

## Strategic Considerations

### Reading Defenders
- Track covering defender position
- Adjust route based on defender's reaction
- Option routes allow mid-route adjustments

### Timing with QB
- Route completion should align with QB's throwing windows
- Arrive at target when QB is ready to throw
- Don't outrun the pattern
- Use the QB's `QbTargetPattern` from the AI state response each tick to align route endpoints with reachable throw cells.

### Reception Advantage Rule
- **Hands attribute does NOT affect receptions**
- Coordinate with blockers to **neutralize defenders** at:
	- **Live ball targets** (empty cell throws)
	- **Direct pass targets** (receiver on target)

### Pass Targeting Integration
- The QB throws by including `"passTarget": [x, y]` (absolute field coordinates) in the moves JSON body.
- Route completion should place the receiver **at or adjacent to** the `passTarget` cell when the ball arrives.
- Only **WR1**, **WR2**, and **RB** are eligible pass receiver positions.
- A pass can only be thrown **once per play** — coordinate route timing with the QB's throw tick.
- See [offense/pass_planning.md](pass_planning.md) for full pass mechanics and API details.

### Safety Awareness
- Deep routes risk interception from safety help
- Understand when safeties shade toward your route
- Be prepared for contact on contested catches
