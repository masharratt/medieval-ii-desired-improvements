# Medieval III Economic Mechanics Implementation Plan

## Executive Summary

This document provides a comprehensive implementation specification for six selected Medieval III economic mechanics that will be integrated into the existing Medieval II Total War modding framework. The mechanics are designed to enhance economic gameplay depth while maintaining compatibility with M2TWEOP capabilities and existing Medieval II systems.

## Selected Mechanics Overview

1. **Guild Monopoly Warfare (#4)** - Economic warfare through guild control
2. **Dynastic Marriage Economics (#6)** - Economic benefits through strategic marriages
3. **Port Taxation & Smuggling Networks (#9)** - Maritime trade control and black markets
4. **Resource Depletion & Discovery (#11)** - Dynamic resource availability
5. **Craft Specialization Bonuses (#13)** - Regional production advantages
6. **Basic Supply & Demand (#14)** - Dynamic pricing based on market conditions

---

## 1. Guild Monopoly Warfare (#4)

### Detailed Gameplay Mechanics

**Core System:**
- Guilds control specific trade goods within regions (silk, spices, dyes, etc.)
- Players can attempt hostile takeovers of guild operations
- Guild wars affect regional economy and diplomatic relations
- Monopolistic control provides significant economic advantages

**Player Interactions:**
- **Guild Acquisition**: Send merchant agents to attempt hostile takeovers
- **Guild Defense**: Station agents to protect existing guild operations
- **Economic Warfare**: Coordinate attacks on enemy guild networks
- **Diplomatic Leverage**: Use guild control to influence trade agreements

### Technical Implementation Requirements

**Building System Extensions:**
```
building guild_hall
{
    levels guild_outpost guild_chapter guild_headquarters
    {
        guild_outpost
        {
            capability
            {
                guild_control_resource silk 25
                guild_defense_bonus 1
                merchant_income_bonus 10
                recruitment_slots 1
            }
            cost 1500
            construction 3
            settlement_min town
        }
        
        guild_chapter  
        {
            capability
            {
                guild_control_resource silk 50
                guild_control_resource spices 25
                guild_defense_bonus 2
                merchant_income_bonus 25
                trade_route_income_bonus 15
                recruitment_slots 2
            }
            cost 4000
            construction 5
            settlement_min large_town
        }
        
        guild_headquarters
        {
            capability
            {
                guild_control_resource silk 75
                guild_control_resource spices 50
                guild_control_resource dyes 25
                guild_defense_bonus 3
                merchant_income_bonus 50
                trade_route_income_bonus 30
                guild_monopoly_warfare_enabled
                recruitment_slots 3
            }
            cost 8000
            construction 8
            settlement_min city
        }
    }
}
```

**New Character Types:**
```
; In export_descr_character_traits.txt
character guild_master
{
    type named character
    actions guild_acquisition, guild_defense, trade_negotiation
    wage_base 200
    wage_additional 50
    
    ancillaries guild_seal, trade_ledgers, merchant_contacts
    
    traits GuildInfluence, TradeExpertise, EconomicWarfare
}
```

**Resource Control System:**
```
; In descr_sm_resources.txt additions
guild_control_modifiers
{
    silk_monopoly      guild_income_multiplier 2.5    trade_value_bonus 8
    spices_monopoly    guild_income_multiplier 2.0    trade_value_bonus 6  
    dyes_monopoly      guild_income_multiplier 1.8    trade_value_bonus 5
    textiles_monopoly  guild_income_multiplier 1.6    trade_value_bonus 4
}
```

### Integration with M2TWEOP Framework

**Campaign Database Extensions (descr_campaign_db.xml):**
```xml
<guilds>
    <max_guilds_per_faction uint="12"/>
    <guild_acquisition_base_chance float="35.0"/>
    <guild_defense_bonus_per_level float="15.0"/>
    <monopoly_income_multiplier float="1.5"/>
    <guild_warfare_diplomacy_penalty int="-2"/>
    <hostile_takeover_cost_base uint="3000"/>
    <guild_network_bonus_threshold uint="5"/>
</guilds>
```

**New Agent Actions:**
- **Guild Acquisition**: Base 35% success, modified by character traits and target defenses
- **Guild Sabotage**: Disrupt enemy guild operations (25% base success)
- **Trade Network Infiltration**: Gain intelligence on enemy guild operations

### UI/UX Design Specifications

**Guild Management Panel:**
- Guild network overview showing controlled resources and regions
- Hostile takeover interface with success probability calculations  
- Guild defense management with agent assignment
- Economic warfare planning tools

**Trade Information Overlay:**
- Visual indicators on campaign map showing guild-controlled resources
- Trade route overlays with monopoly status indicators
- Guild conflict notifications and diplomatic impact warnings

### Database Schema Requirements

**New Tables:**
```sql
CREATE TABLE guild_controls (
    faction_id INTEGER,
    resource_type VARCHAR(32),
    region_id INTEGER,
    control_level INTEGER (0-100),
    defense_rating INTEGER,
    monthly_income INTEGER
);

CREATE TABLE guild_conflicts (
    attacker_faction INTEGER,
    defender_faction INTEGER,
    target_resource VARCHAR(32),
    conflict_start_turn INTEGER,
    diplomatic_penalty INTEGER
);
```

### Lua Scripting Examples

```lua
-- Guild acquisition attempt
function attempt_guild_acquisition(character, target_region, resource_type)
    local base_chance = 35
    local character_bonus = character:get_trait_level("GuildInfluence") * 5
    local defense_penalty = target_region:guild_defense_rating() * -3
    
    local final_chance = base_chance + character_bonus + defense_penalty
    final_chance = math.max(5, math.min(95, final_chance))
    
    if math.random(100) <= final_chance then
        establish_guild_control(character:faction(), target_region, resource_type, 25)
        return true
    end
    return false
end

-- Monthly guild income calculation
function calculate_guild_income(faction)
    local total_income = 0
    local monopoly_bonus = 1.0
    
    for resource, control_data in pairs(faction:guild_controls()) do
        local base_income = resource:trade_value() * control_data.control_level * 10
        if control_data.control_level >= 75 then
            monopoly_bonus = 2.5  -- Full monopoly bonus
        end
        total_income = total_income + (base_income * monopoly_bonus)
    end
    
    faction:add_money(total_income)
end
```

### Timeline and Development Phases

**Phase 1 (Months 1-2)**: Core Infrastructure
- Implement guild building types and capabilities
- Create guild master character type
- Basic acquisition mechanics

**Phase 2 (Months 3-4)**: Advanced Features  
- Guild warfare system
- Monopoly bonuses and penalties
- Diplomatic integration

**Phase 3 (Months 5-6)**: Polish and Balance
- UI implementation
- AI behavior scripting
- Balancing and testing

### Balancing Considerations

**Economic Impact:**
- Guild monopolies should provide 150-250% income bonus for controlled resources
- Hostile takeovers should cost 3000-8000 florins depending on target value
- Diplomatic penalties for guild warfare: -1 to -3 relation points per conflict

**Gameplay Balance:**
- Limit maximum guilds per faction (8-12) to prevent excessive dominance
- Require significant investment in defense to maintain guild control
- Provide counter-strategies for factions without strong guild presence

**Testing Approaches:**
- Economic simulation testing with varying guild control scenarios
- Multiplayer balance testing with guild-focused strategies
- AI behavior verification for guild acquisition priorities
- Performance testing with multiple active guild conflicts

---

## 2. Dynastic Marriage Economics (#6)

### Detailed Gameplay Mechanics

**Core System:**
- Strategic marriages create lasting economic partnerships
- Trade agreements and tariff reductions through marriage alliances
- Dowries and inheritance mechanics affecting faction treasuries
- Economic diplomatic options unlocked through marriage ties

**Player Interactions:**
- **Marriage Negotiations**: Include economic terms in marriage proposals
- **Trade Privileges**: Grant preferential trade rates to marriage allies
- **Economic Inheritance**: Receive economic benefits from deceased spouses' families
- **Commercial Diplomacy**: Use marriage ties to negotiate exclusive trade deals

### Technical Implementation Requirements

**Marriage System Extensions:**
```
; In export_descr_character_traits.txt
trait MarriageWealth
{
    level 1
    {
        description MarriageWealth_1_desc
        effects
        {
            TaxCollection 1
            Trading 1
            faction_trade_income_bonus 5
        }
    }
    level 2
    {
        description MarriageWealth_2_desc  
        effects
        {
            TaxCollection 2
            Trading 2
            faction_trade_income_bonus 10
            diplomatic_trade_bonus 1
        }
    }
    level 3
    {
        description MarriageWealth_3_desc
        effects
        {
            TaxCollection 3
            Trading 3
            faction_trade_income_bonus 20
            diplomatic_trade_bonus 2
            marriage_dowry_bonus 50
        }
    }
}
```

**Economic Ancillaries:**
```
Ancillary trade_agreement_scroll
    Type Diplomatic
    Transferable 1
    Image trade_agreement.tga
    Description trade_agreement_desc
    EffectsDescription trade_agreement_effects_desc
    Effect Trading 3
    Effect TaxCollection 2
    Effect faction_trade_income_bonus 15

Ancillary dowry_chest
    Type Wealth
    Transferable 0
    Image dowry_wealth.tga
    Description dowry_chest_desc
    EffectsDescription dowry_chest_effects_desc
    Effect faction_starting_money 5000
    Effect faction_trade_income_bonus 10
```

### Integration with M2TWEOP Framework

**Campaign Database Extensions:**
```xml
<dynastic_economics>
    <marriage_trade_bonus_base float="0.15"/>
    <dowry_wealth_multiplier float="2.0"/>
    <inheritance_probability float="0.25"/>
    <trade_privilege_discount float="0.20"/>
    <economic_marriage_diplomacy_bonus int="3"/>
    <marriage_wealth_decay_rate float="0.05"/>
</dynastic_economics>
```

**Diplomatic System Integration:**
```xml
<diplomacy>
    <marriage_economic_options>
        <trade_agreement_option bool="true"/>
        <dowry_negotiation bool="true"/>  
        <inheritance_clause bool="true"/>
        <tariff_reduction_pact bool="true"/>
    </marriage_economic_options>
</diplomacy>
```

### UI/UX Design Specifications

**Enhanced Marriage Negotiation Interface:**
- Economic terms panel in marriage proposals
- Dowry amount slider with faction wealth consideration
- Trade agreement checkbox options (tariffs, exclusive deals)
- Projected economic impact calculator

**Dynasty Economic Overview:**
- Family tree with economic contributions highlighted
- Marriage-based trade route visualization
- Inheritance tracking and projections
- Economic diplomatic status indicators

### Database Schema Requirements

```sql
CREATE TABLE marriage_economics (
    marriage_id INTEGER PRIMARY KEY,
    husband_faction INTEGER,
    wife_faction INTEGER,
    dowry_amount INTEGER,
    trade_bonus_percentage REAL,
    active_trade_agreements TEXT,
    inheritance_pending BOOLEAN
);

CREATE TABLE dynastic_trade_bonuses (
    faction_a INTEGER,
    faction_b INTEGER,
    marriage_count INTEGER,
    cumulative_trade_bonus REAL,
    tariff_reduction REAL
);
```

### Lua Scripting Examples

```lua
-- Calculate marriage economic benefits
function calculate_marriage_benefits(marriage)
    local husband_faction = marriage:husband():faction()
    local wife_faction = marriage:wife():faction()
    
    -- Base trade bonus from marriage alliance
    local trade_bonus = 0.15 * marriage:relationship_strength()
    
    -- Apply dowry effects
    if marriage:dowry_amount() > 0 then
        husband_faction:add_money(marriage:dowry_amount())
        local ongoing_bonus = marriage:dowry_amount() * 0.02  -- 2% annual return
        husband_faction:add_trade_income_bonus(ongoing_bonus)
    end
    
    -- Mutual trade benefits
    establish_trade_bonus(husband_faction, wife_faction, trade_bonus)
    
    -- Unlock economic diplomatic options
    enable_economic_diplomacy(husband_faction, wife_faction)
end

-- Handle inheritance events
function process_inheritance_event(deceased_character)
    if deceased_character:has_spouse() then
        local spouse = deceased_character:spouse()
        local inheritor_faction = spouse:faction()
        
        -- Calculate inheritance based on deceased's wealth and traits
        local inheritance = deceased_character:personal_wealth() * 0.3
        inheritance = inheritance + (deceased_character:trait_level("MarriageWealth") * 1000)
        
        inheritor_faction:add_money(inheritance)
        
        -- Add inheritance trait to spouse
        spouse:add_trait("InheritedWealth", 1)
    end
end
```

### Timeline and Development Phases

**Phase 1 (Months 1-2)**: Core Marriage Economics
- Implement dowry system
- Basic trade bonus mechanics
- Marriage wealth traits

**Phase 2 (Months 3-4)**: Advanced Diplomacy
- Economic diplomatic options
- Inheritance system
- Trade agreement mechanisms

**Phase 3 (Months 5-6)**: Integration and Polish
- UI enhancements
- AI diplomatic behavior
- Balance testing and refinement

### Balancing Considerations

**Economic Impact:**
- Marriage trade bonuses: 15-25% increase for allied trade
- Dowry amounts: 2000-10000 florins based on faction wealth
- Inheritance probability: 25-40% chance per turn after spouse death

**Gameplay Balance:**
- Limit simultaneous marriage benefits to prevent exploitation
- Require diplomatic maintenance to preserve economic marriage benefits
- Balance dowry costs against long-term economic gains

---

## 3. Port Taxation & Smuggling Networks (#9)

### Detailed Gameplay Mechanics

**Core System:**
- Ports generate income through taxation of merchant vessels
- Smuggling networks bypass port taxes but require infrastructure investment
- Naval patrol effectiveness determines tax collection and smuggling interdiction
- Black market goods provide high profits but increase public order penalties

**Player Interactions:**
- **Tax Rate Setting**: Adjust port taxes (high revenue vs. trade volume balance)
- **Smuggling Operations**: Establish covert trade routes to avoid enemy taxes
- **Naval Enforcement**: Deploy ships to patrol trade routes and intercept smugglers
- **Black Market Management**: Control contraband trade for enhanced profits

### Technical Implementation Requirements

**Port Building Extensions:**
```
building port_authority
{
    levels customs_house harbor_watch maritime_bureau
    {
        customs_house
        {
            capability
            {
                port_tax_rate 10
                smuggling_detection 25
                naval_patrol_range 2
                trade_income_bonus 15
                public_order_bonus 1
            }
            cost 2000
            construction 4
            settlement_min town
            requires building port
        }
        
        harbor_watch
        {
            capability
            {
                port_tax_rate 20
                smuggling_detection 50
                naval_patrol_range 4
                trade_income_bonus 30
                contraband_interdiction 35
                public_order_bonus 2
            }
            cost 5000
            construction 6
            settlement_min large_town
        }
        
        maritime_bureau
        {
            capability
            {
                port_tax_rate 35
                smuggling_detection 75
                naval_patrol_range 6
                trade_income_bonus 50
                contraband_interdiction 60
                diplomatic_trade_options
                public_order_bonus 3
            }
            cost 12000
            construction 8
            settlement_min city
        }
    }
}

building smugglers_den
{
    levels hidden_dock underground_network smuggling_empire
    {
        hidden_dock
        {
            capability
            {
                contraband_income 500
                smuggling_capacity 25
                detection_avoidance 20
                public_order_penalty -1
            }
            cost 3000
            construction 5
            settlement_min town
            hidden_building true
        }
        
        underground_network
        {
            capability
            {
                contraband_income 1200
                smuggling_capacity 50
                detection_avoidance 40
                black_market_access
                public_order_penalty -2
            }
            cost 7000
            construction 7
            settlement_min large_town
            hidden_building true
        }
        
        smuggling_empire
        {
            capability
            {
                contraband_income 2500
                smuggling_capacity 100
                detection_avoidance 65
                international_smuggling_network
                faction_contraband_bonus 25
                public_order_penalty -3
            }
            cost 15000
            construction 10
            settlement_min city
            hidden_building true
        }
    }
}
```

**New Agent Type:**
```
character smuggler
{
    type spy
    actions establish_smuggling_route, avoid_patrols, contraband_trading
    wage_base 150
    wage_additional 75
    
    ancillaries forged_papers, contraband_goods, bribery_funds
    traits Subterfuge, TradingExpertise, Criminal_Contacts
}
```

### Integration with M2TWEOP Framework

**Campaign Database Extensions:**
```xml
<maritime_trade>
    <base_port_tax_rate float="0.15"/>
    <max_port_tax_rate float="0.45"/>
    <smuggling_base_success float="40.0"/>
    <naval_patrol_effectiveness float="2.5"/>
    <contraband_profit_multiplier float="3.0"/>
    <smuggling_public_order_penalty int="-2"/>
    <port_blockade_income_reduction float="0.75"/>
</maritime_trade>
```

### UI/UX Design Specifications

**Port Management Interface:**
- Tax rate slider with real-time revenue projection
- Trade volume vs. tax rate optimization chart
- Naval patrol deployment interface
- Smuggling detection reports and interdiction options

**Smuggling Network Panel:**
- Covert trade route planning interface
- Risk/reward calculator for contraband operations
- Detection probability indicators
- Black market goods inventory management

### Database Schema Requirements

```sql
CREATE TABLE port_taxation (
    region_id INTEGER,
    faction_id INTEGER,
    tax_rate REAL,
    monthly_revenue INTEGER,
    trade_volume INTEGER,
    patrol_strength INTEGER
);

CREATE TABLE smuggling_routes (
    route_id INTEGER PRIMARY KEY,
    origin_port INTEGER,
    destination_port INTEGER,
    controller_faction INTEGER,
    detection_risk REAL,
    monthly_profit INTEGER,
    contraband_type VARCHAR(32)
);

CREATE TABLE naval_patrols (
    patrol_id INTEGER,
    faction_id INTEGER,
    patrol_region INTEGER,
    ship_count INTEGER,
    effectiveness_rating INTEGER,
    interdiction_success_rate REAL
);
```

### Lua Scripting Examples

```lua
-- Calculate port tax revenue
function calculate_port_revenue(port_region)
    local tax_rate = port_region:port_tax_rate()
    local base_trade_volume = port_region:trade_volume()
    
    -- Trade volume decreases with higher tax rates
    local effective_trade_volume = base_trade_volume * (1 - (tax_rate - 0.15) * 2)
    effective_trade_volume = math.max(effective_trade_volume, base_trade_volume * 0.3)
    
    local tax_revenue = effective_trade_volume * tax_rate
    
    return tax_revenue
end

-- Smuggling detection check
function attempt_smuggling_route(smuggler, origin_port, destination_port)
    local base_detection = 30
    local port_security = destination_port:smuggling_detection_rating()
    local smuggler_skill = smuggler:subterfuge_rating() * 2
    local route_risk = calculate_route_risk(origin_port, destination_port)
    
    local detection_chance = base_detection + port_security + route_risk - smuggler_skill
    detection_chance = math.max(5, math.min(85, detection_chance))
    
    if math.random(100) <= detection_chance then
        -- Smuggling attempt detected
        handle_smuggling_interdiction(smuggler, destination_port)
        return false
    else
        -- Successful smuggling run
        local profit = calculate_contraband_profit(origin_port, destination_port)
        smuggler:faction():add_money(profit)
        return true
    end
end

-- Naval patrol interdiction
function naval_patrol_interdiction(patrol_fleet, target_region)
    local patrol_strength = patrol_fleet:ship_count() * patrol_fleet:average_crew_quality()
    local smuggling_activity = target_region:smuggling_activity_level()
    
    local interdiction_success = (patrol_strength * 0.6) / smuggling_activity
    interdiction_success = math.min(interdiction_success, 0.8)  -- Max 80% interdiction
    
    if math.random() <= interdiction_success then
        local seized_goods_value = smuggling_activity * 150  -- Base contraband value
        patrol_fleet:faction():add_money(seized_goods_value)
        
        -- Reduce smuggling activity in region
        target_region:reduce_smuggling_activity(25)
        
        return true
    end
    
    return false
end
```

### Timeline and Development Phases

**Phase 1 (Months 1-2)**: Basic Port Taxation
- Implement port tax system
- Basic revenue calculation mechanics
- Tax rate adjustment interface

**Phase 2 (Months 3-4)**: Smuggling Networks
- Smuggling route establishment
- Contraband trade mechanics
- Detection and interdiction systems

**Phase 3 (Months 5-6)**: Naval Integration
- Naval patrol mechanics
- Advanced interdiction systems
- AI behavioral scripting

### Balancing Considerations

**Economic Impact:**
- Port taxes: 10-35% of trade value
- Smuggling profits: 200-400% of normal trade goods
- Naval patrol costs: 100-300 florins per turn per ship

**Gameplay Balance:**
- High port taxes reduce legitimate trade volume
- Smuggling carries significant detection risks and public order penalties
- Naval patrols require ongoing investment but provide interdiction capabilities

---

## 4. Resource Depletion & Discovery (#11)

### Detailed Gameplay Mechanics

**Core System:**
- Resources gradually deplete based on exploitation intensity
- New resource deposits discovered through exploration and research
- Depletion affects regional economy and forces strategic adaptation
- Discovery events provide significant economic opportunities

**Player Interactions:**
- **Exploitation Management**: Balance resource extraction rates with sustainability
- **Exploration Missions**: Send agents to discover new resource deposits
- **Technology Research**: Unlock advanced extraction methods and discovery techniques
- **Strategic Planning**: Adapt to changing resource availability across regions

### Technical Implementation Requirements

**Resource Depletion System:**
```
; In descr_sm_resources.txt extensions
resource_depletion_rates
{
    gold        depletion_rate 0.02    discovery_chance 0.15    replenishment_impossible
    silver      depletion_rate 0.025   discovery_chance 0.18    replenishment_impossible  
    iron        depletion_rate 0.015   discovery_chance 0.25    replenishment_rate 0.005
    coal        depletion_rate 0.01    discovery_chance 0.20    replenishment_impossible
    
    ; Renewable resources
    timber      depletion_rate 0.08    discovery_chance 0.35    replenishment_rate 0.12
    fish        depletion_rate 0.05    discovery_chance 0.30    replenishment_rate 0.08
    grain       depletion_rate 0.03    discovery_chance 0.40    replenishment_rate 0.15
    
    ; Luxury resources
    spices      depletion_rate 0.02    discovery_chance 0.10    replenishment_rate 0.01
    silk        depletion_rate 0.025   discovery_chance 0.08    replenishment_rate 0.02
    ivory       depletion_rate 0.04    discovery_chance 0.05    replenishment_impossible
}
```

**Exploration Buildings:**
```
building exploration_guild  
{
    levels surveyors_guild prospectors_guild discovery_academy
    {
        surveyors_guild
        {
            capability
            {
                resource_discovery_chance 15
                exploration_range 2
                depletion_mitigation 10
                recruitment_slots 1
            }
            cost 2500
            construction 4
            settlement_min town
        }
        
        prospectors_guild
        {
            capability
            {
                resource_discovery_chance 30
                exploration_range 4
                depletion_mitigation 20
                advanced_mining_techniques
                recruitment_slots 2
            }
            cost 6000
            construction 6
            settlement_min large_town
        }
        
        discovery_academy
        {
            capability
            {
                resource_discovery_chance 50
                exploration_range 6
                depletion_mitigation 35
                new_extraction_methods
                regional_survey_capability
                recruitment_slots 3
            }
            cost 12000
            construction 8
            settlement_min city
        }
    }
}
```

**New Agent Type:**
```
character explorer
{
    type spy
    actions resource_survey, establish_mining_operation, geological_research
    wage_base 180
    wage_additional 60
    
    ancillaries surveying_instruments, geological_maps, mining_equipment
    traits Exploration, GeologicalKnowledge, Survival
}
```

### Integration with M2TWEOP Framework

**Campaign Database Extensions:**
```xml
<resource_dynamics>
    <base_depletion_rate float="0.02"/>
    <discovery_base_chance float="0.15"/>
    <exploitation_intensity_modifier float="1.5"/>
    <technology_discovery_bonus float="0.25"/>
    <depletion_warning_threshold float="0.25"/>
    <complete_depletion_threshold float="0.05"/>
    <replenishment_season_bonus float="0.3"/>
</resource_dynamics>
```

### UI/UX Design Specifications

**Resource Management Dashboard:**
- Regional resource status indicators with depletion warnings
- Historical extraction charts showing sustainability trends  
- Discovery mission planning interface
- Technology research progress for extraction improvements

**Exploration Interface:**
- World map overlay showing potential discovery sites
- Mission risk/reward calculations for exploration attempts
- Geological survey results and resource probability indicators
- Discovery event notifications with economic impact projections

### Database Schema Requirements

```sql
CREATE TABLE resource_depletion (
    region_id INTEGER,
    resource_type VARCHAR(32),
    current_quantity REAL,
    depletion_rate REAL,
    extraction_intensity INTEGER,
    turns_until_depletion INTEGER
);

CREATE TABLE resource_discoveries (
    discovery_id INTEGER PRIMARY KEY,
    region_id INTEGER,
    resource_type VARCHAR(32),
    discovery_turn INTEGER,
    discoverer_faction INTEGER,
    initial_quantity REAL,
    quality_grade INTEGER
);

CREATE TABLE exploration_missions (
    mission_id INTEGER,
    explorer_character INTEGER,
    target_region INTEGER,
    mission_type VARCHAR(32),
    success_probability REAL,
    potential_rewards TEXT
);
```

### Lua Scripting Examples

```lua
-- Process resource depletion per turn
function process_resource_depletion()
    for region in world_regions() do
        for resource_type, resource_data in pairs(region:resources()) do
            local base_depletion = get_resource_depletion_rate(resource_type)
            local exploitation_modifier = region:exploitation_intensity(resource_type) / 100
            local mitigation_factor = region:depletion_mitigation() / 100
            
            local effective_depletion = base_depletion * (1 + exploitation_modifier) * (1 - mitigation_factor)
            
            resource_data.quantity = resource_data.quantity - effective_depletion
            
            -- Check for complete depletion
            if resource_data.quantity <= 0.05 then
                trigger_resource_depletion_event(region, resource_type)
                remove_resource_from_region(region, resource_type)
            -- Check for warning threshold  
            elseif resource_data.quantity <= 0.25 and not resource_data.warning_sent then
                trigger_depletion_warning(region, resource_type)
                resource_data.warning_sent = true
            end
        end
    end
end

-- Handle resource discovery attempts
function attempt_resource_discovery(explorer, target_region)
    local base_chance = 15
    local explorer_bonus = explorer:trait_level("GeologicalKnowledge") * 5
    local technology_bonus = explorer:faction():technology_level("Mining") * 3
    local regional_modifier = get_regional_discovery_modifier(target_region)
    
    local discovery_chance = base_chance + explorer_bonus + technology_bonus + regional_modifier
    discovery_chance = math.max(5, math.min(60, discovery_chance))
    
    if math.random(100) <= discovery_chance then
        local discovered_resource = determine_discovery_type(target_region)
        local resource_quantity = math.random(50, 200) / 100  -- 0.5 to 2.0 base quantity
        
        add_resource_to_region(target_region, discovered_resource, resource_quantity)
        trigger_discovery_event(explorer, target_region, discovered_resource)
        
        return true
    end
    
    return false
end

-- Calculate economic impact of resource changes
function calculate_resource_economic_impact(region, resource_type, quantity_change)
    local base_trade_value = get_resource_trade_value(resource_type)
    local regional_economy = region:economic_value()
    
    -- Positive change = discovery, negative = depletion
    local economic_impact = quantity_change * base_trade_value * 100
    
    if quantity_change < 0 then
        -- Depletion impact - affects region income and happiness
        region:modify_income(economic_impact)
        region:modify_public_order(math.floor(quantity_change * -10))
    else
        -- Discovery benefit - immediate wealth bonus and ongoing income
        region:faction():add_money(economic_impact * 2)  -- Discovery bonus
        region:modify_income(economic_impact * 0.1)  -- Ongoing benefit
    end
end
```

### Timeline and Development Phases

**Phase 1 (Months 1-3)**: Core Depletion System
- Implement basic resource depletion mechanics
- Create depletion tracking and warning systems
- Basic economic impact calculations

**Phase 2 (Months 4-5)**: Discovery Mechanics
- Resource discovery system implementation
- Explorer agent type and actions
- Discovery event system

**Phase 3 (Months 6-7)**: Integration and Balance
- Technology research integration
- Advanced extraction techniques
- AI adaptation to resource changes

### Balancing Considerations

**Economic Impact:**
- Resource depletion: 1-5% quantity loss per turn based on exploitation
- Discovery bonuses: 2000-8000 florins immediate wealth, ongoing income boost
- Exploration missions: 200-500 florins cost, 15-60% success rates

**Gameplay Balance:**
- Depletion should encourage territorial expansion and economic diversification
- Discovery events provide catch-up mechanisms for economically disadvantaged factions
- Sustainable exploitation should be more profitable long-term than intensive extraction

---

## 5. Craft Specialization Bonuses (#13)

### Detailed Gameplay Mechanics

**Core System:**
- Regions develop specializations in specific crafts based on resources and buildings
- Specialized regions produce higher quality goods with better trade values
- Craft specializations unlock unique units, buildings, and technologies
- Specialization bonuses compound over time with continued investment

**Player Interactions:**
- **Specialization Development**: Focus regional development on specific craft industries  
- **Master Craftsmen Recruitment**: Attract skilled artisans to enhance specialization
- **Trade Network Optimization**: Route specialized goods through optimal trade paths
- **Technology Investment**: Research improvements specific to regional specializations

### Technical Implementation Requirements

**Craft Specialization System:**
```
building craft_specialization
{
    levels artisan_quarter craft_district master_workshop
    {
        artisan_quarter
        {
            capability
            {
                craft_specialization textiles 25
                regional_quality_bonus 10
                trade_value_bonus 15
                specialist_recruitment_slots 1
                happiness_bonus 1
            }
            cost 3000
            construction 4  
            settlement_min town
            requires resource textiles or resource silk or resource cotton
        }
        
        craft_district
        {
            capability  
            {
                craft_specialization textiles 50
                craft_specialization dyes 25
                regional_quality_bonus 25
                trade_value_bonus 30
                unique_unit_recruitment
                specialist_recruitment_slots 2
                happiness_bonus 2
            }
            cost 7000
            construction 6
            settlement_min large_town
        }
        
        master_workshop
        {
            capability
            {
                craft_specialization textiles 75
                craft_specialization dyes 50
                craft_specialization_secondary 25
                regional_quality_bonus 50
                trade_value_bonus 60
                legendary_crafts_production
                master_artisan_recruitment
                specialist_recruitment_slots 3
                happiness_bonus 3
            }
            cost 15000
            construction 8
            settlement_min city
        }
    }
}
```

**Specialization Categories:**
```
; Craft specialization types and bonuses
craft_types
{
    metalworking
    {
        required_resources iron, tin, coal
        quality_bonus_multiplier 1.8
        unique_units armorer_guild_units
        building_bonuses weapon_smith, armor_forge
    }
    
    textiles
    {
        required_resources cotton, silk, wool  
        quality_bonus_multiplier 1.6
        unique_units cloth_merchant_guards
        building_bonuses weaving_hall, dye_works
    }
    
    luxury_goods
    {
        required_resources gold, silver, ivory, amber
        quality_bonus_multiplier 2.2
        unique_units master_jewelers, luxury_traders
        building_bonuses jewelers_guild, luxury_market
    }
    
    shipbuilding
    {
        required_resources timber, iron, tar
        quality_bonus_multiplier 1.9
        unique_units naval_engineers, master_shipwrights
        building_bonuses naval_yard, dry_dock
        requires coastal_region true
    }
}
```

**Master Craftsmen Characters:**
```
character master_craftsman
{
    type named character
    actions enhance_specialization, craft_research, quality_improvement
    wage_base 300
    wage_additional 100
    
    ancillaries master_tools, craft_knowledge, artisan_network
    traits CraftMastery, Innovation, QualityControl
    
    specialization_types metalworking, textiles, luxury_goods, shipbuilding
}
```

### Integration with M2TWEOP Framework

**Campaign Database Extensions:**
```xml
<craft_specialization>
    <specialization_development_rate float="2.0"/>
    <quality_bonus_base float="1.2"/>
    <master_craftsman_bonus float="0.3"/>
    <specialization_decay_rate float="0.5"/>
    <cross_specialization_penalty float="0.7"/>
    <legendary_craft_threshold uint="75"/>
    <specialization_max_level uint="100"/>
</craft_specialization>
```

### UI/UX Design Specifications

**Regional Specialization Panel:**
- Craft specialization progress bars with development status
- Available specialization options based on regional resources
- Master craftsmen assignment interface  
- Quality rating indicators and trade value projections

**Craft Network Overview:**
- Empire-wide specialization distribution map
- Trade route optimization for specialized goods
- Cross-regional craft collaboration opportunities
- Legendary craft production tracking

### Database Schema Requirements

```sql
CREATE TABLE regional_specializations (
    region_id INTEGER,
    craft_type VARCHAR(32),
    specialization_level INTEGER,
    quality_bonus REAL,
    master_craftsmen_count INTEGER,
    legendary_items_produced INTEGER
);

CREATE TABLE craft_technologies (
    faction_id INTEGER,
    craft_type VARCHAR(32),
    technology_level INTEGER,
    research_progress INTEGER,
    unlocked_bonuses TEXT
);

CREATE TABLE master_craftsmen (
    character_id INTEGER,
    craft_specialization VARCHAR(32),
    mastery_level INTEGER,
    assigned_region INTEGER,
    legendary_works_created INTEGER
);
```

### Lua Scripting Examples

```lua
-- Develop regional craft specialization
function develop_craft_specialization(region, craft_type, investment)
    local current_level = region:specialization_level(craft_type)
    local max_development = calculate_max_specialization(region, craft_type)
    
    -- Check resource requirements
    if not region:has_required_resources(craft_type) then
        return false, "Missing required resources for " .. craft_type
    end
    
    -- Calculate development gain
    local base_development = investment / 100
    local master_craftsman_bonus = region:master_craftsmen_count(craft_type) * 0.3
    local building_bonus = region:craft_building_bonus(craft_type)
    
    local development_gain = base_development * (1 + master_craftsman_bonus + building_bonus)
    development_gain = math.min(development_gain, max_development - current_level)
    
    region:increase_specialization(craft_type, development_gain)
    
    -- Check for specialization milestones
    local new_level = current_level + development_gain
    if new_level >= 25 and current_level < 25 then
        unlock_basic_specialization_bonuses(region, craft_type)
    elseif new_level >= 50 and current_level < 50 then
        unlock_advanced_specialization_bonuses(region, craft_type)
    elseif new_level >= 75 and current_level < 75 then
        unlock_master_specialization_bonuses(region, craft_type)
    end
    
    return true, "Specialization developed successfully"
end

-- Calculate specialized goods trade value
function calculate_specialized_trade_value(region, resource_type)
    local base_value = get_base_trade_value(resource_type)
    local specialization_level = region:specialization_level_for_resource(resource_type)
    
    if specialization_level > 0 then
        local quality_multiplier = 1 + (specialization_level / 100)
        local master_bonus = region:master_craftsmen_bonus(resource_type)
        local legendary_bonus = region:legendary_craft_bonus(resource_type)
        
        local specialized_value = base_value * quality_multiplier * (1 + master_bonus + legendary_bonus)
        return specialized_value
    end
    
    return base_value
end

-- Handle master craftsman recruitment
function recruit_master_craftsman(region, craft_type, character_pool)
    local recruitment_cost = 5000 + (region:specialization_level(craft_type) * 50)
    local availability_chance = 30 + (region:craft_reputation(craft_type) * 2)
    
    if math.random(100) <= availability_chance then
        local master_craftsman = create_master_craftsman(craft_type)
        master_craftsman:assign_to_region(region)
        
        -- Apply immediate benefits
        region:increase_specialization_development_rate(craft_type, 0.5)
        region:increase_quality_bonus(craft_type, 0.15)
        
        return master_craftsman, recruitment_cost
    end
    
    return nil, 0
end
```

### Timeline and Development Phases

**Phase 1 (Months 1-3)**: Basic Specialization System
- Implement regional craft specialization mechanics
- Create craft building types and bonuses
- Basic quality improvement calculations

**Phase 2 (Months 4-5)**: Master Craftsmen System
- Master craftsman character type implementation
- Advanced specialization bonuses and unique unlocks
- Legendary craft production mechanics

**Phase 3 (Months 6-7)**: Integration and Polish
- Cross-regional craft collaboration features
- AI specialization development behavior
- Balance refinement and testing

### Balancing Considerations

**Economic Impact:**
- Specialized goods: 150-300% base trade value depending on specialization level
- Master craftsman costs: 5000-12000 florins recruitment, 300-400 monthly wages
- Specialization development: 2000-8000 florins investment for significant progress

**Gameplay Balance:**
- Regions should naturally develop specializations based on available resources
- Multiple specializations possible but with efficiency penalties
- Specialization bonuses should encourage regional economic focus without forcing it

---

## 6. Basic Supply & Demand (#14)

### Detailed Gameplay Mechanics

**Core System:**
- Trade good prices fluctuate based on regional supply and demand
- Over-supply reduces prices while scarcity increases them
- Faction actions and events affect market conditions
- Strategic stockpiling and market manipulation possible

**Player Interactions:**
- **Market Analysis**: Monitor price trends and supply/demand indicators
- **Strategic Trading**: Time trade operations for maximum profit
- **Supply Manipulation**: Control production to influence prices
- **Market Cornering**: Attempt to monopolize specific goods in target markets

### Technical Implementation Requirements

**Supply & Demand System:**
```
; Price calculation system
supply_demand_mechanics
{
    base_price_volatility 0.25
    supply_price_impact -1.5    ; Each unit oversupply reduces price by 1.5%
    demand_price_impact 2.0     ; Each unit demand increases price by 2%
    
    market_categories
    {
        staple_goods        volatility_modifier 0.8    max_price_change 50
        luxury_goods        volatility_modifier 1.5    max_price_change 200
        military_supplies   volatility_modifier 1.2    max_price_change 100
        raw_materials       volatility_modifier 1.0    max_price_change 75
    }
}
```

**Market Building Extensions:**
```  
building trade_exchange
{
    levels market_stall trading_post commercial_exchange
    {
        market_stall
        {
            capability
            {
                market_information_basic
                price_influence_range 1
                trade_volume_bonus 10
                supply_demand_visibility 25
            }
            cost 800
            construction 2
            settlement_min village
        }
        
        trading_post
        {
            capability
            {
                market_information_detailed
                price_influence_range 3
                trade_volume_bonus 25
                supply_demand_visibility 50
                market_manipulation_basic
            }
            cost 2500
            construction 4
            settlement_min town
        }
        
        commercial_exchange
        {
            capability
            {
                market_information_comprehensive
                price_influence_range 5
                trade_volume_bonus 50
                supply_demand_visibility 100
                market_manipulation_advanced
                futures_trading_access
            }
            cost 8000
            construction 6
            settlement_min large_town
        }
    }
}
```

**Market Specialist Character:**
```
character market_analyst
{
    type spy
    actions market_research, price_manipulation, commodity_speculation  
    wage_base 200
    wage_additional 80
    
    ancillaries market_ledgers, trade_contacts, commodity_samples
    traits MarketKnowledge, TradingExpertise, Economic_Intelligence
}
```

### Integration with M2TWEOP Framework

**Campaign Database Extensions:**
```xml
<supply_demand>
    <price_update_frequency uint="2"/>
    <base_demand_growth_rate float="1.02"/>
    <supply_shock_threshold float="0.3"/>
    <demand_shock_threshold float="0.4"/>
    <market_memory_duration uint="10"/>
    <speculation_impact_modifier float="1.25"/>
    <seasonal_demand_variation bool="true"/>
</supply_demand>
```

### UI/UX Design Specifications

**Market Information Panel:**
- Real-time price charts with historical trends
- Supply/demand indicators for each trade good
- Market opportunity alerts and profit projections
- Regional price comparison tools

**Trading Interface:**
- Advanced trade deal calculator with profit projections
- Market timing recommendations based on price trends  
- Speculation and futures trading options
- Portfolio management for commodity positions

### Database Schema Requirements

```sql
CREATE TABLE market_prices (
    region_id INTEGER,
    resource_type VARCHAR(32),
    current_price REAL,
    base_price REAL,
    supply_level INTEGER,
    demand_level INTEGER,
    price_trend VARCHAR(16)
);

CREATE TABLE supply_demand_history (
    region_id INTEGER,
    resource_type VARCHAR(32),
    turn_number INTEGER,
    supply_quantity INTEGER,
    demand_quantity INTEGER,
    market_price REAL
);

CREATE TABLE commodity_positions (
    faction_id INTEGER,
    resource_type VARCHAR(32),
    quantity_held INTEGER,
    average_purchase_price REAL,
    unrealized_profit REAL
);
```

### Lua Scripting Examples

```lua
-- Calculate current market price based on supply and demand
function calculate_market_price(region, resource_type)
    local base_price = get_base_trade_value(resource_type)
    local supply = region:resource_supply(resource_type)
    local demand = region:resource_demand(resource_type)
    
    -- Calculate supply/demand ratio
    local supply_demand_ratio = supply / math.max(demand, 1)
    
    -- Price adjustment based on ratio
    local price_modifier = 1.0
    if supply_demand_ratio > 1.2 then
        -- Oversupply - reduce price
        price_modifier = 1.0 - math.min((supply_demand_ratio - 1) * 0.5, 0.8)
    elseif supply_demand_ratio < 0.8 then
        -- Undersupply - increase price  
        price_modifier = 1.0 + math.min((1 - supply_demand_ratio) * 0.75, 2.0)
    end
    
    -- Apply volatility and market category modifiers
    local category_modifier = get_commodity_category_volatility(resource_type)
    local random_variation = (math.random() - 0.5) * 0.1 * category_modifier
    
    local final_price = base_price * price_modifier * (1 + random_variation)
    
    -- Clamp to reasonable bounds
    local max_change = get_max_price_change_percent(resource_type) / 100
    final_price = math.max(base_price * (1 - max_change), 
                          math.min(base_price * (1 + max_change), final_price))
    
    return final_price
end

-- Update regional supply and demand levels
function update_supply_demand_levels()
    for region in world_regions() do
        for resource_type in region:available_resources() do
            -- Calculate supply based on production and imports
            local production = region:resource_production(resource_type)
            local imports = region:resource_imports(resource_type)
            local total_supply = production + imports
            
            -- Calculate demand based on population, buildings, and exports
            local population_demand = region:population() * get_per_capita_demand(resource_type)
            local building_demand = region:building_resource_demand(resource_type)
            local export_demand = region:resource_exports(resource_type)
            local total_demand = population_demand + building_demand + export_demand
            
            region:set_supply_level(resource_type, total_supply)
            region:set_demand_level(resource_type, total_demand)
            
            -- Update market price
            local new_price = calculate_market_price(region, resource_type)
            region:set_market_price(resource_type, new_price)
            
            -- Check for supply/demand shocks
            check_market_shock_conditions(region, resource_type)
        end
    end
end

-- Handle market manipulation attempts
function attempt_market_manipulation(character, target_region, resource_type, manipulation_type)
    local base_success = 40
    local character_skill = character:trait_level("MarketKnowledge") * 5
    local market_size_penalty = target_region:market_size() * -2
    local manipulation_cost = calculate_manipulation_cost(target_region, resource_type, manipulation_type)
    
    local success_chance = base_success + character_skill + market_size_penalty
    success_chance = math.max(10, math.min(80, success_chance))
    
    if character:faction():money() >= manipulation_cost then
        character:faction():subtract_money(manipulation_cost)
        
        if math.random(100) <= success_chance then
            apply_market_manipulation_effect(target_region, resource_type, manipulation_type)
            return true
        end
    end
    
    return false
end
```

### Timeline and Development Phases

**Phase 1 (Months 1-2)**: Basic Price System
- Implement supply/demand calculation mechanics
- Basic price fluctuation system
- Market information interface

**Phase 2 (Months 3-4)**: Advanced Market Features
- Market manipulation mechanics
- Speculation and futures trading
- Market analyst character implementation

**Phase 3 (Months 5-6)**: Integration and Polish
- Seasonal demand variations
- AI market behavior scripting  
- Balance testing and refinement

### Balancing Considerations

**Economic Impact:**
- Price fluctuations: ±50-200% of base value depending on commodity category
- Market manipulation costs: 1000-5000 florins per attempt
- Speculation profits: 25-150% returns on successful market timing

**Gameplay Balance:**
- Staple goods should have lower volatility than luxury items
- Market manipulation should be expensive and risky
- Supply/demand shocks should create meaningful strategic opportunities

---

## Integration Strategy & Master Timeline

### Cross-System Integration Points

**Database Integration:**
All systems share common data structures for regions, factions, and resources, enabling seamless interaction between mechanics.

**Economic Synergies:**
- Guild monopolies affect supply/demand calculations
- Marriage trade bonuses modify port taxation rates
- Resource depletion impacts craft specialization development
- Smuggling networks bypass guild taxation systems

**UI Integration:**
Unified economic dashboard providing:
- Real-time economic overview across all systems
- Cross-system impact analysis and projections
- Strategic planning tools incorporating all six mechanics
- Performance metrics and economic health indicators

### Master Development Timeline

**Months 1-2: Foundation Phase**
- Core infrastructure development for all systems
- Database schema implementation
- Basic mechanic functionality

**Months 3-4: Feature Development Phase**  
- Advanced features and cross-system integration
- Character types and agent actions
- UI/UX implementation

**Months 5-6: Integration & Polish Phase**
- Cross-mechanic balancing and testing
- AI behavior scripting
- Performance optimization

**Months 7-8: Testing & Refinement Phase**
- Comprehensive gameplay testing
- Balance adjustments based on feedback
- Documentation and deployment preparation

### Quality Assurance & Testing Strategy

**Automated Testing:**
- Economic simulation scripts testing various scenarios
- Balance verification algorithms
- Performance benchmarking tools

**Manual Testing:**
- Comprehensive gameplay testing with focus on economic strategies
- Multiplayer balance verification
- AI behavior evaluation

**Community Testing:**
- Closed beta with experienced Medieval II modding community
- Feedback integration and iterative refinement
- Public release preparation

This implementation plan provides a roadmap for creating a sophisticated economic system that enhances Medieval II Total War's strategic depth while maintaining compatibility with existing modding frameworks and player expectations.