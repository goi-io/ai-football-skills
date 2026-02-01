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

### Corner Hotspots (±10 points)
| Location | Coordinates | Offense Value | Defense Value |
|----------|-------------|---------------|---------------|
| Northwest | (-5, +5) | +10 | -10 |
| Northeast | (+5, +5) | +10 | -10 |
| Southwest | (-5, -5) | -10 | +10 |
| Southeast | (+5, -5) | -10 | +10 |

### Prime Hotspots (±20 points)
| Location | Coordinates | Offense Value | Defense Value |
|----------|-------------|---------------|---------------|
| North Center | (0, +5) | +20 | -20 |
| South Center | (0, -5) | -20 | +20 |

### Hotspot Map
```
       -5    -4    -3    -2    -1     0    +1    +2    +3    +4    +5
      ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  +5  │ +10 │     │     │     │     │ +20 │     │     │     │     │ +10 │  ← Offense Scoring Zone
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +4  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +3  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +2  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  +1  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
   0  │     │     │     │     │     │  ●  │     │     │     │     │     │  ← Line of Scrimmage
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -1  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -2  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -3  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -4  │     │     │     │     │     │     │     │     │     │     │     │
      ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -5  │ -10 │     │     │     │     │ -20 │     │     │     │     │ -10 │  ← Defense Scoring Zone
      └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
            (Values shown from Offense perspective)
```

### Hotspot Rules
1. **Ball carrier must reach hotspot** to score bonus points
2. **Neutralization denies bonus:** If tackled ON a hotspot, score = 0
3. **Squatting penalty:** Staying on a hotspot for consecutive ticks = penalty
4. **Prime hotspots** (center) are worth 2x corner hotspots

## Strategic Considerations

### Offensive Strategy
- **Primary targets:** North hotspots (+10, +20, +10 at Y=+5)
- **Avoid:** South hotspots (negative points for offense)
- **Best path:** Advance Y while avoiding tackles
- **Prime target:** (0, +5) for maximum +20 points

### Defensive Strategy
- **Protect:** North hotspots to deny offense bonus points
- **Force turnovers:** Near south hotspots for +10/+20 points
- **Coverage priority:** Prime hotspot (0, +5) highest priority

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
    """Returns point value for offense. Negate for defense."""
    if not is_hotspot(x, y):
        return 0
    if x == 0:  # Prime hotspot
        return 20 if y == 5 else -20
    else:  # Corner hotspot
        return 10 if y == 5 else -10
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
2. **6 hotspots total:** 4 corners (±10 pts) + 2 prime (±20 pts)
3. **Offense scores north** (Y=+5), **Defense scores south** (Y=-5)
4. **Prime hotspot (0, +5)** is the highest-value target
5. **Neutralization on hotspot = 0 points** (critical rule)
