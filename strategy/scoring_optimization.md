# SKILL: Scoring Optimization

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Strategy  
**Required By:** All agents (Offense & Defense)

## Purpose
Maximize points through strategic positioning and play decisions.

## Scoring Concepts

### Offensive Scoring
- Points based on ball advancement
- Advancing into defensive territory scores points
- Losing ground into offensive territory loses points
- Hotspot zones provide bonus points

### Defensive Scoring
- Points from interceptions based on interception location
- Deeper interceptions (further from line) score more
- Hotspot zones provide bonus points for defense too

### Hotspot Zones
The field has special scoring locations:
- **High-value hotspots:** Significant bonus points
- **Standard hotspots:** Moderate bonus points
- **Penalty zones:** Point deductions for offense

Exact hotspot locations and values are provided in game state.

## Offensive Optimization

### Maximizing Gain
- Advance as far as possible
- Target hotspot zones when reachable
- Balance advancement vs. risk

### Risk vs. Reward
- Deep passes: Higher potential, higher risk
- Short passes: Lower potential, safer
- Consider time remaining and score situation
- The QB can throw from **tick 2 through tick 8** only — factor this window into pass play value calculations.
- Use the QB's `QbTargetPattern` from the AI state response each tick to identify reachable throw cells when weighing pass value.

### Hotspot Targeting
- Identify reachable hotspots
- Calculate probability of reaching them
- Weigh bonus value against interception risk

## Defensive Optimization

### Preventing Scores
- Keep ball carrier in low-scoring zones
- Protect hotspot areas
- Force negative plays when possible

### Interception Scoring
- Position for interception opportunities
- Deeper interceptions score more
- Balance ball-hawking vs. coverage
- Use the QB's `QbTargetPattern` from the AI state response each tick to prioritize high-value interception zones.

### Hotspot Defense
- Priority coverage of high-value zones
- Don't leave hotspots unprotected
- Force offense to less valuable areas

## Strategic Planning

### Score Differential
- **Ahead:** Play conservatively, protect lead
- **Behind:** Take risks for bigger plays
- **Close game:** Balanced approach

### Time Remaining
- **Early:** Standard optimization
- **Late:** Urgent optimization based on score

### Expected Value
- Consider probability of outcomes
- Weigh positive vs. negative scenarios
- Make decisions that maximize expected points
