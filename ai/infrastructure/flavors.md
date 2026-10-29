# Flavor production, propagation and consumption

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

Which flavor value does a particular consumer actually read?

Personality, active player state, recipient-local latest state, grand-strategy transformations and object coefficients are distinct signals. Callback invocation does not prove the consumer reads the layer that triggered it.

## Reference synthesis

The stable distinction is the path, not the FLAVOR_X name. Base personality is initialized from leader/minor/default data and can be randomized; active player state and city/latest recipients receive different mutations; grand-strategy reads are a parallel transformation. Object rows are coefficients joining a consumer's chosen runtime signal.

Economic/military player deltas, economic/military city deltas, city strategies and city specialization must be traced to the actual recipient. Stored signed deltas, getter clamps, reversible start/end changes, initialization/new-city replay and serialized caches are separate contracts. Direct belief personality reads and policy recomputation can bypass a recipient's propagated latest values. Dynamic flavor proposals below are design hypotheses, not implemented signals.

Canonical related owners: [Strategic pressure and manager outputs](<../strategies/managers.md>), [Unit production and recommendation](<../units/production.md>), [Belief selection](<../beliefs/selection.md>), [Policies, branches, tenets and ideology](<../policies/selection.md>), [Shared weighted selection and seeded draws](<weighted-selection.md>), [Database loading, options and effective-data provenance](<../../architecture/database-loading.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- player init registers recipients; `CvEconomicAI::UpdateEconomicStrategies` and `CvMilitaryAI::UpdateMilitaryStrategies` start/end strategy deltas; city strategies and specializations adjust city-local flavors; `CvFlavorRecipient::ChangeFlavors` and `SetFlavors` call virtual `FlavorUpdate`.

Decision role: Flavor manager classes are not option selectors. They propagate personality, strategy, and city-local flavor values into specialist score caches used by policy, tech, desired-wonder, and city production specialists.

- Decision/process: flavor propagation is not a final candidate selector. It is the runtime process that loads database flavor rows, initializes personality flavor vectors, applies strategy/city/specialization deltas, and rebuilds recipient-local scoring caches for later AI decisions.

- core ownership is `CvFlavorManager` and `CvFlavorRecipient` in `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp` / `CvFlavorManager.h`. Main inspected recipients are `CvPlayerTechs`, `CvPlayerPolicies`, `CvWonderProductionAI`, and `CvCityStrategyAI`; city production sub-caches are owned by `CvBuildingProductionAI`, `CvUnitProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`.

- `CvPlayer::init(...)` registers player-level recipients with `CvFlavorManager::AddFlavorRecipient(...)`; `CvCity::init(...)` registers each city's `CvCityStrategyAI`; `CvPlayer::initCity(...)` calls `CvCityStrategyAI::UpdateFlavorsForNewCity()` after founding. runtime deltas are applied when `CvEconomicAI::UpdateEconomicStrategies()`, `CvMilitaryAI::UpdateMilitaryStrategies()`, `CvCityStrategyAI::DoTurn()`, and `CvCityStrategyAI::SetSpecialization(...)` start or end strategies/specializations.

## Candidates and gates

### `DYNAMIC_FLAVOR_RANGED_ROLE_DEFICIT`

`CvMilitaryAI` already has need/enough ranged strategies in default mode and finer archer/siege/skirmisher strategies under `MOD_AI_UNIT_PRODUCTION`; `CvUnitProductionAI::CheckUnitBuildSanity` already reads those strategy flags.

DESIGN_HYPOTHESIS: Producer: `CvMilitaryAI`, based on ranged/archer/siege/skirmisher ratios and current war posture. Consumer: `CvFlavorManager::ChangeCityFlavors` into city `CvUnitProductionAI::AddFlavorWeights`; optional tech/policy consumers only after calibration. Scope: military-strategy-derived city flavor, temporary until composition recovers.

Expected effect: raise production priority for underrepresented ranged roles before the local sanity layer, making the behavior visible in flavor logs and production base weights.

RISK: Double-counting with existing `MILITARYAISTRATEGY_NEED_RANGED` bonuses and broad `FLAVOR_RANGED` unit rows. Implementation difficulty: small-to-medium DLL plus data rows. Should complement existing filters first; only replace direct `+100` after logging validates equivalent behavior.

### `DYNAMIC_FLAVOR_COUNTER_INTELLIGENCE`

Policy flavor bucketing treats `FLAVOR_ESPIONAGE` as diplomacy value in `CvPolicyAI::WeighPolicy`; city/economic strategy code has espionage flavor threshold references in `CvCityStrategyAI.cpp` and `CvEconomicAI.cpp`; the user example of a leading civ without espionage traits maps to existing espionage flavor concepts.

DESIGN_HYPOTHESIS: Producer: diplomacy/economic strategy layer, based on being a tech/culture/diplomacy leader, spy pressure, lack of espionage trait bonuses, and known rivals. Consumer: policy scoring, building scoring for espionage buildings, possibly tech scoring. Scope: diplomacy-derived or economic-strategy-derived player flavor.

Expected effect: shift leaders toward defensive espionage infrastructure/policies when exposed, without hardcoding each consumer.

### `DYNAMIC_FLAVOR_NAVAL_COMPOSITION_DEFICIT`

Default unit scoring uses recommended navy counts and `FLAVOR_NAVAL`/`FLAVOR_NAVAL_RECON`; `MOD_AI_UNIT_PRODUCTION` adds naval melee/ranged/submarine role strategy helpers and local unit sanity checks (`CvUnitProductionAI.cpp`, `CvMilitaryAI.cpp`).

DESIGN_HYPOTHESIS: Producer: `CvMilitaryAI`, per naval role and water-body relevance. Consumer: city-local unit production flavors for coastal cities only. Scope: military-strategy-derived plus city-local.

Expected effect: coastal cities with relevant naval access favor missing naval roles earlier in base weighting.

RISK: High if implemented as a global player flavor; landlocked cities and irrelevant seas would receive noise. Implementation difficulty: medium DLL change because city/water-body scoping is required. Should complement, not replace, hard naval feasibility and area checks.

### `DYNAMIC_FLAVOR_WONDER_RACE_PRESSURE`

`CvBuildingProductionAI::CheckBuildingBuildSanity` subtracts direct penalties for met rivals constructing the same world wonder, while `CvWonderProductionAI::ChooseWonder` has desired-wonder flavor weights and duration reweighting.

DESIGN_HYPOTHESIS: Producer: wonder/building production layer or diplomacy/intel layer when rivals are building contested wonders. Consumer: desired-wonder scoring and final building scoring. Scope: temporary player/city-local pressure for a specific wonder class, not broad `FLAVOR_WONDER`.

Expected effect: make "race or abandon" pressure explicit and logged.

RISK: This may not fit normal `FlavorTypes` because it is object-specific. A generic `FLAVOR_WONDER` boost would overbuild all wonders. Implementation difficulty: large if done cleanly; likely better as named score component rather than flavor.

### `DYNAMIC_FLAVOR_HAPPINESS_CRISIS_SUPPRESSION`

`CvPlayerPolicies::AddFlavorAsStrategies` already boosts `FLAVOR_HAPPINESS` when happiness is below current unhappiness, and `(2) Vox Populi/Database Changes/AI/StrategyChanges.sql` includes a negative growth city flavor for critical happiness need.

DESIGN_HYPOTHESIS: Producer: economic strategy happiness crisis. Consumer: city production, policy, tech, and maybe belief scoring. Scope: economic-strategy-derived player/city flavor.

Expected effect: unify positive happiness seeking and growth suppression across systems.

RISK: Could over-suppress growth if city-local food/poverty logic already handles the problem. Implementation difficulty: data-only for existing strategy flavor rows where consumers already listen; DLL change only if belief scoring should read latest active flavors.

These are small, source-grounded suggestions, not redesigns.

1. Proposed name: `FLAVOR_UNIT_ESCORT`
   - Decision problem: settler escort preference is currently hardcoded around `UNITAI_DEFENSE`, `UNITAI_COUNTER`, and `UNITAI_ATTACK` in `CvUnitProductionAI::CheckUnitBuildSanity`.
   - Consumed by: `CvUnitProductionAI::CheckUnitBuildSanity` in the settler-on-city-plot escort boost.
   - Level: strategy-derived or city-local; it should react to expansion pressure and local danger more than fixed leader personality.
   - Existing path suggesting need: the source already has a special escort boost and a `MOD_AI_UNIT_PRODUCTION` exception for counter units.
   - Should not collapse into: `FLAVOR_DEFENSE`, because city/empire defense and settler escort are different local needs.
   - Risk: medium double-counting with `FLAVOR_EXPANSION`, danger, and settler-skipped counters.
   - Implementation: likely DLL change plus data rows, because no current generic flavor hook isolates escort scoring.

2. Proposed name: `FLAVOR_AIR_DEFENSE_LOCAL`
   - Decision problem: anti-air demand is partly driven by opponent air counts and strategy flags inside `CheckUnitBuildSanity`, not by a city-local flavor.
   - Consumed by: `CvUnitProductionAI::CheckUnitBuildSanity` around `UNITAI_COUNTER`, `UNITAI_DEFENSE_AIR`, and opponent-air scoring.
   - Level: city-local or strategy-derived.
   - Existing path suggesting need: `MOD_AI_UNIT_PRODUCTION` comments treat `UNITAI_COUNTER` as AA-only, while default flavor data uses broader defense/ranged/mobile rows.
   - Should not collapse into: `FLAVOR_ANTIAIR`, because that is leader/object preference, not local exposure.
   - Risk: high overfitting if it duplicates tactical opponent-air scoring.
   - Implementation: DLL change likely; data-only rows cannot currently express local air threat.

3. Proposed name: `FLAVOR_NAVAL_ROLE_BALANCE`
   - Decision problem: default naval production uses broad `FLAVOR_NAVAL`/`FLAVOR_NAVAL_RECON` plus hardcoded recommended navy counts; the mod attempts role splits for naval melee/ranged/submarine.
   - Consumed by: `CvMilitaryAI` need/enough role strategies and `CvUnitProductionAI::CheckUnitBuildSanity`.
   - Level: strategy-derived.
   - Existing path suggesting need: `MOD_AI_UNIT_PRODUCTION` role-specific naval helper functions.
   - Should not collapse into: `FLAVOR_NAVAL`, because broad naval interest does not encode composition.
   - Risk: medium; can double-count with recommended navy size and water-map strategy.
   - Implementation: data-only only if the missing flavor rows and strategy table rows already exist externally; in this repo, robust support needs DLL/data cleanup.

## Scoring and inputs

| Bypass | Decision system | Input used instead of flavor | Evaluation |
|---|---|---|---|
| `CvGameReligions::GetAvailable*Beliefs` and `CvUnit::CanFoundReligion`/`CanEnhanceReligion` (`CvReligionClasses.cpp`, `CvUnit.cpp`) | Belief candidate generation | belief category flags, already-used beliefs, civilization requirements, prophet/holy-city legality, Lua/game events | reasonable legality/candidate filters. RISK converting these to flavor would hide hard rules and mod hooks. |
| `CvReligionAI::ScoreBelief` starting-pantheon special case (`CvReligionClasses.cpp`) | Belief scoring | `CvPlayerTraits::StartsWithPantheon` and `CvBeliefEntry::IsAIGoodStartingPantheon` set scores to `1000` or `1` | special-case trait behavior. Candidate for dynamic flavor only if "starting pantheon" becomes a general early-game preference; otherwise reasonable local heuristic. RISK broad flavor conversion weakens trait-specific intent. |
| `CvReligionAI::CalculatePlotWeightsForBeliefSelection`, `ScoreBeliefAtPlot`, `ScoreBeliefAtCity` (`CvReligionClasses.cpp`) | Belief scoring | plot resources/features/improvements, city population, trade routes, specialists, buildings, holy-city/coastal/river state | local value estimation. Should mostly stay outside flavor; flavor can weight yields, but local eligibility/value is necessary. RISK loss of locality. |
| `CvPolicyAI::DoChooseIdeology` (`CvPolicyAI.cpp`) | Ideology branch choice | vassal/team ideology, relations, public opinion, free tenets, branch building happiness, victory branch mapping | parallel deterministic scoring system. Some diplomacy-derived pressure could feed dynamic flavors, but ideology identity and public-opinion constraints are too specific for generic flavor alone. RISK unintended ideology switches. |
| `CvPolicyAI::WeightPolicyAttributes` (`CvPolicyAI.cpp`) | Policy/tenet scoring | direct policy effects, city/map/unit/building/religion/specialist/resource state | effect estimator. Reasonable complement to flavors; replacing it with flavors would lose policy-specific effect valuation. |
| `CvBuildingProductionAI::CheckBuildingBuildSanity` (`CvBuildingProductionAI.cpp`) | Building/wonder scoring | city danger/falling, best-wonder-city, rival construction, religion availability, policy synergy, city geography, maintenance, local yields | necessary local/situational layer. Some recurring motives can become dynamic city flavors, but hard gates and rival timing should remain local. RISK double-counting with `FLAVOR_WONDER`, defense flavors, and grand-strategy building value. |
| `CvPlayer::SetBestWonderCities` (`CvPlayer.cpp`) | Wonder city assignment | `canConstruct`, puppet/annexable state, turns left, started production, per-wonder city competition | placement/feasibility layer. Bad candidate for generic flavor; it assigns a specific city for a specific building class. |
| `CvCityAI::AI_chooseProduction` (`CvCityAI.cpp`) | Production selection | preserve spaceship parts/current wonders/buildings; force desired wonder with engineer | production-control heuristic. Should not be replaced by flavor; it protects queues and one-off wonder timing. |
| `CvCity::GetUnitForOperation` and `CvMilitaryAI::GetUnitTypeForArmy` (`CvCity.cpp`, `CvMilitaryAI.cpp`) | Unit candidate injection | operation slots, formation primary/secondary `UnitAITypes`, muster landmass/ocean access, army open slots | role/operation requirement. A flavor boost can prioritize role types, but operation legality and slot filling must stay explicit. |
| `CvUnitProductionAI::CheckUnitBuildSanity` (`CvUnitProductionAI.cpp`) | Unit scoring/filtering | supply caps, maintenance, bankruptcy, valid placement plots, garrisons, recommended army/navy/explorer counts, resources, war state, tactical zone, city siege, city buildings | broad parallel heuristic layer. Some count-based pressure is candidate for dynamic flavor integration; hard caps and local feasibility should remain outside. RISK severe double-counting and invalid local choices. |
| `CvDiplomacyAI` victory helpers used by policy/building/unit scoring (`CvPolicyAI.cpp`, `CvBuildingProductionAI.cpp`, `CvUnitProductionAI.cpp`) | Multiple scoring systems | `IsGoingFor*Victory`, `IsCloseTo*Victory`, proximity/approach/war state | direct strategic state. Candidate for producer-side flavor deltas where it reflects durable orientation; immediate relationship facts should remain direct. |
| `MOD_AI_UNIT_PRODUCTION` role need/enough checks (`CvMilitaryAI.cpp`, `CvUnitProductionAI.cpp`) | Unit role composition | explicit unit role counts and strategy flags; new flavor names in helper thresholds | partial flavor extension plus direct filters. Candidate for cleaner dynamic flavor integration, but audited repository data is underwired. RISK missing flavor rows and opaque hard filters. |

INFERRED: These bypasses are not all flaws. Many are boundaries between preference, legality, and tactical feasibility. The useful architectural target is not "make every input a flavor"; it is "make durable, cross-system preferences explicit as dynamic flavors, while keeping local facts local."

- `CvFlavorManager::GetPersonalityIndividualFlavor` reads randomized base personality.

- `CvFlavorRecipient::GetLatestFlavorValue` reads recipient-local latest flavors after strategy and specialization deltas.

Scoring shape: `CvFlavorRecipient::SetFlavors` replaces a recipient's latest flavor vector and calls virtual `FlavorUpdate`. `CvFlavorRecipient::ChangeFlavors` adds deltas without clamping stored negative values, then calls `FlavorUpdate`; `GetLatestFlavorValue(eFlavor, false)` clamps negative reads to zero. Specialist `FlavorUpdate` implementations rebuild cached weights, such as `CvWonderProductionAI::FlavorUpdate`, `CvPlayerPolicies::FlavorUpdate`, and `CvCityStrategyAI::FlavorUpdate`.

- Beliefs: belief candidate generation and scoring bypass generic flavor propagation for hard facts such as belief category flags, already-used beliefs, civilization requirements, plot/resource/improvement values, city state, religion state, neighbor/warmonger checks, and `CvPlayerTraits::StartsWithPantheon` / `CvBeliefEntry::IsAIGoodStartingPantheon` special scoring.

- Policies: policy selection bypasses propagated latest values in several ways: `CvPlayerPolicies::CanAdoptPolicy(...)` / `CanUnlockPolicyBranch(...)` legality, branch continuation and prerequisites, `CvPolicyAI::WeightPolicyAttributes(...)` direct effect valuation, ideology choice/switch logic, situational gold/happiness/war boosts in `AddFlavorAsStrategies(...)`, and hardcoded flavor-name buckets in `WeighPolicy(...)`.

- Wonders/buildings: city production directly gates/scales buildings with `CvCity::canConstruct(...)`, danger/falling-city checks, population/capital checks, best-wonder-city checks, rival construction, religion usefulness, policy synergies, local geography/yields, maintenance, duration reweighting, current-production inertia, and seeded final selection in `CvCityStrategyAI::ChooseProduction(...)`.

- Units: unit production directly uses `CvCity::canTrain(...)`, operation slots from `CvCity::GetUnitForOperation()`, army slots from `CvMilitaryAI::GetUnitTypeForArmy(...)`, supply and maintenance limits, under-siege/underdeveloped city rules, valid placement plots, strategic resources, recommended army/navy/explorer counts, tactical zone, role counts, traits, beliefs, policies, diplomacy, duration, and current-production inertia.

## Data and configuration

- Flavor type names: database table `Flavors`; copied into `GC.getFlavorTypes()` by `CvDllDatabaseUtility::PrefetchGlobalDefines` in `CvGameCoreDLL_Expansion2/CvDllDatabaseUtility.cpp`.
- General loader helper: `CvDatabaseUtility::SetFlavors` in `CvGameCoreDLL_Expansion2/CvDatabaseUtility.cpp`, which reads `FlavorType` and `Flavor` columns keyed through the `Flavors` table.
- Leader/personality flavors: `CvLeaderHeadInfo::CacheResults` loads `Leader_Flavors` in `CvGameCoreDLL_Expansion2/CvInfos.cpp`; VP rewrites many rows in `(2) Vox Populi/Database Changes/AI/LeaderFlavorSweeps.sql`.
- Minor-civ flavors: `CvMinorCivInfo::CacheResults` loads `MinorCivilization_Flavors` in `CvGameCoreDLL_Expansion2/CvMinorCivAI.cpp`.
- Unit flavors: `CvUnitEntry::CacheResults` loads `Unit_Flavors` in `CvGameCoreDLL_Expansion2/CvUnitClasses.cpp`; VP rewrites them in `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql`.
- Building/wonder flavors: `CvBuildingEntry::CacheResults` loads `Building_Flavors` in `CvGameCoreDLL_Expansion2/CvBuildingClasses.cpp`; VP rewrites them in `(2) Vox Populi/Database Changes/AI/BuildingFlavorSweeps.sql`.
- Policy flavors: `CvPolicyEntry::CacheResults` loads `Policy_Flavors` in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`; VP rewrites them in `(2) Vox Populi/Database Changes/AI/PolicyFlavorSweeps.sql`.
- Tech flavors: `CvTechEntry::CacheResults` loads `Technology_Flavors` in `CvGameCoreDLL_Expansion2/CvTechClasses.cpp`; VP rewrites them in `(2) Vox Populi/Database Changes/AI/TechFlavorSweeps.sql`.
- Strategy/specialization flavor deltas: `CvEconomicAIStrategyXMLEntry::CacheResults`, `CvMilitaryAIStrategyXMLEntry::CacheResults`, `CvAICityStrategyXMLEntry::CacheResults`, and `CvCitySpecializationXMLEntry::CacheResults` load their `*_Flavors` tables in `CvEconomicAI.cpp`, `CvMilitaryAI.cpp`, `CvCityStrategyAI.cpp`, and `CvCitySpecializationAI.cpp`.

INFERRED: Base Civ V XML/schema files defining the original `Flavors` table and some default rows are not fully represented in the VP mod folders. Current DLL behavior still proves that `Flavors` is database-driven because the DLL counts and copies the table at runtime.

- Database to enum-name cache: `CvDllDatabaseUtility::PrefetchGlobalDefines` sets `GC.setNumFlavorTypes(kUtility.MaxRows("Flavors"))`, allocates `CvString[iNumFlavors]`, and fills each ID with `Flavors.Type`.
- Object flavor arrays: each `Cv*Entry::CacheResults` uses `CvDatabaseUtility::SetFlavors`, storing an integer array indexed by `FlavorTypes`.
- Player flavor manager: `CvPlayer::init` initializes `m_pFlavorManager` and registers non-city recipients `m_pPlayerTechs`, `m_pPlayerPolicies`, and `m_pWonderProductionAI`.
- City flavor recipient: `CvCity::init` registers `m_pCityStrategyAI` with the owner `CvFlavorManager`; city deserialization re-adds it without propagating values.
- Savegame state: `CvFlavorManager::Serialize` stores `m_piPersonalityFlavor` and `m_piActiveFlavor`; `CvFlavorRecipient` latest values are serialized by recipients such as `CvPlayerTechs`, `CvPlayerPolicies`, and `CvCityStrategyAI`.

`CvFlavorManager::Init` fills `m_piPersonalityFlavor` from leader flavors for non-human major civs, from minor-civ flavors for minors, and from `DEFAULT_FLAVOR_VALUE` for human AI-diplomacy players. Non-zero AI personality flavors are randomized by `CvFlavorManager::RandomizeWeights` using `FLAVOR_RANDOMIZATION_RANGE`, `PERSONALITY_FLAVOR_MIN_VALUE`, and `PERSONALITY_FLAVOR_MAX_VALUE`.

`CvFlavorManager::AdjustWeightsForMap` modifies only `FLAVOR_EXPANSION` and `FLAVOR_GROWTH` based on land tiles per major civ, then calls `ResetToBasePersonality`.

Flavor type identity is database-driven at runtime but hardcoded by string at many call sites. The enum is open (`FlavorTypes` has no compiled list except `NO_FLAVOR`), but C++ frequently calls `GC.getInfoTypeForString("FLAVOR_*")` or compares `GC.getFlavorTypes(eFlavor)` to literal strings.

Object-to-flavor values are database-driven. The DLL does not hardcode unit/building/policy flavor values in the inspected production, belief, policy, and wonder paths; it reads object arrays populated from `*_Flavors` tables.

Interpretation is hardcoded. Examples:
- `CvReligionAI::ScoreYieldForReligionTimes100` maps yields to specific flavor names and constants.
- `CvPolicyAI::WeighPolicy` classifies flavor names into conquest, diplomacy, science, culture, or generic buckets by string comparison.
- `CvTechEntry::GetGrandStrategyPriority` uses hardcoded flavor-name groups for diplo, conquest, culture, science, and trait modifiers.
- `CvMilitaryAI` `MOD_AI_UNIT_PRODUCTION` helper functions hardcode role-specific flavor names.

- Macro: `CvGameCoreDLL_Expansion2/CustomMods.h` defines `MOD_AI_UNIT_PRODUCTION` as `gCustomMods.isAI_UNIT_PRODUCTION()`.
- Cache: `CvGameCoreDLL_Expansion2/CustomMods.cpp` uses `MOD_OPT_CACHE(AI_UNIT_PRODUCTION)`.
- Default option: `(1) Community Patch/Database Changes/NewCustomModOptions.xml` sets `AI_UNIT_PRODUCTION` to `0`.

| Folder / data area | Relevant tables and rows | Current-source consumer |
|---|---|---|
| `(1) Community Patch/Database Changes/AI/` | patches `Leader_Flavors`, `AIGrandStrategy_Flavors`, `AIGrandStrategy_FlavorMods`, `AIEconomicStrategy_Player_Flavors`, `AIEconomicStrategy_City_Flavors`, `AIMilitaryStrategy_Player_Flavors`, `AIMilitaryStrategy_City_Flavors`, `AICityStrategy_Flavors`, and `CitySpecialization_Flavors` in files such as `CoreLeaderFlavorChanges.sql`, `CoreStrategyChanges.sql`, and `CoreCitySpecializationChanges.sql`. | `CvLeaderHeadInfo::CacheResults` (`CvGameCoreDLL_Expansion2/CvInfos.cpp`), `CvGrandStrategyAIType::CacheResults` (`CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp`), `CvEconomicAIStrategyXMLEntry::CacheResults`, `CvMilitaryAIStrategyXMLEntry::CacheResults`, `CvAICityStrategyEntry::CacheResults`, and `CvCitySpecializationXMLEntry::CacheResults`. |

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvMinorCivInfo::CacheResults` (`CvGameCoreDLL_Expansion2/CvMinorCivAI.cpp`) | `MinorCivilization_Flavors` | Minor-civ personality flavors in `CvFlavorManager::Init`. |

- Flavor IDs/names: `FlavorTypes` is an open enum with only `NO_FLAVOR = -1` in `CvGameCoreDLLUtil/include/CvEnums.h`; `CvDllDatabaseUtility::PrefetchGlobalDefines()` in `CvGameCoreDLL_Expansion2/CvDllDatabaseUtility.cpp` sets `GC.setNumFlavorTypes(kUtility.MaxRows("Flavors"))` and fills `GC.getFlavorTypes()` from database table `Flavors`.

- Generic loader: `CvDatabaseUtility::SetFlavors(...)` in `CvGameCoreDLL_Expansion2/CvDatabaseUtility.cpp` populates arrays by joining the caller's `FlavorType` column through `Flavors` and reading `Flavor`.

- Personality sources: `CvLeaderHeadInfo::CacheResults(...)` loads `Leader_Flavors` in `CvGameCoreDLL_Expansion2/CvInfos.cpp`; `CvMinorCivInfo::CacheResults(...)` loads `MinorCivilization_Flavors` in `CvGameCoreDLL_Expansion2/CvMinorCivAI.cpp`. Relevant source-data maps include `(1) Community Patch/Database Changes/AI/LeaderTableChanges.sql`, `(1) Community Patch/Database Changes/AI/CoreLeaderFlavorChanges.sql`, and `(2) Vox Populi/Database Changes/AI/LeaderFlavorSweeps.sql`.

- Object sources: object flavor rows load through `CvUnitEntry::CacheResults(...)` from `Unit_Flavors`, `CvBuildingEntry::CacheResults(...)` from `Building_Flavors`, `CvPolicyEntry::CacheResults(...)` from `Policy_Flavors`, `CvTechEntry::CacheResults(...)` from `Technology_Flavors`, `CvProjectEntry::CacheResults(...)` from `Project_Flavors`, and `CvProcessInfo::CacheResults(...)` from `Process_Flavors`. Relevant VP maps include `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql`, `BuildingFlavorSweeps.sql`, `PolicyFlavorSweeps.sql`, `TechFlavorSweeps.sql`, `ProjectFlavorChanges.sql`, and `ProcessFlavorSweeps.sql`.

- Strategy/specialization sources: `CvEconomicAIStrategyXMLEntry::CacheResults(...)` loads `AIEconomicStrategy_Player_Flavors` and `AIEconomicStrategy_City_Flavors`; `CvMilitaryAIStrategyXMLEntry::CacheResults(...)` loads `AIMilitaryStrategy_Player_Flavors` and `AIMilitaryStrategy_City_Flavors`; `CvAICityStrategyXMLEntry::CacheResults(...)` loads `AICityStrategy_Flavors`; `CvCitySpecializationXMLEntry::CacheResults(...)` loads `CitySpecialization_Flavors`. Relevant source-data maps include `(1) Community Patch/Database Changes/AI/CoreStrategyChanges.sql`, `(1) Community Patch/Database Changes/AI/CoreCitySpecializationChanges.sql`, `(2) Vox Populi/Database Changes/AI/StrategyChanges.sql`, `(2) Vox Populi/Database Changes/AI/StrategyFlavorSweeps.sql`, and `(2) Vox Populi/Database Changes/AI/CitySpecializationFlavorChanges.sql`.

- Beliefs: source search found no `Belief_Flavors` loader or `CvBeliefEntry` flavor cache in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`. Belief AI uses personality flavors directly; belief object rows are not part of the generic object-flavor cache path.

- Personality initialization: `CvFlavorManager::Init(CvPlayer*)` copies leader flavors for major AI players, minor-civ flavors for minors, and `DEFAULT_FLAVOR_VALUE` for human AI-diplomacy players. Non-zero AI personality values are randomized by `CvFlavorManager::RandomizeWeights()` using `FLAVOR_RANDOMIZATION_RANGE`, `PERSONALITY_FLAVOR_MIN_VALUE`, `PERSONALITY_FLAVOR_MAX_VALUE`, and seed `0xe655df8f` mixed with player ID and flavor index.

- Base broadcast: after initialization, `CvFlavorManager::BroadcastBaseFlavors()` calls `CvFlavorRecipient::SetFlavors(...)` on registered recipients, and `CvFlavorManager::ResetToBasePersonality()` copies personality values to `m_piActiveFlavor`.

- Recipient registration: `CvPlayer::init(...)` registers `m_pPlayerTechs`, `m_pPlayerPolicies`, and `m_pWonderProductionAI`; `CvCity::init(...)` registers `m_pCityStrategyAI`. Load paths re-register with propagation disabled: `CvPlayer::Serialize(...)` and `CvCity::Serialize(...)` call `AddFlavorRecipient(..., false)`.

- Save/load state: `CvFlavorManager::Serialize(...)` stores `m_piPersonalityFlavor` and `m_piActiveFlavor`; recipient latest values are stored by recipient serializers such as `CvCityStrategyAI::Serialize(...)`, `CvPlayerPolicies::Serialize(...)`, and production AI serializers reached from those classes. RISK changing array size/order or recipient serialization affects save compatibility.

- Map adjustment: `CvFlavorManager::AdjustWeightsForMap()` changes `FLAVOR_EXPANSION` and `FLAVOR_GROWTH` based on land tiles per major civ, then calls `ResetToBasePersonality()`.

## Observability and validation obligations

Logging: `CvFlavorRecipient::SetFlavors` and `ChangeFlavors` call virtual `LogFlavorChange`. `CvCityStrategyAI::LogFlavorChange` logs city flavor deltas used by production caches. `CvFlavorManager::ChangeActivePersonalityFlavors` calls `LogActivePersonalityChange`.

- `CvFlavorManager::LogActivePersonalityChange(...)` writes player active-flavor changes to `FlavorAILog.csv` or split player logs; `CvCityStrategyAI::LogFlavorChange(...)` logs city flavor deltas. `CvPlayerPolicies::LogFlavorChange(...)` is intentionally empty in audited source.

## Version differences, risks and unresolved questions

### `DYNAMIC_FLAVOR_COUNTER_INTELLIGENCE`

RISK: OPEN_QUESTION because current spy/counter-intelligence production and policy paths need a focused source pass before implementation. Risk of double-counting with existing espionage strategy thresholds and policy flavor rows. Implementation difficulty: medium DLL change.

1. / OPEN_QUESTION: Build a complete merged database flavor inventory: `Flavors`, all `*_Flavors` tables, and missing `MOD_AI_UNIT_PRODUCTION` role flavor names. Expected value: highest, because C++ references currently cannot be calibrated without data.

2. / OPEN_QUESTION: Trace every producer of `CvFlavorManager::ChangeActivePersonalityFlavors`, `ChangeCityFlavors`, and direct `CvCityStrategyAI` flavor changes, including logs and lifetimes. Expected value: high, because dynamic flavor architecture depends on producer clarity.

4. OPEN_QUESTION: Investigate espionage/counter-intelligence production, policy, tech, and diplomacy paths before adding any dynamic espionage flavor. Expected value: high because it tests the user's example against source.

7. OPEN_QUESTION: Review whether belief scoring should ever read active/latest dynamic flavors instead of only personality flavors and grand-strategy priorities. Expected value: medium; belief timing is infrequent and changing it may have large strategic effects.

Patch-risk note: Adding a new flavor row is data-visible to generic weighted caches, but it has no semantic meaning unless object rows use it and some consumer either generically multiplies it or hardcoded logic interprets its name.

Safe-ish patch points:
- Data-only changes to existing `*_Flavors` rows for units/buildings/policies/techs, if the target flavor already exists and consumer semantics are understood.
- Small scoring constants in local sanity functions after verifying candidate generation and selection are unchanged.
- Documentation/logging around `FlavorUpdate`, `AddFlavorWeights`, and strategy activation paths.

Dangerous patch points:
- Reordering or redefining `FlavorTypes`/database `Flavors.ID`: open enum indices are used across cached arrays and saves.
- `CvFlavorManager::Serialize`, recipient latest-flavor serialization, or `CvEnumMap<FlavorTypes, int>` layout: savegame-sensitive.
- `CvFlavorManager::ChangeActivePersonalityFlavors`, `ChangeCityFlavors`, and `CvFlavorRecipient::ChangeFlavors`: temporary strategy deltas rely on exact add/remove symmetry.
- `CvCityStrategyAI::FlavorUpdate`: affects buildings, units, projects, and processes at once.
- `CvUnitProductionAI::CheckUnitBuildSanity` and `CvBuildingProductionAI::CheckBuildingBuildSanity`: shared by normal production and helper recommendation paths.
- Hardcoded string comparisons in `CvPolicyAI`, `CvTechClasses`, `CvGrandStrategyAI`, and `CvMilitaryAI`: typo or missing database row can silently produce inert logic or `NO_FLAVOR` access risk.
- `MOD_AI_UNIT_PRODUCTION` role flavors: current repo lacks matching flavor data rows, so enabling or extending this path needs database verification first.

INFERRED: The base game's original `Flavors` table rows are not enumerated in this pass because the repo search did not expose a complete base XML definition. The DLL loading path is verified, but a complete flavor-name inventory should be generated from a built database or full merged XML if exact row IDs are needed.

INFERRED: The exact `AIMilitaryStrategy_*_Flavors` rows for `MOD_AI_UNIT_PRODUCTION` role strategies were not found in repository XML/SQL. If those strategies are supplied by an external modmod or generated database, the appendix should be updated from that merged database.

INFERRED: Tech AI and event/advisor flavor consumers were only mapped where they intersected the requested systems. A separate scoring-infrastructure doc should cover `CvTechAI`, `CvAdvisorRecommender`, `CvPlayerAI` event choice, contracts, builder tasking, and site evaluation in the same level of detail.

INFERRED: Some flavor-name comparisons duplicate or contain legacy quirks, such as `CvPolicyAI::WeighPolicy` checking `FLAVOR_I_LAND_TRADE_ROUTE` twice and `FLAVOR_I_SEA_TRADE_ROUTE` twice. This document records the pattern but does not evaluate every typo-like branch.

`CvFlavorManager::ChangeActivePersonalityFlavors`, `CvFlavorManager::ChangeCityFlavors`, `CvFlavorRecipient::ChangeFlavors`, and recipient `FlavorUpdate` methods are shared flavor propagation hubs in `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp` and recipient classes.

- RISK `CvFlavorManager` / `CvFlavorRecipient` propagation: `FlavorTypes` is database-backed and array-indexed; active personality, city flavor, recipient latest values, and production AI caches are serialized. Negative deltas require exact start/end symmetry.

Patch-risk notes: RISK: `FlavorTypes` is an open database-backed enum in `CvGameCoreDLLUtil/include/CvEnums.h`; array indices, serialized `CvFlavorManager` personality/active values, recipient latest values, and add/remove delta symmetry are savegame-sensitive. STALE_OR_MISMATCH: Several C++ flavor-name checks for role flavors are present, but repository XML/SQL does not define all of those names; do not assume new flavor names have active data without merged database verification.

- OPEN_QUESTION / DESIGN_HYPOTHESIS: many bypasses are intentional boundaries between preference and feasibility. Durable medium-term pressure can fit flavor deltas; legality, one-turn tactical facts, and object-specific races usually need explicit code paths.

- RISK `FlavorTypes` is database-indexed and array-backed. Reordering `Flavors.ID`, changing `GC.getNumFlavorTypes()`, or altering serialized `CvEnumMap<FlavorTypes, int>` state can affect saves and runtime cache indexing.

- RISK negative deltas are legal in `CvFlavorRecipient::ChangeFlavors(...)` and repository data includes negative city deltas such as `(2) Vox Populi/Database Changes/AI/StrategyChanges.sql` setting `AIEconomicStrategy_City_Flavors` `FLAVOR_GROWTH = -10` for `ECONOMICAISTRATEGY_NEED_HAPPINESS_CRITICAL`. Exact start/end symmetry matters.

- RISK `CvCityStrategyAI::FlavorUpdate()` rebuilds building, unit, project, and process base weights together; changing a city flavor producer can shift the whole buildable competition, not only one object class.

- RISK hardcoded flavor-name comparisons in `CvReligionAI`, `CvPolicyAI`, `CvTechEntry::GetGrandStrategyPriority(...)`, `CvMilitaryAI`, and related helpers silently depend on database row names existing.

- STALE_OR_MISMATCH / RISK `MOD_AI_UNIT_PRODUCTION` role flavor names documented in existing flavor docs remain source-referenced, but this repository's default custom-mod option disables the gated behavior and repository searches did not find complete matching flavor rows for all role-split names.

## Source and dependency lookup

Beliefs use flavors as motive/yield multipliers, not as object flavor rows. `CvReligionAI::ScoreYieldForReligionTimes100` maps yields to personality flavors such as `FLAVOR_GROWTH`, `FLAVOR_PRODUCTION`, `FLAVOR_GOLD`, `FLAVOR_SCIENCE`, `FLAVOR_CULTURE`, and `FLAVOR_RELIGION`; `ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, and `ScorePantheonBeliefForPlayer` read additional personality flavors (`CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`). Belief candidate generation and final application are separate from flavor.

Policies are one of the strongest flavor integrations. `CvPlayerPolicies::AddFlavorAsStrategies` chooses personality or personality-plus-grand-strategy flavor values, adds situational boosts, and calls `CvPolicyAI::AddFlavorWeights`; `CvPolicyAI::WeighPolicy` then adds grand-strategy/victory logic, flavor-category bucketing, policy-attribute estimates, branch continuation, and tenet/finisher modifiers (`CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, `CvPolicyAI.cpp`).

Wonders/buildings use flavors twice. `CvWonderProductionAI::AddFlavorWeights` creates a desired-wonder cache from building flavor rows, while final city production uses `CvBuildingProductionAI::AddFlavorWeights` through `CvCityStrategyAI::FlavorUpdate`. `CvBuildingProductionAI::CheckBuildingBuildSanity` then adds best-city, desired-wonder, religion, policy, war, threat, geography, yield, and duration logic (`CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`, `CvBuildingProductionAI.cpp`).

Unit production uses flavors as base weights. `CvCityStrategyAI::FlavorUpdate` sends city-local latest flavors into `CvUnitProductionAI::AddFlavorWeights`, which stores `unit flavor * city flavor` in `m_UnitAIWeights`. `CvUnitProductionAI::CheckUnitBuildSanity` then converts the base weight with `sqrti(10 * iFlavorWeight)` and applies many direct constraints and bonuses (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvUnitProductionAI.cpp`).

Grand/economic/military/city strategies already act as dynamic flavor producers. `CvEconomicAI` and `CvMilitaryAI` call `CvFlavorManager::ChangeActivePersonalityFlavors` and `ChangeCityFlavors` when strategies start/end; city strategies and specializations directly adjust `CvCityStrategyAI::m_piLatestFlavorValues`; `CvGrandStrategyAI::GetPersonalityAndGrandStrategy` combines personality with active grand-strategy flavor mods (`CvEconomicAI.cpp`, `CvMilitaryAI.cpp`, `CvCityStrategyAI.cpp`, `CvGrandStrategyAI.cpp`).

Personality/latest flavor split is hard to reason about. Systems using `CvFlavorManager::GetPersonalityIndividualFlavor` ignore active strategy deltas, while recipients using `CvFlavorRecipient::GetLatestFlavorValue` see propagated deltas. Belief scoring reads personality flavors directly, so economic/military/city flavor deltas do not naturally affect belief choices (`CvFlavorManager.cpp`, `CvReligionClasses.cpp`).

Flavor interpretation is hardcoded by string in several systems. `CvPolicyAI::WeighPolicy`, `CvTechEntry::GetGrandStrategyPriority`, and `CvMilitaryAI` role helpers compare literal flavor names. This allows database-driven flavor rows but makes semantic coverage difficult to audit (`CvPolicyAI.cpp`, `CvTechClasses.cpp`, `CvMilitaryAI.cpp`).

Scale and normalization vary by consumer. Religion yield scoring uses constants like `FLAVOR_CULTURE * 100` and `FLAVOR_RELIGION * 110`; unit/building sanity converts raw flavor products with `sqrti(10 * weight)`; policy flavor weights are propagated through prerequisites; tech AI normalizes current flavor values against the largest flavor (`CvReligionClasses.cpp`, `CvUnitProductionAI.cpp`, `CvBuildingProductionAI.cpp`, `CvPolicyAI.cpp`, `CvTechClasses.cpp`).

Some broad flavor names are overloaded. `FLAVOR_OFFENSE`, `FLAVOR_DIPLOMACY`, `FLAVOR_RANGED`, `FLAVOR_WONDER`, and `FLAVOR_NAVAL` are consumed by multiple systems with different meanings, from object base weights to strategy priority buckets to belief scoring and operation boosts.

`MOD_AI_UNIT_PRODUCTION` role flavors are weakly wired in this repository. C++ references `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, and `FLAVOR_FIGHTER`, but repository XML/SQL search did not find definitions or object rows for those flavor names. This makes the extension conceptually flavor-driven but data-incomplete.

INFERRED: Some dynamic strategy changes plausibly should affect beliefs but currently do not through the flavor-recipient path. Example: a player with active conquest or defense pressure can have city production flavors changed by military/economic strategies, but `CvReligionAI` mostly reads personality flavors and grand-strategy priorities directly, not recipient latest values.

INFERRED: Traceability is weak because dynamic flavor changes are logged by `LogFlavorChange` paths, but downstream score logs often show final candidate scores rather than a normalized decomposition of "base flavor product, dynamic delta, local heuristic."

The mod option is disabled by default (`(1) Community Patch/Database Changes/NewCustomModOptions.xml`, `AI_UNIT_PRODUCTION = 0`), so it is not part of the default unit-production pipeline unless enabled externally.

It introduces a more flavor-driven design shape in `CvMilitaryAI.cpp`: role-composition strategy tests use `CvGrandStrategyAI::GetPersonalityAndGrandStrategy` for role flavors such as archer, siege, skirmisher, naval melee/ranged, submarine, bomber, and fighter. This matches the hypothesis that "too few archer-type units could trigger an archer/ranged-unit priority."

It also adds direct local unit-production effects in `CvUnitProductionAI::CheckUnitBuildSanity`: need strategies add `100`, enough strategies can return `SR_BALANCE`, and `UNITAI_COUNTER` is excluded from settler escort scoring when the mod is enabled. These are not pure flavor propagation; they are strategy flags directly altering local scoring/filtering.

RISK: The extension improves role specificity but makes traceability more opaque. Role need/enough checks are in `CvMilitaryAI`, role suppression/boosts are in `CvUnitProductionAI`, role semantics also appear in `CvTacticalAI`, `CvHomelandAI`, and `CvEconomicAI`, and the new flavor names appear underwired in repository data.

RISK: Some added logic can be isolated or duplicated. The new role names are checked in policy and tech flavor categorization (`CvPolicyAI::WeighPolicy`, `CvTechEntry::GetGrandStrategyPriority`), role strategy helpers (`CvMilitaryAI.cpp`), advisor categorization (`CvAdvisorRecommender.cpp`), and unit sanity (`CvUnitProductionAI.cpp`). Without complete data and logging, it is hard to tell which layer is actually driving behavior.

DESIGN_HYPOTHESIS: If revived, `MOD_AI_UNIT_PRODUCTION` should not remain only local to unit production. Its durable role deficits should be expressed as logged military-strategy-derived city flavor deltas, consumed by unit weights and maybe tech/policy priorities. Its immediate "enough role, stop building this now" constraints should remain local `CvUnitProductionAI` filters to prevent overproduction.

Legality/candidate filters should stay outside flavors. Examples: `CvCity::canTrain`, `CvCity::canConstruct`, `CvPlayerPolicies::CanAdoptPolicy`, `CvPlayerPolicies::CanUnlockPolicyBranch`, `CvGameReligions::GetAvailable*Beliefs`, `CvUnit::CanFoundReligion`, and Lua/game-event vetoes.

Hard strategic resource constraints should stay explicit. `CvUnitProductionAI::CheckUnitBuildSanity` handles unavailable resources and spaceship aluminum reservation; `CvUnitProductionAI::RecommendUnit(..., bAllowStrategicResource=false)` rejects strategic-resource units for certain role requests.

Highly local city constraints should stay local. Under-siege noncombat rejection, danger/falling city checks, valid placement plots, garrison state, air slots, puppet restrictions, local water access, best-wonder-city flags, and local production turns are not safely expressible as global flavor.

Immediate tactical emergencies should stay outside broad flavor. `CvTacticalAI` emergency purchase behavior and one-turn city defense needs require local tactical context and should not leak into long-lived personality/state flavors.

DESIGN_HYPOTHESIS: Flavors are best for durable preferences and medium-term pressure, not legality, one-turn tactics, object-specific races, or constraints where wrong abstraction hides why a candidate was impossible.

3. Decompose `CvUnitProductionAI::CheckUnitBuildSanity` into named score components and identify which are hard filters, local heuristics, strategy-derived pressure, or potential dynamic flavor deltas. Expected value: high for unit-production architecture.

5. DESIGN_HYPOTHESIS: Prototype documentation-only "flavor signal registry" before code changes: for each flavor, list producers, consumers, scale, scope, lifetime, and log file. Expected value: high for traceability with low behavior risk.

6. RISK: Audit double-counting cases: `FLAVOR_OFFENSE`, `FLAVOR_RANGED`, `FLAVOR_NAVAL`, `FLAVOR_DIPLOMACY`, `FLAVOR_WONDER`, and `FLAVOR_HAPPINESS`. Expected value: medium-high before adding new dynamic flavor producers.

STALE_OR_MISMATCH: Several `MOD_AI_UNIT_PRODUCTION` flavor names (`FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, `FLAVOR_FIGHTER`) are referenced in C++ but are not defined or assigned in any repository XML/SQL found by source search. If enabled without external data, those `GC.getInfoTypeForString(...)` calls can resolve to `NO_FLAVOR`, so the source intent and repository data appear incomplete on that point.

- `CvFlavorManager::GetPersonalityIndividualFlavor(FlavorTypes)`: raw randomized personality value; used directly by religion scoring, diplomacy, site evaluation, builder tasking, settlement/AI helpers, and some unit logic.
- `CvFlavorManager::GetPersonalityFlavorForDiplomacy(FlavorTypes)`: rescales personality flavor to diplomacy's 1-10 scale.
- `CvFlavorManager::ChangeActivePersonalityFlavors(...)`: applies player-level strategy deltas to non-city flavor recipients only.
- `CvFlavorManager::ChangeCityFlavors(...)`: applies city-level strategy deltas to city recipients only.
- `CvFlavorRecipient::SetFlavors(...)`: replaces a recipient's local flavor vector and immediately calls virtual `FlavorUpdate`.
- `CvFlavorRecipient::ChangeFlavors(...)`: adds deltas and immediately calls `FlavorUpdate`; negative values are stored, but `GetLatestFlavorValue(..., false)` clamps negative reads to zero.
- `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FlavorTypes, bool)`: returns personality plus active grand-strategy flavor mod, optionally blending toward the active grand strategy's main flavor.
- Object APIs: `CvUnitEntry::GetFlavorValue`, `CvBuildingEntry::GetFlavorValue`, `CvPolicyEntry::GetFlavorValue`, `CvTechEntry::GetFlavorValue`, `CvProjectEntry::GetFlavorValue`, `CvProcessInfo::GetFlavorValue`, and similar entry methods read cached object flavor arrays.

| System | Flavor source | Cache/consumer | Selection effect |
|---|---|---|---|
| Tech AI | recipient latest flavors plus grand strategy normalization | `CvPlayerTechs::FlavorUpdate`, `CvTechAI::AddFlavorWeights` | tech weights; outside this doc's primary scope |
| Policy AI | personality before Medieval; personality plus grand strategy after | `CvPlayerPolicies::AddFlavorAsStrategies`, `CvPolicyAI::AddFlavorWeights`, `CvPolicyAI::WeighPolicy` | branch/policy/tenet scores |
| Desired wonder | player flavor recipient latest values | `CvWonderProductionAI::FlavorUpdate`, `CvWonderProductionAI::AddFlavorWeights` | desired wonder candidate weight |
| City buildings/wonders | city flavor recipient latest values | `CvCityStrategyAI::FlavorUpdate`, `CvBuildingProductionAI::AddFlavorWeights` | initial building weight before sanity/duration |
| City units | city flavor recipient latest values | `CvCityStrategyAI::FlavorUpdate`, `CvUnitProductionAI::AddFlavorWeights` | initial unit weight before sanity/duration |
| Projects/processes | city flavor recipient latest values | `CvProjectProductionAI::AddFlavorWeights`, `CvProcessProductionAI::AddFlavorWeights` | initial production candidate weight |
| Beliefs | personality flavors and grand-strategy priorities | `CvReligionAI::ScoreYieldForReligionTimes100`, `ScoreBeliefAtCity`, `ScoreBeliefForPlayer` | belief score components |
| Grand strategy | personality flavors, strategy flavor values/mods | `CvGrandStrategyAI::GetBaseGrandStrategyPriority`, `GetPersonalityAndGrandStrategy` | grand-strategy priority and downstream flavor modulation |
| Economic/military/city strategies | strategy flavor tables | `CvEconomicAI`, `CvMilitaryAI`, `CvCityStrategyAI` activation/end paths | temporary player/city flavor deltas |

Belief choice does not use `CvBeliefEntry` flavor arrays. Instead, `CvReligionAI` directly reads personality flavors and game state in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`.

- `CvReligionAI::ScoreYieldForReligionTimes100` maps `YIELD_FOOD` to `FLAVOR_GROWTH * 50`, production to `FLAVOR_PRODUCTION * 50`, gold to `FLAVOR_GOLD * 50`, science to `FLAVOR_SCIENCE * 80`, culture to `FLAVOR_CULTURE * 100`, and faith to `FLAVOR_RELIGION * 110`.
- `CvReligionAI::ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, and `ScorePantheonBeliefForPlayer` read flavors such as `FLAVOR_GREAT_PEOPLE`, `FLAVOR_OFFENSE`, `FLAVOR_DEFENSE`, `FLAVOR_CITY_DEFENSE`, `FLAVOR_HAPPINESS`, `FLAVOR_DIPLOMACY`, and `FLAVOR_WONDER`.
- `CvReligionAI::ScoreBeliefForPlayer` uses grand-strategy priorities for conquest, culture, diplomacy, and science rather than only raw flavors.

INFERRED: In beliefs, flavors mostly act as yield or motive multipliers. They do not generate candidates and do not apply selected beliefs; candidate generation and application are separate religion paths documented in `docs-ai/ai-beliefs.md`.

`CvPlayerPolicies::FlavorUpdate` delegates to `CvPlayerPolicies::AddFlavorAsStrategies` in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`. That function resets `CvPolicyAI`, reads each flavor, adds situational boosts for losing money, unhappiness, and war, then calls `CvPolicyAI::AddFlavorWeights`.

`CvPolicyAI::AddFlavorWeights` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp` multiplies each policy's `CvPolicyEntry::GetFlavorValue(eFlavor)` by the current flavor weight, propagates some value backward through prerequisites with `WeightPrereqs` and `PropagateWeights`, and stores cached policy weights.

`CvPolicyAI::WeighPolicy` then reinterprets policy flavor rows by string names. Military/naval/air flavors feed conquest, `FLAVOR_DIPLOMACY`/gold/trade/espionage feed diplomacy, `FLAVOR_SCIENCE`/`FLAVOR_SPACESHIP` feed science, `FLAVOR_GREAT_PEOPLE`/`FLAVOR_WONDER`/`FLAVOR_ARCHAEOLOGY` feed culture, and growth/happiness/infrastructure/religion/expansion/production/tile-improvement flavors feed generic weight.

STALE_OR_MISMATCH: `CvPolicyAI::WeighPolicy` includes the new `MOD_AI_UNIT_PRODUCTION` flavor names in always-compiled string comparisons, but this repository does not define those flavor rows in XML/SQL. They are therefore inert unless external data adds them.

Desired-wonder flavor scoring uses `CvWonderProductionAI::FlavorUpdate` and `CvWonderProductionAI::AddFlavorWeights` in `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`. `AddFlavorWeights` only applies to `CvWonderProductionAI::IsWonder(kBuilding)` entries and accumulates `CvBuildingEntry::GetFlavorValue(eFlavor) * iWeight`.

Final city building and wonder production uses `CvCityStrategyAI::FlavorUpdate` and `CvBuildingProductionAI::AddFlavorWeights` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` and `CvBuildingProductionAI.cpp`. The raw flavor sum is only an initial value; `CvBuildingProductionAI::CheckBuildingBuildSanity` converts it with `sqrti(10 * iValue)`, then applies best-wonder-city checks, desired-wonder bonuses, war/threat penalties, religion/policy/yield/geography modifiers, duration reweighting, and final production selection.

INFERRED: `FLAVOR_WONDER` has two meanings in this area: it is both a general leader/object preference used in object flavor products and a city-specialization input in `CvCitySpecializationAI::WeightProductionSubtypes`. These are related but not identical decision layers.

Default unit production has three flavor entry points:
- City flavor vector: `CvCityStrategyAI::FlavorUpdate` sends current city flavors to `CvUnitProductionAI::AddFlavorWeights`.
- Unit database rows: `CvUnitProductionAI::AddFlavorWeights` multiplies each `CvUnitEntry::GetFlavorValue(eFlavor)` by the current city flavor value and stores it in `m_UnitAIWeights`.
- Operation/army wrapper scores: `CvCityStrategyAI::ChooseProduction` adds operation and army buildables using base constants plus `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)` and the recommended unit's flavor weight.

Generic unit candidate generation in `CvCityStrategyAI::ChooseProduction` loops all units, requires `CvCity::canTrain`, and uses `CvUnitProductionAI::GetWeight(eUnit)` as the precheck weight. `CvUnitProductionAI::GetWeight` returns cached flavor weight, except barbarian cities return `1`.

`CvUnitProductionAI::RecommendUnit` filters by `CvUnitEntry::GetUnitAIType(eUnitAIType)`, optional strategic-resource rejection, and `CvCity::canTrain`. If cached flavor weight is zero, it falls back to the unweighted sum of all `CvUnitEntry::GetFlavorValue(iFlavorLoop)` values, then reweights by turns and returns the top unit after stable sort.

`CvUnitProductionAI::CheckUnitBuildSanity` is the main default scoring layer. It rejects nonpositive flavor weight, converts base flavor weight with `sqrti(10 * iFlavorWeight)`, then applies many hard constraints and bonuses from supply, maintenance, war state, unit role, resource availability, city state, strategy flags, traits, religion, diplomacy, underrepresented unit types, current production, and duration.

Flavor inputs in the default pipeline are therefore local base scoring inputs, not final selectors. Candidate generation is mostly `canTrain` plus operation/army needs; flavor affects initial candidate weights and some operation/army boosts; constraints and non-flavor strategy logic can discard or dominate flavor-preferred candidates.

In `CvUnitProductionAI::CheckUnitBuildSanity`, settler escort scoring includes `UNITAI_COUNTER` only when `!MOD_AI_UNIT_PRODUCTION`; with the mod enabled, the escort boost applies to `UNITAI_DEFENSE` and `UNITAI_ATTACK` but not counter units. Classification: scoring change. Scope: local to unit sanity scoring. Coherence: INFERRED coherent with later comments that counter units are treated as anti-air-specialized.

In `CvUnitProductionAI::CheckUnitBuildSanity`, the `MOD_AI_UNIT_PRODUCTION - NEW STRATEGIES` block reads role-specific military strategies for ranged, city bombard, skirmisher, fast attack, attack sea, assault sea, submarine, attack air, and defense air. Need strategies add `100`; enough strategies return `SR_BALANCE` for most roles. Classification: scoring and constraint/filter change. Scope: local sanity result, but strategy activation comes from wider `CvMilitaryAI`.

In `CvMilitaryAI`, default `MILITARYAISTRATEGY_NEED_RANGED` and `ENOUGH_RANGED` tests are disabled when `MOD_AI_UNIT_PRODUCTION` is enabled; the mod enables finer helper tests for archer, siege, skirmisher, naval melee, naval ranged, submarine, bomber, and fighter strategies.

STALE_OR_MISMATCH: Those helper tests call `CvGrandStrategyAI::GetPersonalityAndGrandStrategy((FlavorTypes)GC.getInfoTypeForString("FLAVOR_*"))` for the new role names, but repository data search found no definitions for those flavor names. This makes the mod appendix incomplete in the audited tree unless external database content supplies those rows.

`CvMilitaryAI::IsTestStrategy_EnoughMobileUnits` and `IsTestStrategy_NeedMobileUnits` change ratio logic under `MOD_AI_UNIT_PRODUCTION` from mobile / (melee + mobile) to mobile / melee. Classification: strategy input change that can indirectly affect production through strategy flags.

Related gated role semantics appear outside city production: `CvTacticalAI.cpp` treats counter/AA differently for emergency purchase and formation placement; `CvHomelandAI.cpp` excludes `UNITAI_COUNTER` from sentry suitability; `CvEconomicAI.cpp` allows `UNITAI_SUBMARINE` as a naval explorer only under the mod.

`CvTechEntry::GetGrandStrategyPriority` in `CvGameCoreDLL_Expansion2/CvTechClasses.cpp` includes comments `AI_UNIT_PRODUCTION : New flavors` and checks the new role flavor names, but this block is not gated by `MOD_AI_UNIT_PRODUCTION`.

`CvPolicyAI::WeighPolicy` includes the new role flavor names in always-compiled conquest/naval/air bucket logic.

`CvGrandStrategyAI.cpp` and `CvAdvisorRecommender.cpp` include gated `MOD_AI_UNIT_PRODUCTION` classification of new role flavor names.

INFERRED: Internally, the mod's design is coherent if the missing flavor rows exist externally: split broad ranged/naval/air pressure into role-specific composition strategies and let those strategies add or suppress unit roles. In this repository alone, the flavor-name data dependency is unresolved.

- Personality vs active/latest flavors: `GetPersonalityIndividualFlavor` ignores active strategy deltas, while recipient `GetLatestFlavorValue` includes propagated deltas. Systems that directly read personality flavors, such as belief yield scoring, will not react to temporary strategy flavor changes.
- `FLAVOR_RANGED` is broad. It appears on ranged units and siege-like units in `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql`, while `MOD_AI_UNIT_PRODUCTION` tries to split archer/siege/skirmisher but lacks repository data rows for those split flavors.
- `FLAVOR_OFFENSE` is overloaded. It contributes to unit object weights, operation/army buildable boosts, policy conquest classification, belief war scoring, grand-strategy priority, and tech conquest priority.
- `FLAVOR_DIPLOMACY` is not only diplomacy. It weights diplomatic units, policy diplomacy buckets, religion trade-route scoring, leader strategy identity, and event/contract choices.
- Negative city strategy flavors are legal. `CvFlavorRecipient::ChangeFlavors` stores negative latest values; default reads clamp them to zero, but adding/removing deltas without preserving exact signs can permanently skew temporary effects.

INFERRED:
- Some non-flavor logic behaves like hidden flavors: recommended army/navy/explorer counts, underrepresented unit-type bonuses, city specialization, grand-strategy victory interests, and strategy need/enough flags are all preference systems that can outweigh generic flavor rows.
- Double-counting risk is highest where a leader's flavor both raises a strategy threshold and boosts the object score that satisfies that strategy. Unit production's broad military flavors plus military strategy bonuses are the main example.

| Function / enum / table | Decisions affected |
|---|---|
| `CvFlavorManager::Init(...)`, `BroadcastBaseFlavors()`, `AddFlavorRecipient(...)`, `ChangeActivePersonalityFlavors(...)`, `ChangeCityFlavors(...)`, `GetPersonalityIndividualFlavor(...)` (`CvFlavorManager.cpp`) | flavor initialization, propagation, and direct personality flavor access. |
| `CvFlavorRecipient::SetFlavors(...)`, `ChangeFlavors(...)`, `GetLatestFlavorValue(...)` (`CvFlavorManager.cpp/.h`) | recipient-local flavor state and cache update trigger. |

- personality sources: `Leader_Flavors` in `CvLeaderHeadInfo::CacheResults` (`CvGameCoreDLL_Expansion2/CvInfos.cpp`) and `MinorCivilization_Flavors` in `CvMinorCivInfo::CacheResults` (`CvGameCoreDLL_Expansion2/CvMinorCivAI.cpp`).

- propagation: `CvFlavorManager::Init`, `RandomizeWeights`, `BroadcastBaseFlavors`, `ChangeActivePersonalityFlavors`, `ChangeCityFlavors`; recipient methods `CvFlavorRecipient::SetFlavors`, `ChangeFlavors`, `GetLatestFlavorValue`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvFlavorManager::ChangeActivePersonalityFlavors(...)` / `ChangeCityFlavors(...)` | player and city flavor recipient cache updates. | RISK start/end delta symmetry and serialized recipient values are savegame-sensitive. |

`FlavorTypes` is an open enum with `NO_FLAVOR = -1` in `CvGameCoreDLLUtil/include/CvEnums.h`; actual flavor identity is runtime database data from `Flavors`. C++ then treats `FlavorTypes` as array indices in `CvEnumMap<FlavorTypes, int>` and object flavor arrays.

| Flavor table | Loader | Producer/consumer path |
|---|---|---|
| `Flavors` | `CvDllDatabaseUtility::PrefetchGlobalDefines` | Defines runtime IDs/names; consumed by `GC.getInfoTypeForString("FLAVOR_*")`, `GC.getFlavorTypes(eFlavor)`, `CvDatabaseUtility::SetFlavors`, and hardcoded string comparisons. |
| `Leader_Flavors` | `CvLeaderHeadInfo::CacheResults` | `CvFlavorManager::Init` fills `m_piPersonalityFlavor`; `CvFlavorManager::RandomizeWeights` may randomize non-zero AI personality flavors unless disabled by diplomacy option logic. |
| `MinorCivilization_Flavors` | `CvMinorCivInfo::CacheResults` | Minor-civ flavor initialization in `CvFlavorManager::Init`. |
| `Unit_Flavors` | `CvUnitEntry::CacheResults` | `CvUnitProductionAI::AddFlavorWeights` multiplies unit flavor by city latest flavor; `RecommendUnit` falls back to unweighted unit flavor sums if cached weight is zero. |
| `Building_Flavors` | `CvBuildingEntry::CacheResults` | `CvBuildingProductionAI::AddFlavorWeights` for final city building/wonder weights; `CvWonderProductionAI::AddFlavorWeights` for desired-wonder weights. |
| `Policy_Flavors` | `CvPolicyEntry::CacheResults` | `CvPolicyAI::AddFlavorWeights`, `WeightPrereqs`, `PropagateWeights`, and later `CvPolicyAI::WeighPolicy` string-bucket reinterpretation. |
| `Technology_Flavors` | `CvTechEntry::CacheResults` | `CvPlayerTechs::FlavorUpdate` -> `CvTechAI::AddFlavorWeights` -> `CvTechAI::ChooseNextTech`; outside this atlas's main decision pages but important for data path completeness. |
| `Project_Flavors` | `CvProjectEntry::CacheResults` | `CvProjectProductionAI` weights in city production. |
| `Process_Flavors` | process info loader / `CvProcessProductionAI` | `CvProcessProductionAI` weights in city production. |
| `AIGrandStrategy_Flavors`, `AIGrandStrategy_FlavorMods` | `CvGrandStrategyAIType::CacheResults` | Grand strategy scoring and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy`. |
| `AIEconomicStrategy_*_Flavors`, `AIMilitaryStrategy_*_Flavors`, `AICityStrategy_Flavors`, `CitySpecialization_Flavors` | respective strategy/specialization `CacheResults` functions | Dynamic flavor deltas applied through `CvFlavorManager::ChangeActivePersonalityFlavors`, `ChangeCityFlavors`, and direct `CvCityStrategyAI` local changes. |
| `EventChoiceFlavors`, `CityEventChoiceFlavors`, `Contract_Flavor`, `Improvement_Flavors` | `CvInfos.cpp`, `CvContractClasses.cpp`, `CvImprovementClasses.cpp` | flavor-backed systems exist but are not primary targets of this page; cite before importing into AI decision docs. |

Runtime flavor access has three important layers:

- `CvFlavorManager::GetPersonalityIndividualFlavor` reads randomized base personality (`CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`).

- `CvFlavorRecipient::GetLatestFlavorValue` reads recipient-local latest flavors after propagated deltas; negative values clamp to zero unless `bAllowNegative` is true.

- `CvGrandStrategyAI::GetPersonalityAndGrandStrategy` combines personality with active grand-strategy flavor mods (`CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp`).

RISK: Do not treat "flavor" as one value. Belief scoring often reads personality flavors directly; city production reads city recipient latest flavors; policy and tech caches read recipient flavors; grand-strategy consumers use a separate blended helper.

`CvFlavorRecipient` and `CvFlavorManager` are declared in `CvGameCoreDLL_Expansion2/CvFlavorManager.h` and implemented in `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`. `CvWonderProductionAI` inherits from `CvFlavorRecipient`; `CvPlayerPolicies`, `CvPlayerTechs`, and `CvCityStrategyAI` also receive flavor updates.

Inputs consumed: `CvFlavorManager::Init` fills personality values from `Leader_Flavors`, `MinorCivilization_Flavors`, or default values; `RandomizeWeights` applies personality randomization; `AdjustWeightsForMap` modifies expansion/growth by map conditions. Strategy systems call `ChangeActivePersonalityFlavors` for non-city recipients and `ChangeCityFlavors` for city recipients. Flavor IDs are database-driven by the `Flavors` table and object rows such as `Unit_Flavors`, `Building_Flavors`, `Policy_Flavors`, `Technology_Flavors`, `Project_Flavors`, `Process_Flavors`, `AIEconomicStrategy_*_Flavors`, `AIMilitaryStrategy_*_Flavors`, `AICityStrategy_Flavors`, and `CitySpecialization_Flavors`.

Selection mechanism: Not applicable directly. INFERRED: Selection changes happen downstream because recipients rebuild cached option weights and later selectors use those caches.

`CvCityStrategyAI` is a `CvFlavorRecipient` with serialized `m_piLatestFlavorValues`. `CvFlavorRecipient::GetLatestFlavorValue(FlavorTypes, bool)` clamps negative values to zero unless `bAllowNegative` is true.

`CvCityStrategyAI::FlavorUpdate()` resets `CvBuildingProductionAI`, `CvUnitProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`, then loops all `FlavorTypes` and broadcasts each latest city flavor to `AddFlavorWeights(...)` on each production AI.

`CvFlavorManager::ChangeCityFlavors(const CvEnumMap<FlavorTypes, int>&, const char*, bool)` applies deltas only to flavor recipients where `CvFlavorRecipient::IsCity()` is true. City strategy and specialization flavor changes are logged with `CvCityStrategyAI::LogFlavorChange(...)`.

`CvAICityStrategyEntry::CacheResults(...)` loads `AICityStrategy_Flavors` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`; `CvCitySpecializationXMLEntry::CacheResults(...)` loads `CitySpecialization_Flavors` in `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`.

Object flavor tables feeding cached production weights are loaded in `CvUnitEntry::CacheResults(...)` from `Unit_Flavors`, `CvBuildingEntry::CacheResults(...)` from `Building_Flavors`, `CvProjectEntry::CacheResults(...)` from `Project_Flavors`, and `CvProcessInfo::CacheResults(...)` from `Process_Flavors`.

Relevant source-data maps include `(1) Community Patch/Database Changes/AI/CoreStrategyChanges.sql`, `(1) Community Patch/Database Changes/AI/CoreCitySpecializationChanges.sql`, `(2) Vox Populi/Database Changes/AI/StrategyChanges.sql`, `(2) Vox Populi/Database Changes/AI/StrategyFlavorSweeps.sql`, `(2) Vox Populi/Database Changes/AI/CitySpecializationFlavorChanges.sql`, `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql`, `(2) Vox Populi/Database Changes/AI/BuildingFlavorSweeps.sql`, `(2) Vox Populi/Database Changes/AI/ProjectFlavorChanges.sql`, and `(2) Vox Populi/Database Changes/AI/ProcessFlavorSweeps.sql`.

RISK: `CvCityStrategyAI::Serialize(...)` and each production AI `Serialize(...)` make flavor and cached production-weight state savegame-sensitive. Enum/database ordering for `FlavorTypes`, `UnitTypes`, `BuildingTypes`, `ProjectTypes`, and `ProcessTypes` is also sensitive.

- Player strategy deltas: `CvEconomicAI::UpdateEconomicStrategies()` and `CvMilitaryAI::UpdateMilitaryStrategies()` copy each active strategy's `GetPlayerFlavorValue(...)` rows into `m_aiTempFlavors` and call `CvFlavorManager::ChangeActivePersonalityFlavors(...)` on strategy start; on strategy end they apply the negated rows. `CvMilitaryAI` additionally dirties city specializations when a strategy requires it.

- City strategy deltas: `CvCityStrategyAI::DoTurn()` starts/ends `AICityStrategyTypes`, directly adds/removes `CvAICityStrategyEntry::GetFlavorValue(...)` from `m_piLatestFlavorValues`, logs with `LogFlavorChange(...)`, then calls `FlavorUpdate()`.

- New-city deltas: `CvCityStrategyAI::UpdateFlavorsForNewCity()` loops active economic and military strategies and adds their city flavor rows to the new city's latest flavor vector. OPEN_QUESTION this function applies current player strategy city deltas but does not call `FlavorUpdate()` in the inspected body; initial registration already propagated base personality values.

- Specialization deltas: `CvCityStrategyAI::SetSpecialization(...)` calls `SpecializationFlavorChange(false, old)` and `SpecializationFlavorChange(true, new)`. `SpecializationFlavorChange(...)` adds/removes `CvCitySpecializationXMLEntry::GetFlavorValue(...)` rows and calls `FlavorUpdate()`.

- Grand strategy flavor mods: `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FlavorTypes, bool)` is a parallel access path, not a recipient delta. It returns personality plus active `AIGrandStrategy_FlavorMods`, clamps at zero, and can blend toward the active grand strategy's main `AIGrandStrategy_Flavors` value.

- Active player path: `CvFlavorManager::ChangeActivePersonalityFlavors(...)` updates `m_piActiveFlavor`, logs through `LogActivePersonalityChange(...)`, then sends `CvFlavorRecipient::ChangeFlavors(...)` only to non-city recipients.

- City path: `CvFlavorManager::ChangeCityFlavors(...)` sends `CvFlavorRecipient::ChangeFlavors(...)` only to city recipients.

- Recipient semantics: `CvFlavorRecipient::SetFlavors(...)` replaces a recipient's latest vector; `CvFlavorRecipient::ChangeFlavors(...)` adds deltas without range enforcement so temporary negative effects can be reversed exactly; both call virtual `FlavorUpdate()`. `CvFlavorRecipient::GetLatestFlavorValue(..., false)` clamps negative reads to zero.

- Policies: `CvPlayerPolicies::FlavorUpdate()` calls `CvPlayerPolicies::AddFlavorAsStrategies(POLICY_WEIGHT_PROPAGATION_PERCENT)`. That function resets `CvPolicyAI`, uses `CvFlavorManager::GetPersonalityIndividualFlavor(...)` before Medieval and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` from Medieval onward, adds direct situational boosts for `ECONOMICAISTRATEGY_LOSING_MONEY`, unhappiness, and war, then calls `CvPolicyAI::AddFlavorWeights(...)`. INFERRED non-city strategy deltas still call this recipient's `FlavorUpdate()`, but the recomputation deliberately ignores `GetLatestFlavorValue(...)` in favor of long-term personality/grand-strategy values.

- Desired wonders: `CvWonderProductionAI::FlavorUpdate()` loops latest recipient flavors and calls `CvWonderProductionAI::AddFlavorWeights(...)`; `AddFlavorWeights(...)` multiplies each wonder `CvBuildingEntry::GetFlavorValue(eFlavor)` by the latest weight and accumulates `m_WonderAIWeights`.

- City production: `CvCityStrategyAI::FlavorUpdate()` resets `CvBuildingProductionAI`, `CvUnitProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`, then broadcasts each latest city flavor through each sub-AI's `AddFlavorWeights(...)`. Those functions multiply object rows from `Building_Flavors`, `Unit_Flavors`, `Project_Flavors`, and `Process_Flavors` by the city flavor value and cache base weights.

- Techs: `CvPlayerTechs::FlavorUpdate()` and `CvTechAI::AddFlavorWeights(...)` use the same recipient/cache pattern for `Technology_Flavors`; tech is outside this decision record's consumer focus but confirms the shared infrastructure.

- Belief selection: `CvReligionAI::ScoreYieldForReligionTimes100(...)`, `ScorePantheonBeliefAtCity(...)`, `ScoreBeliefAtCity(...)`, and `ScorePantheonBeliefForPlayer(...)` read `CvFlavorManager::GetPersonalityIndividualFlavor(...)` for names such as `FLAVOR_GROWTH`, `FLAVOR_PRODUCTION`, `FLAVOR_GOLD`, `FLAVOR_SCIENCE`, `FLAVOR_CULTURE`, `FLAVOR_RELIGION`, `FLAVOR_GREAT_PEOPLE`, `FLAVOR_OFFENSE`, `FLAVOR_DEFENSE`, `FLAVOR_CITY_DEFENSE`, `FLAVOR_HAPPINESS`, `FLAVOR_DIPLOMACY`, and `FLAVOR_WONDER`. These consumers bypass recipient latest values and object flavor tables.

- Policy selection: policy base weights come from `CvPlayerPolicies::AddFlavorAsStrategies(...)` and `CvPolicyAI::AddFlavorWeights(...)`, which multiplies `CvPolicyEntry::GetFlavorValue(eFlavor)` by the chosen current flavor value and propagates prerequisite value with `CvPolicyAI::WeightPrereqs(...)`. `CvPolicyAI::WeighPolicy(...)` also reinterprets `Policy_Flavors` by hardcoded `GC.getFlavorTypes(...) == "FLAVOR_*"` buckets for conquest, diplomacy, science, culture, and generic weights.

- Wonder/building choice: desired-wonder scoring uses `CvWonderProductionAI::FlavorUpdate()` / `AddFlavorWeights(...)` over `Building_Flavors` for wonder entries. Final city building/wonder production uses `CvBuildingProductionAI::GetWeight(...)` as the precheck value, then `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` converts the raw flavor product with `sqrti(10 * iValue)` before applying local and strategic heuristics.

- Unit production: generic city unit candidates read `CvUnitProductionAI::GetWeight(UnitTypes)` from `Unit_Flavors * latest city flavor`; operation/army wrappers also read `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)`. `CvUnitProductionAI::CheckUnitBuildSanity(...)` rejects nonpositive flavor weight, converts with `sqrti(10 * iFlavorWeight)`, then applies the main unit-specific sanity layer.

GUIDE_PATTERN: The guide says AI flavors are often used in scoring. The database table `Leader_Flavors` assigns AI civilizations values from 1 to 10 for categories such as `FLAVOR_GROWTH` and `FLAVOR_CULTURE`. Flavor values can be accessed through `CvFlavorManager` and `GetPersonalityIndividualFlavor(...)`.

GUIDE_PATTERN: The guide's pantheon scoring example for `FaithFromKills` uses a conditional gate and a normalized multiplication:

GUIDE_PATTERN: - positive score only if player is a warmonger and has at least one neighbor

GUIDE_PATTERN: - multiplier includes offense flavor

GUIDE_PATTERN: - multiplier includes faith per kill from the belief

GUIDE_PATTERN: - multiplier includes faith yield valuation

GUIDE_PATTERN: - multiplier includes neighbor count capped at 3

GUIDE_PATTERN: - division normalizes the result relative to other belief scores

GUIDE_PATTERN: This is important for later flavor disentangling. Flavors may be raw personality values, but scoring functions often combine them with yield valuation, game-state conditions, strategic goals, and arbitrary normalization divisors.

GUIDE_PATTERN: The flavor system is a central input to many scores. The guide states that leader flavors are stored in the database table `Leader_Flavors`, commonly as values from 1 to 10. Code can access them through `CvFlavorManager`, for example:

GUIDE_PATTERN:

```cpp
CvFlavorManager* pFlavorManager = kPlayer.GetFlavorManager();
int iGoldFlavor = pFlavorManager->GetPersonalityIndividualFlavor(
    (FlavorTypes)GC.getInfoTypeForString("FLAVOR_GOLD"));
```

GUIDE_PATTERN: When documenting any scoring function, record each flavor used, its source, and how it is combined with other terms. Also distinguish static leader personality flavor from dynamic game-state strategy inputs.

GUIDE_PATTERN: The guide's pantheon `FaithFromKills` example is a useful template: the score is conditional on warmonger status and neighbors, then multiplies offense flavor, belief value, faith yield valuation, and neighbor count before dividing to normalize. This implies that flavor is rarely the whole decision. It is usually one multiplier among game-state gates, database values, and yield valuation.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U current unit flavor contracts

Partial source verification: `src-20261029-vp-c285a18`, pass `audit-20261029-task1u-01`. `CvCityStrategyAI::FlavorUpdate` (349) resets unit cache, then loops all flavor indices and calls AddFlavorWeights with **city recipient GetLatestFlavorValue**. `CvFlavorRecipient::GetLatestFlavorValue` (CvFlavorManager.cpp:89) returns max(0,stored signed latest) unless allow-negative; stored signed deltas are not clamped on update. Object coefficient from `Unit_Flavors(UnitType,FlavorType,Flavor)` is loaded by `CvUnitEntry::CacheResults`, CvUnitClasses.cpp:438 via SetFlavors; role membership loads Unit_AITypes at453. Exact effective coefficients for selected profiles remain unestablished.

Recipient baseline: AddFlavorRecipient propagates personality vector when requested/initialized. ChangeActivePersonalityFlavors mutates active player state and sends only to **non-city** recipients; ChangeCityFlavors sends only to cities. Thus active player strategy deltas cannot be universally added to city unit weights. New-city initialization replays active economic/military **city** deltas (CvCityStrategyAI.cpp:385); local city strategies and specialization deltas remain distinct producers. Full initialization/randomization/map-adjustment term graph and all transition timings still PARTIAL.

Independent GS path (`CvGrandStrategyAI::GetPersonalityAndGrandStrategy`,1598): active GS → max(0,GS flavorMod + personality); optional boost if main GS flavor>0 → min(10,(GS main + adjusted +1)/2); otherwise adjusted. No active GS returns personality. CvGrandStrategyAI.h:58 declares bBoostGSMainFlavor=false. The single-argument unit consumers above therefore use the adjusted personality+GS modifier without the optional main-flavor averaging boost; they do not read city latest. Consumers: operation/army OFFENSE wrappers, settler EXPANSION sanity, composition DEFENSE/OFFENSE and UPAI role ratios. Army/navy producer also reads **direct personality NAVAL**, bypassing recipient latest/active. Recommendation's cache==0 fallback sums only object coefficients, with no runtime flavor multiplier.

UPAI supplied NewStrategies adds eight flavor IDs 38–45, role city strategy deltas ±150 and player deltas ±15, while NewLeaderFlavors/Triggers provide role personality rows and NewUnitFlavors object coefficients. Those are source intentions; absent effective UPAI DB and manifest mismatch prevent asserting final loaded rows. Role ratio helpers read personality+GS, not the ±150 latest-city signal; generic weight consumes city signal × object coefficient; sanity independently adds100 or rejects ENOUGH. This is three distinct mechanisms.

## Task 1U-B flavor lifecycle closure

5.4.8 src-20261029-vp-548 / audit-20261029-task1u-b revalidates the previous unit cache and bypass contracts. Relevant source: FlavorManager.cpp29–196,233–435; CityStrategyAI.cpp349–497,1407–1705; EconomicAI.cpp74–130, strategy adoption/reversal738–764; MilitaryAI.cpp52–108,1950–2220; Infos.cpp7973; GrandStrategyAI.cpp42–44,1598–1618. These are current unit-consumer contracts only.

`infra.unit-flavor-paths:f001` object data: Unit_Flavors(UnitType,FlavorType,Flavor)→UnitEntry CacheResults438→SetFlavors→PopulateArrayByValue joins Flavors.ID. Missing array yields0 at GetFlavorValue; rows with nonexistent UnitType never load to a real entry. Unit_AITypes joins UnitAIInfos with NUM_UNITAI_TYPES; fixed enum bounds matter. Leader_Flavors loads in Infos7973; minor personality uses minor entry flavors. Strategy loaders: AIEconomicStrategy_Player_Flavors/City_Flavors; AIMilitaryStrategy_Player_Flavors/City_Flavors; AICityStrategy_Flavors; CitySpecialization_Flavors; GS flavor/mod tables AIGrandStrategy_Flavors/AIGrandStrategy_FlavorMods. Key column is each owning Type, value Flavor. Effective rows remain profile evidence, not a single SQL occurrence.

`f002` personality initialization: live nonbarbarian player with taken/computer slot; AI diplomacy uses personality leader's cached row (minor uses minor rows), human diplomacy uses DEFAULT_FLAVOR_VALUE for every flavor. Reset vectors0. Nonzero AI personality values randomized inclusively base±max(FLAVOR_RANDOMIZATION_RANGE,0), clamp min/max personality Defines (max constrained1..100; min1..max); zero stays0. Seed raw0xe655df8f.mix(playerID).mix(flavorIndex), with game map/turn RNG context. Map adjustment: int((log10(landTiles/aliveMajors)−FLAVOR_STANDARD_LOG10_TILES_PER_PLAYER)*FLAVOR_EXPANDGROW_COEFFICIENT), add expansion with max cap, subtract growth with min floor; reset active to base and broadcast. Personality and active vectors serialize; these are source initialization draws, not final city-selection draws.

`f003` recipient baseline and deltas: AddFlavorRecipient optionally SetFlavors(personality); BroadcastBaseFlavors sets every registered recipient; ResetToBasePersonality copies active and broadcasts base. SetFlavors overwrites latest, logs differences, invokes FlavorUpdate. ChangeFlavors adds signed deltas without storage clamp then invokes update. Active-player mutation broadcasts only noncity recipients; city mutation only cities. New city replays currently active economic/military city deltas. Economic/military adoption adds table player/city vectors, reversal subtracts exact vectors; adopted flags/turns serialize. Military gates include minor eligibility, prereq/obsolete tech, first turn, elapsed check interval/minimum duration and MilitaryStrategyCanActivate veto. Economic equivalent includes EconomicStrategyCanActivate. City strategy update uses its own timing/flags/predicate and CityStrategyCanActivate, then applies local vector/reversal. These managers are independent strategy-state decision interfaces; log exact adopted state/delta/age rather than infer activation from flavors.

`f004` specialization: SetSpecialization removes old specialization vector and adds new vector directly to stored city latest; marks production dirty. **It does not itself call FlavorUpdate.** Subsequent actual flavor updates rebuild cache; current stored latest and cached unit sum may therefore have different generations. City latest vector and sub-AI caches both serialize. Do not reconstruct a cache by blindly multiplying a later latest read. New-city/rebuild timing must be correlated at actual callback.

`f005` consuming cache: each FlavorUpdate resets cache to0, reads max(0,cityStoredLatest[f]) using default allowNegative=false, accumulates UnitFlavor[u,f]*read[f]. Sanity root is later. RecommendUnit zero-cache fallback sums only object coefficients; it does not multiply personality/active/latest. Barbarian GetWeight returns CODE_CONSTANT1.

`f006` bypass: single-argument GetPersonalityAndGrandStrategy has boost=false (header58): no GS→personality; active GS→max(0,personality+GS flavorMod). Optional true branch averages with positive GS main flavor and caps10; it is not selected by the audited single-argument consumers. OFFENSE wrappers, settler EXPANSION, land/naval allocation OFFENSE/DEFENSE, recon RECON and role ratios consume this layer; allocation NAVAL consumes direct personality. UPAI strategy city±150, player±15, leader coefficients and unit role coefficients enter these separate routes; no universal sum of every layer exists.

UNRESOLVED_EXTERNAL loaded coefficient/profile values and runtime cache age require logging; the symbolic unit signal routing, resets/reversal/clamps and bypass semantics are now mapped. Broader AI flavor consumers remain outside this scope.
