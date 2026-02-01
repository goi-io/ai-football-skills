# SKILL: Movement Planning

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Foundation  
**Required By:** All agents (Offense & Defense)

## Purpose
Plan player movements and understand mobility constraints.

## Movement Fundamentals

### Movement Rate
- Players can move **one grid cell per tick** (in any direction)
- Players may stay in place
- Movement is constrained by grid boundaries

### Movement Directions
Players can move in cardinal and diagonal directions:
- **Cardinal:** North, South, East, West
- **Diagonal:** NE, NW, SE, SW
- **Stay:** Remain in current position

### Movement Constraints

#### Grid Boundaries
- Players cannot move outside the field boundaries
- The game state provides boundary information

#### Position-Based Constraints
- Certain positions (linemen) have restricted movement zones
- These players cannot advance beyond designated areas
- Agents receive constraint violations from the game engine

## Strategic Movement

### Offensive Movement Goals
| Situation | Strategy |
|-----------|----------|
| Ball carrier | Advance toward scoring zones |
| Route running | Create separation, reach targets |
| Blocking | Position to protect teammates |

### Defensive Movement Goals
| Situation | Strategy |
|-----------|----------|
| Coverage | Shadow assigned receiver |
| Pursuit | Angle toward ball carrier |
| Pass rush | Move toward QB or blocking lanes |

## Path Planning Concepts

### Reachability
- Determine if a player can reach a target position
- Consider time remaining and distance
- Account for movement constraints

### Optimal Paths
- Direct paths are fastest when unobstructed
- Diagonal movement can be efficient
- Obstacles may require alternate routing

### Pursuit Angles
- Defenders should cut off ball carrier paths
- Anticipate opponent's likely movement
- Position to intercept rather than chase

## Agent Decision Making

When planning movement, consider:
1. **Target position:** Where does this player need to go?
2. **Time available:** Can they reach it in time?
3. **Obstacles:** Are there blocking players in the way?
4. **Constraints:** Does this position have movement limits?
5. **Priority:** Is this move more important than alternatives?
