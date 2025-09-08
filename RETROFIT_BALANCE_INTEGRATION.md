# Retrofit Mod Balance Integration Guide

## Overview
This document outlines the safe integration of Retrofit Mod balance improvements into Medieval III without downloading potentially compromised files. We'll implement the known balance changes manually.

## Retrofit Mod Balance Improvements (Known Changes)

### 1. Unit Rebalancing
- **Upkeep Cost Reduction**: Approximately halved army maintenance costs
- **Enhanced Recruitment**: Nearly all faction units made recruitable (including generals)
- **AI Unit Recruitment**: Improved AI unit composition and recruitment logic

### 2. Campaign AI Improvements
- **Strategic AI**: Better long-term planning and expansion patterns
- **Diplomatic AI**: More realistic diplomatic behavior
- **Economic AI**: Improved resource management and building construction

### 3. Technical Improvements
- **Boiling Oil**: Fixed implementation for siege defense
- **Bug Fixes**: Various engine-level fixes and stability improvements

## Manual Implementation Strategy

### Phase 1: Upkeep Cost Modifications
**File to Modify**: `data/export_descr_unit.txt`

```
; Example changes to unit costs
stat_cost 1, 580, 290, 75, 100, 580  ; Original
stat_cost 1, 580, 145, 75, 100, 580  ; Retrofit (halved upkeep)
```

### Phase 2: AI Recruitment Improvements
**File to Modify**: `data/descr_sm_factions.txt`

```
; Enhanced AI recruitment pools
faction	england
ai_do_not_attack_faction	papal_states
```

### Phase 3: Building Cost Adjustments
**File to Modify**: `data/export_descr_buildings.txt`

```
; Reduced construction costs for key buildings
cost 400  ; Original
cost 300  ; Retrofit adjustment
```

## Attribution Requirements
From the original mod documentation:
> "Other mods may include content of the Retrofit Mod in whole or in part. Please acknowledge credit to the Retrofit Mod when Retrofit Mod content is used in other mods."

## Implementation Status

### Completed:
- [ ] Upkeep cost reductions
- [ ] AI recruitment improvements
- [ ] Building cost adjustments
- [ ] Boiling oil fixes

### Pending:
- [ ] Battle map integration
- [ ] Advanced AI tweaks
- [ ] Bug fixes implementation

## Legal Compliance
- **License**: Community-permissive with attribution requirement
- **Attribution**: "Balance improvements adapted from Retrofit Mod by Unspoken Knight"
- **Usage**: Allowed for derivative works per community permissions

## Next Steps
1. Implement upkeep cost reductions across all units
2. Apply AI recruitment improvements
3. Test balance changes with M2TWEOP
4. Document all modifications for transparency