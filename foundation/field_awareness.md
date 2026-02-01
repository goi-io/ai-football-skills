# SKILL: Field Awareness

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Foundation  
**Required By:** All agents (Offense & Defense)

## Purpose
Understand the GOI playing field geometry, territories, and spatial constraints.

## Core Knowledge

### Field Overview
- The GOI field is a grid-based rectangular playing surface
- The field has a clear center/origin point
- All cells on the grid are valid playing positions (no out-of-bounds)

### Territories
- **Offensive Territory:** The side where offense starts
- **Defensive Territory:** The side offense tries to advance into
- **Line of Scrimmage:** The dividing line between territories

### Visual Concept
```
[DEFENSIVE TERRITORY - Offense scores here]
        ↑ Offense advances north ↑
═══════════ LINE OF SCRIMMAGE ═══════════
        ↓ Offense starts south ↓
[OFFENSIVE TERRITORY - Offense starts here]
```

### Hotspot Zones
The field contains special scoring zones:
- **High-value zones** at key field positions offer bonus points
- **Penalty zones** where ball loss results in point deduction
- Strategic agents should know hotspot locations from game state

## Strategic Considerations

### Territory Awareness
- Track which territory the ball carrier is in
- Monitor distance from line of scrimmage
- Understand scoring implications of each territory

### Positional Analysis
- Evaluate player positions relative to hotspots
- Assess coverage gaps based on field position
- Plan routes considering field geometry

### Zone Classification
- **Safe zones:** Low contest areas for possession
- **Contested zones:** Multiple players competing
- **High-value zones:** Scoring opportunities
- **Danger zones:** Risk of turnover or point loss

## Agent Response Pattern

When analyzing field positions, agents should consider:
1. Current territory of ball/players
2. Distance to scoring opportunities
3. Nearby threats and defenders
4. Optimal path to high-value zones
