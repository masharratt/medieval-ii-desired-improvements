# Medieval III Enhanced Merchant Systems Implementation Specification

## Overview

This document provides comprehensive technical implementation details for the expanded merchant systems in Medieval III mod. These systems transform merchants from passive income generators into active economic empire builders, integrating seamlessly with M2TWEOP's framework while providing rich strategic gameplay.

## Selected Systems for Implementation

1. **Craft Master Recruitment & Artisan Management (#7)**
2. **Port City Development & Maritime Infrastructure (#9)**
3. **Quality Branding & Luxury Market Creation (#14)**
4. **Expanded Merchant Apprenticeship & Skill Development (#20)**

---

## 1. Craft Master Recruitment & Artisan Management

### Technical Implementation

#### Database Schema Extensions
```sql
-- Master craftsman tracking
CREATE TABLE master_craftsmen (
    master_id INTEGER PRIMARY KEY,
    name VARCHAR(100),
    specialty VARCHAR(50), -- weaponsmith, architect, shipwright, etc.
    skill_level INTEGER, -- 1-10 scale
    personality_type VARCHAR(50), -- perfectionist, innovative, temperamental, etc.
    current_employer_faction INTEGER,
    contract_end_turn INTEGER,
    age INTEGER,
    reputation_score INTEGER,
    exclusive_techniques TEXT,
    workshop_requirements TEXT,
    annual_salary INTEGER,
    loyalty_level INTEGER
);

-- Craftsman contracts and employment
CREATE TABLE craftsman_contracts (
    contract_id INTEGER PRIMARY KEY,
    master_id INTEGER,
    merchant_id INTEGER,
    signing_bonus INTEGER,
    annual_salary INTEGER,
    contract_length INTEGER,
    start_turn INTEGER,
    end_turn INTEGER,
    exclusivity_clause BOOLEAN,
    workshop_provided BOOLEAN,
    status VARCHAR(20) -- active, completed, breached
);

-- Workshop and production facilities
CREATE TABLE merchant_workshops (
    workshop_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    settlement_id INTEGER,
    workshop_type VARCHAR(50),
    facility_level INTEGER, -- 1-5 development level
    construction_cost INTEGER,
    maintenance_cost INTEGER,
    assigned_master INTEGER,
    production_bonus INTEGER,
    capacity INTEGER,
    last_upgrade_turn INTEGER
);

-- Branded products and quality tracking
CREATE TABLE branded_products (
    product_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    product_name VARCHAR(100),
    base_resource VARCHAR(50),
    quality_level INTEGER,
    brand_recognition INTEGER, -- 0-100 scale
    price_premium INTEGER, -- percentage above base price
    production_cost_modifier INTEGER,
    market_demand INTEGER,
    counterfeit_risk INTEGER
);
```

#### M2TWEOP Character Extensions
**File: `data/export_descr_character_traits.txt`**
```
;--- Master Craftsman Traits ---
Trait MasterCraftsman
    Characters merchant
    
    Level MasterWeaponsmith
        Description MasterWeaponsmith_desc
        EffectsDescription MasterWeaponsmith_effects_desc
        Threshold 1
        Effect TradeIncome 2
        Effect LocalPopulationLoyalty 1
        
    Level MasterArchitect  
        Description MasterArchitect_desc
        EffectsDescription MasterArchitect_effects_desc
        Threshold 1
        Effect Construction 1
        Effect TradeIncome 3
        
    Level MasterShipwright
        Description MasterShipwright_desc
        EffectsDescription MasterShipwright_effects_desc
        Threshold 1
        Effect NavalCommand 1
        Effect TradeIncome 2

Trait CraftworkQuality
    Characters merchant
    
    Level StandardQuality
        Description StandardQuality_desc
        Threshold 1
        
    Level SuperiorQuality
        Description SuperiorQuality_desc
        Threshold 1
        Effect TradeIncome 3
        
    Level MasterworkQuality
        Description MasterworkQuality_desc
        Threshold 1
        Effect TradeIncome 5
        Effect Influence 1
```

#### Lua Scripting Framework
**File: `data/scripts/campaign/merchant_artisans.lua`**
```lua
-- Master craftsman recruitment system
function AttemptMasterRecruitment(merchant_character, master_id, offer_package)
    local master = GetMasterCraftsman(master_id)
    local recruitment_chance = CalculateRecruitmentChance(merchant_character, master, offer_package)
    
    -- Base success factors
    local base_chance = 30
    local salary_modifier = (offer_package.annual_salary - master.expected_salary) / master.expected_salary * 20
    local workshop_modifier = offer_package.workshop_provided and 15 or -10
    local reputation_modifier = merchant_character.reputation / 10
    local competition_penalty = GetCompetingOffers(master_id) * -5
    
    local final_chance = math.min(base_chance + salary_modifier + workshop_modifier + reputation_modifier + competition_penalty, 85)
    
    if final_chance > math.random(1, 100) then
        CreateCraftsmanContract(merchant_character, master, offer_package)
        TriggerEvent("master_craftsman_recruited", merchant_character.faction, master.name)
        return true
    else
        return false
    end
end

-- Workshop production and quality management
function ProcessWorkshopProduction(workshop_id)
    local workshop = GetWorkshop(workshop_id)
    local assigned_master = GetAssignedMaster(workshop)
    
    if assigned_master then
        local base_production = workshop.capacity
        local quality_modifier = assigned_master.skill_level / 10
        local personality_effects = ApplyPersonalityEffects(assigned_master)
        
        -- Production calculation
        local production_quantity = base_production * personality_effects.speed_modifier
        local quality_level = math.min(assigned_master.skill_level + personality_effects.quality_bonus, 10)
        
        -- Create branded products if quality threshold met
        if quality_level >= 7 then
            CreateBrandedProduct(workshop.merchant_id, workshop.workshop_type, quality_level)
        end
        
        return {
            quantity = production_quantity,
            quality = quality_level,
            value_modifier = quality_modifier * 1.5
        }
    else
        return {
            quantity = workshop.capacity * 0.6, -- Reduced without master
            quality = 3, -- Standard quality
            value_modifier = 1.0
        }
    end
end

-- Master craftsman aging and succession
function ProcessMasterAging(master_id)
    local master = GetMasterCraftsman(master_id)
    master.age = master.age + 1
    
    -- Peak years (30-45), decline after 50
    if master.age > 50 then
        local decline_rate = (master.age - 50) * 0.1
        master.skill_level = math.max(master.skill_level - decline_rate, 3)
        
        -- Succession opportunities
        if master.age > 60 and math.random(1, 100) < 15 then
            TriggerSuccessionEvent(master_id)
        end
    end
    
    -- Innovation chances for innovative masters
    if master.personality_type == "innovative" and math.random(1, 100) < 8 then
        DevelopNewTechnique(master_id)
    end
end

-- Talent poaching and contract competition
function AttemptTalentPoaching(poaching_merchant, target_master, poaching_offer)
    local current_contract = GetCurrentContract(target_master)
    local current_employer = GetMerchant(current_contract.merchant_id)
    
    -- Calculate poaching success chance
    local offer_improvement = (poaching_offer.total_package - current_contract.total_value) / current_contract.total_value
    local loyalty_resistance = target_master.loyalty_level * 0.1
    local reputation_factor = (poaching_merchant.reputation - current_employer.reputation) * 0.05
    
    local poaching_chance = math.min(offer_improvement * 50 - loyalty_resistance + reputation_factor, 75)
    
    if poaching_chance > math.random(1, 100) then
        -- Trigger counter-offer opportunity for current employer
        TriggerCounterOfferEvent(current_employer, target_master, poaching_offer)
        return true
    else
        -- Increase master's loyalty to current employer
        ModifyMasterLoyalty(target_master, 5)
        return false
    end
end
```

#### UI Implementation
**Master Craftsman Recruitment Interface**
- **Available Masters Panel**: List masters by specialty, skill level, and availability
- **Recruitment Negotiation Dialog**: Salary slider, workshop requirements, contract terms
- **Workshop Management Screen**: Assign masters, monitor production, upgrade facilities
- **Master Biography Panel**: Skills, personality, work history, and current projects

---

## 2. Port City Development & Maritime Infrastructure

### Technical Implementation

#### Database Schema Extensions
```sql
-- Port infrastructure development
CREATE TABLE port_infrastructure (
    infrastructure_id INTEGER PRIMARY KEY,
    settlement_id INTEGER,
    infrastructure_type VARCHAR(50), -- harbor, warehouse, lighthouse, shipyard
    development_level INTEGER, -- 1-5 upgrade levels
    owner_merchant_id INTEGER,
    construction_cost INTEGER,
    maintenance_cost INTEGER,
    revenue_generation INTEGER,
    capacity INTEGER,
    construction_start_turn INTEGER,
    completion_turn INTEGER,
    status VARCHAR(20) -- planning, construction, operational, damaged
);

-- Waterfront property ownership
CREATE TABLE waterfront_properties (
    property_id INTEGER PRIMARY KEY,
    settlement_id INTEGER,
    property_type VARCHAR(50), -- warehouse, wharf, office, shop
    owner_merchant_id INTEGER,
    purchase_cost INTEGER,
    annual_rent INTEGER,
    size_rating INTEGER,
    location_quality INTEGER, -- proximity to deep water, main trade routes
    current_usage VARCHAR(50),
    rental_income INTEGER
);

-- Port traffic and commerce tracking
CREATE TABLE port_activity (
    activity_id INTEGER PRIMARY KEY,
    settlement_id INTEGER,
    turn_number INTEGER,
    ships_docked INTEGER,
    cargo_volume INTEGER,
    trade_value INTEGER,
    major_routes_active INTEGER,
    weather_delays INTEGER,
    infrastructure_usage INTEGER
);

-- Maritime business relationships
CREATE TABLE maritime_contracts (
    contract_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    client_type VARCHAR(50), -- ship_captain, trading_company, naval_fleet
    service_type VARCHAR(50), -- docking, repair, storage, provisioning
    contract_value INTEGER,
    duration_turns INTEGER,
    start_turn INTEGER,
    renewal_option BOOLEAN,
    exclusivity_clause BOOLEAN
);
```

#### Building System Extensions
**File: `data/export_descr_buildings.txt`**
```
;--- Maritime Infrastructure Buildings ---
building merchant_harbor
{
    levels harbor_basic harbor_improved harbor_advanced harbor_master harbor_grand
    
    harbor_basic
    {
        capability
        {
            trade_fleet_capacity 2
            ship_repair_rate 0.1
            storage_capacity 5000
        }
        construction 3
        cost 2000
        settlement_min town
        upgrades
        {
            harbor_improved
        }
    }
    
    harbor_improved  
    {
        capability
        {
            trade_fleet_capacity 4
            ship_repair_rate 0.15
            storage_capacity 8000
            lighthouse_effect 0.25
        }
        construction 4
        cost 4000
        settlement_min town
        upgrades
        {
            harbor_advanced
        }
    }
    
    harbor_advanced
    {
        capability
        {
            trade_fleet_capacity 6
            ship_repair_rate 0.2  
            storage_capacity 12000
            lighthouse_effect 0.4
            shipyard_access true
        }
        construction 6
        cost 8000
        settlement_min large_town
        upgrades
        {
            harbor_master
        }
    }
}

building merchant_warehouse_complex
{
    levels warehouse_basic warehouse_advanced warehouse_master
    
    warehouse_basic
    {
        capability
        {
            storage_capacity 3000
            spoilage_reduction 0.15
            seasonal_storage true
        }
        construction 2
        cost 1500
        settlement_min town
    }
    
    warehouse_advanced
    {
        capability
        {
            storage_capacity 6000
            spoilage_reduction 0.25
            seasonal_storage true
            bulk_processing true
        }
        construction 3
        cost 3000
        settlement_min town
    }
}
```

#### Port Development Lua Framework
**File: `data/scripts/campaign/port_development.lua`**
```lua
-- Port infrastructure development system
function DevelopPortInfrastructure(merchant_id, settlement_id, infrastructure_type, investment_level)
    local settlement = GetSettlement(settlement_id)
    local merchant = GetMerchant(merchant_id)
    
    -- Calculate development costs and timeline
    local base_cost = GetInfrastructureCost(infrastructure_type, investment_level)
    local location_modifier = GetLocationCostModifier(settlement)
    local competition_modifier = GetCompetitionModifier(settlement_id)
    
    local final_cost = base_cost * location_modifier * competition_modifier
    local construction_time = CalculateConstructionTime(infrastructure_type, investment_level)
    
    if merchant.treasury >= final_cost then
        CreateInfrastructureProject(merchant_id, settlement_id, infrastructure_type, final_cost, construction_time)
        DeductMerchantTreasury(merchant_id, final_cost)
        
        -- Notify other merchants of new competition
        NotifyLocalMerchants(settlement_id, "new_infrastructure_development", merchant_id)
        
        return true
    else
        return false, "insufficient_funds"
    end
end

-- Waterfront property acquisition
function BidOnWaterfrontProperty(merchant_id, property_id, bid_amount)
    local property = GetWaterfrontProperty(property_id)
    local competing_bids = GetCompetingBids(property_id)
    
    -- Auction mechanics
    if bid_amount > property.current_high_bid then
        property.current_high_bid = bid_amount
        property.high_bidder = merchant_id
        
        -- Notify competing merchants
        for _, competing_merchant in pairs(competing_bids) do
            TriggerEvent("outbid_notification", competing_merchant, property_id)
        end
        
        return true
    else
        return false, "bid_too_low"
    end
end

-- Port revenue generation and management
function ProcessPortRevenue(merchant_id)
    local merchant_infrastructure = GetMerchantInfrastructure(merchant_id)
    local total_revenue = 0
    
    for _, infrastructure in pairs(merchant_infrastructure) do
        local base_revenue = infrastructure.revenue_generation
        local usage_rate = CalculateUsageRate(infrastructure)
        local maintenance_cost = infrastructure.maintenance_cost
        
        -- Revenue modifiers
        local weather_modifier = GetWeatherModifier(infrastructure.settlement_id)
        local competition_modifier = GetPortCompetitionModifier(infrastructure.settlement_id)
        local quality_modifier = infrastructure.development_level * 0.1
        
        local period_revenue = base_revenue * usage_rate * weather_modifier * competition_modifier * (1 + quality_modifier) - maintenance_cost
        
        total_revenue = total_revenue + math.max(period_revenue, 0)
        
        -- Infrastructure degradation
        ProcessInfrastructureMaintenance(infrastructure)
    end
    
    return total_revenue
end

-- Maritime contract management
function NegotiateMaritimeContract(merchant_id, client_id, service_type, terms)
    local merchant = GetMerchant(merchant_id)
    local client = GetMaritimeClient(client_id)
    
    -- Contract negotiation factors
    local service_availability = CheckServiceAvailability(merchant_id, service_type)
    local reputation_modifier = merchant.reputation / 100
    local competition_factor = GetServiceCompetition(merchant.settlement_id, service_type)
    local client_urgency = GetClientUrgency(client_id)
    
    local negotiation_success = CalculateNegotiationSuccess(reputation_modifier, competition_factor, client_urgency, terms)
    
    if negotiation_success > 70 then
        CreateMaritimeContract(merchant_id, client_id, service_type, terms)
        return true, "contract_accepted"
    elseif negotiation_success > 40 then
        return false, "counter_offer_required"
    else
        return false, "contract_rejected"
    end
end
```

---

## 3. Quality Branding & Luxury Market Creation

### Technical Implementation

#### Database Schema Extensions
```sql
-- Brand development and management
CREATE TABLE merchant_brands (
    brand_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    brand_name VARCHAR(100),
    product_category VARCHAR(50),
    established_turn INTEGER,
    recognition_level INTEGER, -- 0-100 regional recognition
    quality_consistency INTEGER, -- track quality maintenance
    market_presence INTEGER, -- geographic spread
    premium_percentage INTEGER, -- price premium over base products
    brand_reputation INTEGER,
    counterfeiting_attempts INTEGER,
    authentication_methods TEXT
);

-- Product quality tracking and certification
CREATE TABLE product_quality_records (
    record_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    brand_id INTEGER,
    product_batch VARCHAR(50),
    quality_score INTEGER,
    production_turn INTEGER,
    inspector_id INTEGER,
    customer_feedback INTEGER,
    defect_rate REAL,
    market_response VARCHAR(50)
);

-- Luxury market segments
CREATE TABLE luxury_markets (
    market_id INTEGER PRIMARY KEY,
    settlement_id INTEGER,
    market_segment VARCHAR(50), -- noble_court, military_elite, wealthy_merchants
    market_size INTEGER,
    purchasing_power INTEGER,
    preferences TEXT,
    seasonal_demand_modifier REAL,
    competition_level INTEGER,
    entry_barriers INTEGER
);

-- Brand protection and enforcement
CREATE TABLE brand_protection (
    protection_id INTEGER PRIMARY KEY,
    brand_id INTEGER,
    protection_type VARCHAR(50), -- legal, physical, technological
    investment_cost INTEGER,
    effectiveness_rating INTEGER,
    coverage_area INTEGER,
    maintenance_cost INTEGER,
    last_enforcement_turn INTEGER
);
```

#### Quality Control System
**File: `data/scripts/campaign/quality_branding.lua`**
```lua
-- Brand establishment and development
function EstablishProductBrand(merchant_id, product_category, brand_name, initial_investment)
    local merchant = GetMerchant(merchant_id)
    
    -- Validate brand creation requirements
    if merchant.treasury < initial_investment then
        return false, "insufficient_funds"
    end
    
    if GetMerchantProductQuality(merchant_id, product_category) < 7 then
        return false, "insufficient_quality_history"
    end
    
    -- Create new brand
    local brand = {
        merchant_id = merchant_id,
        brand_name = brand_name,
        product_category = product_category,
        recognition_level = 15, -- Starting recognition
        quality_consistency = 85, -- Based on past performance
        market_presence = 1, -- Local market only initially
        premium_percentage = 10 -- Starting premium
    }
    
    CreateBrand(brand)
    DeductMerchantTreasury(merchant_id, initial_investment)
    
    return true, brand.brand_id
end

-- Quality consistency monitoring
function ProcessQualityControl(merchant_id, production_batch)
    local merchant = GetMerchant(merchant_id)
    local quality_systems = GetQualityControlSystems(merchant_id)
    
    -- Base quality from production
    local base_quality = CalculateProductionQuality(production_batch)
    
    -- Quality control modifiers
    local inspector_bonus = quality_systems.master_inspector and 1.2 or 1.0
    local material_bonus = quality_systems.premium_materials and 1.15 or 1.0
    local process_bonus = quality_systems.monitoring_systems and 1.1 or 1.0
    
    local final_quality = base_quality * inspector_bonus * material_bonus * process_bonus
    
    -- Record quality for brand tracking
    RecordProductQuality(merchant_id, production_batch, final_quality)
    
    -- Update brand consistency scores
    UpdateBrandConsistency(merchant_id, final_quality)
    
    return final_quality
end

-- Brand recognition and market penetration
function ProcessBrandRecognition(brand_id)
    local brand = GetBrand(brand_id)
    local merchant = GetMerchant(brand.merchant_id)
    
    -- Factors affecting brand recognition
    local quality_factor = brand.quality_consistency / 100
    local market_presence_factor = brand.market_presence * 0.1
    local marketing_investment = GetMarketingInvestment(brand_id) / 1000
    local word_of_mouth = CalculateWordOfMouth(brand_id)
    
    -- Recognition growth calculation
    local recognition_growth = (quality_factor + market_presence_factor + marketing_investment + word_of_mouth) * 2
    
    brand.recognition_level = math.min(brand.recognition_level + recognition_growth, 100)
    
    -- Premium pricing ability increases with recognition
    if brand.recognition_level > 50 then
        brand.premium_percentage = math.min(brand.premium_percentage + 2, 75)
    end
    
    -- Market expansion opportunities
    if brand.recognition_level > 70 and brand.market_presence < 5 then
        TriggerMarketExpansionOpportunity(brand_id)
    end
end

-- Luxury market penetration
function PenetrateLuxuryMarket(merchant_id, target_market_segment, marketing_strategy)
    local merchant = GetMerchant(merchant_id)
    local market = GetLuxuryMarket(target_market_segment)
    
    -- Market entry requirements
    local quality_threshold = GetMarketQualityThreshold(target_market_segment)
    local merchant_reputation = merchant.reputation
    local brand_recognition = GetHighestBrandRecognition(merchant_id)
    
    if GetMerchantAverageQuality(merchant_id) < quality_threshold then
        return false, "quality_insufficient"
    end
    
    if brand_recognition < market.entry_barriers then
        return false, "insufficient_brand_recognition"
    end
    
    -- Market penetration attempt
    local penetration_chance = CalculateMarketPenetrationChance(merchant, market, marketing_strategy)
    
    if penetration_chance > math.random(1, 100) then
        EstablishLuxuryMarketPresence(merchant_id, target_market_segment)
        return true, "market_entry_successful"
    else
        return false, "market_entry_failed"
    end
end

-- Anti-counterfeiting and brand protection
function ProcessBrandProtection(brand_id)
    local brand = GetBrand(brand_id)
    local protection_systems = GetBrandProtection(brand_id)
    
    -- Counterfeiting threat assessment
    local base_threat = brand.premium_percentage * 0.5 -- Higher premiums attract counterfeiters
    local market_size_factor = brand.market_presence * 0.3
    local protection_effectiveness = CalculateProtectionEffectiveness(protection_systems)
    
    local counterfeiting_risk = math.max(base_threat + market_size_factor - protection_effectiveness, 0)
    
    -- Process counterfeiting attempts
    if counterfeiting_risk > math.random(1, 100) then
        local counterfeiting_damage = ProcessCounterfeitingAttempt(brand_id)
        
        -- Damage brand reputation and recognition
        brand.recognition_level = math.max(brand.recognition_level - counterfeiting_damage, 0)
        brand.premium_percentage = math.max(brand.premium_percentage - (counterfeiting_damage / 2), 0)
        
        -- Trigger enforcement opportunities
        TriggerBrandEnforcementEvent(brand_id)
    end
end
```

---

## 4. Expanded Merchant Apprenticeship & Skill Development

### Technical Implementation

#### Enhanced Character System
**File: `data/export_descr_character_traits.txt`**
```
;--- Enhanced Merchant Skills ---
Trait MerchantMastery
    Characters merchant
    
    Level Apprentice
        Description ApprenticeTrader_desc
        EffectsDescription ApprenticeTrader_effects_desc
        Threshold 1
        Effect Finance 1
        Effect TradeIncome 1
        
    Level Journeyman
        Description JourneymanTrader_desc
        EffectsDescription JourneymanTrader_effects_desc  
        Threshold 1
        Effect Finance 2
        Effect TradeIncome 2
        Effect Influence 1
        
    Level Master
        Description MasterTrader_desc
        EffectsDescription MasterTrader_effects_desc
        Threshold 1
        Effect Finance 3
        Effect TradeIncome 3
        Effect Influence 2
        Effect LocalPopulationLoyalty 1
        
    Level Grandmaster
        Description GrandmasterTrader_desc
        EffectsDescription GrandmasterTrader_effects_desc
        Threshold 1
        Effect Finance 5
        Effect TradeIncome 5
        Effect Influence 3
        Effect LocalPopulationLoyalty 2

Trait SpecializationMastery
    Characters merchant
    
    Level ResourceDevelopment
        Description ResourceDevelopment_desc
        Threshold 1
        Effect Mining 2
        Effect Agriculture 1
        Effect TradeIncome 1
        
    Level NetworkBuilding
        Description NetworkBuilding_desc
        Threshold 1  
        Effect Influence 2
        Effect PublicSecurity 1
        Effect TradeIncome 1
        
    Level MaritimeTrade
        Description MaritimeTrade_desc
        Threshold 1
        Effect NavalCommand 1
        Effect TradeIncome 2
        Effect LocalPopulationLoyalty 1

Trait RegionalExpertise
    Characters merchant
    
    Level ItalianCities
        Description ItalianCities_desc
        Threshold 1
        Effect Influence 2
        Effect TradeIncome 2
        
    Level HanseaticLeague
        Description HanseaticLeague_desc
        Threshold 1
        Effect NavalCommand 1
        Effect TradeIncome 2
        
    Level IslamicTerritories
        Description IslamicTerritories_desc
        Threshold 1
        Effect Finance 2
        Effect TradeIncome 1
```

#### Skill Development Database
```sql
-- Enhanced merchant skill system
CREATE TABLE merchant_skills (
    skill_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    skill_category VARCHAR(50), -- trade_mastery, resource_development, network_building
    skill_name VARCHAR(50),
    current_level INTEGER,
    experience_points INTEGER,
    training_investment INTEGER,
    last_training_turn INTEGER,
    specialization_bonuses TEXT,
    mastery_achievements TEXT
);

-- Training programs and mentorship
CREATE TABLE merchant_training (
    training_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    training_type VARCHAR(50),
    mentor_id INTEGER,
    training_location INTEGER, -- settlement_id for specialized schools
    start_turn INTEGER,
    duration_turns INTEGER,
    cost INTEGER,
    completion_bonus TEXT,
    status VARCHAR(20) -- enrolled, active, completed, failed
);

-- Guild advancement and leadership
CREATE TABLE guild_positions (
    position_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    guild_type VARCHAR(50),
    position_rank INTEGER, -- 1=member, 5=grandmaster
    appointment_turn INTEGER,
    responsibilities TEXT,
    authority_level INTEGER,
    annual_stipend INTEGER,
    political_influence INTEGER
);

-- Achievement and recognition system
CREATE TABLE merchant_achievements (
    achievement_id INTEGER PRIMARY KEY,
    merchant_id INTEGER,
    achievement_type VARCHAR(50),
    achievement_name VARCHAR(100),
    earned_turn INTEGER,
    achievement_bonus TEXT,
    reputation_gain INTEGER,
    prerequisite_met BOOLEAN
);
```

#### Skill Development Lua Framework
**File: `data/scripts/campaign/merchant_skills.lua`**
```lua
-- Advanced skill development system
function ProcessSkillDevelopment(merchant_id)
    local merchant = GetMerchant(merchant_id)
    local skills = GetMerchantSkills(merchant_id)
    
    for _, skill in pairs(skills) do
        -- Experience gain from successful activities
        local activity_experience = CalculateActivityExperience(merchant_id, skill.skill_category)
        skill.experience_points = skill.experience_points + activity_experience
        
        -- Check for level advancement
        local level_threshold = GetLevelThreshold(skill.current_level)
        if skill.experience_points >= level_threshold then
            AdvanceSkillLevel(merchant_id, skill.skill_name)
            TriggerSkillAdvancementEvent(merchant_id, skill.skill_name, skill.current_level + 1)
        end
        
        -- Apply skill bonuses to merchant activities
        ApplySkillBonuses(merchant_id, skill)
    end
end

-- Specialized training programs
function EnrollInTrainingProgram(merchant_id, training_type, location_id)
    local merchant = GetMerchant(merchant_id)
    local training_program = GetTrainingProgram(training_type, location_id)
    
    -- Check eligibility requirements
    if merchant.skill_level < training_program.minimum_level then
        return false, "insufficient_skill_level"
    end
    
    if merchant.treasury < training_program.cost then
        return false, "insufficient_funds"
    end
    
    -- Check availability and competition
    local available_slots = training_program.capacity - GetCurrentEnrollment(training_program.id)
    if available_slots <= 0 then
        return false, "program_full"
    end
    
    -- Enroll merchant
    CreateTrainingEnrollment(merchant_id, training_program)
    DeductMerchantTreasury(merchant_id, training_program.cost)
    
    return true, training_program.id
end

-- Master merchant advancement and leadership
function ProcessGuildAdvancement(merchant_id)
    local merchant = GetMerchant(merchant_id)
    local guild_memberships = GetGuildMemberships(merchant_id)
    
    for _, membership in pairs(guild_memberships) do
        local advancement_eligibility = CheckAdvancementEligibility(merchant_id, membership.guild_type)
        
        if advancement_eligibility.eligible then
            local advancement_chance = CalculateAdvancementChance(merchant, membership)
            
            if advancement_chance > math.random(1, 100) then
                AdvanceGuildPosition(merchant_id, membership.guild_type)
                GrantPositionBenefits(merchant_id, membership.guild_type, membership.position_rank + 1)
                
                -- Grandmaster privileges
                if membership.position_rank + 1 >= 5 then
                    GrantGrandmasterPrivileges(merchant_id, membership.guild_type)
                end
            end
        end
    end
end

-- Regional specialization development
function DevelopRegionalExpertise(merchant_id, target_region, specialization_type)
    local merchant = GetMerchant(merchant_id)
    local current_expertise = GetRegionalExpertise(merchant_id)
    
    -- Requirements for specialization
    local time_requirement = GetTimeInRegion(merchant_id, target_region)
    local success_requirement = GetRegionalSuccessRate(merchant_id, target_region)
    local investment_requirement = GetSpecializationInvestment(merchant_id, specialization_type)
    
    if time_requirement >= 20 and success_requirement >= 0.7 and investment_requirement >= 5000 then
        -- Grant regional specialization
        GrantRegionalExpertise(merchant_id, target_region, specialization_type)
        
        -- Apply specialization bonuses
        local bonuses = GetSpecializationBonuses(target_region, specialization_type)
        ApplyPermanentBonuses(merchant_id, bonuses)
        
        -- Unlock advanced regional activities
        UnlockRegionalActivities(merchant_id, target_region)
        
        return true
    else
        return false, "requirements_not_met"
    end
end

-- Achievement and recognition system
function ProcessMerchantAchievements(merchant_id)
    local merchant = GetMerchant(merchant_id)
    local potential_achievements = GetPotentialAchievements(merchant_id)
    
    for _, achievement in pairs(potential_achievements) do
        if CheckAchievementCriteria(merchant_id, achievement) then
            GrantAchievement(merchant_id, achievement)
            
            -- Achievement bonuses
            ApplyAchievementBonus(merchant_id, achievement.bonus_type)
            
            -- Reputation and recognition
            ModifyMerchantReputation(merchant_id, achievement.reputation_gain)
            
            -- Special unlocks
            if achievement.unlocks then
                UnlockSpecialAbilities(merchant_id, achievement.unlocks)
            end
            
            TriggerAchievementEvent(merchant_id, achievement.name)
        end
    end
end
```

---

## 5. Integration Systems

### Cross-System Data Flow

#### Merchant Portfolio Management
```lua
-- Comprehensive merchant empire management
function ProcessMerchantEmpire(merchant_id)
    local merchant = GetMerchant(merchant_id)
    
    -- Process all merchant activities
    local workshop_revenue = ProcessWorkshopProduction(merchant_id)
    local port_revenue = ProcessPortRevenue(merchant_id)
    local brand_income = ProcessBrandedProductSales(merchant_id)
    local training_costs = ProcessTrainingInvestments(merchant_id)
    
    -- Calculate total empire performance
    local total_revenue = workshop_revenue + port_revenue + brand_income - training_costs
    
    -- Update merchant progression
    ProcessSkillDevelopment(merchant_id)
    ProcessGuildAdvancement(merchant_id)
    ProcessMerchantAchievements(merchant_id)
    
    -- Update empire-wide effects
    UpdateRegionalEconomicImpact(merchant_id)
    UpdateFactionEconomicContribution(merchant.faction_id)
    
    return total_revenue
end
```

---

## 6. UI/UX Implementation

### Enhanced Merchant Interface

#### Merchant Empire Dashboard
- **Portfolio Overview**: Revenue streams, investments, skill progression
- **Master Craftsman Panel**: Employed artisans, contracts, production quality
- **Infrastructure Map**: Port developments, warehouse locations, trade routes
- **Brand Management**: Brand recognition, quality metrics, market presence
- **Training Center**: Available programs, skill development, guild advancement

#### Specialized Management Screens
- **Workshop Management**: Assign masters, monitor quality, upgrade facilities  
- **Port Operations**: Infrastructure development, contract management, revenue tracking
- **Brand Portfolio**: Quality control, market expansion, anti-counterfeiting
- **Skill Development**: Training enrollment, mentor relationships, achievement progress

---

## 7. Development Timeline

### Phase 1: Foundation Systems (Months 1-3)
- Enhanced character traits and skill system
- Basic database schema implementation
- Core UI framework development
- Workshop and artisan management basics

### Phase 2: Advanced Features (Months 4-6)  
- Port development and maritime infrastructure
- Brand management and quality control systems
- Training programs and guild advancement
- Cross-system integration

### Phase 3: Refinement & Balancing (Months 7-8)
- Gameplay balance testing and adjustment
- UI/UX optimization and polish
- AI behavior integration and testing
- Performance optimization

---

## 8. Balancing Considerations

### Economic Balance
- Investment costs should provide clear but not overwhelming advantages
- Multiple viable progression paths (specialization vs. diversification)
- Risk/reward balance encourages active management
- Long-term investments should outperform short-term speculation

### Gameplay Balance  
- Meaningful choices between different development strategies
- Competition mechanics prevent single-player domination
- Failure recovery options maintain player engagement
- Success should feel earned rather than automatic

### Technical Balance
- Database queries optimized for performance
- Memory usage controlled through efficient data structures
- Save/load compatibility maintained throughout development
- Modular design allows selective feature implementation

This implementation specification provides a comprehensive technical framework for transforming Medieval III merchants into sophisticated economic empire builders while maintaining game balance and medieval authenticity.