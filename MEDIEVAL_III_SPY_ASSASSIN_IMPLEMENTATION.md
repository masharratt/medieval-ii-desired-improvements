# Medieval III: Enhanced Spy and Assassin Systems
## Comprehensive Technical Implementation Document

### Version: 1.0
### Date: September 2025
### Target: M2TWEOP Framework Integration

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Military Intelligence & Battle Preparation](#military-intelligence--battle-preparation)
3. [Sabotage Operations & Infrastructure](#sabotage-operations--infrastructure)
4. [Information Brokerage & Intelligence Markets](#information-brokerage--intelligence-markets)
5. [Agent Specialization & Training Schools](#agent-specialization--training-schools)
6. [Expanded Assassination System](#expanded-assassination-system)
7. [Database Schema](#database-schema)
8. [Implementation Timeline](#implementation-timeline)
9. [Balancing Framework](#balancing-framework)

---

## System Overview

This document outlines the technical implementation of five interconnected spy and assassin systems for Medieval III, building upon the existing Medieval II Total War engine and M2TWEOP framework. The systems are designed to provide deep strategic gameplay while maintaining historical authenticity and balanced mechanics.

### Core Design Principles
- **Historical Authenticity**: All mechanics reflect medieval espionage and assassination practices
- **Strategic Depth**: Multiple interconnected systems that reward long-term planning
- **Balanced Risk/Reward**: High-risk operations provide proportional benefits
- **Player Agency**: Multiple approaches to achieve objectives
- **Integration**: Systems work together and enhance existing gameplay

### Technical Foundation
- **Engine**: Medieval II Total War with M2TWEOP Framework
- **Scripting**: Lua 5.1+ for dynamic content and event handling
- **Configuration**: Extended .txt files following M2TW conventions
- **UI**: Custom interface elements using existing M2TW UI framework
- **Database**: SQLite integration for persistent data storage

---

## Military Intelligence & Battle Preparation

### Overview
This system transforms spies from simple reconnaissance units into sophisticated military intelligence operatives capable of providing decisive advantages in warfare through infiltration, intelligence gathering, and psychological warfare.

### Core Mechanics

#### Army Infiltration System
Spies can embed themselves within enemy armies to gather intelligence over time. The longer a spy remains undetected, the more valuable information they acquire.

**Technical Implementation:**

```lua
-- File: /data/scripts/campaign/mod_military_intelligence.lua
function infiltrate_army(spy_character, target_army)
    local infiltration_data = {
        spy_id = spy_character.character_record.character_id,
        target_army_id = target_army.army_id,
        infiltration_level = 0,
        detection_risk = calculate_base_detection_risk(spy_character, target_army),
        intelligence_gathered = {},
        start_turn = cm:turn_number()
    }
    
    -- Store infiltration data in campaign database
    save_infiltration_data(infiltration_data)
    
    -- Trigger UI notification
    trigger_event("spy_infiltration_begun", spy_character, target_army)
end

function process_infiltration_turn(spy_id)
    local infiltration = get_infiltration_data(spy_id)
    if not infiltration then return end
    
    -- Increase infiltration level
    infiltration.infiltration_level = infiltration.infiltration_level + 1
    
    -- Calculate intelligence gathering
    local intelligence_chance = calculate_intelligence_chance(infiltration)
    if math.random() < intelligence_chance then
        gather_intelligence(infiltration)
    end
    
    -- Check for detection
    local detection_chance = calculate_detection_chance(infiltration)
    if math.random() < detection_chance then
        handle_spy_detection(infiltration)
    end
end
```

#### Intelligence Types and Values

**Basic Intelligence (Turns 1-3):**
- Army composition overview
- Approximate unit counts
- General's basic attributes
- Movement patterns

**Detailed Intelligence (Turns 4-7):**
- Exact unit types and experience levels
- General's detailed traits and skills
- Army morale and loyalty status
- Supply situation assessment

**Strategic Intelligence (Turns 8+):**
- Planned attack targets
- Reinforcement schedules
- Weaknesses in command structure
- Opportunities for sabotage

#### Pre-Battle Advantages

When engaging an army that has been infiltrated, players receive tactical benefits:

```lua
-- File: /data/scripts/battle/mod_infiltration_bonuses.lua
function apply_infiltration_bonuses(battle_context)
    local infiltration_data = get_battle_infiltration_data(battle_context)
    if not infiltration_data then return end
    
    local bonuses = calculate_infiltration_bonuses(infiltration_data.intelligence_level)
    
    -- Apply tactical bonuses
    if bonuses.deployment_advantage then
        extend_deployment_zone(battle_context.allied_alliance, bonuses.deployment_extension)
    end
    
    if bonuses.ambush_opportunity then
        trigger_ambush_deployment(battle_context)
    end
    
    if bonuses.morale_warfare then
        apply_enemy_morale_penalty(battle_context.enemy_alliance, bonuses.morale_penalty)
    end
    
    -- Display intelligence report UI
    show_intelligence_report(infiltration_data.intelligence_gathered)
end
```

#### Supply Line Disruption

Advanced intelligence operations can target enemy supply lines, affecting army movement and effectiveness:

**Configuration File Extension:**
```txt
# File: /data/descr_character.txt (Extended)
type            military_scout

actions         moving_normal, spying, infiltrate_army, disrupt_supplies, gather_intelligence
wage_base       150
starting_action_points  100
specialization_bonus    military_intelligence

# Specialization effects
trait_effects   infiltration_skill +2, detection_resistance +1, supply_disruption +1
```

#### Morale Warfare Operations

Psychological warfare capabilities that can demoralize enemy forces before battle:

```lua
function execute_morale_warfare(spy_character, target_army, operation_type)
    local success_chance = calculate_morale_operation_success(
        spy_character.attributes.subterfuge,
        target_army.general.attributes.command,
        operation_type
    )
    
    if roll_success(success_chance) then
        local morale_impact = {
            propaganda_campaign = -10,
            desertion_encouragement = -15,
            false_intelligence = -8,
            supply_rumors = -12
        }
        
        apply_army_morale_modifier(target_army, morale_impact[operation_type])
        create_news_event("morale_warfare_success", spy_character, target_army)
    else
        handle_morale_operation_failure(spy_character, target_army)
    end
end
```

### Database Schema

```sql
-- Intelligence Operations Table
CREATE TABLE intelligence_operations (
    operation_id INTEGER PRIMARY KEY AUTOINCREMENT,
    spy_character_id INTEGER NOT NULL,
    target_army_id INTEGER NOT NULL,
    operation_type VARCHAR(50) NOT NULL,
    start_turn INTEGER NOT NULL,
    current_level INTEGER DEFAULT 0,
    intelligence_data TEXT, -- JSON formatted
    detection_risk REAL DEFAULT 0.1,
    status VARCHAR(20) DEFAULT 'active',
    FOREIGN KEY (spy_character_id) REFERENCES characters(character_id),
    FOREIGN KEY (target_army_id) REFERENCES armies(army_id)
);

-- Gathered Intelligence Table
CREATE TABLE gathered_intelligence (
    intelligence_id INTEGER PRIMARY KEY AUTOINCREMENT,
    operation_id INTEGER NOT NULL,
    intelligence_type VARCHAR(50) NOT NULL,
    intelligence_data TEXT NOT NULL,
    gathered_turn INTEGER NOT NULL,
    reliability_score REAL DEFAULT 1.0,
    FOREIGN KEY (operation_id) REFERENCES intelligence_operations(operation_id)
);
```

### UI/UX Specifications

#### Intelligence Report Interface
- **Location**: Campaign map overlay when selecting infiltrated army
- **Components**: 
  - Intelligence level indicator (Basic/Detailed/Strategic)
  - Gathered information panels with reliability indicators
  - Recommended actions based on intelligence
  - Risk assessment for continued operations

#### Operation Management Panel
- **Location**: Spy character details panel
- **Components**:
  - Active operations list
  - Operation status indicators
  - Intelligence gathering progress bars
  - Detection risk warnings

---

## Sabotage Operations & Infrastructure

### Overview
This system expands sabotage beyond simple settlement disruption to include targeted infrastructure attacks with strategic consequences, multi-stage operation planning, and dynamic risk/reward scaling.

### Target Categories

#### Economic Infrastructure
High-value targets that affect regional economy and settlement growth:

**Granaries and Food Storage:**
```lua
-- File: /data/scripts/campaign/mod_sabotage_economic.lua
function sabotage_granary(assassin_character, target_settlement)
    local sabotage_data = {
        target_type = "granary",
        difficulty = calculate_granary_difficulty(target_settlement),
        success_chance = calculate_success_chance(assassin_character, "economic_sabotage"),
        consequences = {
            food_shortage_turns = math.random(3, 8),
            population_growth_penalty = -50,
            public_order_impact = -15,
            economic_damage = target_settlement.wealth * 0.15
        },
        detection_consequences = {
            diplomatic_penalty = -2,
            assassin_elimination_chance = 0.3
        }
    }
    
    execute_sabotage_operation(sabotage_data)
end
```

**Mills and Production Centers:**
```txt
# File: /data/sabotage_targets.txt
target_type granary
    difficulty_base 40
    difficulty_modifiers settlement_level +10, garrison_strength +5, walls +15
    success_effects food_shortage 5, growth_penalty -50, order_penalty -10
    failure_effects diplomatic_relations -2, agent_exposure_risk 30%
    duration_turns 4-8
    economic_impact 15%

target_type mill
    difficulty_base 35
    difficulty_modifiers industrial_buildings +8, population_density +3
    success_effects production_penalty -25%, trade_income -20%
    failure_effects relations_penalty -1, detection_risk 25%
    duration_turns 3-6
    economic_impact 10%

target_type market
    difficulty_base 45
    difficulty_modifiers merchant_presence +12, wealth_level +8
    success_effects trade_routes_disrupted 3, income_loss -30%
    failure_effects merchant_guild_hostility, detection_risk 40%
    duration_turns 2-5
    economic_impact 20%
```

#### Military Infrastructure
Strategic targets affecting military capability:

**Armories and Weapon Storage:**
```lua
function sabotage_armory(assassin_character, target_settlement)
    local armory_data = get_settlement_armory_data(target_settlement)
    local sabotage_impact = {
        weapon_quality_reduction = 15, -- Reduces unit effectiveness
        recruitment_cost_increase = 25, -- More expensive to recruit units
        equipment_shortage_duration = math.random(4, 10),
        affected_unit_types = get_armory_dependent_units(target_settlement)
    }
    
    if execute_sabotage_check(assassin_character, "armory", armory_data.security_level) then
        apply_armory_sabotage_effects(target_settlement, sabotage_impact)
        create_event("armory_sabotaged", {
            settlement = target_settlement,
            assassin = assassin_character,
            impact = sabotage_impact
        })
    end
end
```

**Stables and Military Training:**
```lua
function sabotage_stables(assassin_character, target_settlement)
    local stable_effects = {
        cavalry_recruitment_disabled = true,
        cavalry_unit_effectiveness = -20,
        duration_turns = math.random(3, 7),
        affected_buildings = {"stables", "knights_stables", "destrier_stables"}
    }
    
    -- Check for specialized cavalry units in training
    local training_queue = get_settlement_training_queue(target_settlement)
    for _, unit in pairs(training_queue) do
        if unit.category == "cavalry" then
            stable_effects.training_disruption = unit.remaining_turns + 2
        end
    end
    
    execute_infrastructure_sabotage(assassin_character, target_settlement, stable_effects)
end
```

### Multi-Stage Operation Planning

Complex sabotage operations that require multiple phases and coordination:

```lua
-- File: /data/scripts/campaign/mod_multi_stage_sabotage.lua
function plan_multi_stage_operation(planner_character, target_settlement, operation_template)
    local operation = {
        operation_id = generate_unique_id(),
        planner = planner_character,
        target = target_settlement,
        stages = {},
        current_stage = 1,
        total_stages = operation_template.stage_count,
        risk_escalation = operation_template.risk_progression,
        reward_scaling = operation_template.reward_multipliers
    }
    
    -- Generate stage requirements
    for i = 1, operation.total_stages do
        local stage = {
            stage_number = i,
            required_agents = operation_template.stages[i].agent_requirements,
            target_infrastructure = operation_template.stages[i].targets,
            success_requirements = operation_template.stages[i].success_conditions,
            failure_consequences = operation_template.stages[i].failure_effects,
            stage_risk_modifier = operation_template.risk_progression[i]
        }
        table.insert(operation.stages, stage)
    end
    
    register_multi_stage_operation(operation)
    return operation.operation_id
end
```

**Example Multi-Stage Operation: "Cripple Settlement Defense"**
```txt
# File: /data/multi_stage_operations.txt
operation_template cripple_settlement_defense
    description "Systematic destruction of settlement's defensive capabilities"
    total_stages 4
    base_difficulty 60
    completion_time_estimate 8-15_turns

    stage 1 "Reconnaissance Phase"
        required_agents 1 spy
        objectives gather_garrison_intelligence, identify_weak_points
        success_conditions intelligence_level >= detailed
        failure_consequences operation_exposure_risk +10%
        risk_modifier 0.8

    stage 2 "Supply Disruption"
        required_agents 1 assassin
        objectives sabotage_armory, disrupt_weapon_supplies
        success_conditions armory_damage >= 50%
        failure_consequences agent_exposure_risk 25%, diplomatic_penalty -1
        risk_modifier 1.2

    stage 3 "Communication Breakdown"
        required_agents 1 assassin, 1 spy
        objectives eliminate_messengers, sabotage_signal_towers
        success_conditions communication_efficiency <= 30%
        failure_consequences operation_discovery_risk 40%
        risk_modifier 1.5

    stage 4 "Final Assault Preparation"
        required_agents 2 assassin
        objectives gate_sabotage, wall_weakening, garrison_demoralization
        success_conditions defensive_strength <= 60%
        failure_consequences total_operation_failure, all_agents_compromised
        risk_modifier 2.0

    completion_rewards
        siege_duration -40%
        defensive_bonus -50%
        garrison_morale -25
        siege_casualties_enemy +30%
        fame_gain 500
        experience_bonus_all_participants +2_levels
```

### Dynamic Risk/Reward Scaling

The system adjusts difficulty and rewards based on multiple factors:

```lua
function calculate_dynamic_risk_reward(operation_data, settlement_context)
    local base_difficulty = operation_data.base_difficulty
    local risk_modifiers = {
        settlement_wealth = settlement_context.wealth_level * 0.1,
        garrison_strength = settlement_context.garrison_size * 0.05,
        fortification_level = settlement_context.wall_level * 10,
        previous_sabotage_attempts = get_settlement_sabotage_history(settlement_context) * 15,
        faction_security_focus = get_faction_security_investment(settlement_context.owner) * 0.2,
        regional_stability = calculate_regional_stability(settlement_context.region) * -0.1
    }
    
    local adjusted_difficulty = base_difficulty
    for modifier_name, value in pairs(risk_modifiers) do
        adjusted_difficulty = adjusted_difficulty + value
    end
    
    -- Calculate proportional rewards
    local reward_multiplier = math.max(0.5, adjusted_difficulty / base_difficulty)
    local rewards = {
        experience_gain = operation_data.base_exp * reward_multiplier,
        monetary_reward = operation_data.base_payment * reward_multiplier,
        reputation_gain = operation_data.base_reputation * reward_multiplier,
        strategic_advantage = operation_data.base_advantage * reward_multiplier
    }
    
    return adjusted_difficulty, rewards
end
```

### Database Schema

```sql
-- Sabotage Operations Table
CREATE TABLE sabotage_operations (
    operation_id INTEGER PRIMARY KEY AUTOINCREMENT,
    operation_type VARCHAR(100) NOT NULL,
    assassin_character_id INTEGER NOT NULL,
    target_settlement_id INTEGER NOT NULL,
    target_infrastructure VARCHAR(100) NOT NULL,
    planned_turn INTEGER NOT NULL,
    execution_turn INTEGER,
    completion_turn INTEGER,
    difficulty_rating INTEGER NOT NULL,
    success_probability REAL NOT NULL,
    operation_status VARCHAR(20) DEFAULT 'planning',
    stage_data TEXT, -- JSON for multi-stage operations
    risk_factors TEXT, -- JSON formatted risk data
    reward_data TEXT, -- JSON formatted reward data
    FOREIGN KEY (assassin_character_id) REFERENCES characters(character_id),
    FOREIGN KEY (target_settlement_id) REFERENCES settlements(settlement_id)
);

-- Infrastructure Damage Tracking
CREATE TABLE infrastructure_damage (
    damage_id INTEGER PRIMARY KEY AUTOINCREMENT,
    settlement_id INTEGER NOT NULL,
    building_type VARCHAR(100) NOT NULL,
    damage_level INTEGER NOT NULL, -- 0-100 scale
    damage_effects TEXT, -- JSON formatted effects
    repair_cost INTEGER NOT NULL,
    repair_time_turns INTEGER NOT NULL,
    damage_turn INTEGER NOT NULL,
    repaired_turn INTEGER,
    FOREIGN KEY (settlement_id) REFERENCES settlements(settlement_id)
);
```

---

## Information Brokerage & Intelligence Markets

### Overview
This system creates a dynamic economy around intelligence gathering and distribution, where information becomes a tradeable commodity with fluctuating values based on relevance, rarity, and strategic importance.

### Market Dynamics

#### Intelligence Pricing System
Information value fluctuates based on multiple factors:

```lua
-- File: /data/scripts/campaign/mod_intelligence_markets.lua
function calculate_intelligence_value(intelligence_data, market_context)
    local base_value = {
        troop_movements = 100,
        army_composition = 150,
        diplomatic_relations = 200,
        settlement_defenses = 180,
        faction_finances = 250,
        planned_attacks = 400,
        succession_disputes = 300,
        technological_developments = 350
    }
    
    local value_modifiers = {
        freshness_factor = calculate_freshness_multiplier(intelligence_data.age_in_turns),
        relevance_factor = calculate_relevance_multiplier(intelligence_data, market_context.buyer_faction),
        rarity_factor = calculate_rarity_multiplier(intelligence_data.type, market_context.available_intelligence),
        strategic_importance = calculate_strategic_value(intelligence_data, market_context.current_conflicts),
        market_saturation = calculate_saturation_penalty(intelligence_data.type, market_context.recent_sales)
    }
    
    local final_value = base_value[intelligence_data.type]
    for modifier_name, multiplier in pairs(value_modifiers) do
        final_value = final_value * multiplier
    end
    
    return math.floor(final_value)
end
```

#### Regional Intelligence Hubs
Major cities serve as information trading centers:

```txt
# File: /data/intelligence_hubs.txt
# Regional Intelligence Hubs Configuration

hub_city venice
    region northern_italy
    specialization maritime_intelligence, trade_networks
    market_size large
    base_demand_multiplier 1.4
    information_types_preferred naval_movements, trade_routes, diplomatic_missions
    security_level high
    broker_network_quality excellent
    operating_costs_modifier 1.2

hub_city paris
    region northern_france
    specialization court_intrigue, political_intelligence
    market_size very_large
    base_demand_multiplier 1.6
    information_types_preferred succession_disputes, court_scandals, diplomatic_secrets
    security_level very_high
    broker_network_quality exceptional
    operating_costs_modifier 1.5

hub_city constantinople
    region anatolia
    specialization military_intelligence, eastern_politics
    market_size large
    base_demand_multiplier 1.3
    information_types_preferred army_movements, siege_preparations, eastern_diplomacy
    security_level moderate
    broker_network_quality good
    operating_costs_modifier 1.0

hub_city cairo
    region egypt
    specialization trade_intelligence, desert_routes
    market_size moderate
    base_demand_multiplier 1.1
    information_types_preferred caravan_routes, spice_trade, nomad_movements
    security_level low
    broker_network_quality fair
    operating_costs_modifier 0.8
```

#### Broker Network System
Intelligence brokers facilitate information trade:

```lua
function establish_broker_network(character, hub_city)
    local network_data = {
        broker_character_id = character.character_id,
        hub_city_id = hub_city.settlement_id,
        network_size = 1, -- Starts small, grows over time
        reputation_level = 0, -- Affects trust and pricing
        specialization_areas = {}, -- Developed based on traded intelligence types
        active_contracts = {},
        client_base = {},
        established_turn = cm:turn_number()
    }
    
    -- Calculate establishment costs
    local establishment_cost = calculate_broker_establishment_cost(hub_city)
    if character.faction.treasury >= establishment_cost then
        character.faction:treasury_mod(-establishment_cost)
        save_broker_network(network_data)
        trigger_event("broker_network_established", character, hub_city)
        return true
    end
    
    return false
end

function process_intelligence_auction(hub_city, intelligence_item)
    local auction_data = {
        item = intelligence_item,
        hub = hub_city,
        bidders = get_interested_factions(intelligence_item),
        starting_bid = calculate_minimum_bid(intelligence_item),
        auction_duration_turns = 2,
        current_highest_bid = 0,
        highest_bidder = nil
    }
    
    -- Notify potential bidders
    for _, faction in pairs(auction_data.bidders) do
        if faction.has_broker_network_in_city(hub_city) then
            notify_faction_of_auction(faction, auction_data)
        end
    end
    
    register_active_auction(auction_data)
end
```

### Disinformation Campaigns

Advanced intelligence operations can spread false information:

```lua
function launch_disinformation_campaign(spy_character, target_faction, disinformation_type)
    local campaign_data = {
        operative = spy_character,
        target = target_faction,
        disinformation_content = generate_disinformation_content(disinformation_type),
        credibility_rating = calculate_disinformation_credibility(spy_character),
        distribution_method = select_optimal_distribution_method(target_faction),
        success_metrics = {
            belief_threshold = 0.6, -- Percentage of target faction that must believe
            duration_turns = math.random(3, 8),
            strategic_impact_required = calculate_minimum_impact()
        }
    }
    
    execute_disinformation_campaign(campaign_data)
end

-- Disinformation types and effects
local disinformation_effects = {
    false_army_movements = {
        effect = "misdirect_military_planning",
        impact_duration = 5,
        detection_difficulty = 30,
        strategic_value = "high"
    },
    
    fabricated_diplomatic_agreements = {
        effect = "disrupt_alliance_formation",
        impact_duration = 8,
        detection_difficulty = 45,
        strategic_value = "very_high"
    },
    
    economic_misinformation = {
        effect = "influence_trade_decisions",
        impact_duration = 4,
        detection_difficulty = 25,
        strategic_value = "moderate"
    },
    
    fake_succession_crisis = {
        effect = "internal_faction_instability",
        impact_duration = 10,
        detection_difficulty = 50,
        strategic_value = "extreme"
    }
}
```

### Database Schema

```sql
-- Intelligence Market Data
CREATE TABLE intelligence_market_items (
    item_id INTEGER PRIMARY KEY AUTOINCREMENT,
    intelligence_type VARCHAR(100) NOT NULL,
    source_character_id INTEGER,
    hub_city_id INTEGER NOT NULL,
    intelligence_data TEXT NOT NULL, -- JSON formatted
    base_value INTEGER NOT NULL,
    current_market_price INTEGER,
    age_in_turns INTEGER DEFAULT 0,
    reliability_score REAL DEFAULT 1.0,
    market_status VARCHAR(20) DEFAULT 'available',
    listed_turn INTEGER NOT NULL,
    sold_turn INTEGER,
    buyer_faction_id INTEGER,
    FOREIGN KEY (source_character_id) REFERENCES characters(character_id),
    FOREIGN KEY (hub_city_id) REFERENCES settlements(settlement_id),
    FOREIGN KEY (buyer_faction_id) REFERENCES factions(faction_id)
);

-- Broker Networks
CREATE TABLE broker_networks (
    network_id INTEGER PRIMARY KEY AUTOINCREMENT,
    character_id INTEGER NOT NULL,
    hub_city_id INTEGER NOT NULL,
    network_size INTEGER DEFAULT 1,
    reputation_level INTEGER DEFAULT 0,
    specialization_data TEXT, -- JSON formatted
    established_turn INTEGER NOT NULL,
    total_transactions INTEGER DEFAULT 0,
    total_revenue INTEGER DEFAULT 0,
    FOREIGN KEY (character_id) REFERENCES characters(character_id),
    FOREIGN KEY (hub_city_id) REFERENCES settlements(settlement_id)
);

-- Intelligence Auctions
CREATE TABLE intelligence_auctions (
    auction_id INTEGER PRIMARY KEY AUTOINCREMENT,
    item_id INTEGER NOT NULL,
    hub_city_id INTEGER NOT NULL,
    starting_bid INTEGER NOT NULL,
    current_highest_bid INTEGER DEFAULT 0,
    highest_bidder_faction_id INTEGER,
    auction_end_turn INTEGER NOT NULL,
    auction_status VARCHAR(20) DEFAULT 'active',
    FOREIGN KEY (item_id) REFERENCES intelligence_market_items(item_id),
    FOREIGN KEY (hub_city_id) REFERENCES settlements(settlement_id),
    FOREIGN KEY (highest_bidder_faction_id) REFERENCES factions(faction_id)
);
```

### UI/UX Specifications

#### Intelligence Market Interface
- **Location**: Hub city settlement view with dedicated "Intelligence Market" tab
- **Components**:
  - Available intelligence listings with value indicators
  - Active auction display with countdown timers
  - Broker network management panel
  - Market trend analysis charts
  - Intelligence portfolio management

#### Auction Interface
- **Location**: Modal overlay when participating in auctions
- **Components**:
  - Intelligence preview with reliability indicators
  - Current bid status and remaining time
  - Bidding interface with suggested bid amounts
  - Competitor activity indicators

---

## Agent Specialization & Training Schools

### Overview
This system expands agent development beyond simple skill progression to include specialized training paths, regional expertise, and institutional knowledge through dedicated training facilities.

### Specialist Agent Types

#### Court Infiltrators
Masters of social manipulation and political espionage:

```txt
# File: /data/descr_character.txt (Extended Specialist Types)

type            court_infiltrator
actions         moving_normal, spying, infiltrate_court, manipulate_politics, gather_court_intelligence
wage_base       300
starting_action_points  90
specialization_bonus   court_intrigue

trait_effects   charm +3, subterfuge +2, political_knowledge +3
special_abilities court_access, noble_disguise, rumor_spreading
training_requirements noble_etiquette_school, completed_missions >= 5
advancement_path spy -> court_infiltrator -> master_manipulator

faction         venice
strat_model     venetian_court_agent
dictionary      court_infiltrator_name

faction         france
strat_model     french_court_agent
dictionary      court_infiltrator_name

faction         hre
strat_model     imperial_court_agent
dictionary      court_infiltrator_name
```

**Specialized Abilities:**
```lua
-- Court Infiltrator Special Actions
function infiltrate_royal_court(infiltrator, target_settlement)
    if not target_settlement.has_royal_court then
        return false, "No royal court present"
    end
    
    local infiltration_success = calculate_court_infiltration_chance(
        infiltrator.traits.charm,
        infiltrator.traits.political_knowledge,
        target_settlement.court_security_level,
        infiltrator.has_noble_connections()
    )
    
    if roll_success(infiltration_success) then
        establish_court_presence(infiltrator, target_settlement)
        unlock_court_intelligence_gathering(infiltrator, target_settlement)
        return true, "Successfully infiltrated court"
    else
        handle_court_infiltration_failure(infiltrator, target_settlement)
        return false, "Infiltration attempt failed"
    end
end

function manipulate_court_politics(infiltrator, target_faction, manipulation_type)
    local manipulation_effects = {
        succession_influence = {
            effect = "alter_succession_preferences",
            difficulty = 70,
            impact = "major_political_shift",
            duration = "permanent"
        },
        
        advisor_corruption = {
            effect = "corrupt_royal_advisors",
            difficulty = 45,
            impact = "policy_influence",
            duration = "medium_term"
        },
        
        marriage_arrangement = {
            effect = "influence_diplomatic_marriages",
            difficulty = 55,
            impact = "alliance_disruption",
            duration = "long_term"
        },
        
        scandal_creation = {
            effect = "fabricate_court_scandal",
            difficulty = 40,
            impact = "reputation_damage",
            duration = "short_term"
        }
    }
    
    execute_political_manipulation(infiltrator, target_faction, manipulation_effects[manipulation_type])
end
```

#### Military Scouts
Specialists in battlefield intelligence and army reconnaissance:

```txt
type            military_scout
actions         moving_normal, spying, army_reconnaissance, battlefield_analysis, supply_line_tracking
wage_base       200
starting_action_points  120
specialization_bonus   military_intelligence

trait_effects   command +1, subterfuge +2, logistics +3
special_abilities rapid_movement, terrain_expertise, combat_assessment
training_requirements military_academy, veteran_status
advancement_path spy -> military_scout -> intelligence_commander

faction         england
strat_model     english_military_scout
battle_model    English_Scout
dictionary      military_scout_name

faction         france
strat_model     french_military_scout  
battle_model    French_Scout
dictionary      military_scout_name
```

**Specialized Abilities:**
```lua
function conduct_battlefield_analysis(scout, target_region)
    local analysis_data = {
        terrain_advantages = analyze_terrain_features(target_region),
        defensive_positions = identify_strategic_locations(target_region),
        supply_routes = map_supply_lines(target_region),
        weather_patterns = predict_weather_impact(target_region),
        local_resources = catalog_available_resources(target_region)
    }
    
    local analysis_quality = calculate_analysis_quality(
        scout.traits.logistics,
        scout.experience_level,
        scout.has_terrain_expertise(target_region.climate)
    )
    
    return generate_battlefield_intelligence_report(analysis_data, analysis_quality)
end

function track_army_movements(scout, target_army, tracking_duration)
    local tracking_data = {
        movement_patterns = {},
        supply_requirements = {},
        strategic_objectives = {},
        vulnerabilities = {},
        tracking_start_turn = cm:turn_number()
    }
    
    for turn = 1, tracking_duration do
        local turn_data = gather_army_movement_data(scout, target_army, turn)
        table.insert(tracking_data.movement_patterns, turn_data)
        
        if turn >= 3 then -- Pattern recognition after 3 turns
            tracking_data.predicted_destination = predict_army_destination(tracking_data.movement_patterns)
        end
        
        if turn >= 5 then -- Strategic analysis after 5 turns
            tracking_data.strategic_assessment = analyze_strategic_intentions(tracking_data)
        end
    end
    
    return tracking_data
end
```

#### Economic Saboteurs
Specialists in disrupting enemy economies and trade networks:

```txt
type            economic_saboteur
actions         moving_normal, sabotage, disrupt_trade_routes, manipulate_markets, industrial_espionage
wage_base       250
starting_action_points  100
specialization_bonus   economic_warfare

trait_effects   subterfuge +3, trade_knowledge +2, technical_skill +2
special_abilities market_manipulation, trade_route_identification, industrial_sabotage
training_requirements merchant_guild_training, sabotage_expertise
advancement_path assassin -> economic_saboteur -> trade_war_master

faction         venice
strat_model     venetian_trade_saboteur
dictionary      economic_saboteur_name
```

**Specialized Abilities:**
```lua
function disrupt_trade_network(saboteur, target_trade_route)
    local disruption_methods = {
        caravan_sabotage = {
            success_chance = 0.6,
            impact_duration = 4,
            economic_damage = 0.25,
            detection_risk = 0.3
        },
        
        market_manipulation = {
            success_chance = 0.4,
            impact_duration = 6,
            economic_damage = 0.35,
            detection_risk = 0.2
        },
        
        warehouse_sabotage = {
            success_chance = 0.5,
            impact_duration = 3,
            economic_damage = 0.4,
            detection_risk = 0.4
        }
    }
    
    local selected_method = choose_optimal_disruption_method(saboteur, target_trade_route)
    execute_trade_disruption(saboteur, target_trade_route, disruption_methods[selected_method])
end

function conduct_industrial_espionage(saboteur, target_settlement, technology_focus)
    local espionage_targets = {
        military_technology = {
            difficulty = 60,
            reward = "unlock_enemy_unit_types",
            discovery_penalty = "major_diplomatic_incident"
        },
        
        agricultural_techniques = {
            difficulty = 35,
            reward = "farming_efficiency_bonus",
            discovery_penalty = "minor_diplomatic_penalty"
        },
        
        construction_methods = {
            difficulty = 45,
            reward = "building_cost_reduction",
            discovery_penalty = "trade_relationship_damage"
        }
    }
    
    execute_industrial_espionage_mission(saboteur, target_settlement, espionage_targets[technology_focus])
end
```

#### Diplomatic Observers
Specialists in monitoring and influencing international relations:

```txt
type            diplomatic_observer
actions         moving_normal, spying, monitor_diplomacy, influence_negotiations, gather_treaty_intelligence
wage_base       280
starting_action_points  90
specialization_bonus   diplomatic_intelligence

trait_effects   subterfuge +2, languages +3, political_knowledge +2
special_abilities diplomatic_immunity, treaty_analysis, negotiation_influence
training_requirements diplomatic_academy, language_mastery
advancement_path diplomat -> diplomatic_observer -> master_diplomat

faction         papal_states
strat_model     papal_diplomatic_observer
dictionary      diplomatic_observer_name
```

### Regional Training Centers

Specialized facilities that provide enhanced training and regional expertise:

```txt
# File: /data/training_schools.txt

training_school venetian_intelligence_academy
    location venice
    building_requirements spy_network, library, diplomatic_quarter
    construction_cost 8000
    maintenance_cost 200_per_turn
    specializations court_infiltration, maritime_intelligence
    training_bonus_court_infiltrators +2_levels
    regional_expertise italian_states, byzantine_empire, mediterranean_trade
    graduation_traits venetian_connections, maritime_knowledge
    capacity 4_agents_simultaneously
    training_duration 8_turns

training_school french_royal_spy_school  
    location paris
    building_requirements castle, court, spy_network
    construction_cost 10000
    maintenance_cost 300_per_turn
    specializations court_intrigue, political_manipulation
    training_bonus_court_infiltrators +3_levels
    regional_expertise western_europe, papal_politics
    graduation_traits royal_court_access, noble_etiquette
    capacity 3_agents_simultaneously
    training_duration 10_turns

training_school english_military_intelligence_college
    location london
    building_requirements barracks, spy_network, armorer
    construction_cost 6000
    maintenance_cost 150_per_turn
    specializations military_scouting, battlefield_intelligence
    training_bonus_military_scouts +2_levels
    regional_expertise british_isles, northern_france
    graduation_traits terrain_expertise_temperate, siege_analysis
    capacity 5_agents_simultaneously
    training_duration 6_turns
```

**Training School Implementation:**
```lua
-- File: /data/scripts/campaign/mod_training_schools.lua
function enroll_agent_in_training_school(agent_character, school_id, specialization)
    local school_data = get_training_school_data(school_id)
    local enrollment_cost = calculate_enrollment_cost(agent_character, school_data, specialization)
    
    if not validate_enrollment_requirements(agent_character, school_data) then
        return false, "Agent does not meet enrollment requirements"
    end
    
    if school_data.current_students >= school_data.capacity then
        return false, "School at maximum capacity"
    end
    
    if agent_character.faction.treasury < enrollment_cost then
        return false, "Insufficient funds for enrollment"
    end
    
    local training_data = {
        agent_id = agent_character.character_id,
        school_id = school_id,
        specialization = specialization,
        enrollment_turn = cm:turn_number(),
        completion_turn = cm:turn_number() + school_data.training_duration,
        progress = 0,
        training_bonuses = calculate_training_bonuses(school_data, specialization)
    }
    
    agent_character.faction:treasury_mod(-enrollment_cost)
    register_agent_training(training_data)
    school_data.current_students = school_data.current_students + 1
    
    return true, "Agent successfully enrolled"
end

function process_training_completion(training_data)
    local agent = get_character_by_id(training_data.agent_id)
    local school = get_training_school_data(training_data.school_id)
    
    -- Apply training bonuses
    for trait, bonus in pairs(training_data.training_bonuses) do
        agent:add_trait_points(trait, bonus)
    end
    
    -- Grant specialization abilities
    agent:add_specialization(training_data.specialization)
    
    -- Award graduation traits
    for _, trait in pairs(school.graduation_traits) do
        agent:add_trait(trait, 1)
    end
    
    -- Update school statistics
    school.graduates = school.graduates + 1
    school.current_students = school.current_students - 1
    
    -- Create graduation event
    trigger_event("agent_training_completed", agent, school, training_data.specialization)
end
```

### Database Schema

```sql
-- Agent Specializations
CREATE TABLE agent_specializations (
    specialization_id INTEGER PRIMARY KEY AUTOINCREMENT,
    character_id INTEGER NOT NULL,
    specialization_type VARCHAR(100) NOT NULL,
    specialization_level INTEGER DEFAULT 1,
    acquired_turn INTEGER NOT NULL,
    training_school_id INTEGER,
    experience_points INTEGER DEFAULT 0,
    special_abilities TEXT, -- JSON formatted list of unlocked abilities
    FOREIGN KEY (character_id) REFERENCES characters(character_id),
    FOREIGN KEY (training_school_id) REFERENCES training_schools(school_id)
);

-- Training Schools
CREATE TABLE training_schools (
    school_id INTEGER PRIMARY KEY AUTOINCREMENT,
    settlement_id INTEGER NOT NULL,
    school_name VARCHAR(200) NOT NULL,
    school_type VARCHAR(100) NOT NULL,
    construction_turn INTEGER NOT NULL,
    building_cost INTEGER NOT NULL,
    maintenance_cost INTEGER NOT NULL,
    current_students INTEGER DEFAULT 0,
    maximum_capacity INTEGER NOT NULL,
    total_graduates INTEGER DEFAULT 0,
    specializations_offered TEXT, -- JSON formatted
    regional_expertise TEXT, -- JSON formatted
    training_bonuses TEXT, -- JSON formatted
    FOREIGN KEY (settlement_id) REFERENCES settlements(settlement_id)
);

-- Training Records
CREATE TABLE agent_training_records (
    record_id INTEGER PRIMARY KEY AUTOINCREMENT,
    character_id INTEGER NOT NULL,
    school_id INTEGER NOT NULL,
    specialization VARCHAR(100) NOT NULL,
    enrollment_turn INTEGER NOT NULL,
    completion_turn INTEGER NOT NULL,
    training_cost INTEGER NOT NULL,
    final_bonuses TEXT, -- JSON formatted bonuses received
    graduation_status VARCHAR(50) DEFAULT 'in_progress',
    FOREIGN KEY (character_id) REFERENCES characters(character_id),
    FOREIGN KEY (school_id) REFERENCES training_schools(school_id)
);
```

---

## Expanded Assassination System - Officer Targeting

### Overview
This system transforms assassination from simple character elimination to sophisticated military decapitation strikes, targeting the complex hierarchies within enemy armies to achieve strategic advantages through precise elimination of key personnel.

### Military Hierarchy Targeting

#### Officer Classification System
Different officer types provide varying strategic value when eliminated:

```txt
# File: /data/military_hierarchy.txt

officer_type commanding_general
    strategic_value 1000
    elimination_difficulty 85
    replacement_time 12_turns
    elimination_effects army_morale -40, command_effectiveness -60%, tactical_coordination -50%
    protection_level very_high
    bodyguard_strength 8-12_elite_units
    detection_countermeasures extensive

officer_type field_marshal
    strategic_value 800  
    elimination_difficulty 75
    replacement_time 8_turns
    elimination_effects multi_army_coordination -30%, strategic_planning -40%
    protection_level high
    bodyguard_strength 5-8_veteran_units
    detection_countermeasures significant

officer_type major_commander
    strategic_value 600
    elimination_difficulty 60
    replacement_time 6_turns
    elimination_effects army_cohesion -25%, unit_effectiveness -20%
    protection_level moderate
    bodyguard_strength 3-5_regular_units
    detection_countermeasures moderate

officer_type captain_specialist
    strategic_value 300
    elimination_difficulty 40
    replacement_time 4_turns  
    elimination_effects specialist_unit_effectiveness -50%, tactical_options -1
    protection_level low
    bodyguard_strength 1-2_bodyguards
    detection_countermeasures minimal

officer_type siege_engineer
    strategic_value 400
    elimination_difficulty 35
    replacement_time 8_turns
    elimination_effects siege_effectiveness -60%, construction_speed -75%
    protection_level low
    bodyguard_strength 0-1_bodyguards
    detection_countermeasures minimal_specialized

officer_type quartermaster
    strategic_value 350
    elimination_difficulty 30
    replacement_time 5_turns
    elimination_effects supply_efficiency -40%, movement_speed -20%, attrition +50%
    protection_level very_low
    bodyguard_strength 0_bodyguards
    detection_countermeasures none
```

#### Strategic Elimination Campaigns
Multi-target assassination campaigns that systematically weaken enemy military capability:

```lua
-- File: /data/scripts/campaign/mod_officer_elimination.lua
function plan_decapitation_campaign(assassin_character, target_army, campaign_objectives)
    local campaign_data = {
        assassin = assassin_character,
        target_army = target_army,
        primary_objectives = identify_high_value_targets(target_army),
        secondary_objectives = identify_supporting_targets(target_army),
        elimination_sequence = optimize_elimination_order(target_army),
        timeline_estimate = calculate_campaign_duration(target_army),
        risk_assessment = analyze_campaign_risks(assassin_character, target_army),
        success_probability = calculate_overall_success_chance(assassin_character, target_army)
    }
    
    -- Validate campaign feasibility
    if campaign_data.success_probability < 0.2 then
        return false, "Campaign success probability too low"
    end
    
    register_elimination_campaign(campaign_data)
    return true, campaign_data
end

function execute_targeted_assassination(assassin_character, target_officer, elimination_method)
    local elimination_methods = {
        direct_assault = {
            success_base = 0.3,
            detection_risk = 0.8,
            casualty_risk = 0.6,
            time_required = 1,
            noise_level = "high"
        },
        
        poison_assassination = {
            success_base = 0.6,
            detection_risk = 0.3,
            casualty_risk = 0.1,
            time_required = 3,
            noise_level = "low"
        },
        
        staged_accident = {
            success_base = 0.4,
            detection_risk = 0.2,
            casualty_risk = 0.2,
            time_required = 5,
            noise_level = "none"
        },
        
        sniper_elimination = {
            success_base = 0.7,
            detection_risk = 0.5,
            casualty_risk = 0.3,
            time_required = 2,
            noise_level = "medium"
        }
    }
    
    local method_data = elimination_methods[elimination_method]
    local modified_success_chance = calculate_modified_success_chance(
        assassin_character,
        target_officer,
        method_data
    )
    
    if roll_success(modified_success_chance) then
        execute_successful_elimination(assassin_character, target_officer, method_data)
    else
        handle_elimination_failure(assassin_character, target_officer, method_data)
    end
end
```

#### Officer Protection Measures
Dynamic security systems that adapt to assassination threats:

```lua
function implement_officer_protection(faction, threat_level)
    local protection_measures = {
        low_threat = {
            bodyguard_increase = 1,
            movement_restrictions = false,
            counterintelligence_active = false,
            cost_multiplier = 1.1
        },
        
        medium_threat = {
            bodyguard_increase = 3,
            movement_restrictions = true,
            counterintelligence_active = true,
            decoy_deployment = true,
            cost_multiplier = 1.4
        },
        
        high_threat = {
            bodyguard_increase = 6,
            movement_restrictions = true,
            counterintelligence_active = true,
            decoy_deployment = true,
            secure_locations_only = true,
            communication_encryption = true,
            cost_multiplier = 2.0
        },
        
        critical_threat = {
            bodyguard_increase = 10,
            movement_restrictions = true,
            counterintelligence_active = true,
            decoy_deployment = true,
            secure_locations_only = true,
            communication_encryption = true,
            trusted_personnel_only = true,
            emergency_protocols = true,
            cost_multiplier = 3.0
        }
    }
    
    local measures = protection_measures[threat_level]
    apply_protection_measures_to_faction_officers(faction, measures)
    faction:treasury_mod(-(calculate_protection_costs(faction, measures)))
end

function detect_assassination_attempt(target_officer, assassin_character, elimination_method)
    local detection_factors = {
        officer_protection_level = get_officer_protection_level(target_officer),
        assassin_skill = assassin_character.traits.subterfuge,
        method_stealth_rating = get_method_stealth_rating(elimination_method),
        counterintelligence_presence = get_counterintelligence_strength(target_officer.army),
        previous_attempt_awareness = get_faction_assassination_awareness(target_officer.faction)
    }
    
    local detection_chance = calculate_detection_probability(detection_factors)
    
    if roll_success(detection_chance) then
        trigger_assassination_detection(target_officer, assassin_character)
        implement_emergency_protection_protocols(target_officer)
        alert_counterintelligence_network(target_officer.faction, assassin_character.faction)
        return true
    end
    
    return false
end
```

#### Counter-Intelligence Networks
Defensive systems that hunt enemy agents:

```lua
-- Counter-Intelligence Implementation
function establish_counterintelligence_network(faction, region, funding_level)
    local network_data = {
        faction_id = faction.faction_id,
        region_id = region.region_id,
        funding_level = funding_level,
        agent_count = calculate_agent_count(funding_level),
        detection_capability = calculate_detection_rating(funding_level),
        investigation_speed = calculate_investigation_speed(funding_level),
        active_investigations = {},
        identified_threats = {},
        establishment_turn = cm:turn_number()
    }
    
    register_counterintelligence_network(network_data)
    spawn_counterintelligence_agents(network_data)
    return network_data
end

function investigate_enemy_agent_activity(network, suspected_agent)
    local investigation_data = {
        network_id = network.network_id,
        target_agent_id = suspected_agent.character_id,
        investigation_start_turn = cm:turn_number(),
        evidence_level = 0,
        investigation_methods = {},
        confirmation_threshold = 70, -- Evidence level required for action
        estimated_completion_turns = math.random(3, 8)
    }
    
    -- Different investigation methods
    local investigation_methods = {
        surveillance = {
            evidence_gain_per_turn = 15,
            detection_risk = 0.1,
            cost_per_turn = 50
        },
        
        background_check = {
            evidence_gain_per_turn = 8,
            detection_risk = 0.05,
            cost_per_turn = 25
        },
        
        infiltration = {
            evidence_gain_per_turn = 25,
            detection_risk = 0.3,
            cost_per_turn = 100
        },
        
        informant_network = {
            evidence_gain_per_turn = 12,
            detection_risk = 0.2,
            cost_per_turn = 75
        }
    }
    
    register_active_investigation(investigation_data)
    select_investigation_methods(investigation_data, investigation_methods)
end
```

### Database Schema

```sql
-- Military Officers Table
CREATE TABLE military_officers (
    officer_id INTEGER PRIMARY KEY AUTOINCREMENT,
    character_id INTEGER UNIQUE,
    army_id INTEGER NOT NULL,
    officer_type VARCHAR(100) NOT NULL,
    rank_level INTEGER NOT NULL,
    strategic_value INTEGER NOT NULL,
    protection_level VARCHAR(50) NOT NULL,
    bodyguard_count INTEGER DEFAULT 0,
    specialized_skills TEXT, -- JSON formatted
    elimination_difficulty INTEGER NOT NULL,
    replacement_estimated_turns INTEGER NOT NULL,
    FOREIGN KEY (character_id) REFERENCES characters(character_id),
    FOREIGN KEY (army_id) REFERENCES armies(army_id)
);

-- Elimination Campaigns
CREATE TABLE elimination_campaigns (
    campaign_id INTEGER PRIMARY KEY AUTOINCREMENT,
    assassin_character_id INTEGER NOT NULL,
    target_army_id INTEGER NOT NULL,
    campaign_status VARCHAR(50) DEFAULT 'planning',
    primary_targets TEXT, -- JSON formatted list of officer IDs
    secondary_targets TEXT, -- JSON formatted list of officer IDs
    elimination_sequence TEXT, -- JSON formatted sequence plan
    start_turn INTEGER NOT NULL,
    estimated_completion_turn INTEGER,
    success_probability REAL NOT NULL,
    eliminated_officers TEXT, -- JSON formatted list of completed eliminations
    FOREIGN KEY (assassin_character_id) REFERENCES characters(character_id),
    FOREIGN KEY (target_army_id) REFERENCES armies(army_id)
);

-- Counter-Intelligence Networks
CREATE TABLE counterintelligence_networks (
    network_id INTEGER PRIMARY KEY AUTOINCREMENT,
    faction_id INTEGER NOT NULL,
    region_id INTEGER NOT NULL,
    funding_level VARCHAR(50) NOT NULL,
    agent_count INTEGER NOT NULL,
    detection_rating INTEGER NOT NULL,
    active_investigations_count INTEGER DEFAULT 0,
    successful_detections INTEGER DEFAULT 0,
    establishment_turn INTEGER NOT NULL,
    operational_status VARCHAR(50) DEFAULT 'active',
    FOREIGN KEY (faction_id) REFERENCES factions(faction_id),
    FOREIGN KEY (region_id) REFERENCES regions(region_id)
);

-- Agent Investigations
CREATE TABLE agent_investigations (
    investigation_id INTEGER PRIMARY KEY AUTOINCREMENT,
    network_id INTEGER NOT NULL,
    target_agent_character_id INTEGER NOT NULL,
    investigation_start_turn INTEGER NOT NULL,
    current_evidence_level INTEGER DEFAULT 0,
    investigation_methods TEXT, -- JSON formatted methods being used
    estimated_completion_turn INTEGER,
    investigation_status VARCHAR(50) DEFAULT 'active',
    final_outcome VARCHAR(100),
    FOREIGN KEY (network_id) REFERENCES counterintelligence_networks(network_id),
    FOREIGN KEY (target_agent_character_id) REFERENCES characters(character_id)
);
```

---

## Database Schema

### Core System Tables

```sql
-- Enhanced Characters Table (extends existing)
ALTER TABLE characters ADD COLUMN specialization_type VARCHAR(100);
ALTER TABLE characters ADD COLUMN specialization_level INTEGER DEFAULT 0;
ALTER TABLE characters ADD COLUMN training_school_graduated VARCHAR(200);
ALTER TABLE characters ADD COLUMN regional_expertise TEXT; -- JSON formatted
ALTER TABLE characters ADD COLUMN operation_history TEXT; -- JSON formatted
ALTER TABLE characters ADD COLUMN reputation_score INTEGER DEFAULT 0;

-- Master Operations Tracking
CREATE TABLE operations_master (
    operation_id INTEGER PRIMARY KEY AUTOINCREMENT,
    operation_type VARCHAR(100) NOT NULL,
    operation_subtype VARCHAR(100),
    initiating_character_id INTEGER NOT NULL,
    target_type VARCHAR(100) NOT NULL, -- 'army', 'settlement', 'character', 'faction'
    target_id INTEGER NOT NULL,
    operation_status VARCHAR(50) DEFAULT 'planning',
    start_turn INTEGER NOT NULL,
    estimated_completion_turn INTEGER,
    actual_completion_turn INTEGER,
    success_level VARCHAR(50), -- 'critical_success', 'success', 'partial_success', 'failure', 'critical_failure'
    strategic_impact_score INTEGER DEFAULT 0,
    operation_data TEXT, -- JSON formatted operation-specific data
    FOREIGN KEY (initiating_character_id) REFERENCES characters(character_id)
);

-- Dynamic Events System
CREATE TABLE dynamic_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT,
    event_type VARCHAR(100) NOT NULL,
    triggering_operation_id INTEGER,
    affected_factions TEXT, -- JSON formatted list of faction IDs
    event_magnitude VARCHAR(50) NOT NULL, -- 'minor', 'moderate', 'major', 'critical'
    event_duration_turns INTEGER DEFAULT 1,
    event_start_turn INTEGER NOT NULL,
    event_end_turn INTEGER,
    event_effects TEXT, -- JSON formatted effects
    cascade_events TEXT, -- JSON formatted list of triggered events
    FOREIGN KEY (triggering_operation_id) REFERENCES operations_master(operation_id)
);

-- Faction Intelligence Ratings
CREATE TABLE faction_intelligence_ratings (
    rating_id INTEGER PRIMARY KEY AUTOINCREMENT,
    faction_id INTEGER NOT NULL,
    intelligence_category VARCHAR(100) NOT NULL,
    current_rating INTEGER NOT NULL,
    maximum_rating INTEGER NOT NULL,
    last_updated_turn INTEGER NOT NULL,
    rating_modifiers TEXT, -- JSON formatted temporary modifiers
    historical_peaks TEXT, -- JSON formatted historical high ratings
    FOREIGN KEY (faction_id) REFERENCES factions(faction_id)
);
```

---

## Implementation Timeline

### Phase 1: Foundation Systems (Months 1-3)
**Priority: Core Infrastructure**

**Month 1: Database and Core Framework**
- Implement all database schemas
- Create base Lua script framework
- Extend existing character system for specializations
- Basic UI framework modifications

**Month 2: Military Intelligence System**
- Army infiltration mechanics
- Intelligence gathering algorithms
- Pre-battle advantage system
- Basic intelligence UI

**Month 3: Agent Specialization Framework**
- Four specialist agent types implementation
- Training school buildings and mechanics
- Specialization advancement system
- Agent management UI improvements

### Phase 2: Market and Sabotage Systems (Months 4-6)
**Priority: Economic and Infrastructure Systems**

**Month 4: Information Brokerage System**
- Intelligence market mechanics
- Broker network establishment
- Pricing algorithms and market dynamics
- Market interface development

**Month 5: Sabotage Operations**
- Infrastructure targeting system
- Multi-stage operation planning
- Dynamic risk/reward calculations
- Sabotage effects implementation

**Month 6: Integration and Balancing**
- System integration testing
- Initial balance pass
- Player feedback integration
- Performance optimization

### Phase 3: Advanced Systems (Months 7-9)
**Priority: Advanced Mechanics and Assassination**

**Month 7: Disinformation Campaigns**
- Disinformation content generation
- Credibility and distribution systems
- Counter-propaganda mechanics
- Advanced intelligence UI

**Month 8: Officer Targeting System**
- Military hierarchy implementation
- Targeted assassination mechanics
- Counter-intelligence networks
- Protection measure systems

**Month 9: Counter-Intelligence and Security**
- Full counter-intelligence implementation
- Dynamic threat response systems
- Investigation and detection mechanics
- Security escalation protocols

### Phase 4: Polish and Launch (Months 10-12)
**Priority: Refinement and Release Preparation**

**Month 10: Balance and Refinement**
- Comprehensive balance testing
- AI behavior improvements
- System interaction refinements
- Performance optimization

**Month 11: UI/UX Polish**
- Complete UI overhaul and polish
- Accessibility improvements
- Tutorial system development
- Help system implementation

**Month 12: Launch Preparation**
- Final testing and bug fixes
- Documentation completion
- Community beta testing
- Launch preparation and marketing

---

## Balancing Framework

### Core Balance Principles

#### Risk/Reward Scaling
All operations follow a fundamental risk/reward curve:

```lua
-- Base Balance Formula
function calculate_risk_reward_balance(operation_difficulty, potential_impact)
    local risk_factor = operation_difficulty / 100 -- Normalize to 0-1 scale
    local reward_multiplier = 1 + (risk_factor * 2) -- Linear scaling with minimum 1x
    local failure_penalty_multiplier = risk_factor * 1.5 -- Failure costs scale with risk
    
    return {
        success_reward = potential_impact * reward_multiplier,
        failure_penalty = potential_impact * failure_penalty_multiplier * 0.5,
        experience_gain = math.floor(operation_difficulty / 10)
    }
end
```

#### Economic Balance Parameters

**Agent Costs and Maintenance:**
```txt
# Economic Balance Configuration
agent_base_costs
    spy_recruitment 400
    assassin_recruitment 600
    specialist_recruitment_multiplier 1.5
    training_school_multiplier 2.0

agent_maintenance_costs
    spy_base_wage 100
    assassin_base_wage 200
    specialist_wage_multiplier 1.4
    active_operation_bonus 50%

operation_costs
    intelligence_gathering_base 50_per_turn
    sabotage_operation_base 200_per_attempt  
    assassination_attempt_base 300_per_attempt
    multi_stage_operation_multiplier 1.8
```

#### Difficulty Scaling Factors

**Dynamic Difficulty Adjustment:**
```lua
function calculate_dynamic_difficulty(base_difficulty, context_factors)
    local difficulty_modifiers = {
        target_faction_security_rating = context_factors.security * 0.1,
        previous_operation_awareness = context_factors.awareness * 0.15,
        agent_reputation_penalty = context_factors.agent_fame * 0.05,
        regional_stability_bonus = (100 - context_factors.stability) * 0.08,
        seasonal_modifiers = context_factors.season_difficulty,
        technology_level_difference = context_factors.tech_diff * 0.12
    }
    
    local final_difficulty = base_difficulty
    for modifier_name, value in pairs(difficulty_modifiers) do
        final_difficulty = final_difficulty + value
    end
    
    -- Ensure difficulty stays within reasonable bounds
    return math.max(10, math.min(95, final_difficulty))
end
```

#### Success Rate Calculations

**Probability Formulas:**
```lua
-- Master success calculation function
function calculate_operation_success_probability(agent, operation, target_context)
    local base_success = operation.base_success_rate
    
    -- Agent skill contributions
    local skill_bonus = (
        agent.subterfuge * operation.subterfuge_weight +
        agent.command * operation.command_weight +
        agent.specialization_bonus * operation.specialization_weight
    ) / 3
    
    -- Environmental factors
    local environmental_modifiers = {
        target_security = -target_context.security_level * 2,
        weather_conditions = get_weather_modifier(target_context.region, cm:turn_number()),
        political_climate = get_political_stability_modifier(target_context),
        agent_fatigue = -agent.operation_count_recent * 5,
        equipment_quality = agent.equipment_bonus or 0
    }
    
    -- Calculate final probability
    local total_modifiers = skill_bonus
    for modifier_name, value in pairs(environmental_modifiers) do
        total_modifiers = total_modifiers + value
    end
    
    local final_probability = math.max(0.05, math.min(0.95, 
        (base_success + total_modifiers) / 100
    ))
    
    return final_probability
end
```

### Faction-Specific Balance

Different factions have varying strengths and weaknesses in espionage:

```txt
# Faction Intelligence Profiles
faction_intelligence_profiles

    faction venice
        specialization trade_intelligence, maritime_operations
        base_bonuses court_infiltration +15%, market_manipulation +20%
        penalties military_intelligence -10%
        unique_abilities merchant_spy_networks, diplomatic_immunity
        training_cost_modifier 0.9

    faction england  
        specialization military_intelligence, battlefield_analysis
        base_bonuses army_infiltration +20%, officer_targeting +15%
        penalties court_intrigue -15%
        unique_abilities longbow_assassins, castle_intelligence_networks
        training_cost_modifier 1.1

    faction france
        specialization court_intrigue, political_manipulation
        base_bonuses diplomatic_espionage +25%, succession_influence +20%
        penalties economic_sabotage -10%
        unique_abilities royal_court_access, noble_connections
        training_cost_modifier 1.2
```

### Anti-Exploitation Measures

#### Operation Cooldowns and Limitations
```lua
-- Prevent operation spamming
function enforce_operation_limitations(agent, operation_type)
    local limitations = {
        assassination_attempts = {
            cooldown_turns = 4,
            max_concurrent = 1,
            failure_penalty_turns = 8
        },
        
        major_sabotage = {
            cooldown_turns = 6,
            max_concurrent = 2,
            failure_penalty_turns = 12
        },
        
        intelligence_operations = {
            cooldown_turns = 2,
            max_concurrent = 3,
            failure_penalty_turns = 4
        }
    }
    
    return validate_operation_constraints(agent, operation_type, limitations[operation_type])
end
```

#### Escalation Penalties
```lua
-- Escalating consequences for repeated operations
function calculate_escalation_penalties(faction, target_faction, operation_history)
    local escalation_level = count_recent_operations(faction, target_faction, 20) -- Last 20 turns
    
    local penalties = {
        diplomatic_relations = -escalation_level * 2,
        operation_difficulty_increase = escalation_level * 5,
        counter_intelligence_attention = escalation_level * 10,
        agent_detection_risk_increase = escalation_level * 3
    }
    
    return penalties
end
```

---

## Development Configuration Files

### File Structure
```
/data/
├── enhanced_spy_systems/
│   ├── core/
│   │   ├── mod_military_intelligence.lua
│   │   ├── mod_sabotage_operations.lua
│   │   ├── mod_intelligence_markets.lua
│   │   ├── mod_agent_specialization.lua
│   │   └── mod_officer_targeting.lua
│   ├── config/
│   │   ├── sabotage_targets.txt
│   │   ├── training_schools.txt
│   │   ├── intelligence_hubs.txt
│   │   ├── military_hierarchy.txt
│   │   └── faction_intelligence_profiles.txt
│   ├── ui/
│   │   ├── intelligence_market_interface.xml
│   │   ├── operation_planning_panel.xml
│   │   ├── agent_specialization_ui.xml
│   │   └── counter_intelligence_display.xml
│   └── database/
│       └── enhanced_spy_systems_schema.sql
├── descr_character.txt (extended)
├── export_descr_buildings.txt (extended)
└── scripts/
    └── campaign/
        └── enhanced_spy_init.lua
```

This comprehensive implementation document provides a complete technical specification for implementing the five enhanced spy and assassin systems in Medieval III. The systems are designed to work together seamlessly while maintaining game balance and providing deep strategic gameplay options.

Each system includes detailed technical implementation, database requirements, UI specifications, and balancing considerations. The modular design allows for phased implementation while ensuring all systems integrate properly with the existing Medieval II Total War engine and M2TWEOP framework.