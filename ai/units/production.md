# Unit production and recommendation

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

Why is a particular unit recommended, scored and ultimately trained?

The recovered reference distinguishes cached unit weights, role recommendations and sanity scoring from the final shared city selector. Operation demand, flavor propagation and queue application have linked owners; this document owns unit-specific candidate and score behavior.

## Reference synthesis

The unit base weight is described as an accumulation of object flavor coefficients multiplied by effective city/latest flavor reads. This cache is a preference input, not the final choice. `RecommendUnit` filters for a requested UnitAI, can fall back to an unweighted flavor sum when the cached weight is zero, applies its own turns-left weighting and returns the top recommendation. Generic city candidates instead enter shared competition after trainability/automation checks and positive cached weight.

`CheckUnitBuildSanity` is a separate shared path: the recovered descriptions include the integer-root conversion `sqrti(10 * iFlavorWeight)`, hard rejection codes and direct situational valuation. Operation, purchase and free-unit parameters materially change its context. These reports do not enumerate every term. Preserve that limitation when filling the normalized score schema.

A recommendation is not a queue decision. City sanity, duration, continuation, fallback and thresholded selection can still change the outcome. Completion and operation assignment must be traced separately from selection.

Canonical related owners: [Operation and army demand reaching unit production](<operation-demand.md>), [City production selection](<../production/city-selection.md>), [Flavor production, propagation and consumption](<../infrastructure/flavors.md>), [Production queues and creation](<../../mechanics/production/queue-and-creation.md>), [Role strategies and the VP+UPAI profile](<role-strategies.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- City founding/init path: `CvCity::init(...)` calls `AI_chooseProduction(false, false)` for non-human AI city production after `AI_init()` (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- Normal production-turn path: `CvCity::doProduction(bool bAllowNoProduction)` calls `AI_chooseProduction(false, false)` when city production is AI-controlled or automated and the city has no production, is maintaining a process, or `AI_isChooseProductionDirty()` is true (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- Queue cleanup/finish path: `CvCity::doCheckProduction(...)` can call `chooseProduction()` for human-controlled cities that need a choice, updates obsolete queued units to available upgrades, and returns into the `CvCity::doProduction(...)` path where AI-controlled cities may call `AI_chooseProduction(...)` if needed (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- Automation path: `CvCity::setProductionAutomated(...)` calls `AI_chooseProduction(...)` if automation leaves the city without production (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- Specialization path: `CvCityStrategyAI::SetSpecialization(...)` marks production dirty; `CvCitySpecializationAI::DoTurn()` can mark cities dirty under siege and can interrupt building production with `AI_chooseProduction(m_bInterruptWonders, m_bInterruptBuildings)` (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`).

`CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)` first preserves spaceship-part production, avoids interrupting buildings/wonders unless allowed, optionally starts the desired wonder when an engineer is available, and otherwise calls `m_pCityStrategyAI->ChooseProduction(NO_BUILDING, NO_UNIT, bInterruptBuildings, bInterruptWonders)` (`CvGameCoreDLL_Expansion2/CvCityAI.cpp`).

Decision role: `CvUnitProductionAI` scores trainable unit types and recommends a highest-weight unit for requested `UnitAITypes`. Normal city production selection is still owned by `CvCityStrategyAI::ChooseProduction`, where unit candidates compete with buildings, projects, and processes.

- AI unit production is one branch of the shared city production selector, not a standalone final queue decision. `CvCityStrategyAI::ChooseProduction(BuildingTypes, UnitTypes, bool, bool)` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` mixes unit candidates with buildings, projects, and processes in `CvWeightedVector<CvCityBuildable>`. Unit candidates enter as three `CityBuildableType` values from `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`: `CITY_BUILDABLE_UNIT`, `CITY_BUILDABLE_UNIT_FOR_OPERATION`, and `CITY_BUILDABLE_UNIT_FOR_ARMY`.

- final normal queue ownership is `CvCityStrategyAI::ChooseProduction(...)`; immediate city entry is `CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)` in `CvGameCoreDLL_Expansion2/CvCityAI.cpp`; unit base-weight and sanity helper ownership is `CvUnitProductionAI` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`; operation-slot demand is exposed through `CvCity::GetUnitForOperation()` in `CvGameCoreDLL_Expansion2/CvCity.cpp` and `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity*, bool&)` in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`; army-fill demand is exposed through `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`.

- `CvCity::doProduction(bool bAllowNoProduction)` in `CvGameCoreDLL_Expansion2/CvCity.cpp` calls `AI_chooseProduction(false, false)` for AI-controlled or automated production when the city has no production, is maintaining a process, or `CvCityAI::AI_isChooseProductionDirty()` is true. `CvCityAI::AI_chooseProduction(...)` normally delegates to `CvCityStrategyAI::ChooseProduction(NO_BUILDING, NO_UNIT, bInterruptBuildings, bInterruptWonders)` after preserving spaceship parts/current buildings/wonders and checking its narrow desired-wonder engineer path. `docs-ai/atlas/01-turn-lifecycle.md` is consistent with audited source: `CvEconomicAI::DoTurn()`, `CvMilitaryAI::DoTurn()`, `CvReligionAI::DoTurn()`, `CvCitySpecializationAI::DoTurn()`, threat/danger updates, and city-local `CvCityStrategyAI::DoTurn()` run before normal city production.

## Candidates and gates

`CvCityStrategyAI::ChooseProduction(...)` resets `m_Buildables` and `m_BuildablesPrecheck`, then adds unit candidates before buildings/projects/processes (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`).

Operation unit candidates:
- `CvCity::GetUnitForOperation()` peeks at `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, verifies the operation and army, checks city access to the operation muster landmass/ocean, reads the formation slot's primary and secondary `UnitAITypes`, and calls `CvUnitProductionAI::RecommendUnit(...)` (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- If a unit is found, `ChooseProduction(...)` adds `CITY_BUILDABLE_UNIT_FOR_OPERATION` with base `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, offense flavor from `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)`, skipped-operation count from `CvMilitaryAI`, and the unit's flavor weight (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`).

Army-fill candidates:
- `CvMilitaryAI::GetUnitTypeForArmy(CvCity* pCity)` scans army open slots, tries primary then secondary slot `UnitAITypes` through `CvUnitProductionAI::RecommendUnit(...)`, and prefers an army with fewer open slots (`CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`).
- `ChooseProduction(...)` adds this as `CITY_BUILDABLE_UNIT_FOR_ARMY` with `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT` plus offense flavor multiplier (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`).

Generic unit candidates:
- `ChooseProduction(...)` loops `GC.GetGameUnits()->GetNumUnits()`.
- It requires `m_pCity->canTrain(unit, continue-current-production)`.
- Human automated cities skip most units; under `MOD_BALANCE_VP`, they may build noncombat units with a work rate.
- The candidate type is `CITY_BUILDABLE_UNIT`, with turns from `getProductionTurnsLeft(unit, 0)` and initial weight from `CvUnitProductionAI::GetWeight(unit)`.
- Only positive-weight candidates enter `m_BuildablesPrecheck`.

`CvUnitProductionAI::RecommendUnit(UnitAITypes eUnitAIType, bool bAllowStrategicResource)` is a helper for operation/army and other purchase/religion/policy callers. It loops all units, requires `CvUnitEntry::GetUnitAIType(eUnitAIType)`, optionally rejects units with a strategic resource type, requires `CvCity::canTrain(...)`, falls back to sum of unit flavors if the cached unit weight is zero, reweights by turns left, stable-sorts, logs, and returns the top unit only (`CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`).

The production code uses both combat/domain fields and `UnitAITypes`:
- Combat classification: `GetCombat() > 0 || GetRangedCombat() > 0`.
- Supply need: combat or ranged combat and not `IsNoSupply()`.
- Domain classification: `GetDomainType()` branches for `DOMAIN_LAND`, `DOMAIN_SEA`, and `DOMAIN_AIR`.
- Default role: `CvUnitEntry::GetDefaultUnitAIType()` is passed to `pushOrder(ORDER_TRAIN, eUnitType, eUnitAI, ...)`.
- Multi-role eligibility: `CvUnitEntry::GetUnitAIType(eUnitAIType)` is used by `RecommendUnit(...)` and religion combat-bonus checks.

Important default `UnitAITypes` observed directly in production scoring include `UNITAI_EXPLORE`, `UNITAI_CITY_SPECIAL`, `UNITAI_CITY_BOMBARD`, `UNITAI_CARRIER_SEA`, `UNITAI_ATTACK_AIR`, `UNITAI_DEFENSE_AIR`, `UNITAI_MESSENGER`, `UNITAI_SETTLE`, `UNITAI_ARCHAEOLOGIST`, `UNITAI_WORKER_SEA`, `UNITAI_WORKER`, `UNITAI_GENERAL`, `UNITAI_ADMIRAL`, `UNITAI_ATTACK`, `UNITAI_DEFENSE`, `UNITAI_COUNTER`, `UNITAI_RANGED`, `UNITAI_SKIRMISHER`, `UNITAI_FAST_ATTACK`, `UNITAI_ATTACK_SEA`, `UNITAI_ASSAULT_SEA`, and `UNITAI_SUBMARINE` (`CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`).

- `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot` finds the next needed operation slot for a city; `CvCity::GetUnitForOperation` reads primary/secondary `UnitAITypes` from `CvArmyAI::GetSlotInfo` and calls `CvUnitProductionAI::RecommendUnit`.

- Operation units: `CvCityStrategyAI::ChooseProduction(...)` calls `CvCity::GetUnitForOperation()` first. `CvCity::GetUnitForOperation()` calls `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, requires a valid operation slot and same muster city, checks `CvAIOperation::GetMusterPlot()` landmass/ocean access with `CvCity::HasAccessToLandmassOrOcean(...)`, reads `CvArmyAI::GetSlotInfo(...)`, and calls `CvUnitProductionAI::RecommendUnit(...)` for the slot's primary `UnitAITypes` and then secondary `UnitAITypes`. If a unit is found, `ChooseProduction(...)` pushes `CITY_BUILDABLE_UNIT_FOR_OPERATION` with `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)`, `CvMilitaryAI::GetNumberOfTimesOpsBuildSkippedOver()`, `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`, and `CvUnitProductionAI::GetWeight(UnitTypes)`. It then bumps skipped-operation pressure with `CvMilitaryAI::BumpNumberOfTimesOpsBuildSkippedOver()`.

- Army-fill units: `CvCityStrategyAI::ChooseProduction(...)` next calls `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`. That function scans player `CvArmyAI` objects, reads required open slots with `CvArmyAI::GetOpenSlots(true)`, reads `CvFormationSlotEntry` role fields through `CvArmyAI::GetSlotInfo(...)`, asks `CvUnitProductionAI::RecommendUnit(...)` for primary then secondary `UnitAITypes`, and prefers a unit from the army with the fewest open slots. If found, `ChooseProduction(...)` pushes `CITY_BUILDABLE_UNIT_FOR_ARMY` with `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT` plus `FLAVOR_OFFENSE * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`.

- Generic unit candidates: `CvCityStrategyAI::ChooseProduction(...)` loops `GC.GetGameUnits()->GetNumUnits()`, skips `eIgnoreUnit`, requires `CvCity::canTrain(UnitTypes, bContinueCurrentProduction)`, applies automated-human filters, and pushes `CITY_BUILDABLE_UNIT` only when `CvUnitProductionAI::GetWeight(UnitTypes)` is positive. Generic units do not require an operation slot or army slot.

- Helper recommendations: `CvUnitProductionAI::RecommendUnit(UnitAITypes eUnitAIType, bool bAllowStrategicResource)` is not the final normal selector. It is a helper used by operation, army, emergency purchase, and other paths. It loops all units, requires `CvUnitEntry::GetUnitAIType(eUnitAIType)`, optionally rejects strategic-resource units when `bAllowStrategicResource` is false, requires `CvCity::canTrain(...)`, falls back to the unweighted sum of all `CvUnitEntry::GetFlavorValue(...)` rows if the city cached weight is zero, reweights by turns left with `CityStrategyAIHelpers::ReweightByTurnsLeft(...)`, stable-sorts, logs through `CvUnitProductionAI::LogPossibleBuilds(...)`, and returns the top unit.

## Scoring and inputs

Base flavor scoring:
- `CvUnitProductionAI::Reset()` initializes one `m_UnitAIWeights` entry per unit with zero weight.
- `CvUnitProductionAI::AddFlavorWeights(FlavorTypes eFlavor, int iWeight)` adds `unit.GetFlavorValue(eFlavor) * iWeight`.
- `CvUnitProductionAI::GetWeight(UnitTypes eUnit)` returns cached weight, except barbarian cities return `1`.

Sanity scoring in `CvUnitProductionAI::CheckUnitBuildSanity(...)`:
- Rejects nonpositive input weight as `SR_IMPOSSIBLE`.
- Converts base weight to `sqrti(10 * iFlavorWeight)`.
- Rejects null unit entries.
- Rejects non-free purchases that cannot be gold-purchased, and non-free non-purchase puppet city training.
- Under siege, rejects noncombat units.
- Rejects units in underdeveloped cities with fewer than two buildings unless desperate or purchasing.
- Rejects units when an upgraded class can be trained somewhere.
- Penalizes or rejects combat units near bankruptcy.
- Rejects land/sea combat units if city garrison plus nearby placement search finds no valid plot.
- Counts explorers, land units, and sea units including units in production; adjusts current production out of counts when not purchasing.
- Rejects supplied land/naval/explorer units once recommended army/navy/explorer counts are reached.
- For minor civs, applies separate era/city-count caps and recently-bullied bonuses.
- Handles resource availability, spaceship aluminum reservation, unit investments, city-special/support/suicide units, policy-unlocked or move-after-purchase units, carriers, explorers, naval/land/air/AA demand, conquest strategy, production modifiers, diplomatic units, spaceship parts, religious combat beliefs, civilian great-person-like effects, settlers, archaeologists, work boats, workers, war boosters, best military city bonuses, promotion/unique unit bonuses, operation bonuses, underrepresented unit types, tiny army/navy pressure, siege desperation, barbarian strategy, current-production inertia, instant yield-on-completion, free-unit floor, purchase-in-siege bonus, occupied-city penalty, and final `max(1, score)`.

Negative return codes are `ProductionSkipReason` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`: `SR_IMPOSSIBLE`, `SR_UNITSUPPLY`, `SR_MAINTENANCE`, `SR_STRATEGY`, `SR_USELESS`, and `SR_BALANCE`. `CvCityStrategyAI::LogInvalidItem(...)` maps these to log strings.

Duration scoring:
- `CvCityStrategyAI::ReweightByDuration(...)` calls `CityStrategyAIHelpers::ReweightByTurnsLeft(weight, turns)` on final buildables.
- `CvUnitProductionAI::RecommendUnit(...)` also reweights candidate unit weights by turns before selecting a top recommendation.

Flavor sources entering unit weights:
- City strategy flavor state enters through `CvCityStrategyAI::FlavorUpdate()` and `CvUnitProductionAI::AddFlavorWeights(...)`.
- Economic and military strategies propagate city flavor deltas via `CvFlavorManager::ChangeCityFlavors(...)` when strategies start/end (`CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`).
- Existing active economic/military strategies are applied to new cities by `CvCityStrategyAI::UpdateFlavorsForNewCity()`.
- City strategies apply local flavor deltas and call `FlavorUpdate()` in `CvCityStrategyAI::DoTurn()`.
- City specializations apply local flavor deltas in `CvCityStrategyAI::SpecializationFlavorChange(...)`.

Tactical/homeland/military/economic inputs directly used by production scoring:
- `CvTacticalAI` dominance zone: workers/work boats are rejected unless the city zone is friendly (`GetTacticalAnalysisMap()->GetZoneByCity(...)` in `CheckUnitBuildSanity(...)`).
- `CvMilitaryAI`: recommended land army/navy/explorer counts, number of civs at war, war type, need naval/air-carrier strategies, barbarian strategies, operation skipped counts, settler skipped counts, underrepresented unit types, AA count, and players at war feed scoring or constraints.
- `CvEconomicAI`: `ECONOMICAISTRATEGY_GS_CONQUEST`, `ECONOMICAISTRATEGY_ENOUGH_EXPANSION`, `ECONOMICAISTRATEGY_NEED_ARCHAEOLOGISTS`, `ECONOMICAISTRATEGY_NEED_DIPLOMATS`, `ECONOMICAISTRATEGY_NEED_DIPLOMATS_CRITICAL`, and spaceship strategy checks directly affect unit scoring.

Victory and grand-strategy inputs:
- `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)` boosts operation/army candidates in `ChooseProduction(...)`.
- `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_EXPANSION)` drives settler scoring.
- `CvDiplomacyAI::IsGoingForSpaceshipVictory()`, `IsCloseToSpaceshipVictory()`, and `IsGoingForCultureVictory()` affect aluminum reservation, spaceship-part scoring, and archaeologist scoring.

Policy/ideology/belief/religion inputs directly entering unit production:
- `CvUnitEntry::GetPolicyType()` and `CvUnitEntry::CanMoveAfterPurchase()` add gold-surplus purchase-oriented bonuses for units unlocked by policy or able to move after purchase.
- `CvPlayerAI::GetUnitClassProductionModifier(...)` adds a small production-modifier bonus.
- State religion beliefs are read through `CvPlayerReligions::GetStateReligion()`, `CvGameReligions::GetReligion(...)`, `CvReligion::m_Beliefs`, and `CvBeliefEntry` fields such as `GetFaithFromKills()`, `GetCombatModifierEnemyCities()`, `GetCombatModifierFriendlyCities()`, `GetCombatVersusOtherReligionOwnLands()`, `GetCombatVersusOtherReligionTheirLands()`, and `GetYieldFromKills(...)`.

Civilization, leader trait, unique unit/building, and personality inputs:
- Unit flavors in `CvUnitEntry` combine with city flavor/personality state.
- `CvPlayerTraits` methods directly used include `GetMinorInfluencePerGiftedUnit()`, `IsDiplomat()`, `IsTourism()`, `IsExpansionist()`, `IsNerd()`, `IsSmaller()`, `IsWarmonger()`, `IsExpansionWLTKD()`, `HasFreePromotionUnitClass(...)`, and `HasFreePromotionUnitCombat(...)`.
- Unique units are detected by `CvCivilizationInfo::isCivilizationUnitOverridden(unitClass)` and get a bonus.
- City/building-derived free promotions and best military city selection influence combat-unit scoring.

Diplomacy and other-player behavior:
- War state: `GetPlayersAtWarWith()`, `GetNumberCivsAtWarWith(false)`, and `GetWarType(...)` affect land/naval/war boosts.
- Friendliness: `GetUnfriendlyMajors()` increases war-type weighting when the war opponent is unfriendly.
- Proximity and enemy air: `CvDiplomacyAI::IsPlayerValid(...)`, `CvPlayerAI::GetProximityToPlayer(...)`, and opponent air-unit counts drive AA scoring.
- Minor-civ pressure: `CvMinorCivAI::IsRecentlyBulliedByAnyMajor()` boosts minor military demand.
- City-state diplomacy: `HasMetValidMinorCiv()`, paper from CS alliances, and diplomatic unit influence values affect messenger scoring.

City/local constraints:
- `CvCity::canTrain(...)` is the first generic unit gate.
- `CvCity::IsCanPurchase(...)` gates purchase scoring.
- Population, under siege, garrison, local valid plots, city building count, ocean connection, air slots, local units on the city plot, food turns, capital status, occupation, city threat, trade priorities, city investments, and reachable water resources are direct scoring inputs.

Strategic resource constraints:
- Generic training relies on `CvCity::canTrain(...)`; `CheckUnitBuildSanity(...)` separately rejects units whose `GetResourceType()` has no available resource and rewards surplus.
- Aluminum is reserved when spaceship victory is active/near and the unit is not itself a spaceship project.
- `RecommendUnit(..., bAllowStrategicResource=false)` rejects units with any resource type before `canTrain(...)`.

- `CvCityStrategyAI::FlavorUpdate` sends latest city flavors to `CvBuildingProductionAI`, `CvUnitProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`.

Scoring shape: `GetWeight` returns cached unit flavor weight, except barbarian cities return `1`. `CheckUnitBuildSanity` rejects nonpositive input and hard skip conditions using `ProductionSkipReason` (`SR_IMPOSSIBLE`, `SR_UNITSUPPLY`, `SR_MAINTENANCE`, `SR_STRATEGY`, `SR_USELESS`, `SR_BALANCE` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`), converts flavor weight with `sqrti(10 * iFlavorWeight)`, then adds local, strategic, role, resource, war, religion, trait, operation, and current-production bonuses.

- Base unit weights: `CvCityStrategyAI::FlavorUpdate()` resets `CvUnitProductionAI` and calls `CvUnitProductionAI::AddFlavorWeights(FlavorTypes, int)` for every latest city flavor. `CvUnitProductionAI::AddFlavorWeights(...)` multiplies each `CvUnitEntry::GetFlavorValue(eFlavor)` by the city flavor value and accumulates it in `m_UnitAIWeights`; `CvUnitProductionAI::GetWeight(UnitTypes)` returns that cached value, except barbarian cities return `1`.

- Operation/army wrapper scores: operation and army candidates add direct `FLAVOR_OFFENSE` from `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` before sanity scoring. Operation candidates also add skipped-operation pressure and the recommended unit's cached weight; army candidates do not add the unit cached weight in the inspected wrapper.

- Sanity scoring: after `m_BuildablesPrecheck.StableSortItems()`, unit candidates are re-scored by `CvUnitProductionAI::CheckUnitBuildSanity(UnitTypes eUnit, bool bForOperation, int iFlavorWeight, bool bForPurchase, bool bFree)`. It rejects nonpositive weight, converts base weight with `sqrti(10 * iFlavorWeight)`, null-checks `CvUnitEntry`, then applies hard filters and bonuses for purchase legality, puppet training, siege/desperation, underdeveloped cities, obsolete upgrade classes, maintenance/bankruptcy, valid placement plots, supply/recommended army/navy/explorer counts, resources, war state, tactical zone, unit role, domain, city state, strategies, traits, beliefs, policies, diplomacy, current production, and production modifiers.

- Skip codes: negative sanity returns are `ProductionSkipReason` values in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`: `SR_IMPOSSIBLE`, `SR_UNITSUPPLY`, `SR_MAINTENANCE`, `SR_STRATEGY`, `SR_USELESS`, and `SR_BALANCE`. `CvCityStrategyAI::LogInvalidItem(...)` maps these codes for production logs.

- Duration scoring: `CvCityStrategyAI::ReweightByDuration(CvWeightedVector<CvCityBuildable>&)` reweights surviving unit/building/project/process candidates by turns left before final sorting. `CvUnitProductionAI::RecommendUnit(...)` separately applies turn-left reweighting inside the helper recommendation list.

- Flavor inputs: unit object flavor values come from `CvUnitEntry::GetFlavorValue(...)`, loaded by `CvUnitEntry::CacheResults(...)` from `Unit_Flavors` in `CvGameCoreDLL_Expansion2/CvUnitClasses.cpp`. Runtime city flavor values are latest `CvCityStrategyAI` recipient values, modified by economic, military, city-strategy, and city-specialization flavor deltas through `CvFlavorManager::ChangeCityFlavors(...)`, `CvFlavorRecipient::ChangeFlavors(...)`, `CvCityStrategyAI::DoTurn()`, and `CvCityStrategyAI::SpecializationFlavorChange(...)`. Relevant source-data map: `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql`, strategy flavor files under `(1) Community Patch/Database Changes/AI/` and `(2) Vox Populi/Database Changes/AI/`, and `CitySpecialization_Flavors`.

- Direct non-flavor inputs: unit production directly reads city legality/state (`CvCity::canTrain`, current queue, turns left, puppet/automation, siege, buildings, garrison, local plots, air slots), player military/economic state (`CvMilitaryAI` recommended land/navy/explorer sizes, strategies, army slots, operation skipped count; `CvEconomicAI` strategies and purchase state), grand strategy (`CvGrandStrategyAI::GetPersonalityAndGrandStrategy`), diplomacy/victory state (`CvDiplomacyAI` spaceship/culture/war/proximity helpers), resources, traits (`CvPlayerTraits`), religion beliefs (`CvReligion`, `CvBeliefEntry` combat/yield fields), policies, unit classes, domains, and default/multiple `UnitAITypes`.

- Data tables / entries / enums: important source classes and entries include `CvCityStrategyAI`, `CvCityAI`, `CvCity`, `CvCityBuildable`, `CvUnitProductionAI`, `CvUnitEntry`, `CvUnitXMLEntries`, `CvPlayerAI`, `CvMilitaryAI`, `CvAIOperation`, `CvArmyAI`, `CvFormationSlotEntry`, `CvGrandStrategyAI`, `CvFlavorManager`, and `CvWeightedVector<CvCityBuildable>`. enum/order-sensitive types include `CityBuildableType`, `ProductionSkipReason`, `UnitTypes`, `UnitClassTypes`, `UnitAITypes`, `DomainTypes`, `ResourceTypes`, `FlavorTypes`, and `OrderTypes` (`ORDER_TRAIN`) in `CvGameCoreDLLUtil/include/CvEnums.h` and `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`.

- MOD option note: `CvGameCoreDLL_Expansion2/CustomMods.h` defines `MOD_AI_UNIT_PRODUCTION` as `gCustomMods.isAI_UNIT_PRODUCTION()`, but `(1) Community Patch/Database Changes/NewCustomModOptions.xml` sets `AI_UNIT_PRODUCTION` to `0`. STALE_OR_MISMATCH / RISK: source comments and enum annotations for role-split unit AI/flavors remain, but default repository behavior does not enable the gated production strategy blocks unless the custom mod option is changed.

## Data and configuration

- Macro definition: `CvGameCoreDLL_Expansion2/CustomMods.h` defines `MOD_AI_UNIT_PRODUCTION` as `gCustomMods.isAI_UNIT_PRODUCTION()`.
- Option declaration/cache: `MOD_OPT_DECL(AI_UNIT_PRODUCTION)` in `CustomMods.h`; `MOD_OPT_CACHE(AI_UNIT_PRODUCTION)` in `CvGameCoreDLL_Expansion2/CustomMods.cpp`.
- Default database value: `(1) Community Patch/Database Changes/NewCustomModOptions.xml` sets `AI_UNIT_PRODUCTION` to `0`.
- Classification: constraint/filter and flavor/strategy input switch, disabled by default.
- Internal coherence: disabled default means MOD-gated blocks are not part of default production behavior unless a mod option changes the database value.

| Folder / data area | Relevant tables and rows | Current-source consumer |
|---|---|---|
| `(2) Vox Populi/Database Changes/Units/` | modifies `Units`, `UnitClasses`, `UnitClass_AITypes`, `Unit_AITypes`, formations (`NewUnitFormations.xml`), missions, unit costs/stats, default AI roles, prerequisite techs/resources, and policy/city-state/barbarian unit variants. | `CvUnitEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvUnitClasses.cpp`), city legality `CvCity::canTrain`, production scoring `CvUnitProductionAI::CheckUnitBuildSanity`, role recommendation `CvUnitProductionAI::RecommendUnit`, and operation formation consumers `CvAIOperation` / `CvArmyAI`. |

Unit data path is `Units` / `Unit_Flavors` / `UnitClass_AITypes` / `Unit_AITypes` / prereq-resource-tech fields -> `CvUnitEntry::CacheResults` -> city flavor cache in `CvUnitProductionAI::AddFlavorWeights` -> candidate generation in `CvCityStrategyAI::ChooseProduction` using `CvCity::canTrain` -> scoring in `CvUnitProductionAI::CheckUnitBuildSanity` -> queue application through `CvCity::pushOrder(ORDER_TRAIN, ...)`.

Operation/army demand adds formation data to the path: `NewUnitFormations.xml`, `MultiUnitFormations`, and `MultiUnitFormation_SlotEntries` feed `CvArmyAI::SetFormation`; unfilled slots are queued in `CvAIOperation::m_viListOfUnitsWeStillNeedToBuild`; cities query via `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot` and convert roles through `CvCity::GetUnitForOperation` / `CvUnitProductionAI::RecommendUnit`.

| Option / macro | Default in repo | Verified AI relevance |
|---|---:|---|
| `AI_UNIT_PRODUCTION` / `MOD_AI_UNIT_PRODUCTION` | `0` in `(1) Community Patch/Database Changes/NewCustomModOptions.xml` | Changes unit-production role semantics and military strategy tests in `CvUnitProductionAI::CheckUnitBuildSanity` and `CvMilitaryAI`. |

## Observability and validation obligations

- `CvCityStrategyAI::LogPossibleBuilds(...)` logs `PRE` and `POST` buildable vectors to `CityStrategyAIProductionLog...csv`.
- `CvCityStrategyAI::LogCityProduction(...)` logs chosen type, description, rush/no-rush, era, turns, GPT, and current random seed.
- `CvCityStrategyAI::LogInvalidItem(...)` logs skipped buildables and maps negative sanity return codes.
- `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes eUnitAIType)` logs `RecommendUnit(...)` special-request candidate lists to the city strategy log.
- `CvCityStrategyAI::LogFlavorChange(...)` logs flavor deltas from strategies/specializations that later affect unit weights.

- `CvUnitProductionAI::LogPossibleBuilds`, `CvCityStrategyAI::LogPossibleBuilds`, `LogCityProduction`, and `LogInvalidItem`.

Logging: `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)` logs `RecommendUnit` candidates and special requests. Normal production logging uses `CvCityStrategyAI::LogPossibleBuilds`, `LogInvalidItem`, and `LogCityProduction`.

- `CvCityStrategyAI::LogPossibleBuilds(...)` logs `PRE` and `POST` city buildable vectors, including unit candidates, to `CityStrategyAIProductionLog...csv`.

- `CvCityStrategyAI::LogInvalidItem(...)` logs candidates rejected by unit sanity checks and records mapped `ProductionSkipReason` text.

- `CvCityStrategyAI::LogCityProduction(...)` logs the final selected buildable type, description, rush flag, era, turns, GPT, and current random seed.

- `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)` logs the helper `RecommendUnit(...)` candidate list for special role requests.

- `CvCityStrategyAI::LogFlavorChange(...)` logs city flavor deltas that later feed `CvUnitProductionAI::AddFlavorWeights(...)`.

## Version differences, risks and unresolved questions

Patch-risk note: `UnitAITypes` is an enum in `CvGameCoreDLLUtil/include/CvEnums.h`; `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` are marked as new for `MOD_AI_UNIT_PRODUCTION`. Enum ordering is savegame/sync-sensitive.

INFERRED:
- `CvCity::canTrain(...)` likely carries many database, tech, resource, civilization replacement, Lua/event, and city-state constraints. This pass treats it as a source-verified gate but does not expand its internals because that dependency path is broad.
- Unit creation side effects after `ORDER_TRAIN` are not fully documented here. A separate pass should trace `CvCity::produce(...)`, `CvCity::CreateUnit(...)`, free promotions, supply counters, operation assignment, and Lua/game events.
- Database definitions for all `AIMilitaryStrategy_City_Flavors`, `AIEconomicStrategy_City_Flavors`, `AICityStrategy_Flavors`, and unit `Flavors` were not enumerated. The code path that consumes them is verified.
- Operation slot generation and army formation composition were only traced from the production-facing methods. Formation planning itself is outside this document unless it becomes necessary for operation-unit candidate generation.

- Savegame-sensitive: `CvUnitProductionAI::Serialize(...)` stores `m_UnitAIWeights`; `CvCityStrategyAI::Serialize(...)` serializes the production AI subobjects.
- Enum-sensitive: `CityBuildableType`, `ProductionSkipReason`, `UnitAITypes`, `DomainTypes`, `FlavorTypes`, and unit/building/project/process enums should not be reordered casually.
- Determinism-sensitive: final city production uses `CvWeightedVector::ChooseAbovePercentThreshold(...)` with seed `0x0e36d18b` mixed with city ID. `RecommendUnit(...)` uses stable sort and top pick, not threshold randomness.
- XML/schema-sensitive: unit production depends on unit flavor rows, default/multiple `UnitAITypes`, unit classes, traits, strategies, city specializations, resources, promotions, beliefs, and custom-mod options.
- Behavioral coupling: unit candidates compete with buildings, projects, and processes. Changing unit scoring can change non-unit production frequency.
- `MOD_AI_UNIT_PRODUCTION` is marked "SLATED FOR DELETION" in `CustomMods.h`, disabled by default in XML, yet its enum additions and some comments remain in always-compiled source.

- Source functions: `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_ARMY` from `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`. In the later sanity switch, `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` share a branch that calls `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` and requires `thisOperationSlot.IsValid() && bCitySameAsMuster` before `CvUnitProductionAI::CheckUnitBuildSanity(..., bForOperation=true, ...)`. Purchase/hurry sanity in the same file treats `CITY_BUILDABLE_UNIT_FOR_ARMY` separately with `bForOperation=false`.

- Smallest possible patch point: The `CvCityStrategyAI::ChooseProduction(...)` sanity switch for `CITY_BUILDABLE_UNIT_FOR_ARMY`; separate it from `CITY_BUILDABLE_UNIT_FOR_OPERATION` and mirror the purchase path's `CheckUnitBuildSanity(eUnitType, false, weight, false, false)` if source owners confirm design intent.

- Smallest possible patch point: Data-only if the feature is meant to be enabled: add `Flavors` and object/strategy rows. DLL patch points only after data is verified are `CvMilitaryAI` role strategy helpers and `CvUnitProductionAI::CheckUnitBuildSanity(...)`.

- Runtime/logging test: With `AI_UNIT_PRODUCTION` enabled in a controlled database, log `GC.getInfoTypeForString(...)` results for each role flavor and observe `MILITARYAISTRATEGY_NEED_*` / `ENOUGH_*` strategy activation and `CvUnitProductionAI` `SR_BALANCE` returns.

- Source functions: `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::LogPossibleBuilds(...)` logs `PRE` and `POST` buildable weights and `LogInvalidItem(...)` logs negative skip reasons. `CvCityStrategyAI::LogFlavorChange(...)` logs city flavor deltas. Scoring transformations occur across `CvUnitProductionAI::GetWeight(...)`, `CvBuildingProductionAI::GetWeight(...)`, `CheckUnitBuildSanity(...)`, `CheckBuildingBuildSanity(...)`, and `CvCityStrategyAI::ReweightByDuration(...)`.

- Risk of double-counting or unintended downstream effects: RISK a scoring boost in `CheckUnitBuildSanity(...)` can affect normal unit production, operation units, army units, gold hurry, tactical purchase, and faith purchase. A purchase can also consume resources/treasury before later city production or movement decisions.

Patch-risk notes: RISK: Unit scoring competes with all other city production. `CvUnitProductionAI::Serialize` stores `m_UnitAIWeights`; `UnitAITypes` is an enum in `CvGameCoreDLLUtil/include/CvEnums.h`; unit database rows include `Unit_Flavors`, role flags, default AI, resources, promotions, and policy gates. `MOD_AI_UNIT_PRODUCTION` is disabled by default in `(1) Community Patch/Database Changes/NewCustomModOptions.xml`, but always-compiled enum/support code and some gated branches remain.

- OPEN_QUESTION: `CITY_BUILDABLE_UNIT_FOR_ARMY` is injected separately, but the normal post-precheck branch in `CvCityStrategyAI::ChooseProduction(...)` handles `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` together and requires `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` plus same muster city before calling `CheckUnitBuildSanity(..., bForOperation=true, ...)`. This may mean army-fill candidates without a current operation slot are filtered in normal queue selection. The source behavior is verified; design intent is not.

- RISK unit production competes with all city buildable classes. Changing unit candidate weights or sanity filters changes the frequency of buildings, wonders, projects, and processes as well as unit mix.

- RISK deterministic behavior depends on seeded city production selection `0x0e36d18b` mixed with city ID, stable sort order, and database-backed unit/flavor ordering.

- RISK savegame-sensitive state includes `CvCityStrategyAI::Serialize(...)`, `CvUnitProductionAI::Serialize(...)`, city order queues, unit-class making counts, and operation/army state. Enum ordering for `UnitAITypes`, `CityBuildableType`, `ProductionSkipReason`, and database-backed `UnitTypes`/`FlavorTypes` is sensitive.

- RISK `CvUnitProductionAI::CheckUnitBuildSanity(...)` is shared by normal generic units, operation/army candidates, purchase helpers, and faith/economic/military emergency paths. A change intended for normal queue production can affect purchases.

- OPEN_QUESTION trace full `CvCity::canTrain(...)` if a later record needs complete legality, Lua/game-event, resource, civilization replacement, and schema gates.

- DESIGN_HYPOTHESIS a future score-component appendix should break `CvUnitProductionAI::CheckUnitBuildSanity(...)` into hard filters, local city constraints, strategic pressure, role-composition pressure, flavor-derived base score, and application-only effects.

## Source and dependency lookup

Unresolved questions for focused pass:
- Trace `CvCity::GetUnitForOperation()` and `CvMilitaryAI::GetUnitTypeForArmy(...)` to distinguish requested unit roles from generic production.
- Break down `CvUnitProductionAI::CheckUnitBuildSanity(...)` into supply, maintenance, war state, domain, exploration, settler, and military strategy gates.
- Map `UnitAITypes` assumptions and any savegame-sensitive enum usage.
- Trace unit creation side effects after `ORDER_TRAIN` completion.

The main unit scoring gate is `CvUnitProductionAI::CheckUnitBuildSanity(UnitTypes eUnit, bool bForOperation, int iFlavorWeight, bool bForPurchase, bool bFree)`. It converts the raw flavor weight with `sqrti(10 * iFlavorWeight)`, applies hard negative skip reasons from `ProductionSkipReason`, then adds local, strategic, diplomatic, religious, trait, supply, resource, domain, and war bonuses before returning `max(1, iFlavorWeight)`.

`MOD_AI_UNIT_PRODUCTION` is disabled in the shipped custom-mod option XML: `(1) Community Patch/Database Changes/NewCustomModOptions.xml` has `<Row Class="6" Name="AI_UNIT_PRODUCTION" Value="0"/>`. Therefore this document treats MOD-gated behavior as an appendix, not as default behavior.

`CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, `CvUnitProductionAI::CheckUnitBuildSanity(...)`, land war settler escort boost.
- Surrounding default logic: land combat demand computes `iWarValue`; default adds extra weight for city-plot settlers when candidate default AI is `UNITAI_DEFENSE`, `UNITAI_COUNTER`, or `UNITAI_ATTACK`.
- MOD change: when enabled, `UNITAI_COUNTER` is excluded from that settler escort boost.
- Classification: scoring change.
- Coherence: INFERRED coherent if MOD reinterprets `UNITAI_COUNTER` as anti-air/specialized rather than generic escort; potentially harms default-style counter-unit escorting if database still uses `UNITAI_COUNTER` for normal defensive units.

`CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, `CvUnitProductionAI::CheckUnitBuildSanity(...)`, "NEW STRATEGIES" block.
- Surrounding default logic: after standard war/current-production/instant-yield/free/purchase bonuses, combat units receive role-specific bonuses or `SR_BALANCE` rejections only when MOD is enabled.
- MOD change: reads need/enough military strategies for `UNITAI_RANGED`, `UNITAI_CITY_BOMBARD`, `UNITAI_SKIRMISHER`, `UNITAI_FAST_ATTACK`, `UNITAI_ATTACK_SEA`, `UNITAI_ASSAULT_SEA`, `UNITAI_SUBMARINE`, `UNITAI_ATTACK_AIR`, and `UNITAI_DEFENSE_AIR`; adds `100` for need strategies or returns `SR_BALANCE` for enough strategies, except naval melee "enough" is commented out.
- Classification: scoring change and constraint/filter change.
- Coherence: comments say it considers "too much or too few of a unit." INFERRED likely improves role balance when corresponding strategy counts/flavors are maintained; it can hard-filter units even if other scoring inputs strongly prefer them.

`CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, strategy activation dispatch in `CvMilitaryAI::DoTurn()`-area loop.
- Surrounding default logic: default ranged need/enough strategies are only evaluated when `!MOD_AI_UNIT_PRODUCTION`; mobile/air/anti-air/nuke/carrier strategies remain default.
- MOD change: enables separate archer, siege, skirmisher, naval melee, naval ranged, submarine, bomber, and fighter need/enough strategies.
- Classification: flavor/strategy input change.
- Coherence: INFERRED coherent with the scoring block above; default `MILITARYAISTRATEGY_NEED_RANGED`/`ENOUGH_RANGED` is replaced by finer categories, but only if XML strategies exist and unit counts are maintained.

`CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `MilitaryAIHelpers::IsTestStrategy_EnoughMobileUnits(...)` and `IsTestStrategy_NeedMobileUnits(...)`.
- Surrounding default logic: default ratio is mobile / (melee + mobile).
- MOD change: ratio becomes mobile / melee.
- Classification: flavor/strategy input change.
- Coherence: INFERRED coherent as a stricter composition check against melee baseline; can produce stronger need/enough flips for small melee counts.

`CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp` and `CvGameCoreDLL_Expansion2/CvMilitaryAI.h`, helper declarations/definitions for new need/enough strategies.
- Surrounding default logic: existing helper functions use `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` flavors and current unit counts.
- MOD change: adds helpers for `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, and `FLAVOR_FIGHTER`.
- Classification: flavor/strategy input change.
- Coherence: internally coherent by direct control flow; INFERRED thresholds vary by role and are plausible role-ratio bands.

`CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp`, policy flavor priority scan.
- Surrounding default logic: grand strategy policy priority already counts military flavors such as offense, defense, naval, naval recon, ranged, and air.
- MOD change: when enabled, additionally counts policy flavors for archer, siege, skirmisher, naval melee, naval ranged, submarine, fighter, and bomber.
- Classification: flavor/strategy input change, indirect to production through grand-strategy/personality flavor state.
- Coherence: INFERRED coherent with role-split unit production flavors; indirect effect only.

`CvGameCoreDLL_Expansion2/CvTechClasses.cpp`, tech grand-strategy priority.
- Surrounding default logic: conquest-focused tech priority counts military flavors including offense, military training, mobile, ranged, defense, and air.
- MOD change: always-commented as "AI_UNIT_PRODUCTION : New flavors" and includes naval, archer, siege, skirmisher, naval melee, naval ranged, submarine, bomber, and fighter flavor names in conquest tech priority. This block is not gated by `MOD_AI_UNIT_PRODUCTION`.
- Classification: flavor/strategy input change; not a runtime MOD gate.
- Coherence: STALE_OR_MISMATCH: comment names AI_UNIT_PRODUCTION, but source does not guard it with `MOD_AI_UNIT_PRODUCTION`. Treat as default current behavior, not appendix-only behavior, if documenting tech choice.

`CvGameCoreDLLUtil/include/CvEnums.h` and `CvGameCoreDLL_Expansion2/CvGameCoreUtils.cpp`.
- Surrounding default logic: `UnitAITypes` enum and `getUnitAIString(...)` map UnitAI values to debug strings.
- MOD change: `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` are annotated "New UNITAI for MOD_AI_UNIT_PRODUCTION"; `getUnitAIString(...)` handles them.
- Classification: enum/schema support and logging/debug support.
- Coherence: always compiled; not unreachable. Patch risk is high because enum ordering and database mappings are sensitive.

`CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `EconomicAIHelpers::IsPotentialNavalExplorer(UnitAITypes eType)`.
- Surrounding default logic: potential naval explorers are attack sea, reserve sea, and assault sea UnitAI types.
- MOD change: submarines are potential naval explorers only when enabled.
- Classification: flavor/strategy input change, indirect to production through exploration demand rather than `ChooseProduction(...)` scoring.
- Coherence: INFERRED plausible but not directly tied to default city production in inspected code.

`CvGameCoreDLL_Expansion2/CvHomelandAI.cpp`, sentry suitability.
- Surrounding default logic: land fortifyable defense/attack/counter units can be sentries.
- MOD change: `UNITAI_COUNTER` is excluded when enabled.
- Classification: indirect role-behavior change, not production selection.
- Coherence: INFERRED coherent with "counter is AA only" interpretation seen elsewhere; not a production block.

`CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, emergency purchase and formation placement.
- Surrounding default logic: emergency city defense can buy ranged, naval melee, counter, or defense units; ranged units are second/third line by range.
- MOD change: emergency purchase uses `UNITAI_DEFENSE` instead of randomized counter/defense because comment says "Counter is AA only now"; AA/intercept-capable units can be third-line.
- Classification: final selection change for emergency purchase, and tactical role change; not the default `ChooseProduction(...)` queue path.
- Coherence: INFERRED coherent with counter-as-AA interpretation.

`CvGameCoreDLL_Expansion2/CvAdvisorRecommender.cpp`.
- Surrounding default logic: advisor recommendation flavor categorization has default branches.
- MOD change: comments mention "New flavors" under `MOD_AI_UNIT_PRODUCTION`.
- Classification: logging/UI recommendation aid, not city production selection.
- Coherence: likely not relevant to AI production decisions inspected here.

The only MOD-gated blocks directly changing city unit-production scoring are in `CvUnitProductionAI::CheckUnitBuildSanity(...)`, with supporting strategy activation in `CvMilitaryAI`. Because `AI_UNIT_PRODUCTION` is disabled by default in `NewCustomModOptions.xml`, the default pipeline should be documented without those role-split enough/need filters.

INFERRED tweak candidates from inspected code only:
- If enabling `MOD_AI_UNIT_PRODUCTION`, audit XML/database support for all referenced strategy names and flavors before trusting the new scoring block.
- If `UNITAI_COUNTER` is no longer generic counter/defense under the enabled MOD, the direct exclusions in production/tactical/homeland code are internally consistent. If database units still use `UNITAI_COUNTER` for ordinary land counters, those exclusions would likely underuse them as escorts/sentries.

The atlas is decision-first, not file-first, because the important AI behaviors cross file boundaries. A single decision usually has separate trigger, candidate generation, scoring, selection, application, logging, and side-effect paths. For example, city unit production starts from `CvCity::doProduction` in `CvGameCoreDLL_Expansion2/CvCity.cpp`, enters `CvCityAI::AI_chooseProduction` in `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, scores candidates in `CvCityStrategyAI::ChooseProduction` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, delegates unit sanity to `CvUnitProductionAI::CheckUnitBuildSanity` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, and finally queues units through `CvCity::pushOrder` in `CvCity.cpp`.

Operation unit demand is stored in `CvAIOperation::m_viListOfUnitsWeStillNeedToBuild`, filled by `CvAIOperation::GrabUnitsFromTheReserves` when required formation slots cannot be filled from existing units. Cities inspect this demand through `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot` and `CvCity::GetUnitForOperation`, then choose a trainable unit through `CvUnitProductionAI::RecommendUnit`.

| Class / manager | Decisions affected |
|---|---|
| `CvUnitProductionAI` (`CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, `.h`) | unit base weights, `RecommendUnit(...)`, `CheckUnitBuildSanity(...)`, and unit production logging. Affects generic unit production, operation/army unit demand, emergency purchases, faith/economic hurry, and advisor/helper valuation. |

| Function / enum / table | Decisions affected |
|---|---|
| `CvUnitProductionAI::GetWeight(...)`, `AddFlavorWeights(...)`, `RecommendUnit(...)`, `CheckUnitBuildSanity(...)` (`CvUnitProductionAI.cpp`) | unit base weights, role recommendation, normal production sanity, operation/army sanity, purchase/hurry sanity. |

- type-specific scoring: `CvUnitProductionAI::GetWeight` / `CheckUnitBuildSanity`; `CvBuildingProductionAI::GetWeight` / `CheckBuildingBuildSanity`; `CvProjectProductionAI::GetWeight` / `CheckProjectBuildSanity`; `CvProcessProductionAI::GetWeight` / `CheckProcessBuildSanity`; duration reweighting in `CvCityStrategyAI::ReweightByDuration`.

- generic units: `CvCityStrategyAI::ChooseProduction`, `CITY_BUILDABLE_UNIT`, `CvCity::canTrain`, `CvUnitProductionAI::GetWeight`, `CvUnitProductionAI::CheckUnitBuildSanity`.

- operation units: `CvCity::GetUnitForOperation`, `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot`, `CvUnitProductionAI::RecommendUnit`, `CITY_BUILDABLE_UNIT_FOR_OPERATION`.

- unit/building selection helpers: `CvMilitaryAI::BuyEmergencyUnit`, `BuyEmergencyBuilding`; `CvCityStrategyAI::ChooseHurry`; `CvUnitProductionAI::RecommendUnit`; `CvUnitProductionAI::CheckUnitBuildSanity(..., bForPurchase=true)`; `CvBuildingProductionAI::CheckBuildingBuildSanity`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvUnitProductionAI::CheckUnitBuildSanity(...)` | generic unit production, operation/army production sanity, operation direct queue path, economic hurry, religion faith purchase, military emergency purchase, policy/process helper valuation. | RISK a unit-scoring change can affect normal queues and immediate purchases. |
| `CvUnitProductionAI::RecommendUnit(...)` | operation slot role conversion, army slot role conversion, military emergency purchases, tactical emergency purchases. | RISK role-helper top-picks do not use the same threshold selection as normal production. |

- Keep the warning that `CvUnitProductionAI::RecommendUnit(...)` is a helper, not the normal final selector. Operation, army, emergency, and hurry paths call it, but normal queued production still competes in `CvCityStrategyAI::ChooseProduction(...)` as `CvCityBuildable` entries.

- Reject/defer patching `CvUnitProductionAI::CheckUnitBuildSanity(...)` based only on atlas production records. it is shared by generic units, operation/army buildables, purchases, emergency paths, and faith/economic hurry paths. Runtime candidate logs must identify whether a bad outcome comes from base flavor weight, sanity score, duration reweighting, current-production inertia, threshold randomness, or application.

`CvUnitProductionAI` is declared in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.h` and implemented in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`. It is owned by `CvCityStrategyAI`.

Inputs consumed: `CvCityStrategyAI::FlavorUpdate` broadcasts latest city flavors into `CvUnitProductionAI::AddFlavorWeights`. `AddFlavorWeights` multiplies `CvUnitEntry::GetFlavorValue(eFlavor)` by current city flavor. `RecommendUnit` consumes requested `UnitAITypes`, optional strategic-resource allowance, `CvUnitEntry::GetUnitAIType`, `CvCity::canTrain`, cached unit weights or fallback raw flavor sums, and turns-left reweighting. `CheckUnitBuildSanity` consumes `CvUnitEntry`, `CvCity::canTrain`/purchase state, supply, maintenance, resources, valid placement plots, city threat/siege/development, unit role/domain/default AI, economic and military strategies, recommended army/navy/explorer counts, diplomacy/war state, beliefs, policies, traits, and current production.

Selection mechanism: Generic units enter `CvCityStrategyAI::ChooseProduction` by looping `GC.GetGameUnits()->GetNumUnits()`, requiring `CvCity::canTrain`, using `GetWeight`, then passing through `CheckUnitBuildSanity` and final city production threshold selection. Operation/army callers use `RecommendUnit`, which stable-sorts and returns the top candidate, then the selected unit still competes in city production as `CITY_BUILDABLE_UNIT_FOR_OPERATION` or `CITY_BUILDABLE_UNIT_FOR_ARMY`.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `AI_UNIT_PRODUCTION` | Enable option with role strategies/flavors in DB, run military strategy update and unit production scoring. | Save before AI production choice; reload and compare selected unit role and strategies. | `CvMilitaryAI::UpdateMilitaryStrategies`, `CvUnitProductionAI::CheckUnitBuildSanity`. | `CvMilitaryAI.cpp:2061`, `CvUnitProductionAI.cpp:1580` |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U current-source score graph

Initial evidence `src-20261029-vp-c285a18` / `audit-20261029-task1u-01`; all43 records revalidated unchanged against authoritative `src-20261029-vp-548` / `audit-20261029-task1u-b` in [source snapshots](../source-snapshots.md#task-1u-b-authoritative-vp-548-identity). VERIFIED_IN_SOURCE for the expressions below; effective loaded values UNKNOWN except the separately identified reference DB. Prior narrative above retains its historical scope. This is a partial provenance pass: an accessor named below is an explicit unresolved upstream leaf unless its producer contract is described in the linked owner. It is not a complete score graph under the maintained schema.

Demand → city/latest cache plus independent manager/GS state → generic/operation/army candidates → sanity → shared duration/continuation/threshold selection → ORDER_TRAIN. Operation fulfillment is a separate state transition, not implied by the buildable tag. See [operation bridge](operation-demand.md#task-1u-current-operation-bridge), [shared selection](../production/city-selection.md#task-1u-current-unit-selection), [feature matrix](role-strategies.md#task-1u-feature-and-profile-matrix), and [flavor paths](../infrastructure/flavors.md#task-1u-current-unit-flavor-contracts).

### Term-record inheritance

The following bounded term inventory uses `units.city-training:t001` onward. Order is source order; branches retain source short circuit/else order. Scope is one city/candidate/invocation (indexed promotions/beliefs/yields/flavors where specified). Inputs named as native getters are unresolved producer leaves unless linked to a current contract. Runtime observations/effective values are UNKNOWN. Literal numbers and enum sentinels are CODE_CONSTANT; named GD reads are DB_DEFINE; unit/formation/flavor coefficients are DB_VALUE/FLAVOR_OBJECT_ROW; state/counts are RUNTIME_STATE or DERIVED_METRIC; strategy predicates MANAGER_STATE; native permission queries LEGALITY; direct trait/policy/belief reads TRAIT_POLICY_RELIGION. These immediate origins do not resolve every upstream dependency.

All integer divisions truncate toward zero; `sqrti` is an integer root; the duration helper uses floating `pow` followed by `int`. No source overflow safety is asserted for arithmetic not explicitly widened. Cache reset occurs in FlavorUpdate; cached weights serialize. Sanity locals initialize on each call and do not persist. Conditions and outputs in the table inherit parameters `(eUnit,bForOperation,iFlavorWeight,bForPurchase,bFree)`. All sanity return sentinels reject at ordinary caller precheck (`>0` required), but [empty-sanity fallback](../production/city-selection.md#task-1u-current-unit-selection) can restore candidates. Logging is NONE for individual numeric terms, PARTIAL for aggregate/rejection outcomes; see [obs007](../logging-coverage.md#obs007). None of these records claims that unlogged positive legality passes can be reconstructed.

Source locations below refer to `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp` unless another file is given. `B` means local `iBonus`, initially zero; `F` means local input flavor score after root. All `B +=` rows consume the preceding executed version of B and feed the next executed row; early returns skip remaining rows. This explicitly preserves overwrite and floor behavior.

| ID | Source / operation / expression and exact branch | Origin / unresolved input boundaries |
|---|---|---|
| <a id="t001"></a>t001 | Reset: cache[unit]=0; AddFlavorWeights: cache[u]+=UnitFlavor[u,f]*cityLatest[f], all units, skip zero multiplier/null entry | FLAVOR_OBJECT_ROW × FLAVOR_RECIPIENT_LATEST; [flavor contract](../infrastructure/flavors.md#task-1u-current-unit-flavor-contracts) |
| <a id="t002"></a>t002 | GetWeight: barbarian owner returns 1, else cached weight | CODE_CONSTANT / stored cache |
| <a id="t003"></a>t003 | RecommendUnit: role<=NO_UNITAI → NO_UNIT; enumerate unit IDs; require role membership, allowed resource flag, canTrain with continue-current parameter | Unit_AITypes, Units.ResourceType, native legality |
| <a id="t004"></a>t004 | RecommendUnit: raw cache==0 → sum all object flavor coefficients; duration-transform; keep >0; stable descending sort; first or NO_UNIT | No personality multiplier in fallback; deterministic top pick, unit enumeration breaks equal-weight ties; no sanity here |
| <a id="t005"></a>t005 | Sanity: B=0; input<1→SR_IMPOSSIBLE; F=sqrti(10*input); null entry→SR_IMPOSSIBLE | root + CODE_CONSTANT; input is wrapper/cache/caller value, not always flavor-only |
| <a id="t006"></a>t006 | !free && purchase requires IsCanPurchase(true,true,unit,...,GOLD); !free && !purchase && puppet rejects; !free && siege && !combat rejects | LEGALITY / puppet / siege; free does not bypass all later gates |
| <a id="t007"></a>t007 | buildings<2 && !siege && !purchase→SR_STRATEGY; any globally trainable allowed class that is upgrade of candidate→SR_STRATEGY | Building count and GetUnitClassTrainingAllowedAnywhere producer unresolved |
| <a id="t008"></a>t008 | !siege&&!free, combat&&!noMaintenance&&!trade: avg=unitMaintenance/max(1,totalUnits); bankruptcyTurns(avg)<9 rejects, else <17 B-=avg*25 | Treasury expense/bankruptcy producer unresolved |
| <a id="t009"></a>t009 | Combat+garrison+nonair requires at least one ring0..before ring3 valid empty combat plot of matching land/water domain | placement LEGALITY; ring constants and native move predicate |
| <a id="t010"></a>t010 | explorers=GetNumUnitsWithUnitAI(EXPLORE,true,true); land/sea=military count+combat production; subtract current default-role explorer or combat domain if !purchase; total=land+sea before land-=explorers | Counts include production; different count populations must be retained |
| <a id="t011"></a>t011 | NeedsSupply=(melee>0\|\|ranged>0)&&!NoSupply; reject land(nonexplore)/sea/explore when respective count>=recommended count | Recommended manager contracts in managers.md; unrelated to an actual supply number at this stage |
| <a id="t012"></a>t012 | !free&&minor&&needsSupply: total>max(4,(era+3)*max(1,cities)) rejects; sea<=2 B+=max(0,300-sea*100); sea<recommended B+=(recommended-sea)*100, bullied +100, else reject; land analog max(0,800-land*100), deficit*150, bullied+250 | MinorCiv recently-bullied and era/state; caps and deficit terms distinct |
| <a id="t013"></a>t013 | !free&&combat, landmass tiles<=3, land+land garrison+!operation→SR_USELESS | Landmass and garrison; commented inland-sea block is inactive source |
| <a id="t014"></a>t014 | Nonminor: unit ResourceType exists → available(false)>0 B+=5*available else impossible; nonspaceship + pursuing/close spaceship + aluminum requirement>0 + available<=remaining spaceship+core-city need rejects | Units/resource entry + diplomacy + aluminum demand producers unresolved |
| <a id="t015"></a>t015 | Nonminor nonspaceship with ResourceType: B+=max(40,(available-required)*40); invested class B+=500 | Distinct from t014; resource semantics follow getter, not inferred strategic classification |
| <a id="t016"></a>t016 | Nonminor CITY_SPECIAL: count(true), subtract current matching role when !purchase; armies=land/6; if count<armies B+=2000*max(3-count,1), operation +10000 | Integer division; role count/default role |
| <a id="t017"></a>t017 | Nonminor !free city-attack-support&&!operation: bombardCount(true)<sameRoleCount(false) rejects; else B+=10*(5-sameRoleCount) | Negative addition allowed |
| <a id="t018"></a>t018 | Nonminor suicide: nukeDamage>0 +100; else ranged>0 && existingNukes>0 -50; else +25 | Units fields/runtime nuke count |
| <a id="t019"></a>t019 | Nonminor policy-unlocked OR move-after-purchase: surplus=(gold-purchaseCost)/d, d=10 combat war,25 combat peace,50 civilian; positive surplus B+=min(cap,surplus), cap=250/200/150 | Runs even for training; purchase price producer unresolved |
| <a id="t020"></a>t020 | Nonminor carrier: !operation&&!NeedAirCarriers rejects; land explorer B+=positive(recommended-count)*500 | Minor carrier test nested in nonminor block is unreachable; helper need contract unresolved |
| <a id="t021"></a>t021 | Nonminor sea combat requires ocean connection; deficit=(recommended-sea); positive deficit gift-unit trait +75; deficit*=max(1,era); positive&&!barbarian deficit*=5 if naval critical else 2 if naval need; warValue=sum(4 unfriendly matching war-domain,2 other matching,1 other domain), skip minors but divide by max(1,all war players); overall SEA war +10; positive warValue multiplies deficit; B+=deficit*30 | MANAGER_STATE/TRAIT_POLICY_RELIGION; exact domains/count divisor matter |
| <a id="t022"></a>t022 | Nonminor land combat: deficit=(recommended-land); positive gift trait +150; era multiply; positive&&!barbarian warValue same domain rules/divisor as t021, overall LAND +15; default DEFENSE/ATTACK, or COUNTER only with !AI_UNIT_PRODUCTION, add 2 per local unassigned settler or 1 assigned settler; positive warValue multiply; B+=deficit*30 | Gate changes COUNTER treatment; no operation exemption from force caps |
| <a id="t023"></a>t023 | Nonminor air: bomber*2>fighter*3 rejects bomber; converse rejects fighter; accepted fighter/bomber B+=2000+(maxAir-teamAirCount(true))*100; other air +1000 | Role balance distinct from native air-capacity gate |
| <a id="t024"></a>t024 | Nonminor (land interceptRange>0)\|\|baseLandAirDefense>20: each valid neighboring/at-war major with air>0 adds cities if AA<cities and adds cities if theirAir>=ourAir; B+=need*50 | Player validity/proximity/AA producer unresolved |
| <a id="t025"></a>t025 | Nonminor meleeCombat>0 + active GS_CONQUEST B+=50; positive unit-class productionModifier B+=modifier/5 | Not city/latest flavors; direct economic strategy / production modifier |
| <a id="t026"></a>t026 | Nonminor VP messenger: no valid minor met, waiting local messenger, or count(true)>3 rejects; influence=sum unit/player/city free promotion mission influence; NEED_DIPLOMATS ×2 else CRITICAL ×3 else reject; VP CS-alliance paper>0 ×2; B+=influence | Promotion DB and existing state; ordinary need branch takes priority over critical |
| <a id="t027"></a>t027 | Nonminor spaceship part: nonhuman city-production AI rejects (separate spaceship controller); remaining GS_SPACESHIP +10000 else1000; closeVictory +10000 else2500; positive city space modifier ×100 added; B<=0 overwritten1 | Explicit direct-controller boundary; not ordinary AI generic training |
| <a id="t028"></a>t028 | Nonminor meleeCombat>0 valid state-religion beliefs: faithKills/5 for ATTACK/FAST; enemyCityModifier for ATTACK/RANGED/SKIRMISHER/FAST/BOMBARD; friendlyCityModifier for DEFENSE/COUNTER; own lands (otherReligion+general)*2/3 for defense roles; their lands same for attack roles; sum positive yieldKills[y]/5; B+=religiousTotal | Beliefs table effects/validity and role membership; changed since release anchor |
| <a id="t029"></a>t029 | Nonminor trade: population<=4 rejects; B+=land/seaTradePriority*10; war -100; diplomat trait +750 else700 | Trade priority producer unresolved |
| <a id="t030"></a>t030 | Nonminor unit properties each independently +100: goldenAgeTurns, beakerTurns, cultureBoost, messenger role, baseGold, buyCS, baseHurry, general, admiral, greatWork; each adds further100 for matching trait family | Matches respectively tourism/expansionist; nerd; tourism; diplomat; diplomat; diplomat; smaller/warmonger; expansionist/warmonger; diplomat/warmonger; tourism. Do not collapse simultaneous properties |
| <a id="t031"></a>t031 | Settler: foodTurns<=1 rejects; ENOUGH_EXPANSION rejects; city ENOUGH_SETTLERS rejects; local settler rejects; E=GS-adjusted EXPANSION; positive settleQuality(bestPlot) added; earlyExpansion+100; continentFeeder+50; expansionWLTKD+10; empireUnhappy-10; capital&&cities>2 -25+capitalSettlerModifier; E<=0 rejects; E+=skippedSettlers*10; B+=E*23 | GS layer plus direct manager/site/trait effects; best settle plot quality unresolved |
| <a id="t032"></a>t032 | Nonminor archaeologist: !NEED_ARCHAEOLOGISTS rejects; count(true)<=0 +5000; cultureVictory +5000; digTourism>0 +2500; each artifactYield[y]>0 +500; break above CULTURE_LOCAL when !JFD | DB/strategy/trait/runtime; JFD extends yield iteration |
| <a id="t033"></a>t033 | Workboat: existing tactical zone nonfriendly rejects; reachable PT_WORKER_SEA_UNIT_SAFE range12; need subtracts every matching-role unit and other-city production (includes own when purchase); eligible owned unimproved needed-resource plots with nonrazing owner increment; bonus resources require owning/effective city work range; closest=min path length; need<=0 rejects; training B+=1000*need*(localPop+era), purchase B+=1000*need*(capitalPop+era)-closest | Pathfinding/resource-improvement predicates unresolved; unit decrement loop has no explicit owner filter |
| <a id="t034"></a>t034 | Worker: nonfriendly tactical zone or local waiting worker rejects; missing=citiesNeedingImprovements+1-count(true); scale=missing>2?3:1; city ENOUGH rejects; WANT B+=200*missing*scale; NEED B+=500*missing*scale | WANT/NEED additions independent, not else branches |
| <a id="t035"></a>t035 | Nonminor combat: matching overall war domain B+=allEnemies.size()*100; major-war bAtWar: combat +threat, civilian -threat; combat+operation+skippedOps>0 B+=skippedOps*100 | Threat and war classifications separate; bAtWar excludes minor-only wars |
| <a id="t036"></a>t036 | Nonminor combat && cities>1: best city for unitcombat +25*cities; independently best city for domain +25*cities | GetBestMilitaryCity producer unresolved; both bonuses may apply |
| <a id="t037"></a>t037 | Nonminor combat promotion loop: trait free unitclass +100, trait free unitcombat +10, player free promotion +10; stop after accumulated>=100; B+=sum; civilization class override +100 | Not capped exactly100: break after additions; unique bonus has no combat guard inside nonminor block |
| <a id="t038"></a>t038 | Nonminor combat: operation +500; rangedStrength>0 adds rangedStrength; underrepresented default role +200; land lacking land garrison +25 | Direct unit strength and derived composition, not flavor |
| <a id="t039"></a>t039 | LandCount<=cities*2 && land candidate: combat+300 else-300; seaCount<=effectiveCoastalCities && sea: combat+300 else-300; siege combat+500 else-500 | Runs outside nonminor guard |
| <a id="t040"></a>t040 | Combat: ERADICATE_BARBARIANS +250 else CRITICAL +500; combat&&!purchase current same unit war+100 else peace+50, other current combat unit -100 | Else priority matters; current-production score mutation precedes city selector inertia |
| <a id="t041"></a>t041 | Every positive YieldOnCompletion[y] contributes yield*1 to B; free && B<=F overwrites B=F; purchase+combat+siege +250 | DB unit completion yields; free result normally at least 2F after final addition, not merely F |
| <a id="t042"></a>t042 | Combat&&AI_UNIT_PRODUCTION: matching default role NEED +100 else ENOUGH rejects SR_BALANCE; roles RANGED/BOMBARD/SKIRMISHER/FAST/ATTACK_SEA/ASSAULT_SEA/SUBMARINE/ATTACK_AIR/DEFENSE_AIR | Strategies ARCHER/SIEGE/SKIRMISHER/MOBILE/NAVAL_MELEE/NAVAL_RANGED/SUBMARINE/BOMBER/FIGHTER; ATTACK_SEA ENOUGH rejection is commented out; no operation/free exemption |
| <a id="t043"></a>t043 | B<0→SR_UNKNOWN even if F+B positive; otherwise F+=B; occupied&&!NoOccupiedUnhappiness F/=5; return max(1,F) | Final negative-bonus gate, occupation division then floor. Old list omitting SR_UNKNOWN was incomplete |

### Prior-pass unresolved leaves (resolved/bounded by Task 1U-B)

This table enumerates direct sanity branches, not complete transitive provenance. The initial pass left loader/data provenance, personality/strategy transitions, best-city, finance, role diversity, tactical/site/trade/resource/aluminum/threat and purchase inputs unresolved. The Task 1U-B sections close or explicitly bound these former leaves at named source contracts. The current disposition of these former gaps is the Task 1U-B p001–p016 producer/legality/flavor contracts below; untouched historical scope remains PARTIAL. Native player/plot legality and Lua vetoes are shared dependencies, not reconstructed from prerequisites. Non-unit competitor scores remain explicitly unresolved at the shared selector owner.

Completeness search found additional callers: `CvAdvisorRecommender.cpp:96`, `CvPolicyAI.cpp:3798` (free purchase context), `CvReligionClasses.cpp:6974,7496`, `CvEconomicAI.cpp:3246`, `CvMilitaryAI.cpp:576,589`, `CvProcessProductionAI.cpp:444`, and ChooseHurry. Their existence is VERIFIED_IN_SOURCE; full per-caller parameter/legality/application provenance is **not yet mapped**, particularly process scoring which consumes sanity and can compete against units. This was the prior pass's completion blocker; the Task 1U-B caller table below now classifies every current call-site family.

### Current native training gates and application boundaries

Same snapshot/pass as t001–t043; source `CvCity.cpp:8642–8808`, `CvPlayer.cpp:canTrainUnit`14033–14330. All gate outputs here are LEGALITY with DB_VALUE/DB_DEFINE/TRAIT_POLICY_RELIGION/RUNTIME_STATE parents; their deep shared helpers/effective profile remain PARTIAL. Candidate calls use visible=false, ignoreCost=false, purchase=false and continue only for currently producing unit. Application pushOrder calls canTrain with default continue=false after queue clearing, so selection legality is not an application guarantee.

City gates in order: valid entry; player eligibility (city allowed-training class can bypass tech); reject locally trainable upgrades; annexable puppet requires VP work-rate noncombat; purchase-only blocks training; AI nonbarbarian supplied combat requires unitsToSupply<unitsSupplied; Zulu replacement; settlers minimum CITY_MIN_SIZE_FOR_SETTLERS; VP maximum oversupply floor via MAX_UNIT_SUPPLY_PRODMOD/PRODUCTION_PENALTY_PER_UNIT_OVER_SUPPLY; required building classes; team air count<city capacity; plot.canTrain; Lua CityCanTrain test-all veto.

Player gates: valid class; trait NoTrain; gift-only unless barbarian; civilization unit mapping/class replacement; cost!=-1; ResourceType total>0 except barbarian; war-only major war; founders major-only/no-annexing/OCC-human restrictions; policy requirement; NO_RELIGION blocks founder/spreader/heresy units; DEFAULT_MAX_NUM_BUILDERS cap for land workers; AND tech requirements unless override, plus obsolete-tech rejection; spaceship victory validity/project cap/count including making with continue decrement. For !testVisible: unhappy settlers if VERY_UNHAPPY_CANT_TRAIN_SETTLERS; prerequisite project; HasResourceForNewUnit (quantity contract unresolved); global/team/player/per-city class limits with making/continue; no-nukes/nukes validity and league prohibition; special-unit validity; trade slots/routes; Lua PlayerCanTrain veto. This bounds meaningful legality without claiming every plot/resource/league/trade internal predicate decomposed.

Application/creation findings belong to [queue owner](../../mechanics/production/queue-and-creation.md#task-1u-current-unit-queue-and-creation). Ordinary completion does not call the explicit promised-slot finish wrapper in the inspected path. Created units can receive explorer roles, so subsequent reserve suitability can differ from requested/default role. Detailed action selection remains deferred.

## Task 1U-B source and consumer closure

Authoritative source src-20261029-vp-548 / audit-20261029-task1u-b. All **43 stable records t001–t043** above retain their IDs and expressions: entire UnitProductionAI.cpp/.h is materially unchanged from prior pass and selected 5.4.8 bodies were inspected. They are grouped ordered records, not a claim there are only43 elementary arithmetic operations. Current upstream read of each term resolves through [producer contracts](../strategies/managers.md#task-1u-b-producer-contracts-on-548), [flavor lifecycle](../infrastructure/flavors.md#task-1u-b-flavor-lifecycle-closure), legality contract below and [operation lifecycle](operation-demand.md#task-1u-b-lifecycle-closure-on-vp-548). Prior c285a18 is comparison evidence, not authority. Direct scorer branch inventory is closed; transitive whole-engine/independent-decision provenance remains bounded as described, and full city-choice provenance remains partial.

Exact object origins: UnitClasses::CacheResults consumes Units scalar Combat/RangedCombat/Domain/DefaultUnitAI/Class/CombatClass/ResourceType/NoSupply/NoMaintenance/Trade/CityAttackOnly/Suicide/NukeDamageLevel/PolicyType/MoveAfterPurchase/AirInterceptRange/BaseLandAirDefense/GoldenAgeTurns/BaseBeakersTurnsToCount/BaseGold/BaseHurry/CultureBoost and referenced project/role IDs. `CityAttackOnly` loads IsCityAttackSupport (do not invent CityAttackSupport column). Keyed tables: Unit_Flavors.Flavor; Unit_ResourceQuantityRequirements.Cost; Unit_YieldOnCompletion.Yield; Unit_FreePromotions.PromotionType; Unit_ClassUpgrades.UnitClassType; Unit_AITypes.UnitAIType. Getter arrays default according to loader; GetFlavorValue null0, GetUnitAIType nullfalse, resource requirement null−1. IsGreatWorkUnit scans named unit great-work entries; it is not a single bool column. Promotion mission influence loads promotion data; free-promotion membership comes separately from unit, player and city. Belief exact current getters/roles and integer 2/3 remain t028, not the legacy renamed methods.

Term input edges: t007→rules allowed-training state; t008→p004/005; t009→p016 placement; t010→p016 counts; t011/012/020–022→p001/002/003 plus p014 trait; t014/015→p012/013; t016–019→DB/runtime counts/p015; t023/024→role counts and tactical/diplomacy interface; t025→p014; t026→promotion effects/strategy state/p012 paper; t027→p013 and independent spaceship controller; t028→p014 belief validity/rows; t029→p008; t030→p014 classifications; t031→flavor f006+p009/strategy flags/skipped counter; t032→p014; t033/034→p011/p016 improvement-need; t035→p010/war state; t036→p007; t037/038→p014/p006; t039–043→prior B versions, read-time counts/occupation/control and object rows. Literals remain CODE_CONSTANT, no undocumented tunable weight is inferred.

### All recommender/sanity call-site dispositions

Current-source `rg RecommendUnit|CheckUnitBuildSanity` across all gameplay C++ produced the following complete call-site families; definitions are not callers. Parameters shown are operation/purchase/free (defaults false).

| Caller / current source | Classification / unit-side contract |
|---|---|
| CityStrategyAI914/932 normal | IN_SCOPE_CORE: op+army(true,false,false), generic(false,false,false); shared competition/queue |
| CityStrategyAI1332/1344/1355 hurry | IN_SCOPE_INTERFACE: op(true,true,false), army/generic(false,true,false); candidate currency differs from gold predicate inside sanity; no queue implied |
| City34181/34185 GetUnitForOperation | IN_SCOPE_CORE: requested primary then secondary role; recommendation only |
| City15637/15641/15645/15740 CheckForOperationUnits | IN_SCOPE_CORE: primary/secondary; purchase(true,true,false) and short build(true,false,false), constant100; direct commitment distinction |
| City15795 army purchase | IN_SCOPE_INTERFACE: (true,true,false), constant100; acquisition/result plus later recruitment |
| MilitaryAI2779/2782 | IN_SCOPE_CORE: required open-slot primary/secondary recommendation |
| MilitaryAI576/589 BuyEmergencyUnit | IN_SCOPE_INTERFACE: requested role; (false,true,false),100; gold budget priority AI_GOLD_PRIORITY_UNIT then faith fallback, native PurchaseUnit result |
| EconomicAI3246 | IN_SCOPE_INTERFACE: hurry-selected unit, (true,true,false),100; investments bypass creation; successful purchase resets demand counters, not exact promise |
| ReligionClasses6974 | IN_SCOPE_INTERFACE: faith hurry military purchase, repeats (false,true,false),100 despite faith context; matches operation/army unit or other military purchase conditions; acquisition/reserve feedback |
| ReligionClasses7496 | IN_SCOPE_INTERFACE: capital RecommendUnit(ATTACK,true), then per-city faith-native legality/net income/supply gate; shared recommendation no final city competition |
| ProcessProductionAI444 | IN_SCOPE_CORE: capital unit sanity for league reward class proxy1500, (false,false,false); [process proxy contract](../production/city-selection.md#task-1u-b-selector-and-process-interface-closure) |
| PolicyAI3798 | ADJACENT_DEFERRED strategic policy valuation interface: granted unit, (false,true,true),10; does not queue unit. Free does not skip every sanity gate; policy choice internals excluded |
| AdvisorRecommender96 | NON_AI/PRESENTATION_ONLY recommendation scoring using (false,false,false); no automatic training/acquisition ownership |

### Structured native legality contract (5.4.8)

LEGality result is bool; a failed pre-enumeration native gate removes candidate before any sanity log. City8642–8808→Player::canTrainUnit14033–14330→Plot::canTrain14261–14366; exact parameters are preserved by caller. Table links existing source-grounded predicates, not UI guesses at permission.

| Gate family / owner | Input contract / variant / failure and observability |
|---|---|
| Entry/civilization/class/player | Valid entry/class; trait NoTrain; gift-only; civilization replacement mapping; production cost; war-only; founders major/no-annexing/OCC; policy/religion option. Fails false; no per-candidate native eligibility log |
| Tech/allowed-training/upgrade | Units prereq/AND/obsolete tech; city allowed class may bypass tech; locally/globally permitted upgrade classes alter legality/sanity separately; source entries/rules state, not score flavors |
| Resource | ResourceType total and HasResourceForNewUnit p012; UNITS_RESOURCE_QUANTITY_TOTALS changes total requirement branch; continue affects held resources, purchase sets continue=false; separate aluminum scoring reservation |
| Counts/supply/city | Global/team/player/per-city unit-class max including making; supplied AI combat hard cap; settlers city minimum/very unhappy Define; air capacity/team count; city prerequisite buildings; puppet/workrate/VP; purchaseOnly. Failed predicate false; normal application clears queue then rechecks without continue flag |
| Domain/plot | Plot::canTrain: prereqResources requires water-neighbor area resource for sea or land area resources; sea city requires coast/minArea with VP canals flag; land min landmass size; outside-city domain/water/minArea checks. No hidden score contribution |
| Game/native extensions | Victory validity/project caps; no-religion, OCC, no nukes/nukes validity, league nuclear prohibition; special-unit validity/trade capacity+routes; Lua PlayerCanTrain/CityCanTrain test-all veto, registered listener state UNRESOLVED_EXTERNAL |
| Purchase boundary | IsCanPurchase30668: puppet exceptions (trait/unit/city), resistance/raze/placement when testing cost; GOLD canTrain(purchase=true), price>=0, investment not already applied, gold, civilian/combat cooldown and purchase-required building classes. FAITH: price>0, religion/enhancement/belief/era unlock, native training, domain/trait exemptions, local/global cooldown and faith balance; some specific-faith branches return early. Native purchase behavior is separate from sanity's fixed GOLD test |

Complete loaded ruleset, script listener results and live legality are UNRESOLVED_EXTERNAL; source contract is sufficient for candidate exclusion logging. Broader purchase allocation, policy decisions and unit movement remain outside this audit.

Additional candidate semantics: generic cache<=0 prevents enumeration even if a later direct sanity bonus would be large; recommendation fallback applies only cache==0, not negative. Operation primary role recommendation is compared internally only among that role's candidates; secondary runs only if NO_UNIT, not after the recommended primary later fails sanity. Unit object coefficients can therefore matter as candidate gates independently of how small the root-scaled base is in the final score.
