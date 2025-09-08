# Medieval III Diplomatic Systems Implementation Specification

## Overview

This document provides technical implementation details for the enhanced diplomatic systems in Medieval III mod, focusing on the selected mechanics that integrate seamlessly with M2TWEOP's existing framework while maintaining engaging medieval political gameplay.

## Selected Systems for Implementation

1. **Dynamic Succession Crisis Intervention (#2)**
2. **Conditional Treaty Webs (#4)**
3. **Economic Sanctions & Trade Partnerships (#6)**
4. **Hostage Exchange & Noble Prisoner Systems (#7)**
5. **Regional Council Systems (#10)**
6. **Crusade & Jihad Politics (#11)**
7. **Border Incident Escalation (#12) with Player Control**
8. **Seasonal Diplomatic Windows (#13)**
9. **Royal Court Influence System (#14)**
10. **Reputation & Honor Systems (#15)**
11. **Automatic Relationship Modifiers (#16)**

---

## 1. Border Incident Escalation System with Player Control

### Technical Implementation

#### Database Schema Extensions
```sql
-- Border incident tracking
CREATE TABLE border_incidents (
    incident_id INTEGER PRIMARY KEY,
    source_faction_id INTEGER,
    target_faction_id INTEGER,
    incident_type VARCHAR(50),
    severity_level INTEGER,
    gold_cost INTEGER,
    plausible_deniability BOOLEAN,
    discovery_chance INTEGER,
    escalation_chance INTEGER,
    turn_occurred INTEGER
);

-- Village raid capabilities
CREATE TABLE village_raid_settings (
    settlement_id INTEGER PRIMARY KEY,
    raid_encouragement_level INTEGER, -- 0-3 (None/Passive/Active/Aggressive)
    monthly_funding INTEGER,
    militia_strength INTEGER,
    last_raid_turn INTEGER
);
```

#### M2TWEOP Integration
**File: `data/world/maps/campaign/imperial_campaign/descr_settlements.txt`**
```
; Add border raid capability to border settlements
settlement {
    level village
    region_id 45
    ; ... existing settlement data ...
    border_raid_capability {
        max_funding 500
        base_success_rate 60
        detection_modifier -10
    }
}
```

#### Lua Scripting Framework
**File: `data/scripts/campaign/border_incidents.lua`**
```lua
-- Border incident management
function InitiateBorderRaid(settlement_id, target_faction, funding_level)
    local settlement = GetSettlement(settlement_id)
    local raid_strength = CalculateRaidStrength(settlement, funding_level)
    local success_chance = CalculateSuccessChance(settlement, target_faction)
    local detection_chance = CalculateDetectionChance(funding_level)
    
    if success_chance > math.random(1, 100) then
        ExecuteSuccessfulRaid(settlement, target_faction, raid_strength)
        if detection_chance > math.random(1, 100) then
            RevealRaidSource(settlement.faction, target_faction)
        end
    else
        ExecuteFailedRaid(settlement, target_faction)
    end
end

function CalculateEscalationChance(incident_severity, faction_relations)
    local base_chance = incident_severity * 5
    local relation_modifier = (faction_relations + 100) / 10
    return math.min(base_chance + relation_modifier, 85)
end

-- Border incident response system
function HandleBorderIncidentResponse(incident_id, response_type)
    local incident = GetBorderIncident(incident_id)
    
    if response_type == "ignore" then
        -- Relations continue deteriorating
        ModifyFactionRelations(incident.source_faction, incident.target_faction, -2)
    elseif response_type == "demand_explanation" then
        -- Diplomatic penalty if obvious involvement
        TriggerDiplomaticEvent("border_explanation_demand", incident)
    elseif response_type == "retaliate" then
        -- Launch counter-raids
        EnableCounterRaidOptions(incident.target_faction, incident.source_faction)
    elseif response_type == "compensate" then
        -- Pay gold to cool tensions
        DeductFactionTreasury(incident.target_faction, incident.gold_cost * 2)
        ModifyFactionRelations(incident.source_faction, incident.target_faction, 5)
    elseif response_type == "military_response" then
        -- Station troops, risk escalation
        TriggerMilitaryResponseEvent(incident)
    end
end
```

#### UI Implementation
**Border Control Panel (Settlement Screen Extension)**
- Raid Encouragement Slider: 4 positions (None/Passive/Active/Aggressive)
- Monthly Funding Input: 50-500 gold allocation
- Plausible Deniability Indicator: Visual representation of detection risk
- Recent Incident Log: List of successful/failed raids with outcomes

**Incident Response Popup**
- Incident description with severity assessment
- 5 response options with predicted consequences
- Intelligence assessment (if spy network exists in region)
- Diplomatic relationship impact preview

---

## 2. Seasonal Diplomatic Windows

### Technical Implementation

#### Season Management System
```lua
-- Seasonal diplomatic bonuses
local SeasonalModifiers = {
    spring = {
        trade_route_cost_reduction = 0.5,
        agricultural_contract_bonus = 15,
        guild_negotiation_bonus = 10
    },
    summer = {
        military_supply_premium = 25,
        mercenary_contract_bonus = 20,
        siege_equipment_availability = true
    },
    autumn = {
        resource_preorder_bonus = 10,
        luxury_goods_bonus = 15,
        harvest_agreement_bonus = 20
    },
    winter = {
        longterm_pact_bonus = 25,
        infrastructure_partnership_bonus = 30,
        craft_guild_bonus = 15
    }
}

function GetCurrentSeasonModifiers()
    local current_season = GetCurrentSeason()
    return SeasonalModifiers[current_season]
end

-- Seasonal trade agreements
function CreateSeasonalTradeAgreement(faction1, faction2, resource_type, quantity, season)
    local agreement = {
        id = GenerateAgreementID(),
        buyer_faction = faction1,
        seller_faction = faction2,
        resource = resource_type,
        quantity = quantity,
        delivery_season = season,
        price_locked = CalculateSeasonalPrice(resource_type, season),
        status = "pending"
    }
    
    AddSeasonalAgreement(agreement)
    return agreement
end
```

#### Database Extensions
```sql
-- Seasonal trade agreements
CREATE TABLE seasonal_agreements (
    agreement_id INTEGER PRIMARY KEY,
    buyer_faction_id INTEGER,
    seller_faction_id INTEGER,
    resource_type VARCHAR(50),
    quantity INTEGER,
    locked_price INTEGER,
    delivery_season VARCHAR(10),
    agreement_turn INTEGER,
    status VARCHAR(20),
    fulfilled_quantity INTEGER DEFAULT 0
);

-- Guild partnerships
CREATE TABLE guild_partnerships (
    partnership_id INTEGER PRIMARY KEY,
    faction1_id INTEGER,
    faction2_id INTEGER,
    guild_type VARCHAR(50),
    established_season VARCHAR(10),
    partnership_level INTEGER,
    annual_benefits TEXT
);
```

#### Pre-Order System Implementation
**Craft Commission Interface**
- Select partner faction's craftsmen
- Choose item type and quantity
- Set delivery timeline (1-4 seasons)
- Lock in price based on current relations and season
- Track commission progress with status updates

**Resource Pre-Order System**
- Browse available resources from partner factions
- Reserve quantities for future delivery
- Seasonal price modifiers applied automatically
- Payment schedule: 50% upfront, 50% on delivery

---

## 3. Royal Court Influence System + Spy Integration

### Technical Implementation

#### Court Advisor System
```sql
-- Court advisor tracking
CREATE TABLE court_advisors (
    advisor_id INTEGER PRIMARY KEY,
    faction_id INTEGER,
    advisor_type VARCHAR(50), -- chancellor, coin_master, general, priest, spymaster, noble_rep
    name VARCHAR(100),
    influence_level INTEGER,
    personality_traits TEXT,
    corruption_level INTEGER,
    loyalty_level INTEGER,
    current_agenda TEXT,
    weaknesses TEXT
);

-- Influence operations tracking
CREATE TABLE influence_operations (
    operation_id INTEGER PRIMARY KEY,
    target_advisor_id INTEGER,
    source_faction_id INTEGER,
    operation_type VARCHAR(50), -- bribery, blackmail, marriage, gifts
    gold_cost INTEGER,
    success_chance INTEGER,
    influence_gained INTEGER,
    exposure_risk INTEGER,
    turn_executed INTEGER
);
```

#### Court Influence Lua Framework
```lua
-- Court advisor influence system
function CalculateAdvisorInfluence(advisor_id, diplomatic_action)
    local advisor = GetCourtAdvisor(advisor_id)
    local base_influence = advisor.influence_level
    local personality_modifier = GetPersonalityModifier(advisor, diplomatic_action)
    local corruption_modifier = advisor.corruption_level > 50 and 10 or 0
    
    return base_influence + personality_modifier + corruption_modifier
end

function ExecuteInfluenceOperation(operation_type, target_advisor, source_faction, resources)
    local operation = CreateInfluenceOperation(operation_type, target_advisor, source_faction)
    local success_chance = CalculateOperationSuccess(operation, resources)
    
    if success_chance > math.random(1, 100) then
        ApplyInfluenceGain(operation)
        if operation.exposure_risk > math.random(1, 100) then
            TriggerExposureConsequences(operation)
        end
    else
        HandleOperationFailure(operation)
    end
end

-- Assassination integration
function AssassinateCourtAdvisor(target_advisor_id, assassin_agent)
    local advisor = GetCourtAdvisor(target_advisor_id)
    local assassination_difficulty = CalculateAssassinationDifficulty(advisor)
    local success_chance = CalculateAssassinationSuccess(assassin_agent, assassination_difficulty)
    
    if success_chance > math.random(1, 100) then
        RemoveCourtAdvisor(target_advisor_id)
        TriggerSuccessionEvent(advisor.faction_id, advisor.advisor_type)
        ModifySpyEffectiveness(advisor.faction_id, -20, 6) -- Court investigation
        GenerateReplacementAdvisor(advisor.faction_id, advisor.advisor_type)
    else
        ExposeAssassinationAttempt(assassin_agent)
    end
end
```

#### Court Intrigue Events
**Monthly Court Reports**
```lua
function GenerateMonthlyCourtReport(faction_id)
    local court = GetFactionCourt(faction_id)
    local report = {
        dominant_advisor = GetMostInfluentialAdvisor(court),
        current_conflicts = GetAdvisorConflicts(court),
        policy_recommendations = GetPolicyRecommendations(court),
        foreign_influence_detected = DetectForeignInfluence(court)
    }
    
    DisplayCourtReport(faction_id, report)
end
```

---

## 4. Integration Systems

### Cross-System Data Flow

#### Border Incidents → Court Influence
```lua
function HandleBorderIncidentCourtResponse(incident_id)
    local incident = GetBorderIncident(incident_id)
    local target_court = GetFactionCourt(incident.target_faction)
    
    -- High General gains influence advocating military response
    ModifyAdvisorInfluence(target_court.high_general, 10)
    
    -- Master of Coin opposes expensive military action
    ModifyAdvisorInfluence(target_court.master_of_coin, -5)
    
    -- Update diplomatic options based on court balance
    UpdateDiplomaticOptions(incident.target_faction, incident.source_faction)
end
```

#### Seasonal Windows → Court Politics
```lua
function ProcessSeasonalCourtInfluence(faction_id, season)
    local court = GetFactionCourt(faction_id)
    local seasonal_modifiers = GetCurrentSeasonModifiers()
    
    if season == "autumn" then
        -- Master of Coin gains influence during trade season
        ModifyAdvisorInfluence(court.master_of_coin, seasonal_modifiers.harvest_agreement_bonus)
    elseif season == "summer" then
        -- High General gains influence during campaign season
        ModifyAdvisorInfluence(court.high_general, seasonal_modifiers.military_supply_premium)
    end
    
    UpdateSeasonalDiplomaticOptions(faction_id, season)
end
```

---

## 5. UI/UX Implementation

### Main Diplomatic Interface Extensions

#### Court Influence Panel
- **Advisor Grid Display**: 6 advisor portraits with influence meters
- **Influence Operations Menu**: Bribery, gifts, blackmail options with success chances
- **Court Politics Web**: Visual network showing advisor relationships and conflicts
- **Monthly Report Archive**: Historical court reports with trend analysis

#### Border Management Interface
- **Regional Border Map**: Zoom view of border settlements with raid capabilities
- **Incident Timeline**: Chronological list of border events with escalation tracking
- **Response Planning**: Pre-configured response templates for different incident types
- **Intelligence Integration**: Spy report integration showing enemy border activities

#### Seasonal Diplomatic Calendar
- **Season Overview**: Current season bonuses and available diplomatic actions
- **Agreement Tracker**: Active pre-orders and seasonal contracts with delivery status
- **Partnership Management**: Guild relationships and infrastructure projects
- **Seasonal Events**: Upcoming diplomatic assemblies and court meetings

---

## 6. Development Timeline

### Phase 1: Core Infrastructure (Months 1-2)
- Database schema implementation
- Basic Lua scripting framework
- M2TWEOP integration hooks
- Core UI components

### Phase 2: System Implementation (Months 3-5)
- Border incident system
- Court influence mechanics
- Seasonal diplomatic windows
- Cross-system integration

### Phase 3: Advanced Features (Months 6-7)
- Spy/assassin integration
- Complex court intrigue events
- Advanced UI components
- AI behavior modifications

### Phase 4: Testing & Balancing (Month 8)
- Gameplay balancing
- AI testing and adjustment
- Performance optimization
- Community feedback integration

---

## 7. Balancing Considerations

### Economic Balance
- Border raid costs: 50-500 gold (significant but not crippling)
- Court influence operations: 100-1000 gold depending on target and method
- Seasonal agreement benefits: 10-30% bonuses (meaningful but not overpowered)

### Diplomatic Balance
- Relationship impact ranges: -30 to +20 per significant action
- Escalation chances: 15-40% based on severity and existing relations
- Court influence effects: 5-25% modification to diplomatic success rates

### Gameplay Balance
- Multiple viable strategies: Military pressure, economic influence, court intrigue
- Meaningful choices: Each diplomatic action has clear trade-offs
- Long-term consequences: Actions affect relationships for 10+ turns

This implementation specification provides a comprehensive technical framework for enhancing Medieval III's diplomatic systems while maintaining game balance and medieval authenticity.