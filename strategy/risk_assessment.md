# SKILL: Risk Assessment

> **⚠️ INTELLECTUAL PROPERTY NOTICE**  
> GOI™ is proprietary technology owned by **GOI.IO LLC**. Patent Pending: US 63/970524  
> This documentation provides strategic guidance only. Game engine mechanics are proprietary.  
> © 2024-2026 GOI.IO LLC. All rights reserved.

**Category:** Strategy  
**Required By:** All agents (Offense & Defense)

## Purpose
Evaluate play options considering probability of success and failure consequences.

## Risk Factors

### Offensive Risks
- **Interception:** Pass caught by defender
- **Neutralization:** Ball carrier tackled
- **Turnover position:** Losing ball in bad location
- **Time loss:** Wasted ticks on failed plays
- Factor in the QB's `QbTargetPattern` from the AI state response each tick when evaluating pass risk.

### Defensive Risks
- **Big play given up:** Offense scores many points
- **Coverage breakdown:** Receiver gets wide open
- **Overcommitment:** Gamble fails, offense benefits
- **Hotspot exposure:** High-value areas unprotected
- Use the QB's `QbTargetPattern` from the AI state response each tick to gauge likely target windows when choosing coverage risk.

## Risk Categories

### High Risk / High Reward
- Deep passes to hotspot zones
- Aggressive blitzes
- Interception attempts

**When appropriate:**
- Behind in score
- Running out of time
- Favorable matchup

### Low Risk / Low Reward
- Short, safe passes
- Conservative coverage
- Standard blocking assignments

**When appropriate:**
- Ahead in score
- Plenty of time
- Protecting lead

### Moderate Risk
- Medium-depth passes
- Balanced coverage
- Selective engagement

**When appropriate:**
- Close game
- Standard situations
- Balanced approach needed

## Expected Value Analysis

### Concept
- Weight outcomes by probability
- Positive outcomes add value
- Negative outcomes subtract value
- Choose options with best expected value

### Factors
- Probability of success
- Value of success
- Probability of failure
- Cost of failure

## Situational Risk Management

### Score Context
- **Large lead:** Minimize risk
- **Large deficit:** Maximize risk
- **Close game:** Standard risk tolerance

### Time Context
- **Early play:** Full range of options
- **Mid play:** Time pressure rising
- **Late play:** Limited options, urgency

### Game Context
- **Important play:** Consider stakes
- **Recovery possible:** Risk more acceptable
- **No recovery:** Risk less acceptable

## Strategic Principles

### Risk Tolerance Profiles
- **Conservative:** Prioritize safety, accept smaller gains
- **Balanced:** Trade off risk and reward appropriately
- **Aggressive:** Pursue big plays despite risk

### Adapting Risk
- Adjust based on situation
- Don't be predictable
- Read opponent's tendencies

### Team Capabilities
- Higher-skilled teams can take more risks
- Know your strengths and weaknesses
- Play to your advantages
