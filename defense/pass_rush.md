# SKILL: Pass Rush

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Defense  
**Required By:** Defensive Coach Agent

## Purpose
Move defensive players to disrupt passing plays through pressure and lane blocking.

## Applicable Positions
- **TL, TR, C_D** - Defensive linemen (primary rushers, movement restricted)
- **LB** - Linebacker (secondary rusher, full mobility)

## Rush Objectives

### Primary Goals
1. **Block throwing lanes** - Position between QB and targets
2. **Apply pressure** - Force early/bad throws
3. **Contain QB** - Prevent scrambles
4. **Support coverage** - Don't leave gaps

## Rush Tactics

### Lane Blocking
- Position in the path between QB and likely targets
- Force QB to throw around you or pump fake
- Even without getting to QB, disrupts timing

### Direct Pressure
- Move toward QB position
- Force rushed decisions
- Collapse pocket

### Contain
- Maintain outside leverage
- Don't let QB escape to sideline
- Keep QB in pocket for coverage to work

## Strategic Considerations

### Lineman Limitations
- Defensive linemen have restricted movement zones
- Must work within constraints
- Position early in throwing lanes

### Rush-Coverage Balance
- More rushers = more pressure but less coverage
- Fewer rushers = better coverage but more time for QB
- Coordinate rush timing with coverage schemes
- The QB can throw from **tick 2 through tick 8** only — maximize pressure before tick 2 to disrupt the earliest throwing windows.

### Reading the Offense
- Track QB eye movement if available
- Anticipate throw timing based on route development
- Adjust lane blocking based on receiver positions
- Use the QB's `QbTargetPattern` from the AI state response each tick to prioritize lane blocks against likely targets.

## Situational Adjustments

### Quick Pass Anticipated
- Rush immediately to disrupt
- Lane blocking critical early

### Deep Pass Developing
- Maintain rush through play
- Eventually collapse on QB

### QB Scramble
- Contain outside
- Don't overcommit leaving escape route
