# SKILL: Field Awareness

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Foundation  
**Required By:** All agents (Offense & Defense)

## Purpose
Understand the GOI playing field geometry, territories, hotspot locations, and spatial constraints.

## Field Geometry

### Dimensions
- **Grid Size:** 11 × 11 cells
- **Coordinate Range:** X: -5 to +5, Y: -5 to +5
- **Origin (0,0):** Center of the field
- **Total Cells:** 121 playable positions

### Coordinate System
```
       X-AXIS
    -5  ...  0  ...  +5
   ┌─────────────────────┐
+5 │ NW      N       NE  │  ← Defense End Zone (Offense scores here)
   │                     │
 0 │ W    CENTER    E    │  ← Line of Scrimmage
   │                     │
-5 │ SW      S       SE  │  ← Offense Start Zone
   └─────────────────────┘
        Y-AXIS
```

**Direction Reference:**
- **+Y** = North (toward defense/scoring)
- **-Y** = South (toward offense start)
- **+X** = East (right)
- **-X** = West (left)

### Territories
- **Offensive Territory:** Y < 0 (south half)
- **Defensive Territory:** Y > 0 (north half)
- **Line of Scrimmage:** Y = 0 (center row)

## Hotspot Locations & Values

Hotspots are special scoring zones that award bonus points when the ball carrier reaches them.

### Scoring Rules
- **Defense:** All hotspot values are **absolute (positive)**. Defense always benefits from reaching any hotspot.
- **Offense:** North hotspots are positive, but **south hotspots are NEGATIVE** (penalties).

### Corner Hotspots (10 points)
| Location | Coordinates | Offense Value | Defense Value |
|----------|-------------|---------------|---------------|
| Northwest | (-5, +5) | **+10** | +10 |
| Northeast | (+5, +5) | **+10** | +10 |
| Southwest | (-5, -5) | **-10** ⚠️ | +10 |
| Southeast | (+5, -5) | **-10** ⚠️ | +10 |

### Prime Hotspots (20 points)
| Location | Coordinates | Offense Value | Defense Value |
|----------|-------------|---------------|---------------|
| North Center | (0, +5) | **+20** | +20 |
| South Center | (0, -5) | **-20** ⚠️ | +20 |

### Hotspot Map (Offense Perspective)
```
       -5    -4    -3    -2    -1     0    +1    +2    +3    +4    +5
      ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  +5  │ +10 │     │     │     │     │ +20 │     │     │     │     │ +10 │  ← OFFENSE TARGETS (+)
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +4  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +3  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +2  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +1  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
   0  │     │     │     │     │     │  ●  │     │     │     │     │     │  ← LINE OF SCRIMMAGE
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -1  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -2  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -3  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -4  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -5  │ -10 │     │     │     │     │ -20 │     │     │     │     │ -10 │  ← OFFENSE DANGER (−)
      └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
```

### Hotspot Map (Defense Perspective)
```
       -5    -4    -3    -2    -1     0    +1    +2    +3    +4    +5
      ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  +5  │ +10 │     │     │     │     │ +20 │     │     │     │     │ +10 │  ← DEFENSE BONUS (+)
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
   :  │                         ...                                     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
   0  │     │     │     │     │     │  ●  │     │     │     │     │     │  ← LINE OF SCRIMMAGE
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
   :  │                         ...                                     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -5  │ +10 │     │     │     │     │ +20 │     │     │     │     │ +10 │  ← DEFENSE BONUS (+)
      └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
        Defense ALWAYS scores positive (absolute values) at ALL hotspots
```

### Hotspot Rules
1. **Ball carrier must reach hotspot** to score bonus points
2. **Neutralization denies bonus:** If tackled ON a hotspot, score = 0
3. **Squatting penalty:** Staying on a hotspot for consecutive ticks = penalty
4. **Prime hotspots** (center) are worth 2x corner hotspots

## Strategic Considerations

### Offensive Strategy
- **Primary targets:** North hotspots (+10, +20, +10 at Y=+5)
- **⚠️ AVOID:** South hotspots = **NEGATIVE POINTS** for offense!
- **Best path:** Advance Y (north) while avoiding tackles
- **Prime target:** (0, +5) for maximum +20 points
- **Worst case:** Ball carrier at (0, -5) = **-20 points**

### Defensive Strategy
- **ALL hotspots are positive** for defense (absolute values)
- **Any interception** that reaches a hotspot scores points
- **North hotspots:** Worth pursuing if opportunity arises (+10/+20)
- **South hotspots:** Force offense ball carrier here for turnover + points
- **Coverage priority:** Prevent offense from reaching north hotspots

### Distance Calculations
```
Manhattan Distance = |x2 - x1| + |y2 - y1|
Diagonal Distance = max(|x2 - x1|, |y2 - y1|)  // Ticks to reach
```

**Example:** Player at (2, 1) to hotspot (5, 5)
- Manhattan: |5-2| + |5-1| = 3 + 4 = 7
- Diagonal/Ticks: max(3, 4) = 4 ticks to reach

### Zone Classification by Y-coordinate
| Y Range | Zone | Offense Priority | Defense Priority |
|---------|------|------------------|------------------|
| +5 | Scoring Zone | HIGHEST | CRITICAL |
| +3 to +4 | Red Zone | HIGH | HIGH |
| +1 to +2 | Midfield+ | MEDIUM | MEDIUM |
| 0 | Scrimmage | NEUTRAL | NEUTRAL |
| -1 to -2 | Midfield- | LOW | LOW |
| -3 to -4 | Safety Zone | AVOID | OPPORTUNITY |
| -5 | Danger Zone | CRITICAL AVOID | HIGHEST |

## Agent Decision Helpers

### Check if Position is Hotspot
```python
def is_hotspot(x, y):
    return (abs(x) == 5 and abs(y) == 5) or (x == 0 and abs(y) == 5)

def hotspot_value(x, y, side_of_ball):
    """
    Returns point value based on side of ball.
    - OFFENSE: North = positive, South = NEGATIVE
    - DEFENSE: ALL hotspots = POSITIVE (absolute values)
    """
    if not is_hotspot(x, y):
        return 0
    
    # Base value (absolute)
    base = 20 if x == 0 else 10
    
    if side_of_ball == 'defense':
        # Defense ALWAYS scores positive at any hotspot
        return base
    else:
        # Offense: north = positive, south = NEGATIVE
        return base if y == 5 else -base
```

### Nearest Hotspot Calculation
```python
def nearest_scoring_hotspot(x, y):
    """Find closest positive-value hotspot for offense."""
    targets = [(-5, 5), (0, 5), (5, 5)]  # North hotspots
    return min(targets, key=lambda t: max(abs(t[0]-x), abs(t[1]-y)))
```

## Key Takeaways

1. **Field is 11×11** with coordinates from -5 to +5 on both axes
2. **6 hotspots total:** 4 corners (10 pts) + 2 prime center (20 pts)
3. **OFFENSE:** North hotspots = +points, South hotspots = **−points (PENALTY)**
4. **DEFENSE:** ALL hotspots = **+points (absolute values, always positive)**
5. **Prime hotspot (0, +5)** is highest-value for offense (+20)
6. **Prime hotspot (0, -5)** is worst for offense (−20) but great for defense (+20)
7. **Neutralization on hotspot = 0 points** (critical rule)
