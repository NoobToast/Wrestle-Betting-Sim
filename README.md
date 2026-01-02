# Wrestle-Betting-Sim# Wrestling Simulation - Complete Variable Reference

## FIGHTER STATS (Core Attributes)

| Variable | Default Range | What It Changes | Formula/Impact |
|----------|--------------|-----------------|----------------|
| **fighter_power** | 45-85 | • STRIKE damage<br>• SLAM damage<br>• Method odds (slight KO bias) | • STRIKE: `6 + (power × 0.18)`<br>• SLAM: `8 + (power × 0.22)` |
| **fighter_stamina** | 45-85 | • Health pool (HP)<br>• Finish gate (50% depletion)<br>• Emergency finishes at 0 | • At 0 stamina: **8× finish multiplier**<br>• 50% depleted = finishes enabled<br>• Injury reduces by **90%** |
| **fighter_technique** | 45-85 | • Hit accuracy<br>• Botch reduction<br>• SUBMISSION damage<br>• Method odds (SUB bias) | • Accuracy: `55% + (tech/200)` = max 92.5%<br>• Botch: `-(tech/300)`<br>• SUB: `5 + (tech × 0.20)`<br>• Injury reduces by **75%** |
| **fighter_charisma** | 45-85 | • Random damage bonus<br>• Crowd pop moments | • `(charisma/350)` chance for +2 damage<br>• No direct finish impact |
| **fighter_durability** | 45-85 | • Damage resistance<br>• Survivability | • Reduces incoming damage: `-(dur × 0.12)`<br>• Min 1 damage always gets through<br>• Injury reduces by **70%** |
| **fighter_aggression** | 45-85 | • Finisher attempt rate<br>• Botch chance (risk/reward)<br>• DQ probability | • Finisher chance: `4% + (aggr/300)` = max 12%<br>• Botch: `+(aggr/250)`<br>• DQ: `(aggr_A + aggr_B) / 5000` |

---

## MATCH RULES (Pacing & Structure)

| Variable | Default Value | What It Changes | Notes |
|----------|--------------|-----------------|-------|
| **max_exchanges** | 32 | • Maximum exchanges before time limit | • If reached = DRAW<br>• Typical range: 24-40 |
| **min_exchanges_before_finish** | 12 | • Minimum exchanges before finish allowed | • Prevents instant finishes<br>• Auto-drops to 0 at 0 stamina |
| **chaos** | 0.65 | • DQ probability<br>• Match unpredictability | • Range: 0.0 (clean) to 1.0 (chaos)<br>• DQ chance: `min(10%, chaos × 0.18 + aggr_factor)` |
| **pin_allowed** | true | • Enable/disable pinfall finishes | • If false, no PIN finishes possible |
| **sub_allowed** | true | • Enable/disable submission finishes | • If false, no SUB finishes possible |
| **dq_allowed** | true | • Enable/disable disqualifications | • If false, no DQ finishes possible |
| **finish_gates_dropped** | false | • Emergency finish mode | • Auto-sets TRUE when stamina hits 0<br>• Bypasses min_exchanges check |

---

## HIDDEN MODIFIERS (Intel & Fixes)

| Variable | Default/Chance | Range | What It Changes |
|----------|---------------|-------|-----------------|
| **injury_a** | 30% chance | 0.05 - 0.18 | • Reduces ALL Fighter A stats before match:<br>&nbsp;&nbsp;- Stamina: **-90%**<br>&nbsp;&nbsp;- Technique: **-75%**<br>&nbsp;&nbsp;- Durability: **-70%**<br>&nbsp;&nbsp;- Power: **-60%** |
| **injury_b** | 30% chance | 0.05 - 0.18 | • Same as injury_a but for Fighter B |
| **ref_bias_a** | 25% chance | -0.12 to +0.12 | • Positive = favors Fighter A<br>• Negative = favors Fighter B<br>• Multiplies ALL finish probabilities:<br>&nbsp;&nbsp;- `ko_a × (1.0 - bias)`<br>&nbsp;&nbsp;- `ko_b × (1.0 + bias)` |
| **fix_winner** | 12% chance | "A" or "B" | • Predetermined winner for storylines |
| **fix_strength** | If fix active | 0.08 - 0.20 | • Bonus added to LOSER'S finish probability<br>• Tilts match toward scripted outcome |

**Intel Generation:**
- ≥10% injury = shows intel hint (55% chance to show 1-2 hints)
- Hints reveal injuries, ref bias, or fixes to player

---

## COMBAT MECHANICS (Per Exchange)

### Accuracy & Failure

| Mechanic | Formula | Range | Notes |
|----------|---------|-------|-------|
| **Base Accuracy** | `55% + (technique/200)` | 55% - 92% | • Max capped at 92%<br>• Miss = 0 damage, 5 stamina cost |
| **Botch Chance** | `5% + (aggr/250) - (tech/300)` | 2% - 18% | • Botch = 0 damage, 8 stamina cost<br>• High aggr + low tech = more botches |

### Damage Calculation

| Move Type | Base Damage | Stat Scaling | Stamina Cost | Notes |
|-----------|-------------|--------------|--------------|-------|
| **STRIKE** | 6 | `+ (power × 0.18)` | 5 | Quick attacks |
| **SLAM** | 8 | `+ (power × 0.22)` | 6 | Power moves (highest base) |
| **SUB** | 5 | `+ (technique × 0.20)` | 6 | Technical moves |
| **FINISHER** | Base × 1.6 | (any type above) | Base + 3 | **60% damage boost** |

### Damage Modifiers

| Modifier | Effect | Formula |
|----------|--------|---------|
| **Defense Reduction** | Durability reduces damage | `damage - (durability × 0.12)` |
| **Minimum Damage** | Always at least 1 | `max(1, calculated_damage)` |
| **Charisma Bonus** | Random crowd pop | `(charisma/350)` chance for +2 damage |

### Stamina Costs Summary

| Outcome | Stamina Cost (Attacker) |
|---------|------------------------|
| Botch | 8 |
| Miss | 5 |
| STRIKE hit | 5 |
| SLAM hit | 6 |
| SUB hit | 6 |
| Finisher hit | Base + 3 (8-9 total) |

---

## FINISH PROBABILITY SYSTEM

### Base Finish Chances (Scales with Exhaustion)

| Finish Type | Starting % | Max % | Exhaustion Formula |
|-------------|-----------|-------|-------------------|
| **KO** | 0.4% | 35% | `min(35%, 0.4% + (exhaustion × 16%))` |
| **PIN** | 0.3% | 35% | `min(35%, 0.3% + (exhaustion × 14%))` |
| **SUB** | 0.2% | 30% | `min(30%, 0.2% + (exhaustion × 12%))` |
| **DQ** | (chaos-based) | 10% | `min(10%, chaos × 0.18 + aggr_factor)` |

**Exhaustion Calculation:**
```
exhaustion = 1.0 - (current_stamina / starting_stamina)

0.0 = Fresh (100% stamina)
0.5 = Half depleted (50% stamina) ← FINISH GATES DROP
1.0 = Completely exhausted (0% stamina) ← 8× MULTIPLIER
```

### Critical Thresholds

| Stamina % | Effect |
|-----------|--------|
| **100% - 51%** | No finishes possible (above depletion gate) |
| **50%** | Finish gates drop, finishes now possible |
| **0%** | **8× multiplier** to ALL finish chances<br>Emergency finish mode activated |

### Finish Priority Order (Checked Each Exchange)

1. **KO** (Fighter A check)
2. **KO** (Fighter B check)
3. **PIN** (Fighter A, if allowed)
4. **PIN** (Fighter B, if allowed)
5. **SUB** (Fighter A, if allowed)
6. **SUB** (Fighter B, if allowed)
7. **DQ** (Random loser, if allowed)

---

## EXAMPLE CALCULATIONS

### Example Fighter (65 Power, 70 Stamina, 75 Technique, 60 Durability)

**Hit Accuracy:**
```
55% + (75/200) = 55% + 37.5% = 92.5% (capped at 92%)
```

**Botch Chance:**
```
5% + (65/250) - (75/300) = 5% + 26% - 25% = 6%
```

**SLAM Damage:**
```
Base: 8 + (65 × 0.22) = 8 + 14.3 = 22.3
After defense (60 dur): 22.3 - (60 × 0.12) = 22.3 - 7.2 = 15.1 → 15 damage
```

**Finisher Attempt Chance:**
```
4% + (65/300) = 4% + 21.7% = 25.7% (capped at 12%) → 12%
```

### Example Finish Probability (Fighter at 25% stamina = 75% exhausted)

**KO Chance:**
```
Base: 0.4% + (0.75 × 16%) = 0.4% + 12% = 12.4%
With ref bias (+0.10): 12.4% × (1.0 - 0.10) = 11.16%
```

**If at 0% stamina:**
```
12.4% × 8 = 99.2% (capped at 60%) → Match ending very soon!
```

---

## BETTING ODDS ADJUSTMENTS

### Market Vigorish (Bookmaker Edge)

| Market Type | Vig % | Impact |
|-------------|-------|--------|
| Moneyline (ML) | 4-6% | Reduces player edge on straight win bets |
| Over/Under (OU) | 4% | Slight edge on totals markets |
| Method Markets | 6-10% | Higher edge on specific finish types |

### Totals Line Calculation

**Base O/U Line:** 18.5 exchanges (default)

**Factors that increase OVER probability:**
- Close fight (pA near 0.5) → `+18%`
- Lower combined aggression → `up to +4%`
- Lower combined power → `up to +3%`

**Factors that increase UNDER probability:**
- Either fighter injured → `-20%` per injury
- High aggression → `up to -4%`
- High power → `up to -3%`

---

## QUICK REFERENCE - STAT PRIORITIES

| Build Type | Priority Stats | Why |
|------------|---------------|-----|
| **Glass Cannon** | Power 85, Aggr 85, Tech 45 | Max damage, high finisher rate, risky |
| **Tank** | Stamina 85, Dur 85, Tech 70 | Survives long, consistent accuracy |
| **Technical Master** | Tech 85, Stam 75, Dur 70 | 92% accuracy, SUB specialist, minimal botches |
| **Balanced** | All stats 65-70 | No weaknesses, adaptable |
| **Chaos Agent** | Aggr 85, Char 85, Power 75 | Exciting, unpredictable, crowd pops |

---

## MODIFIER COMBINATIONS (Edge Cases)

| Scenario | Combined Effect |
|----------|----------------|
| **Injured + 0 Stamina** | Injury already reduced stamina by 90%, so hits 0 VERY fast → Near-instant finish |
| **Fix + Ref Bias (same side)** | Stacks additively → Overwhelming advantage (can create 80%+ finish odds) |
| **High Chaos + Both High Aggr** | DQ chance approaches 10% per exchange check |
| **Both at 0 Stamina** | Immediate finish for whoever is "less negative" (closer to 0) |
