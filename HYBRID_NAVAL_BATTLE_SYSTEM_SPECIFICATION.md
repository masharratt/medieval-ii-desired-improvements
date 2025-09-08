# Hybrid Naval Battle System - Technical Implementation Specification
## Cross-Game Integration: Medieval III ↔ Empire Total War

---

## System Architecture Overview

### Core Integration Framework
```
┌─────────────────────────────────────────────────────────────────┐
│                Medieval II Campaign Layer                       │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   Economy   │    │ Diplomacy   │    │   Naval     │         │
│  │   System    │    │   System    │    │   Units     │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                M2TWEOP Bridge Layer                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │Battle Data  │    │Translation  │    │Process      │         │  
│  │Extraction   │    │Engine       │    │Management   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                Empire TW Battle Engine                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │3D Naval     │    │Advanced     │    │Battle       │         │
│  │Combat       │    │Physics      │    │Resolution   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

## 1. Battle Detection and Interception

### M2TWEOP Lua Hook Integration
```lua
-- Naval Battle Detection System
function onNavalBattleTriggered(battleContext)
    local battleImportance = calculateBattleImportance(battleContext)
    local playerInvolvement = checkPlayerInvolvement(battleContext)
    local systemAvailability = verifyEmpireTWAvailable()
    
    if battleImportance >= HYBRID_BATTLE_THRESHOLD and 
       playerInvolvement and 
       systemAvailability then
        return presentPlayerChoice(battleContext)
    else
        return proceedWithAutoResolve(battleContext)
    end
end

-- Battle Context Analysis
function calculateBattleImportance(context)
    local importance = 0
    
    -- Fleet size weighting
    importance += (context.attackerFleetSize + context.defenderFleetSize) * 10
    
    -- Cargo value assessment  
    importance += context.totalCargoValue / 1000
    
    -- Strategic location bonus
    if context.nearImportantPort then importance += 50 end
    if context.onMajorTradeRoute then importance += 30 end
    
    -- Admiral involvement
    if context.hasNamedAdmiral then importance += 40 end
    
    return importance
end
```

### Battle Trigger Conditions
```cpp
// C++ Performance-Critical Battle Detection
class NavalBattleDetector {
public:
    struct BattleContext {
        FleetComposition attacker;
        FleetComposition defender; 
        GeographicLocation location;
        WeatherConditions weather;
        EconomicContext cargo;
        PlayerInvolvement player;
    };
    
    bool ShouldTriggerHybridBattle(const BattleContext& context) {
        // Minimum fleet size requirement (prevents minor skirmish interference)
        if (context.attacker.totalShips < 3 && context.defender.totalShips < 3) {
            return false;
        }
        
        // Player must be involved or have significant interest
        if (!context.player.directlyInvolved && 
            !context.player.hasEconomicInterest && 
            !context.player.hasDiplomaticInterest) {
            return false;
        }
        
        // Strategic importance threshold
        return CalculateStrategicValue(context) >= HYBRID_THRESHOLD;
    }
    
private:
    static constexpr int HYBRID_THRESHOLD = 75;
};
```

## 2. Data Translation System

### Medieval II → Empire TW Ship Mapping
```xml
<!-- Ship Translation Configuration -->
<ShipTranslations>
    <!-- Medieval Galleys → Empire Ships -->
    <Translation>
        <MedievalShip>galley</MedievalShip>
        <EmpireEquivalent>5th_rate</EmpireEquivalent>
        <ScalingFactor>0.7</ScalingFactor>
        <SpecialAbilities>ramming_capability</SpecialAbilities>
    </Translation>
    
    <Translation>
        <MedievalShip>war_galley</MedievalShip>
        <EmpireEquivalent>4th_rate</EmpireEquivalent>
        <ScalingFactor>0.8</ScalingFactor>
        <SpecialAbilities>ramming_capability, greek_fire</SpecialAbilities>
    </Translation>
    
    <!-- Medieval Sailing Ships → Empire Ships -->
    <Translation>
        <MedievalShip>cog</MedievalShip>
        <EmpireEquivalent>sloop</EmpireEquivalent>
        <ScalingFactor>0.6</ScalingFactor>
        <CargoCapacity>high</CargoCapacity>
    </Translation>
    
    <Translation>
        <MedievalShip>carrack</MedievalShip>
        <EmpireEquivalent>3rd_rate</EmpireEquivalent>
        <ScalingFactor>0.9</ScalingFactor>
        <SpecialAbilities>exploration_vessel</SpecialAbilities>
    </Translation>
    
    <!-- Specialized Medieval Vessels -->
    <Translation>
        <MedievalShip>dromon</MedievalShip>
        <EmpireEquivalent>frigate</EmpireEquivalent>
        <ScalingFactor>0.75</ScalingFactor>
        <SpecialAbilities>greek_fire, ramming_capability</SpecialAbilities>
        <CulturalBonus>byzantine</CulturalBonus>
    </Translation>
</ShipTranslations>
```

### Cargo and Economic Value Translation
```json
{
  "cargo_translation_system": {
    "trade_goods_mapping": {
      "spices": {
        "empire_equivalent": "luxury_goods",
        "value_multiplier": 2.5,
        "battle_objective": "capture_intact"
      },
      "silk": {
        "empire_equivalent": "luxury_goods", 
        "value_multiplier": 2.0,
        "battle_objective": "capture_intact"
      },
      "grain": {
        "empire_equivalent": "basic_supplies",
        "value_multiplier": 0.8,
        "battle_objective": "deny_enemy"
      },
      "iron": {
        "empire_equivalent": "military_supplies",
        "value_multiplier": 1.5,
        "battle_objective": "capture_priority"
      }
    },
    "cargo_objectives": {
      "merchant_convoy": {
        "primary": "preserve_cargo_ships",
        "secondary": "minimize_cargo_losses", 
        "victory_condition": "cargo_survival_rate > 60%"
      },
      "treasure_fleet": {
        "primary": "defend_flagship",
        "secondary": "escort_survival",
        "victory_condition": "flagship_survives AND cargo_losses < 30%"
      },
      "raiding_attack": {
        "primary": "capture_cargo_vessels",
        "secondary": "minimize_own_losses",
        "victory_condition": "cargo_captured > enemy_ships_lost"
      }
    }
  }
}
```

### Weather and Environment Translation
```cpp
// Weather System Translation
class WeatherTranslator {
public:
    struct EmpireWeatherConfig {
        WindStrength wind_strength;
        WindDirection wind_direction;
        SeaState sea_state;
        Visibility visibility;
        TimeOfDay time;
    };
    
    EmpireWeatherConfig TranslateMedievalWeather(
        MedievalWeatherConditions medieval_weather,
        SeasonalContext season) {
        
        EmpireWeatherConfig config;
        
        switch(medieval_weather.type) {
            case MEDIEVAL_STORM:
                config.wind_strength = STRONG_GALE;
                config.sea_state = VERY_ROUGH;
                config.visibility = POOR;
                // Storms favor smaller, more agile ships
                break;
                
            case MEDIEVAL_CALM:
                config.wind_strength = LIGHT_BREEZE;
                config.sea_state = CALM;
                config.visibility = EXCELLENT;
                // Calm seas favor larger ships with more firepower
                break;
                
            case MEDIEVAL_WINTER:
                config.wind_strength = MODERATE_GALE;
                config.sea_state = ROUGH;
                config.visibility = LIMITED;
                config.time = DAWN; // Shorter days
                break;
        }
        
        return config;
    }
};
```

## 3. External Process Management

### Empire TW Launch and Management
```cpp
// Empire TW Process Management System
class EmpireTWManager {
public:
    struct LaunchParameters {
        std::string scenario_file;
        std::string mod_configuration;
        ProcessPriority priority;
        MemoryLimits memory_limits;
    };
    
    class BattleProcess {
    private:
        HANDLE process_handle;
        std::shared_ptr<NamedPipeComm> communication_pipe;
        BattleMonitor monitor;
        
    public:
        BattleResult LaunchBattle(const LaunchParameters& params) {
            // Create Empire TW scenario file
            std::string scenario_path = GenerateScenario(params);
            
            // Launch Empire TW with specific parameters
            std::string command_line = BuildCommandLine(scenario_path, params);
            
            process_handle = CreateProcessSafe(command_line);
            if (!process_handle) {
                return BattleResult::LAUNCH_FAILED;
            }
            
            // Establish communication
            communication_pipe = CreateCommunicationChannel();
            
            // Monitor battle progress
            return MonitorBattleExecution();
        }
        
        BattleResult MonitorBattleExecution() {
            const int TIMEOUT_MINUTES = 30;
            auto start_time = std::chrono::steady_clock::now();
            
            while (IsProcessRunning(process_handle)) {
                // Check for timeout
                auto current_time = std::chrono::steady_clock::now();
                auto elapsed = std::chrono::duration_cast<std::chrono::minutes>(
                    current_time - start_time);
                    
                if (elapsed.count() > TIMEOUT_MINUTES) {
                    TerminateProcessSafely();
                    return BattleResult::TIMEOUT;
                }
                
                // Check for communication from Empire TW
                if (communication_pipe->HasMessage()) {
                    ProcessEmpireTWMessage();
                }
                
                // Update progress indicators
                UpdateProgressMonitor();
                
                std::this_thread::sleep_for(std::chrono::seconds(1));
            }
            
            return ExtractBattleResult();
        }
    };
};
```

### Battle Scenario Generation
```xml
<!-- Empire TW Scenario Template -->
<EmpireTWScenario>
    <BattleType>naval_engagement</BattleType>
    <Environment>
        <Map>custom_medieval_naval_map</Map>
        <Weather>{WEATHER_CONDITIONS}</Weather>
        <TimeOfDay>{TIME_OF_DAY}</TimeOfDay>
        <SeasonalEffects>{SEASONAL_MODIFIERS}</SeasonalEffects>
    </Environment>
    
    <Factions>
        <Faction id="attacker">
            <Name>{ATTACKER_FACTION_NAME}</Name>
            <Color>{FACTION_COLOR}</Color>
            <Admiral>
                <Name>{ADMIRAL_NAME}</Name>
                <Traits>{ADMIRAL_TRAITS}</Traits>
                <Experience>{EXPERIENCE_LEVEL}</Experience>
            </Admiral>
        </Faction>
        
        <Faction id="defender">
            <Name>{DEFENDER_FACTION_NAME}</Name>
            <Color>{FACTION_COLOR}</Color>
            <Admiral>
                <Name>{ADMIRAL_NAME}</Name>
                <Traits>{ADMIRAL_TRAITS}</Traits>
                <Experience>{EXPERIENCE_LEVEL}</Experience>
            </Admiral>
        </Faction>
    </Factions>
    
    <DeploymentZones>
        <AttackerZone>
            <Position x="{X_COORD}" y="{Y_COORD}"/>
            <Formation>{CONVOY_FORMATION}</Formation>
            <Approach>{ATTACK_VECTOR}</Approach>
        </AttackerZone>
        
        <DefenderZone>
            <Position x="{X_COORD}" y="{Y_COORD}"/>
            <Formation>{DEFENSIVE_FORMATION}</Formation>
            <Objectives>{DEFENSE_PRIORITIES}</Objectives>
        </DefenderZone>
    </DeploymentZones>
    
    <VictoryConditions>
        <Primary>{PRIMARY_OBJECTIVE}</Primary>
        <Secondary>{SECONDARY_OBJECTIVES}</Secondary>
        <TimeLimit>{BATTLE_TIME_LIMIT}</TimeLimit>
    </VictoryConditions>
    
    <SpecialRules>
        <CargoObjectives>{CARGO_PROTECTION_RULES}</CargoObjectives>
        <WeatherEffects>{WEATHER_BATTLE_MODIFIERS}</WeatherEffects>
        <HistoricalAccuracy>{MEDIEVAL_COMBAT_RESTRICTIONS}</HistoricalAccuracy>
    </SpecialRules>
</EmpireTWScenario>
```

## 4. User Interface Integration

### Player Choice Dialog System
```cpp
// Player Choice Interface
class HybridBattleChoiceDialog {
public:
    enum class PlayerChoice {
        QUICK_RESOLVE,
        DETAILED_BATTLE,
        CANCEL
    };
    
    struct BattleInformation {
        FleetStrength attacker_strength;
        FleetStrength defender_strength;
        CargoValue total_cargo_value;
        StrategicImportance importance;
        EstimatedDuration battle_duration;
        DifficultyRating difficulty;
    };
    
    PlayerChoice PresentChoice(const BattleInformation& info) {
        // Create custom dialog window
        auto dialog = CreateModalDialog("Naval Battle Options");
        
        // Battle overview section
        dialog->AddSection("Battle Overview", [&](DialogSection& section) {
            section.AddText(f("Attacker: {} ships", info.attacker_strength.ship_count));
            section.AddText(f("Defender: {} ships", info.defender_strength.ship_count));
            section.AddText(f("Cargo Value: {} florins", info.total_cargo_value));
            section.AddText(f("Strategic Importance: {}", 
                ImportanceToString(info.importance)));
        });
        
        // Time estimate section
        dialog->AddSection("Time Requirements", [&](DialogSection& section) {
            section.AddText(f("Quick Resolve: Instant"));
            section.AddText(f("Detailed Battle: {} minutes", 
                info.battle_duration.minutes));
            section.AddWarning("Empire Total War must be installed and available");
        });
        
        // Choice buttons
        dialog->AddButton("Quick Auto-Resolve", PlayerChoice::QUICK_RESOLVE,
            "Standard Medieval II auto-resolve with enhanced narratives");
        dialog->AddButton("Detailed Naval Battle", PlayerChoice::DETAILED_BATTLE,
            "Launch Empire Total War for full 3D naval combat");
        dialog->AddButton("Cancel", PlayerChoice::CANCEL);
        
        return dialog->ShowModal();
    }
};
```

### Battle Progress Monitor
```cpp
// Real-time battle progress monitoring
class BattleProgressMonitor {
public:
    struct ProgressState {
        Phase current_phase;
        int progress_percentage;
        std::string status_message;
        std::optional<BattleUpdate> latest_update;
    };
    
    enum class Phase {
        LAUNCHING_EMPIRE,
        LOADING_SCENARIO, 
        BATTLE_IN_PROGRESS,
        PROCESSING_RESULTS,
        COMPLETING_INTEGRATION
    };
    
    void DisplayProgressWindow(std::shared_ptr<BattleProcess> battle) {
        auto window = CreateProgressWindow("Naval Battle in Progress");
        
        // Medieval-themed loading animations
        window->SetLoadingAnimation("medieval_ship_battle.gif");
        window->SetBackgroundMusic("medieval_naval_ambient.ogg");
        
        // Progress tracking
        while (battle->IsActive()) {
            ProgressState state = battle->GetProgressState();
            
            window->UpdateProgress(state.progress_percentage);
            window->SetStatusText(state.status_message);
            
            // Show battle updates if available
            if (state.latest_update.has_value()) {
                window->AddBattleUpdate(state.latest_update.value());
            }
            
            // Update every second
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
        
        window->Close();
    }
    
    void AddBattleUpdate(const BattleUpdate& update) {
        std::string formatted_update;
        
        switch(update.type) {
            case BattleUpdate::ENGAGEMENT_STARTED:
                formatted_update = f("Fleets have engaged! {} vs {}", 
                    update.attacker_name, update.defender_name);
                break;
                
            case BattleUpdate::SHIP_SUNK:
                formatted_update = f("{} {} has been sunk!", 
                    update.faction_name, update.ship_name);
                break;
                
            case BattleUpdate::CARGO_CAPTURED:
                formatted_update = f("Valuable cargo has been captured by {}!", 
                    update.faction_name);
                break;
                
            case BattleUpdate::ADMIRAL_KILLED:
                formatted_update = f("Admiral {} has fallen in battle!", 
                    update.admiral_name);
                break;
        }
        
        battle_log.push_back({
            .timestamp = std::chrono::steady_clock::now(),
            .message = formatted_update,
            .importance = update.importance
        });
    }
};
```

## 5. Result Import and Integration

### Battle Outcome Translation
```cpp
// Empire TW Result Processing
class BattleResultProcessor {
public:
    struct EmpireBattleResult {
        VictoryStatus victory_status;
        CasualtyReport casualties;
        CargoStatus cargo_outcomes;
        AdmiralFates admiral_results;
        ExperienceGains experience_earned;
        DiplomaticConsequences diplomatic_impact;
    };
    
    Medieval2CampaignResult TranslateResult(
        const EmpireBattleResult& empire_result,
        const BattleContext& original_context) {
        
        Medieval2CampaignResult campaign_result;
        
        // Translate casualties back to Medieval II units
        campaign_result.casualties = TranslateCasualties(
            empire_result.casualties, original_context.unit_mapping);
            
        // Handle cargo outcomes
        campaign_result.cargo_changes = ProcessCargoOutcomes(
            empire_result.cargo_outcomes, original_context.cargo_manifest);
            
        // Process admiral fate and experience
        campaign_result.character_changes = ProcessCharacterOutcomes(
            empire_result.admiral_results, empire_result.experience_earned);
            
        // Calculate economic impact
        campaign_result.economic_impact = CalculateEconomicConsequences(
            empire_result, original_context.economic_context);
            
        // Determine diplomatic ramifications
        campaign_result.diplomatic_changes = ProcessDiplomaticOutcomes(
            empire_result.diplomatic_impact, original_context.factions);
            
        return campaign_result;
    }
    
private:
    CasualtyReport TranslateCasualties(
        const EmpireCasualties& empire_casualties,
        const UnitMapping& mapping) {
        
        CasualtyReport medieval_casualties;
        
        for (const auto& empire_loss : empire_casualties.ships_lost) {
            // Find corresponding Medieval II ship
            auto medieval_ship = mapping.GetMedievalEquivalent(empire_loss.ship_type);
            
            // Calculate crew losses based on ship type and battle circumstances
            int crew_lost = CalculateCrewLosses(empire_loss, medieval_ship);
            
            medieval_casualties.ships_lost.push_back({
                .ship_type = medieval_ship.type,
                .crew_casualties = crew_lost,
                .experienced_crew_lost = empire_loss.veteran_crew_lost,
                .ship_capture_status = empire_loss.was_captured
            });
        }
        
        return medieval_casualties;
    }
    
    CargoManifest ProcessCargoOutcomes(
        const CargoStatus& empire_cargo,
        const CargoManifest& original_manifest) {
        
        CargoManifest updated_manifest = original_manifest;
        
        // Process captured cargo
        for (const auto& captured : empire_cargo.cargo_captured) {
            auto& cargo_entry = updated_manifest.FindCargo(captured.cargo_id);
            cargo_entry.owner = captured.new_owner;
            cargo_entry.location = captured.current_location;
            cargo_entry.condition = captured.cargo_condition;
        }
        
        // Process destroyed/lost cargo
        for (const auto& lost : empire_cargo.cargo_lost) {
            updated_manifest.RemoveCargo(lost.cargo_id);
        }
        
        return updated_manifest;
    }
};
```

### Campaign Integration
```lua
-- Lua integration for campaign consequence application
function applyHybridBattleResults(battleId, empireBattleResult)
    local campaignResult = translateEmpireResult(empireBattleResult)
    
    -- Apply unit casualties
    for _, casualty in pairs(campaignResult.casualties) do
        local unit = findCampaignUnit(casualty.unitId)
        if unit then
            unit:takeCasualties(casualty.losses)
            if casualty.experienced_crew_lost > 0 then
                unit:reduceExperience(casualty.experienced_crew_lost)
            end
        end
    end
    
    -- Handle cargo transfers
    for _, cargoChange in pairs(campaignResult.cargoChanges) do
        local cargoItem = findCargoItem(cargoChange.cargoId)
        if cargoItem then
            if cargoChange.captured then
                transferCargoToFaction(cargoItem, cargoChange.newOwner)
            elseif cargoChange.destroyed then
                removeCargoFromCampaign(cargoItem)
            end
        end
    end
    
    -- Process character consequences
    for _, characterChange in pairs(campaignResult.characterChanges) do
        local character = findCharacter(characterChange.characterId)
        if character then
            -- Apply experience gains
            character:addExperience(characterChange.experienceGained)
            
            -- Handle potential death or wounding
            if characterChange.killed then
                character:kill("died_in_naval_battle")
            elseif characterChange.wounded then
                character:addTrait("wounded", characterChange.woundSeverity)
            end
            
            -- Add battle-related traits
            for _, trait in pairs(characterChange.traitsGained) do
                character:addTrait(trait.name, trait.level)
            end
        end
    end
    
    -- Apply economic consequences
    local economicImpact = campaignResult.economicImpact
    for factionName, impact in pairs(economicImpact) do
        local faction = getFaction(factionName)
        if faction then
            faction:adjustTreasury(impact.treasuryChange)
            faction:adjustTradeIncome(impact.tradeIncomeChange)
            
            -- Update reputation based on battle outcome
            if impact.reputationChange ~= 0 then
                faction:adjustReputation(impact.reputationChange)
            end
        end
    end
    
    -- Process diplomatic ramifications
    for _, diplomaticChange in pairs(campaignResult.diplomaticChanges) do
        local faction1 = getFaction(diplomaticChange.faction1)
        local faction2 = getFaction(diplomaticChange.faction2)
        
        if faction1 and faction2 then
            -- Adjust relationship based on battle outcome
            adjustFactionRelationship(faction1, faction2, diplomaticChange.relationshipChange)
            
            -- Add diplomatic memory of the battle
            addDiplomaticMemory(faction1, faction2, {
                event = "naval_battle",
                outcome = diplomaticChange.outcome,
                significance = diplomaticChange.significance,
                decay_rate = 0.95 -- Gradually fade over time
            })
        end
    end
    
    -- Generate battle narrative for history
    local narrative = generateBattleNarrative(battleId, empireBattleResult, campaignResult)
    addToFactionHistory(battleId.attackerFaction, narrative.attackerPerspective)
    addToFactionHistory(battleId.defenderFaction, narrative.defenderPerspective)
    
    -- Trigger follow-up events if appropriate
    checkForFollowUpEvents(campaignResult)
end
```

## 6. Error Handling and Fallbacks

### Comprehensive Error Recovery
```cpp
// Error handling and recovery system
class HybridBattleErrorHandler {
public:
    enum class ErrorType {
        EMPIRE_TW_NOT_FOUND,
        EMPIRE_TW_LAUNCH_FAILED,
        SCENARIO_GENERATION_FAILED,
        BATTLE_TIMEOUT,
        RESULT_IMPORT_FAILED,
        SAVE_CORRUPTION_DETECTED,
        COMMUNICATION_FAILURE
    };
    
    struct ErrorContext {
        ErrorType error_type;
        std::string error_message;
        BattleContext battle_context;
        std::chrono::time_point<std::chrono::steady_clock> error_time;
        std::optional<std::string> recovery_suggestion;
    };
    
    BattleResult HandleError(const ErrorContext& context) {
        // Log error for debugging
        LogError(context);
        
        // Attempt automatic recovery based on error type
        switch (context.error_type) {
            case EMPIRE_TW_NOT_FOUND:
                return HandleEmpireNotFound(context);
                
            case EMPIRE_TW_LAUNCH_FAILED:
                return HandleLaunchFailure(context);
                
            case BATTLE_TIMEOUT:
                return HandleBattleTimeout(context);
                
            case RESULT_IMPORT_FAILED:
                return HandleImportFailure(context);
                
            case SAVE_CORRUPTION_DETECTED:
                return HandleSaveCorruption(context);
                
            default:
                return HandleUnknownError(context);
        }
    }
    
private:
    BattleResult HandleEmpireNotFound(const ErrorContext& context) {
        // Present user with options
        auto choice = PresentErrorDialog({
            .title = "Empire Total War Not Found",
            .message = "Empire Total War is required for detailed naval battles.",
            .options = {
                {"Use Enhanced Auto-Resolve", "Continue with improved Medieval II auto-resolve"},
                {"Install Empire TW", "Open Steam store page for Empire Total War"},  
                {"Cancel Battle", "Return to campaign without resolving battle"}
            }
        });
        
        switch (choice) {
            case 0: // Enhanced auto-resolve
                return FallbackToEnhancedAutoResolve(context.battle_context);
            case 1: // Install Empire TW
                LaunchSteamStorePage("empire_total_war");
                return BattleResult::USER_CANCELLED;
            case 2: // Cancel
                return BattleResult::USER_CANCELLED;
        }
    }
    
    BattleResult HandleBattleTimeout(const ErrorContext& context) {
        // Save current Empire TW state if possible
        auto empireSave = AttemptEmpireSaveExtraction();
        
        // Estimate battle outcome based on current state
        if (empireSave.has_value()) {
            return EstimateBattleOutcomeFromPartialResult(empireSave.value());
        }
        
        // Fallback to enhanced auto-resolve with time penalty
        auto autoResolveResult = FallbackToEnhancedAutoResolve(context.battle_context);
        
        // Apply timeout penalty (slight disadvantage to player)
        if (context.battle_context.player_is_attacker) {
            autoResolveResult.ApplyTimeoutPenalty(0.9); // 10% penalty
        }
        
        return autoResolveResult;
    }
    
    BattleResult FallbackToEnhancedAutoResolve(const BattleContext& context) {
        // Use sophisticated auto-resolve with Empire TW-inspired calculations
        auto resolver = EnhancedAutoResolver();
        
        // Apply Empire TW tactical considerations to auto-resolve
        resolver.SetTacticalFactors({
            .weather_impact = CalculateWeatherAdvantage(context.weather),
            .formation_bonus = CalculateFormationAdvantage(context.formations),
            .experience_modifier = CalculateExperienceAdvantage(context.admirals),
            .cargo_objectives = context.cargo_objectives
        });
        
        auto result = resolver.ResolveNavalBattle(context);
        
        // Generate detailed narrative to compensate for lack of visual battle
        result.battle_narrative = GenerateEnhancedNarrative(context, result);
        
        return result;
    }
    
    void CreateSaveBackup(const std::string& backup_reason) {
        auto timestamp = GetCurrentTimestamp();
        auto backup_path = f("saves/backup_before_{}_{}",
            backup_reason, timestamp);
            
        CopySaveFiles(GetCurrentSavePath(), backup_path);
        
        // Store backup metadata
        BackupMetadata metadata{
            .creation_time = timestamp,
            .reason = backup_reason,
            .game_state_hash = CalculateSaveStateHash(),
            .mod_version = GetModVersion()
        };
        
        StoreDackupMetadata(backup_path, metadata);
    }
};
```

## 7. Performance Considerations

### Memory and Resource Management
```cpp
// Performance optimization system
class PerformanceManager {
public:
    struct PerformanceProfile {
        MemoryLimits memory_limits;
        ProcessPriority priorities;
        GraphicsSettings graphics_config;
        NetworkSettings network_config;
    };
    
    void OptimizeForHybridBattles() {
        // Adjust Medieval II memory allocation
        AdjustMedieval2Memory();
        
        // Prepare system for Empire TW launch
        PrepareForEmpireLaunch();
        
        // Optimize inter-process communication
        OptimizeIPCPerformance();
    }
    
private:
    void AdjustMedieval2Memory() {
        // Reduce Medieval II memory usage before launching Empire
        // This is critical for 32-bit compatibility
        
        // Temporarily reduce texture quality
        M2TWEOP.setTemporaryGraphicsReduction(true);
        
        // Clear unnecessary caches
        ClearAssetCaches();
        
        // Minimize background processes
        SuspendNonEssentialSystems();
    }
    
    void PrepareForEmpireLaunch() {
        // Ensure adequate memory for Empire TW
        const size_t EMPIRE_TW_MEMORY_REQUIREMENT = 2048 * 1024 * 1024; // 2GB
        
        size_t available_memory = GetAvailableMemory();
        if (available_memory < EMPIRE_TW_MEMORY_REQUIREMENT) {
            // Attempt to free additional memory
            FreeNonEssentialMemory();
            
            // If still insufficient, warn user
            if (GetAvailableMemory() < EMPIRE_TW_MEMORY_REQUIREMENT) {
                ShowLowMemoryWarning();
            }
        }
        
        // Set Empire TW to use dedicated GPU if available
        ConfigureGPUUsage();
    }
    
    void MonitorPerformanceDuringBattle() {
        auto monitor = PerformanceMonitor();
        
        monitor.TrackMetrics({
            .cpu_usage = true,
            .memory_usage = true,
            .gpu_usage = true,
            .disk_io = true,
            .network_io = false // Not needed for local battles
        });
        
        // Set performance thresholds
        monitor.SetThresholds({
            .max_cpu_usage = 85.0,
            .max_memory_usage_mb = 3800, // Leave room for 32-bit limit
            .max_gpu_usage = 90.0
        });
        
        monitor.SetCallback([](const PerformanceAlert& alert) {
            HandlePerformanceIssue(alert);
        });
        
        monitor.StartMonitoring();
    }
};
```

### Loading Time Optimization
```cpp
// Parallel loading and caching system
class LoadingOptimizer {
public:
    void PreloadEmpireAssets() {
        // Background preloading of Empire TW assets while player makes choice
        std::async(std::launch::async, [this]() {
            PreloadEmpireExecutable();
            PreloadMedievalModAssets();
            PrepareScenarioTemplates();
        });
    }
    
    void OptimizeEmpireLaunch(const BattleContext& context) {
        // Use cached scenario templates
        auto scenario_template = GetCachedScenarioTemplate(context.battle_type);
        
        // Pre-generate commonly used battle configurations
        auto battle_config = GenerateBattleConfig(context, scenario_template);
        
        // Use fast launch parameters
        auto launch_params = CreateOptimizedLaunchParams(battle_config);
        
        // Launch with SSD-optimized settings if available
        if (IsSSDAvailable()) {
            launch_params.use_ssd_optimizations = true;
        }
    }
    
private:
    void PreloadEmpireExecutable() {
        // Load Empire TW into memory cache (Windows file caching)
        auto empire_path = FindEmpireTWExecutable();
        if (empire_path) {
            CacheExecutableInMemory(*empire_path);
        }
    }
    
    void PrepareScenarioTemplates() {
        // Generate common scenario variations in advance
        std::vector<std::future<void>> template_tasks;
        
        for (const auto& battle_type : COMMON_BATTLE_TYPES) {
            template_tasks.push_back(
                std::async(std::launch::async, [this, battle_type]() {
                    GenerateScenarioTemplate(battle_type);
                })
            );
        }
        
        // Wait for all template generation to complete
        for (auto& task : template_tasks) {
            task.wait();
        }
    }
};
```

## 8. Development Timeline and Milestones

### 8-Week Implementation Schedule

#### Week 1-2: Foundation and Research
```
Phase 1A: M2TWEOP Integration Setup (Week 1)
├── Install and configure M2TWEOP development environment
├── Research and document M2TWEOP Lua API naval battle hooks  
├── Create basic battle detection prototype
├── Establish Empire TW installation verification system
└── Set up cross-process communication framework

Phase 1B: Empire TW Analysis and Preparation (Week 2)
├── Analyze Empire TW file structure and modding capabilities
├── Research Empire TW command-line parameters and launch options
├── Create medieval-themed Empire TW mod framework
├── Design ship translation matrix (Medieval → Empire units)
└── Develop basic scenario generation system
```

#### Week 3-4: Core System Development
```
Phase 2A: Battle Detection and Translation Engine (Week 3)
├── Implement M2TWEOP Lua hooks for naval battle interception
├── Create battle context analysis system (importance calculation)
├── Develop Medieval II → Empire TW data translation engine
├── Build ship type mapping and cargo value translation
└── Implement weather and environment translation system

Phase 2B: Process Management and Communication (Week 4)
├── Create Empire TW process launcher and manager
├── Implement inter-process communication system
├── Develop scenario file generation system
├── Create battle progress monitoring framework
└── Build basic error handling and recovery system
```

#### Week 5-6: User Interface and Experience
```
Phase 3A: Player Choice and Progress Systems (Week 5)
├── Design and implement player choice dialog
├── Create battle information presentation system
├── Develop battle progress monitor with medieval theming
├── Implement loading screens and transition effects
└── Build user preference and configuration system

Phase 3B: Result Import and Campaign Integration (Week 6)
├── Create Empire TW battle result extraction system
├── Implement result translation back to Medieval II format
├── Develop campaign consequence calculation engine
├── Create character and unit modification system
└── Build economic and diplomatic impact processors
```

#### Week 7-8: Testing, Polish, and Integration
```
Phase 4A: Comprehensive Testing and Bug Fixing (Week 7)
├── Unit testing for all major system components
├── Integration testing with existing Medieval III systems
├── Performance testing and optimization
├── Error scenario testing and recovery validation
└── Cross-game compatibility verification

Phase 4B: Final Polish and Community Preparation (Week 8)
├── User interface polish and accessibility improvements
├── Performance optimization and memory management
├── Documentation creation for users and developers
├── Community beta testing framework setup
└── Final integration with Phase 2 Medieval III systems
```

### Critical Milestones and Deliverables

#### Milestone 1: Proof of Concept (End Week 2)
```
Deliverables:
✓ M2TWEOP can detect and intercept naval battles
✓ Empire TW can be launched with custom parameters
✓ Basic communication between processes established
✓ Simple battle data extraction working

Success Criteria:
- Naval battle detection accuracy > 95%
- Empire TW launch success rate > 90%
- Basic data exchange functional
- No save game corruption in testing
```

#### Milestone 2: Core System Functional (End Week 4)
```
Deliverables:
✓ Complete battle translation system operational
✓ Empire TW scenarios generated from Medieval II data
✓ Battle results successfully imported back to Medieval II
✓ Error handling prevents system crashes

Success Criteria:
- Battle translation accuracy verified by manual comparison
- Scenario generation produces playable Empire TW battles
- Result import maintains campaign state integrity
- System degrades gracefully when Empire TW unavailable
```

#### Milestone 3: User Experience Complete (End Week 6)
```
Deliverables:
✓ Player choice system fully functional
✓ Battle progress monitoring working smoothly
✓ Campaign integration preserves all game systems
✓ Performance optimized for target hardware

Success Criteria:
- Player choice response time < 2 seconds
- Battle progress updates in real-time
- No noticeable performance degradation
- Integration with cargo/convoy systems verified
```

#### Milestone 4: Production Ready (End Week 8)
```
Deliverables:
✓ All systems tested and stable
✓ Community beta testing framework operational
✓ Documentation complete for users and developers
✓ Integration with Medieval III Phase 2 systems complete

Success Criteria:
- 7-day continuous operation without crashes
- Community beta testing successfully launched
- System performance meets all benchmarks
- Ready for inclusion in Medieval III Phase 2 release
```

### Resource Allocation and Dependencies

#### Development Resources Required
```
Primary Developer: 1 FTE (Full-Time Equivalent)
- C++/Lua programming expertise
- M2TWEOP framework experience  
- Total War modding background
- Cross-platform development skills

Secondary Resources:
- Empire TW modding specialist: 0.25 FTE
- UI/UX designer: 0.25 FTE  
- Quality assurance tester: 0.5 FTE
- Community liaison: 0.1 FTE
```

#### Technical Dependencies
```
Critical Dependencies:
- M2TWEOP 2.1+ stable release
- Empire Total War installation and modding tools
- Visual Studio 2019+ for C++ development
- Modern GPU for AI graphics processing

Integration Dependencies:
- Medieval III Graphics Modernization system
- Medieval III Economic Revolution framework
- Medieval III Naval Warfare systems (cargo, convoys)
- Phase 1 AI graphics pipeline operational
```

#### Risk Mitigation Strategies
```
High-Risk Elements:
1. Empire TW Compatibility Issues
   → Mitigation: Extensive testing with multiple Empire TW versions
   → Fallback: Enhanced auto-resolve system as backup

2. Performance Impact on 32-bit Medieval II
   → Mitigation: Aggressive memory management and optimization
   → Fallback: System requirements validation and warnings

3. Save Game Corruption Risk
   → Mitigation: Comprehensive backup system before battles
   → Fallback: Automatic save recovery and validation

4. Community Adoption Barriers  
   → Mitigation: Optional system with clear value proposition
   → Fallback: Traditional naval warfare enhancements
```

## 9. Testing and Quality Assurance

### Comprehensive Testing Framework

#### Integration Testing
```cpp
// Integration test suite for hybrid naval battle system
class HybridBattleIntegrationTests {
public:
    void RunAllTests() {
        TestBasicBattleFlow();
        TestErrorRecovery();
        TestPerformanceUnderLoad();
        TestCrossGameDataConsistency();
        TestSaveGameIntegrity();
        TestCommunityScenarios();
    }
    
private:
    void TestBasicBattleFlow() {
        auto test_cases = GenerateTestBattles({
            .small_skirmish = {2, 3}, // 2 vs 3 ships
            .medium_engagement = {5, 7}, // 5 vs 7 ships  
            .large_convoy_battle = {12, 8}, // 12 vs 8 ships
            .treasure_fleet_raid = {15, 4} // 15 vs 4 ships (high cargo value)
        });
        
        for (const auto& test_case : test_cases) {
            // Test complete battle flow
            auto result = ExecuteHybridBattle(test_case);
            
            // Validate result consistency
            ASSERT_TRUE(ValidateBattleResult(result, test_case));
            
            // Check campaign state integrity
            ASSERT_TRUE(ValidateCampaignState(result));
            
            // Verify save game compatibility
            ASSERT_TRUE(ValidateSaveGame(result));
        }
    }
    
    void TestErrorRecovery() {
        // Test various failure scenarios
        std::vector<ErrorScenario> error_scenarios = {
            {.type = EMPIRE_TW_NOT_FOUND, .expected_recovery = ENHANCED_AUTO_RESOLVE},
            {.type = EMPIRE_TW_CRASH, .expected_recovery = BATTLE_STATE_RECOVERY},
            {.type = TIMEOUT, .expected_recovery = ESTIMATED_OUTCOME},
            {.type = SAVE_CORRUPTION, .expected_recovery = BACKUP_RESTORE}
        };
        
        for (const auto& scenario : error_scenarios) {
            auto recovery_result = TestErrorScenario(scenario);
            ASSERT_EQ(recovery_result.recovery_method, scenario.expected_recovery);
            ASSERT_FALSE(recovery_result.caused_save_corruption);
        }
    }
    
    void TestPerformanceUnderLoad() {
        // Test system performance with multiple concurrent operations
        const int CONCURRENT_TESTS = 5;
        std::vector<std::future<PerformanceMetrics>> performance_tests;
        
        for (int i = 0; i < CONCURRENT_TESTS; ++i) {
            performance_tests.push_back(
                std::async(std::launch::async, [this]() {
                    return MeasurePerformanceDuringBattle();
                })
            );
        }
        
        // Collect and validate performance results
        for (auto& test : performance_tests) {
            auto metrics = test.get();
            
            ASSERT_LT(metrics.memory_usage_mb, 3800); // 32-bit limit
            ASSERT_LT(metrics.cpu_usage_percent, 85);
            ASSERT_LT(metrics.battle_launch_time_seconds, 30);
        }
    }
    
    void TestCrossGameDataConsistency() {
        // Verify data translation accuracy
        auto test_fleets = GenerateTestFleets();
        
        for (const auto& medieval_fleet : test_fleets) {
            // Translate to Empire TW format
            auto empire_fleet = TranslateToEmpireFormat(medieval_fleet);
            
            // Execute mock battle
            auto empire_result = SimulateEmpireBattle(empire_fleet);
            
            // Translate back to Medieval II
            auto medieval_result = TranslateToMedievalFormat(empire_result);
            
            // Verify consistency
            ASSERT_TRUE(ValidateTranslationConsistency(
                medieval_fleet, medieval_result));
        }
    }
};
```

#### Community Beta Testing Framework
```cpp
// Community testing program management
class CommunityBetaProgram {
public:
    struct BetaTester {
        std::string username;
        ExperienceLevel experience;
        HardwareSpecs hardware;
        std::vector<TestScenario> assigned_tests;
        FeedbackHistory feedback_history;
    };
    
    void LaunchBetaProgram() {
        // Phase 1: Alpha testing (developers and experienced modders)
        LaunchAlphaTesting();
        
        // Phase 2: Closed beta (selected community members)  
        LaunchClosedBeta();
        
        // Phase 3: Open beta (general community)
        LaunchOpenBeta();
    }
    
private:
    void LaunchAlphaTesting() {
        auto alpha_testers = SelectAlphaTesters({
            .criteria = {MODDING_EXPERIENCE, TECHNICAL_BACKGROUND},
            .hardware_requirements = HIGH_END,
            .commitment_level = FULL_TIME_TESTING
        });
        
        for (auto& tester : alpha_testers) {
            AssignTestScenarios(tester, COMPREHENSIVE_TEST_SUITE);
            ProvideTestingTools(tester);
            SetupDirectFeedbackChannel(tester);
        }
    }
    
    void LaunchClosedBeta() {
        auto beta_testers = SelectBetaTesters({
            .criteria = {COMMUNITY_INVOLVEMENT, TOTAL_WAR_EXPERIENCE},
            .hardware_requirements = MINIMUM_SPECS,  
            .commitment_level = REGULAR_TESTING
        });
        
        // Distribute focused test scenarios
        for (auto& tester : beta_testers) {
            auto specialized_tests = CreateSpecializedTestSuite(tester);
            AssignTestScenarios(tester, specialized_tests);
            ProvideUserFriendlyTestingInterface(tester);
        }
    }
    
    std::vector<TestScenario> CreateSpecializedTestSuite(const BetaTester& tester) {
        std::vector<TestScenario> scenarios;
        
        // Hardware-specific scenarios
        if (tester.hardware.memory_gb >= 16) {
            scenarios.push_back(LARGE_BATTLE_STRESS_TEST);
        }
        
        if (tester.hardware.has_ssd) {
            scenarios.push_back(LOADING_TIME_OPTIMIZATION_TEST);
        }
        
        // Experience-based scenarios
        if (tester.experience == NAVAL_WARFARE_EXPERT) {
            scenarios.push_back(HISTORICAL_ACCURACY_VALIDATION);
            scenarios.push_back(TACTICAL_REALISM_ASSESSMENT);
        }
        
        if (tester.experience == PERFORMANCE_TESTING_EXPERT) {
            scenarios.push_back(PERFORMANCE_BENCHMARK_SUITE);
            scenarios.push_back(MEMORY_USAGE_ANALYSIS);
        }
        
        return scenarios;
    }
};
```

### Performance Validation Framework
```cpp
// Automated performance validation system
class PerformanceValidator {
public:
    struct PerformanceBenchmarks {
        // Timing benchmarks (95th percentile targets)
        std::chrono::seconds battle_detection_time{2};
        std::chrono::seconds empire_launch_time{30};
        std::chrono::seconds scenario_generation_time{5};
        std::chrono::seconds result_import_time{10};
        
        // Resource benchmarks
        size_t max_memory_usage_mb{3800}; // 32-bit safety margin
        double max_cpu_usage_percent{85.0};
        size_t max_disk_usage_mb{500}; // Temporary scenario files
        
        // Quality benchmarks  
        double min_translation_accuracy{0.95};
        double min_battle_completion_rate{0.98};
        double min_save_integrity_rate{0.999};
    };
    
    ValidationResult ValidatePerformance() {
        ValidationResult result;
        
        // Run performance test suite
        auto metrics = RunPerformanceTestSuite();
        
        // Validate against benchmarks
        result.timing_validation = ValidateTimingMetrics(metrics.timing);
        result.resource_validation = ValidateResourceUsage(metrics.resources);
        result.quality_validation = ValidateQualityMetrics(metrics.quality);
        
        // Generate performance report
        result.performance_report = GeneratePerformanceReport(metrics);
        
        // Provide optimization recommendations if needed
        if (!result.AllTestsPassed()) {
            result.optimization_recommendations = 
                GenerateOptimizationRecommendations(metrics);
        }
        
        return result;
    }
    
private:
    PerformanceMetrics RunPerformanceTestSuite() {
        PerformanceMetrics metrics;
        
        // Test various battle scenarios under different conditions
        std::vector<TestCondition> conditions = {
            {.system_load = LOW, .battle_size = SMALL},
            {.system_load = MEDIUM, .battle_size = MEDIUM},  
            {.system_load = HIGH, .battle_size = LARGE},
            {.system_load = EXTREME, .battle_size = MAXIMUM}
        };
        
        for (const auto& condition : conditions) {
            auto test_result = RunPerformanceTest(condition);
            metrics.AddTestResult(condition, test_result);
        }
        
        // Calculate aggregate performance statistics
        metrics.CalculateAggregateMetrics();
        
        return metrics;
    }
    
    OptimizationRecommendations GenerateOptimizationRecommendations(
        const PerformanceMetrics& metrics) {
        
        OptimizationRecommendations recommendations;
        
        // Memory optimization recommendations
        if (metrics.peak_memory_usage > benchmarks.max_memory_usage_mb) {
            recommendations.memory_optimizations = {
                "Reduce texture quality during Empire TW battles",
                "Implement more aggressive garbage collection",
                "Consider memory-mapped file approach for large data",
                "Optimize scenario file generation to use less memory"
            };
        }
        
        // CPU optimization recommendations
        if (metrics.average_cpu_usage > benchmarks.max_cpu_usage_percent) {
            recommendations.cpu_optimizations = {
                "Implement parallel processing for data translation",
                "Optimize battle result parsing algorithms",
                "Use hardware-accelerated processing where available",
                "Reduce polling frequency for battle monitoring"
            };
        }
        
        // Loading time optimization recommendations
        if (metrics.average_launch_time > benchmarks.empire_launch_time) {
            recommendations.loading_optimizations = {
                "Implement scenario file caching system",
                "Pre-load Empire TW assets in background",
                "Optimize Empire TW launch parameters",  
                "Use SSD-specific optimizations where available"
            };
        }
        
        return recommendations;
    }
};
```

## 10. Medieval III Systems Integration

### Cargo Capacity System Compatibility
```lua
-- Integration with Medieval III cargo management
function translateCargoForEmpireBattle(medievalFleet)
    local empireFleetData = {}
    
    for _, ship in pairs(medievalFleet.ships) do
        local empireShip = {
            type = translateShipType(ship.type),
            experience = ship.crew.experience,
            cargo_manifest = {}
        }
        
        -- Translate cargo to Empire TW objectives
        for _, cargo in pairs(ship.cargo) do
            local empireObjective = translateCargoToObjective(cargo)
            table.insert(empireShip.cargo_manifest, empireObjective)
            
            -- Adjust ship performance based on cargo load
            if cargo.type == "heavy_goods" then
                empireShip.speed_modifier = empireShip.speed_modifier * 0.9
                empireShip.maneuverability_modifier = empireShip.maneuverability_modifier * 0.85
            elseif cargo.type == "treasure" then
                empireShip.target_priority = "high_value"
                empireShip.capture_bonus = cargo.value * 0.1
            end
        end
        
        -- Calculate cargo vs military balance impact
        local cargo_ratio = calculateCargoRatio(ship)
        if cargo_ratio > 0.7 then
            -- High cargo load reduces combat effectiveness
            empireShip.combat_effectiveness = empireShip.combat_effectiveness * 0.8
            empireShip.crew_size_modifier = 0.6 -- Less crew space
        elseif cargo_ratio < 0.3 then  
            -- Low cargo load allows for more marines
            empireShip.boarding_bonus = 1.3
            empireShip.crew_quality_bonus = 1.2
        end
        
        table.insert(empireFleetData, empireShip)
    end
    
    return empireFleetData
end

function translateCargoToObjective(cargo)
    local objective = {
        id = cargo.id,
        value = cargo.value,
        priority = "medium"
    }
    
    -- Different cargo types create different battle objectives
    if cargo.type == "spices" or cargo.type == "silk" then
        objective.type = "capture_intact"
        objective.priority = "high"
        objective.description = "Valuable trade goods - capture without damage"
    elseif cargo.type == "grain" or cargo.type == "wood" then
        objective.type = "deny_enemy"
        objective.priority = "low"  
        objective.description = "Bulk goods - prevent enemy capture"
    elseif cargo.type == "gold" or cargo.type == "silver" then
        objective.type = "treasure"
        objective.priority = "maximum"
        objective.description = "Precious metals - highest capture priority"
    elseif cargo.type == "weapons" or cargo.type == "iron" then
        objective.type = "military_supplies"
        objective.priority = "high"
        objective.description = "Military supplies - strategic value"
    end
    
    return objective
end
```

### Convoy Formation Translation
```cpp
// Medieval III convoy formation → Empire TW fleet deployment
class ConvoyFormationTranslator {
public:
    struct EmpireDeployment {
        std::vector<ShipPosition> ship_positions;
        FormationBonus formation_bonus;
        TacticalAdvantages advantages;
        VulnerabilityFactors vulnerabilities;
    };
    
    EmpireDeployment TranslateFormation(
        const MedievalConvoyFormation& medieval_formation,
        const BattleEnvironment& environment) {
        
        EmpireDeployment deployment;
        
        switch (medieval_formation.type) {
            case TIGHT_CONVOY:
                deployment = CreateTightFormation(medieval_formation);
                deployment.advantages.mutual_support = 1.3;
                deployment.vulnerabilities.weather_susceptibility = 1.4;
                break;
                
            case SPREAD_CONVOY:
                deployment = CreateSpreadFormation(medieval_formation);
                deployment.advantages.speed = 1.2;
                deployment.vulnerabilities.individual_targeting = 1.3;
                break;
                
            case ESCORT_PATTERN:
                deployment = CreateEscortFormation(medieval_formation);
                deployment.advantages.cargo_protection = 1.4;
                deployment.advantages.coordinated_defense = 1.2;
                break;
                
            case RAIDING_FORMATION:
                deployment = CreateRaidingFormation(medieval_formation);
                deployment.advantages.surprise_attack = 1.5;
                deployment.advantages.target_isolation = 1.3;
                break;
        }
        
        // Apply environmental modifiers
        ApplyEnvironmentalEffects(deployment, environment);
        
        return deployment;
    }
    
private:
    EmpireDeployment CreateTightFormation(const MedievalConvoyFormation& formation) {
        EmpireDeployment deployment;
        
        // Place ships in close formation for mutual support
        float formation_spacing = 200.0f; // meters between ships
        
        for (size_t i = 0; i < formation.ships.size(); ++i) {
            ShipPosition position;
            
            // Calculate position in formation grid
            int row = i / SHIPS_PER_ROW;
            int col = i % SHIPS_PER_ROW;
            
            position.x = col * formation_spacing;
            position.y = row * formation_spacing;
            position.facing = 0; // All ships face same direction
            
            // Ship role assignment
            if (formation.ships[i].type == WARSHIP) {
                position.role = ESCORT;
                position.priority = PROTECT_CARGO_SHIPS;
            } else {
                position.role = CARGO_CARRIER;
                position.priority = STAY_IN_FORMATION;
            }
            
            deployment.ship_positions.push_back(position);
        }
        
        return deployment;
    }
    
    EmpireDeployment CreateRaidingFormation(const MedievalConvoyFormation& formation) {
        EmpireDeployment deployment;
        
        // Raiding formation focuses on isolating and overwhelming targets
        
        // Fast ships form flanking groups
        std::vector<Ship> fast_ships = FilterShipsBySpeed(formation.ships, FAST);
        std::vector<Ship> heavy_ships = FilterShipsBySpeed(formation.ships, SLOW);
        
        // Position fast ships for flanking maneuvers
        for (size_t i = 0; i < fast_ships.size(); ++i) {
            ShipPosition position;
            
            if (i % 2 == 0) {
                // Left flank
                position.x = -500 - (i * 100);
                position.y = 300 + (i * 150);
            } else {
                // Right flank  
                position.x = 500 + (i * 100);
                position.y = 300 + (i * 150);
            }
            
            position.role = FLANKER;
            position.priority = ISOLATE_ENEMY_SHIPS;
            position.special_orders = "attack_when_enemy_spreads";
            
            deployment.ship_positions.push_back(position);
        }
        
        // Position heavy ships for main engagement
        for (size_t i = 0; i < heavy_ships.size(); ++i) {
            ShipPosition position;
            
            position.x = i * 250;
            position.y = 0;
            position.role = MAIN_BATTLE_LINE;
            position.priority = ENGAGE_ENEMY_ESCORTS;
            
            deployment.ship_positions.push_back(position);
        }
        
        return deployment;
    }
};
```

### Weather Effect Integration
```json
{
  "weather_integration_system": {
    "medieval_to_empire_mapping": {
      "storm_conditions": {
        "empire_weather": "heavy_storm",
        "wind_strength": 8,
        "sea_state": "very_rough",
        "visibility": "poor",
        "effects": {
          "small_ships_advantage": 1.3,
          "large_ships_penalty": 0.7,
          "convoy_separation_chance": 0.6,
          "ramming_effectiveness": 0.5
        }
      },
      "calm_seas": {
        "empire_weather": "clear_calm",
        "wind_strength": 2,
        "sea_state": "calm",
        "visibility": "excellent",
        "effects": {
          "galley_advantage": 1.4,
          "sailing_ship_penalty": 0.8,
          "formation_integrity": 1.2,
          "boarding_effectiveness": 1.3
        }
      },
      "winter_conditions": {
        "empire_weather": "winter_storm",
        "wind_strength": 6,
        "sea_state": "rough",
        "visibility": "limited",
        "time_of_day": "dawn",
        "effects": {
          "all_ships_penalty": 0.9,
          "crew_morale_impact": -0.2,
          "battle_duration_modifier": 1.3,
          "escape_chance_bonus": 1.4
        }
      }
    },
    "dynamic_weather_events": {
      "weather_change_during_battle": {
        "trigger_conditions": ["battle_duration > 15_minutes"],
        "possible_changes": ["storm_intensifies", "wind_shifts", "fog_rolls_in"],
        "tactical_implications": [
          "formation_disruption",
          "visibility_reduction", 
          "escape_opportunities"
        ]
      }
    }
  }
}
```

### Economic Consequence Engine
```cpp
// Integration with Medieval III economic systems
class EconomicConsequenceProcessor {
public:
    struct BattleEconomicImpact {
        TradeLossAssessment trade_losses;
        ReputationChanges reputation_impact;
        MarketEffects market_disruption;
        DiplomaticCosts diplomatic_consequences;
        InsurancePayouts insurance_claims;
    };
    
    BattleEconomicImpact CalculateEconomicImpact(
        const HybridBattleResult& battle_result,
        const EconomicContext& pre_battle_context) {
        
        BattleEconomicImpact impact;
        
        // Calculate direct trade losses
        impact.trade_losses = CalculateDirectTradeLosses(battle_result);
        
        // Assess reputation changes based on battle outcome
        impact.reputation_impact = CalculateReputationChanges(battle_result);
        
        // Determine market effects of cargo losses/captures
        impact.market_disruption = CalculateMarketEffects(battle_result);
        
        // Process diplomatic consequences
        impact.diplomatic_consequences = CalculateDiplomaticCosts(battle_result);
        
        // Handle insurance and banking implications
        impact.insurance_claims = ProcessInsuranceClaims(battle_result);
        
        return impact;
    }
    
private:
    TradeLossAssessment CalculateDirectTradeLosses(const HybridBattleResult& result) {
        TradeLossAssessment assessment;
        
        for (const auto& cargo_loss : result.cargo_outcomes.lost_cargo) {
            // Direct monetary loss
            assessment.direct_losses += cargo_loss.value;
            
            // Calculate opportunity cost
            auto trade_route = FindTradeRoute(cargo_loss.origin, cargo_loss.destination);
            if (trade_route.has_value()) {
                float profit_margin = trade_route->GetProfitMargin();
                assessment.opportunity_cost += cargo_loss.value * profit_margin;
            }
            
            // Regional scarcity impact
            if (IsEssentialGood(cargo_loss.type)) {
                auto scarcity_impact = CalculateScarcityImpact(cargo_loss);
                assessment.regional_price_increases += scarcity_impact.price_increase;
                assessment.affected_regions.insert(cargo_loss.destination);
            }
        }
        
        // Calculate insurance premium increases
        assessment.insurance_premium_increase = 
            assessment.direct_losses * RISK_ASSESSMENT_MULTIPLIER;
            
        return assessment;
    }
    
    ReputationChanges CalculateReputationChanges(const HybridBattleResult& result) {
        ReputationChanges changes;
        
        // Base reputation change from battle outcome
        if (result.victory_status == DECISIVE_VICTORY) {
            changes.naval_reputation += 15;
            changes.trade_protection_reputation += 10;
        } else if (result.victory_status == DEFEAT) {
            changes.naval_reputation -= 20;
            changes.trade_protection_reputation -= 15;
        }
        
        // Additional reputation factors
        for (const auto& cargo_saved : result.cargo_outcomes.preserved_cargo) {
            if (cargo_saved.owner != result.player_faction) {
                // Protecting other faction's cargo improves diplomatic relations
                changes.diplomatic_reputation[cargo_saved.owner] += 5;
            }
        }
        
        // Piracy suppression reputation
        if (result.battle_context.enemy_type == PIRATES) {
            changes.anti_piracy_reputation += 10;
            changes.trade_security_reputation += 8;
        }
        
        return changes;
    }
    
    MarketEffects CalculateMarketEffects(const HybridBattleResult& result) {
        MarketEffects effects;
        
        std::unordered_map<TradeGoodType, int> supply_changes;
        
        // Calculate supply disruption
        for (const auto& cargo_loss : result.cargo_outcomes.lost_cargo) {
            supply_changes[cargo_loss.type] -= cargo_loss.quantity;
        }
        
        // Calculate regional price effects
        for (const auto& [good_type, quantity_change] : supply_changes) {
            if (quantity_change < 0) { // Supply decreased
                auto affected_regions = GetRegionsSuppliedBy(good_type, result.battle_location);
                
                for (const auto& region : affected_regions) {
                    float price_increase = CalculatePriceIncrease(
                        good_type, abs(quantity_change), region.demand);
                    
                    effects.regional_price_changes[region.id][good_type] = price_increase;
                    
                    // Cascade effects to related goods
                    auto related_goods = GetRelatedTradeGoods(good_type);
                    for (const auto& related : related_goods) {
                        effects.regional_price_changes[region.id][related] += 
                            price_increase * RELATED_GOOD_CORRELATION;
                    }
                }
            }
        }
        
        return effects;
    }
};
```

This comprehensive Hybrid Naval Battle System specification provides a complete technical framework for integrating Empire Total War's advanced naval combat into Medieval III Total War while maintaining historical authenticity and seamless campaign integration. The system respects the limitations of both engines while maximizing the strategic and tactical depth available to players.