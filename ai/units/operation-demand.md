# Operation and army demand reaching unit production

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

How does an unfilled role become a city-visible request and a production candidate?

Required slots, reserve recruitment, city/muster access and role recommendations bridge operation state to city production. This document owns that decision bridge and its scoring wrappers; the shared operation-state contract lives in architecture.

## Reference synthesis and unresolved bridge

Demand is represented by required unfilled formation slots after reserve recruitment, not by a universal numeric desire score. City/muster/access checks expose a role request; primary recommendation is attempted before secondary fallback, rather than comparing both role scores. The operation wrapper is reported as a Define-backed base plus `(grand-strategy offense + skipped-operation count) * multiplier` plus the unit cache weight. The army wrapper is reported to omit that final unit-cache term. These are distinct paths, not interchangeable summaries.

Two unresolved baseline findings remain explicit. Army-fill candidates are injected independently but the described normal sanity branch rechecks a same-muster operation slot for both army and operation buildables; the critical review disputed calling this a confirmed behavioral bug without runtime evidence. Normal selection queues an operation buildable and resets skip pressure, while explicit slot commitment is visible in the separate city operation check. The recovered audit did not finish proving normal-queue commitment/completion. Do not close that edge by inference.

Canonical related owners: [Operation and army state contracts](<../../architecture/operations-armies.md>), [Operation creation and recruitment planning](<../operations/planning.md>), [Unit production and recommendation](<production.md>), [City production selection](<../production/city-selection.md>), [AI purchasing and emergency acquisition](<../production/purchasing.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- Unit demand created by an AI operation or operation-owned army required formation slot, then exposed to city production as `CITY_BUILDABLE_UNIT_FOR_OPERATION` or `CITY_BUILDABLE_UNIT_FOR_ARMY`. This record covers slot creation, city visibility, production/purchase pressure, and operation bookkeeping; it does not document tactical movement except where movement/recruitment creates or consumes demand.

- demand state is owned by `CvAIOperation` and `CvArmyAI` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`, `CvGameCoreDLL_Expansion2/CvAIOperation.h`, `CvGameCoreDLL_Expansion2/CvArmyAI.cpp`, and `CvGameCoreDLL_Expansion2/CvArmyAI.h`. City-facing conversion is owned by `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity*, bool&)`, `CvCity::GetUnitForOperation()`, `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`, `CvCity::CheckForOperationUnits()`, and `CvCityStrategyAI::ChooseProduction(...)`.

- operations are created by `CvPlayer::addAIOperation(AIOperationTypes, size_t iMaxMissingUnits, PlayerTypes, CvCity*, CvCity*)` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`. Military triggers include `CvMilitaryAI::UpdateOperations()`, `RequestCityAttack(...)`, `RequestPillageAttack(...)`, `RequestNukeAttack(...)`, `SetupInstantDefenses(...)`, `CheckLandDefenses(...)`, `CheckSeaDefenses(...)`, and carrier-group creation in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`. Economic/civilian triggers include `CvEconomicAI` calls to `addAIOperation(...)` for `AI_OPERATION_FOUND_CITY`, `AI_OPERATION_MERCHANT_DELEGATION`, `AI_OPERATION_DIPLOMAT_DELEGATION`, and `AI_OPERATION_MUSICIAN_CONCERT_TOUR`. `CvCity::doTurn()` calls `CvCity::CheckForOperationUnits()` for non-human unit-AI players, and normal queue selection later reaches operation/army candidates through `CvCity::doProduction(...)` -> `CvCityAI::AI_chooseProduction(...)` -> `CvCityStrategyAI::ChooseProduction(...)`, as mapped in `docs-ai/atlas/01-turn-lifecycle.md`.

## Candidates and gates

- `CvPlayer::addAIOperation(...)` creates a concrete operation through `CreateAIOperation(...)`, checks `CvAIOperation::PreconditionsAreMet(...)`, pushes it into `m_AIOperations`, calls `Init(...)`, and aborts/deletes it if invalid, aborted, or if `GetNumUnitsNeededToBeBuilt()` exceeds `iMaxMissingUnits`. `AIOperationTypes` are declared in `CvAIOperation.h` and include `AI_OPERATION_FOUND_CITY`, `AI_OPERATION_PILLAGE_ENEMY`, `AI_OPERATION_CITY_ATTACK_LAND`, `AI_OPERATION_CITY_ATTACK_NAVAL`, `AI_OPERATION_CITY_ATTACK_COMBINED`, `AI_OPERATION_CITY_DEFENSE`, `AI_OPERATION_RAPID_RESPONSE`, `AI_OPERATION_NAVAL_SUPERIORITY`, `AI_OPERATION_NUKE_ATTACK`, `AI_OPERATION_CARRIER_GROUP`, `AI_OPERATION_MUSICIAN_CONCERT_TOUR`, `AI_OPERATION_MERCHANT_DELEGATION`, and `AI_OPERATION_DIPLOMAT_DELEGATION`.

- operation initialization creates army slot state. `CvAIOperationMilitary::Init(...)` and civilian operation init paths call `CvAIOperation::AddArmy(...)`, and `CvArmyAI::SetFormation(MultiunitFormationTypes)` builds `m_FormationEntries` from `CvMultiUnitFormationInfo::getFormationSlotEntry(...)`, preserving each `CvFormationSlotEntry::m_requiredSlot`. `MultiunitFormationTypes` are declared in `CvGameCoreDLLUtil/include/CvEnums.h`, with a source warning that the enum must stay in sync with `CIV5MultiunitFormationTypeInfo.xml` and `CIV5MultiunitFormationInfo.xml`.

- `CvAIOperation::SetUpArmy(...)` sets target and muster plots, sets the army goal plot, sets operation state to `AI_OPERATION_STATE_MOVING_TO_TARGET` when there are no required open slots or `AI_OPERATION_STATE_RECRUITING_UNITS` when required slots are open, then calls `CvAIOperation::GrabUnitsFromTheReserves(...)`. `GrabUnitsFromTheReserves(...)` evaluates existing units with `OperationalAIHelpers::IsUnitSuitableForRecruitment(...)`, sorts scored reserve choices, clears `m_viListOfUnitsWeStillNeedToBuild`, and for every free army slot either assigns a reserve via `FindBestFitReserveUnit(...)` or pushes an `OperationSlot(m_iID, pArmy->GetID(), iSlot)` for required slots with no reserve unit.

- operation demand can also be created after initial setup when assigned units disappear. `CvAIOperation::UnitWasRemoved(int iArmyID, int iSlotID)` pushes `OperationSlot(m_iID, iArmyID, iSlotID)` back into `m_viListOfUnitsWeStillNeedToBuild` if the operation is still in `AI_OPERATION_STATE_RECRUITING_UNITS`.

- the city-facing operation slot is only the front unbuilt slot for an operation. `CvAIOperation::PeekAtNextUnitToBuild()` returns `m_viListOfUnitsWeStillNeedToBuild.front()` or an invalid `OperationSlot`; `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` scans `m_AIOperations`, requires a muster plot, sets `bCitySameAsMuster` only when the city is the muster owning city and has access to the muster landmass/ocean, skips naval operations when the city lacks muster access, requires the army slot still be free, and returns the first slot for which `OperationalAIHelpers::IsSlotRequired(...)` is true.

- `CvCity::GetUnitForOperation()` only returns a unit when `PeekAtNextUnitToBuildForOperationSlot(...)` returns a valid same-muster slot. It reads `CvArmyAI::GetSlotInfo(slotId)` and asks `CvUnitProductionAI::RecommendUnit(slotEntry.m_primaryUnitType, true)`, then `RecommendUnit(slotEntry.m_secondaryUnitType, true)` if needed. `RecommendUnit(UnitAITypes, bool)` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp` loops all `CvUnitEntry` rows, requires `CvUnitEntry::GetUnitAIType(eUnitAIType)`, optionally filters strategic-resource units, requires `CvCity::canTrain(...)`, uses cached `m_UnitAIWeights` or a fallback sum of unit flavors, reweights by turns left, stable-sorts, logs, and returns the top trainable `UnitTypes`.

- `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` is a broader required-army-slot query. It scans all player `CvArmyAI` objects, checks `CvArmyAI::GetOpenSlots(true)`, reads each `CvFormationSlotEntry` through `CvArmyAI::GetSlotInfo(...)`, asks the city `RecommendUnit(...)` for primary then secondary `UnitAITypes`, and prefers the army with the fewest required open slots.

## Scoring and inputs

- `CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_OPERATION` before generic unit loops when `CvCity::GetUnitForOperation()` returns a unit. Its precheck weight is `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT` plus `(CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE) + CvMilitaryAI::GetNumberOfTimesOpsBuildSkippedOver()) * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER` plus `CvUnitProductionAI::GetWeight(eUnitForOperation)`. When the candidate is pushed, `CvMilitaryAI::BumpNumberOfTimesOpsBuildSkippedOver()` is called.

- `CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_ARMY` when `CvMilitaryAI::GetUnitTypeForArmy(GetCity())` returns a unit. Its precheck weight is `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT` plus `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE) * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`. The source defaults for these define-backed constants are in `CvGameCoreDLL_Expansion2/CvGlobals.cpp`: `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT = 5000`, `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER = 250`, and `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT = 750`; `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql` also defines `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`.

- normal production sanity handles `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` in the same branch in `CvCityStrategyAI::ChooseProduction(...)`: it re-peeks `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, requires a valid same-muster operation slot, then calls `CvUnitProductionAI::CheckUnitBuildSanity(eUnitType, true, precheckWeight, false, false)`. If accepted and `CvPlayer::IsEarlyExpansionPhase()` is true, the score is tripled. STALE_OR_MISMATCH: the comment says `CITY_BUILDABLE_UNIT_FOR_ARMY` is a "useful unit", but audited normal-production source still requires a same-muster operation slot before accepting both operation and army buildables; the older broad "army-fill" description is incomplete without this shared validation branch.

- purchase/hurry sanity is different. In `CvCityStrategyAI::ChooseHurry(...)`, `CITY_BUILDABLE_UNIT_FOR_OPERATION` requires a valid same-muster operation slot and calls `CheckUnitBuildSanity(..., bForOperation=true, bForPurchase=true)`, while `CITY_BUILDABLE_UNIT_FOR_ARMY` calls `CheckUnitBuildSanity(..., bForOperation=false, bForPurchase=true)` and does not use the same operation-slot validation branch.

- operation type, target city/plot, muster city/plot, enemy player, operation preconditions, `iMaxMissingUnits`, army formation, required slot flags, primary/secondary `UnitAITypes`, reserve-unit suitability and path reachability, city access to muster landmass/ocean, city trainability/purchase legality, cached city unit weights, `FLAVOR_OFFENSE` from `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`, operation skipped-build count from `CvMilitaryAI`, city treasury/income, war state, and unit production sanity inputs from `CvUnitProductionAI::CheckUnitBuildSanity(...)`.

- city-facing operation demand uses unit flavor weights indirectly through `CvUnitProductionAI::RecommendUnit(...)` and `CvUnitProductionAI::GetWeight(UnitTypes)`, which are populated from city latest flavors and `CvUnitEntry::GetFlavorValue(...)`. The operation/army wrapper weights additionally read `FLAVOR_OFFENSE` through `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`. No source in this trace showed `CvAIOperation::GrabUnitsFromTheReserves(...)` using flavor values; it scores existing reserve units by suitability, power, reachability, own-territory/chokepoint adjustments, and a citadel-border extra score.

- required formation slots, same-muster city checks, landmass/ocean access, naval-operation city access, primary/secondary role fallback, `CvCity::canTrain(...)`, production turns, purchase legality, economic purchase permission, treasury income, current queue already-training guard in `CheckForOperationUnits()`, war-front insertion behavior, early-expansion score tripling, reserve path reachability, operation state, and missing/committed slot queues are direct non-flavor controls.

## Application and state

- normal `CvCityStrategyAI::ChooseProduction(...)` applies a selected `CITY_BUILDABLE_UNIT_FOR_OPERATION` or `CITY_BUILDABLE_UNIT_FOR_ARMY` by calling `CvCity::pushOrder(ORDER_TRAIN, eUnitType, pkUnitInfo->GetDefaultUnitAIType(), ...)`. It resets `CvMilitaryAI::ResetNumberOfTimesOpsBuildSkippedOver()` when the selected buildable type is `CITY_BUILDABLE_UNIT_FOR_OPERATION`. OPEN_QUESTION: this normal production application does not visibly call `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)`; the explicit operation-slot commit path is in `CvCity::CheckForOperationUnits()`.

- `CvCity::CheckForOperationUnits()` commits and completes operation demand through city/operation bookkeeping. When it pushes a non-appended order for an operation slot, it calls `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(thisOperationSlot)` and stores `CvCity::m_unitBeingBuiltForOperation`. If it purchases the unit and `m_unitBeingBuiltForOperation` is valid, it calls `CvPlayerAI::CityFinishedBuildingUnitForOperationSlot(...)`; otherwise it tries to recruit the purchased unit into a recruiting operation through `CvAIOperation::RecruitUnit(...)`, prioritizing operations with fewer required open slots.

- the operation bookkeeping methods move slots between pending and committed lists. `CvAIOperation::CommitToBuildNextUnit(OperationSlot)` moves the matching slot from `m_viListOfUnitsWeStillNeedToBuild` to `m_viListOfUnitsCitiesHaveCommittedToBuild`; `UncommitToBuildUnit(...)` pushes it back to the front of the needed deque; `FinishedBuildingUnit(...)` removes it from committed slots and calls `CheckTransitionToNextStage()` if no unbuilt slots remain. `CvPlayerAI::CityFinishedBuildingUnitForOperationSlot(...)` also calls `CvArmyAI::AddUnit(pThisUnit->GetID(), slotId, true)`.

- cleanup paths uncommit city-promised operation units. `CvCity::kill(...)` and order cancellation/removal code in `CvCity.cpp` call `CvPlayerAI::CityUncommitToBuildUnitForOperationSlot(m_unitBeingBuiltForOperation)` and invalidate the city member when `m_unitBeingBuiltForOperation.IsValid()`.

- `CvAIOperation::BuyFinalUnit()` is an operation-owned immediate-purchase consumer. When an operation is in `AI_OPERATION_STATE_RECRUITING_UNITS` and exactly one unit remains in `m_viListOfUnitsWeStillNeedToBuild`, it chooses a nearby friendly city, reads the missing slot's primary `UnitAITypes`, calls `CvMilitaryAI::BuyEmergencyUnit(...)`, adds the purchased unit to the army slot, pops the missing-slot deque, and calls `CheckTransitionToNextStage()` if no missing or committed units remain. `CvMilitaryAI::MakeEmergencyPurchases()` can call `BuyFinalUnit()`.

- operation demand affects normal city queue competition, direct city-turn purchases/investments, city operation-slot commitment state, army slot filling through `CvArmyAI::AddUnit(...)`, operation transition out of recruiting through `CvAIOperation::CheckTransitionToNextStage()`, and operation aborts when setup has no suitable units. INFERRED because operation/army candidates share the city production vector, increasing their weight changes opportunity cost against buildings, wonders, projects, processes, and generic units.

- Operation side effect: when a selected unit candidate is `CITY_BUILDABLE_UNIT_FOR_OPERATION`, `CvCityStrategyAI::ChooseProduction(...)` calls `CvMilitaryAI::ResetNumberOfTimesOpsBuildSkippedOver()`. INFERRED completed-unit assignment back into the originating operation/army is outside this selector and should be traced through `CvAIOperation::CommitToBuildNextUnit(...)`, `UncommitToBuildUnit(...)`, `FinishedBuildingUnit(...)`, and army recruitment before making stronger lifecycle claims.

## Data and configuration

- source types include `CvAIOperation`, derived operation classes from `CreateAIOperation(...)`, `CvArmyAI`, `CvArmyFormationSlot`, `OperationSlot`, `CvMultiUnitFormationInfo`, `CvFormationSlotEntry`, `CvUnitProductionAI`, `CvUnitEntry`, `CvCityBuildable`, `CvWeightedVector<CvCityBuildable>`, `CvGrandStrategyAI`, `CvMilitaryAI`, and `CvEconomicAI`. enums include `AIOperationTypes`, `AIOperationState`, `ArmyType`, `MultiunitFormationTypes`, `UnitAITypes`, `UnitTypes`, `CityBuildableType`, `OrderTypes`, `FlavorTypes`, `PlayerTypes`, and `YieldTypes`. database/schema assumptions include multi-unit formation XML matching `MultiunitFormationTypes`, unit `UnitAITypes`, `Unit_Flavors`, and define-backed operation/army production weights.

## Observability and validation obligations

- operation setup and reserve search log through `CvAIOperation::LogOperationStart()`, `LogOperationSpecialMessage(...)`, and `LogOperationEnd()` in `CvAIOperation.cpp`; `GrabUnitsFromTheReserves(...)` logs the number of potential reserve units when AI logging is enabled. production-side logs include `CvCityStrategyAI::LogPossibleBuilds(...)`, `LogInvalidItem(...)`, `LogCityProduction(...)`, and `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)`. `CvCity::CheckForOperationUnits()` logs purchase/build actions with `CvCityStrategyAI::LogHurryMessage(...)`.

## Version differences, risks and unresolved questions

Open Question: where exactly are `CvAIOperation::CommitToBuildNextUnit(...)`, `UncommitToBuildUnit(...)`, and `FinishedBuildingUnit(...)` called in the audited source, and are they still connected to city queue completion? The operation queue structures are verified, but this pass did not fully trace their city-production lifecycle.

Open Question: are any `CvArmyAI` objects created outside `CvAIOperation::AddArmy(...)`? This pass found operation-owned army creation and generic player army iteration, but did not prove the absence of all non-operation creators.

Open Question: the full `CvCity::canTrain(...)` and `CvCity::PurchaseUnit(...)` constraint trees are broad and were treated as boundaries. They should be traced separately for database, Lua/event, resource, and synchronization side effects.

Open Question: tactical/homeland movement can affect later production inputs by changing garrisons, danger, exposed-city state, and operation army health, but only direct production/purchase/demand paths were documented here.

savegame-sensitive state includes `CvAIOperation::m_viListOfUnitsWeStillNeedToBuild`, `m_viListOfUnitsCitiesHaveCommittedToBuild`, operation state/abort reason, `CvArmyAI::m_FormationEntries`, military recommended-count fields, and serialized city/unit production flavor weights.

enum-sensitive areas include `AIOperationTypes`, `AIOperationState`, `AIOperationAbortReason`, `ArmyType`, `ArmyAIState`, `MultiunitFormationTypes`, `UnitAITypes`, `CityBuildableType`, `ProductionSkipReason`, and database-backed flavor/strategy/unit enums.

synchronization/determinism-sensitive areas include operation creation IDs from `GC.getGame().GetNextGlobalID()`, seeded random choices in production/purchase helpers, and AI decisions that change unit creation timing.

XML/schema-sensitive inputs include `AIMilitaryStrategy_Player_Flavors`, `AIMilitaryStrategy_City_Flavors`, `AIEconomicStrategy_Player_Flavors`, `AIEconomicStrategy_City_Flavors`, `AICityStrategy_Flavors`, `AIGrandStrategy_Flavors`, `AIGrandStrategy_FlavorMods`, multi-unit formation slot definitions, unit `UnitAITypes`, unit flavors, and custom-mod options such as `MOD_AI_UNIT_PRODUCTION`.

behavioral coupling is high. Operation slot demand, army fill demand, generic unit flavors, economic purchase logic, faith purchases, tactical emergency purchases, and city buildable competition all meet at city/unit selection helpers. Changing one layer can alter not only unit type choice but also whether the city builds a unit instead of a building, project, process, or wonder.

OPEN_QUESTION: `CvCityStrategyAI::ChooseProduction(...)` queues selected `CITY_BUILDABLE_UNIT_FOR_OPERATION` with `CvCity::pushOrder(...)` and resets skipped-operation pressure, but the inspected snippet does not show it calling `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)` on that normal final-selection path. The root-level `CvCity` operation path does commit when it pushes with `bAppend=false`. A future operation-lifecycle page should trace all callers and cancellations for `CommitToBuildNextUnit`, `UncommitToBuildUnit`, and `FinishedBuildingUnit`.

- Risk notes: RISK savegame-sensitive state includes `CvAIOperation::Serialize(...)` visitors for `m_viListOfUnitsWeStillNeedToBuild` and `m_viListOfUnitsCitiesHaveCommittedToBuild`, `CvArmyFormationSlot` serialization in `CvArmyAI.cpp`, and city member `CvCity::m_unitBeingBuiltForOperation`. RISK enum/schema-sensitive areas include `AIOperationTypes`, `MultiunitFormationTypes`, `UnitAITypes`, and formation XML ordering/slot contents. RISK deterministic behavior depends on city production seeded selection and stable sort order in `CvUnitProductionAI::RecommendUnit(...)`. RISK operation-slot bookkeeping is split: normal `ChooseProduction(...)` can queue an operation buildable, while explicit `CityCommitToBuildUnitForOperationSlot(...)` is visible in `CvCity::CheckForOperationUnits()` rather than the normal final selector. RISK city purchase/investment paths can bypass or alter normal queue competition and immediately consume operation demand.

- Known improvement hooks: OPEN_QUESTION verify whether any non-operation standing armies are created outside `CvAIOperation::AddArmy(...)`; current trace found army demand mostly through operation-owned armies but source exposes generic player army iteration. OPEN_QUESTION trace `CvCity::pushOrder(...)`, `CvCity::produce(...)`, and `CvCity::CreateUnit(...)` for a queued normal `CITY_BUILDABLE_UNIT_FOR_OPERATION` to confirm whether and how operation-slot commitment/completion is associated when the unit finishes. DESIGN_HYPOTHESIS a future operation-demand table should map each `AIOperationTypes` value to `OperationalAIHelpers::GetArmyFormationForOpType(...)`, required formation slots, primary/secondary `UnitAITypes`, and source data rows.

## Source and dependency lookup

final queued unit production is still selected by the city production path documented in `docs-ai/ai-unit-production.md`: `CvCity::doProduction(...)` -> `CvCityAI::AI_chooseProduction(...)` -> `CvCityStrategyAI::ChooseProduction(...)` in `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, and `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`.

`CvHomelandAI` is not a direct unit production requester in inspected source. Its `CvHomelandAI::Update(...)` recruits and moves existing units, plans worker improvements, and assigns homeland moves. It can indirectly affect production through city/plot state that other systems read, but this pass found no `CvHomelandAI` call to `pushOrder`, `ChooseProduction`, `PurchaseUnit`, `GetUnitForOperation`, or `GetUnitTypeForArmy`.

there are non-city-queue unit purchases. `CvMilitaryAI::BuyEmergencyUnit(...)`, `CvTacticalAI::PlotEmergencyPurchases(...)`, `CvAIOperation::BuyFinalUnit()`, `CvEconomicAI` hurry/purchase logic, and faith purchases in `CvReligionAI` can call `CvCity::PurchaseUnit(...)`, bypassing `CvCityStrategyAI::ChooseProduction(...)` queue selection while still often sharing `CvUnitProductionAI::RecommendUnit(...)` and/or `CheckUnitBuildSanity(...)`.

operations are created through `CvPlayer::addAIOperation(AIOperationTypes, size_t iMaxMissingUnits, PlayerTypes, CvCity*, CvCity*)` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`. It rejects human unit-AI players, constructs the operation via `CreateAIOperation(...)`, checks `CvAIOperation::PreconditionsAreMet(...)`, pushes the operation into `m_AIOperations`, calls `Init(...)`, and aborts/deletes it if it is invalid, aborted, or needs more built units than `iMaxMissingUnits`.

`CreateAIOperation(...)` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp` maps `AIOperationTypes` to concrete classes including `CvAIOperationCityAttackLand`, `CvAIOperationCityAttackNaval`, `CvAIOperationCityAttackCombined`, `CvAIOperationPillageEnemy`, `CvAIOperationNavalSuperiority`, `CvAIOperationDefendCity`, `CvAIOperationDefenseRapidResponse`, `CvAIOperationNukeAttack`, `CvAIOperationCarrierGroup`, and civilian operations such as `CvAIOperationCivilianFoundCity`, `CvAIOperationCivilianMerchantDelegation`, `CvAIOperationCivilianDiplomatDelegation`, and `CvAIOperationCivilianConcertTour`.

military operations are requested from several upstream sources:

- `CvMilitaryAI::RequestCityAttack(...)`, `RequestPillageAttack(...)`, `RequestNukeAttack(...)`, and `RequestBullyingOperation(...)`.

- `CvMilitaryAI::SetupInstantDefenses(...)`, `CheckLandDefenses(...)`, `CheckSeaDefenses(...)`, `DoCityAttacks(...)`, `DoNuke(...)`, and carrier-group creation in `UpdateOperations()`-area code.

- diplomacy and deal code call `CvMilitaryAI::RequestCityAttack(...)` in `CvGameCoreDLL_Expansion2/CvDiplomacyAI.cpp` and `CvGameCoreDLL_Expansion2/CvDealClasses.cpp`.

- minor-civ AI can request major-player city attacks or city-defense operations in `CvGameCoreDLL_Expansion2/CvMinorCivAI.cpp`.

- economic AI creates civilian operations such as `AI_OPERATION_FOUND_CITY`, `AI_OPERATION_MERCHANT_DELEGATION`, `AI_OPERATION_DIPLOMAT_DELEGATION`, and `AI_OPERATION_MUSICIAN_CONCERT_TOUR` in `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`.

operation initialization creates an army. `CvAIOperationMilitary::Init(...)` chooses target/muster plots, calls `AddArmy(OperationalAIHelpers::GetArmyFormationForOpType(m_eType))`, then `SetUpArmy(...)`. `CvAIOperationCivilian::Init(...)` finds a civilian, target, muster plot, adds an army, places the civilian in slot 0, optionally clears the escort slot, then calls `SetUpArmy(...)`.

`CvAIOperation::AddArmy(...)` creates a `CvArmyAI` through `CvPlayer::addArmyAI()`, stores the army ID in `m_viArmyIDs`, sets the army operation ID, state, `ArmyType`, and `MultiunitFormationTypes`. `CvArmyAI::SetFormation(...)` in `CvGameCoreDLL_Expansion2/CvArmyAI.cpp` reads `CvMultiUnitFormationInfo::getFormationSlotEntry(...)` and creates one `CvArmyFormationSlot` per formation slot, preserving required-slot flags.

`CvAIOperation::SetUpArmy(...)` sets operation target/muster plots, sets the army goal, sets operation state to moving or recruiting depending on required open slots, then calls `GrabUnitsFromTheReserves(...)`. If no suitable unit fills any slot and the army has zero filled slots, it aborts with `AI_ABORT_NO_UNITS`.

`CvAIOperation::GrabUnitsFromTheReserves(...)` scans existing units, filters them through `OperationalAIHelpers::IsUnitSuitableForRecruitment(...)`, scores reachable units, sorts choices, clears `m_viListOfUnitsWeStillNeedToBuild`, and for every free army slot calls `FindBestFitReserveUnit(...)`. Required slots with no reserve unit are pushed into `m_viListOfUnitsWeStillNeedToBuild` as `OperationSlot(m_iID, pArmy->GetID(), iSlot)`.

`CvAIOperation::PeekAtNextUnitToBuild()` returns the first entry in `m_viListOfUnitsWeStillNeedToBuild`. `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity*, bool&)` loops `m_AIOperations`, requires a muster plot, requires city/muster access for naval operations, peeks the next operation slot, verifies the army and free slot, and prefers required slots via `OperationalAIHelpers::IsSlotRequired(...)`.

`CvCity::GetUnitForOperation()` calls `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, requires the city to be the muster city (`bSameCity`) and to have landmass/ocean access, reads `CvArmyAI::GetSlotInfo(slotId)` to get primary/secondary `UnitAITypes`, and asks `CvUnitProductionAI::RecommendUnit(...)` for a trainable unit for the primary role, then secondary role.

`CvCityStrategyAI::ChooseProduction(...)` injects a `CITY_BUILDABLE_UNIT_FOR_OPERATION` candidate if `CvCity::GetUnitForOperation()` returns a unit. Its weight starts with `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, adds `FLAVOR_OFFENSE` from `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`, adds `CvMilitaryAI::GetNumberOfTimesOpsBuildSkippedOver()`, multiplies by `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`, then adds `CvUnitProductionAI::GetWeight(unit)`. When the candidate is pushed, `CvMilitaryAI::BumpNumberOfTimesOpsBuildSkippedOver()` is called; after a selected operation unit is queued, `ResetNumberOfTimesOpsBuildSkippedOver()` is called.

operation city commitment exists but is not visibly called in the inspected `ChooseProduction(...)` snippets. `CvAIOperation::CommitToBuildNextUnit(...)`, `UncommitToBuildUnit(...)`, and `FinishedBuildingUnit(...)` maintain `m_viListOfUnitsWeStillNeedToBuild` and `m_viListOfUnitsCitiesHaveCommittedToBuild`; the city production selection path re-peeks the operation slot before accepting/pushing a candidate.

`CvMilitaryAI::IsBuildingArmy(ArmyType)` scans all player `CvArmyAI` objects and returns true if any matching army has required open slots.

`CvMilitaryAI::GetUnitTypeForArmy(CvCity* pCity)` scans all player armies, reads `CvArmyAI::GetOpenSlots(true)` for required free slots, gets each slot's `CvFormationSlotEntry` from `CvArmyAI::GetSlotInfo(...)`, and asks the city `CvUnitProductionAI::RecommendUnit(slot.m_primaryUnitType, true)` then secondary role. It chooses a trainable unit from the army with the fewest open slots.

this path uses the same army/formation-slot data structures as operations: `CvArmyAI::m_FormationEntries`, `CvArmyFormationSlot`, `CvFormationSlotEntry::m_primaryUnitType`, `m_secondaryUnitType`, and `m_requiredSlot`.

`CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_ARMY` after the operation candidate. The base weight uses `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT` plus `FLAVOR_OFFENSE * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`.

in the normal production precheck switch, both `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` enter the same case and require `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` to return a valid same-muster operation slot before `CheckUnitBuildSanity(..., bForOperation=true, ...)` accepts them. In purchase precheck, `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` are handled separately: operation purchase requires the operation slot, while army purchase calls `CheckUnitBuildSanity(..., bForOperation=false, ..., bForPurchase=true)`.

INFERRED: all armies inspected here are operation-owned, because `CvAIOperation::AddArmy(...)` creates armies and assigns `SetOperationID(m_iID)`. The source also exposes generic player army iteration, so a future pass should confirm whether any non-operation standing armies are created elsewhere. This pass did not find a separate homeland/standing army creator.

`CvHomelandAI::Update(bool bUpdateImprovements)` in `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp` recruits existing available units, optionally plans improvements/worker distribution, finds homeland targets, and assigns homeland moves. It does not directly call city production or purchase methods in inspected source.

`CvHomelandAI::RecruitUnits()` skips units already processed, unable to move, delayed-death units, and units assigned to an army. This means army/operation units are not consumed by homeland move planning once assigned.

`CvHomelandAI` is used for logging by other systems (`LogHomelandMessage(...)`), but those log calls do not make HomelandAI a production requester.

homeland-adjacent defense demand is mostly in `CvMilitaryAI` and `CvTacticalAI`, not `CvHomelandAI`: `CvMilitaryAI::SetupInstantDefenses(...)`, `CheckLandDefenses(...)`, and `CheckSeaDefenses(...)` create defensive operations; `CvTacticalAI::PlotEmergencyPurchases(...)` can buy emergency city defenders.

INFERRED: HomelandAI can indirectly change later demand by moving units, garrisoning, worker-improving, or affecting city/plot safety, but this pass found no direct path from `CvHomelandAI` to unit production scoring.

`CvMilitaryAI::SetRecommendedArmyNavySize()` is the core recommended-count producer. It writes `m_iRecLandUnits`, `m_iRecNavalUnits`, and `m_iRecExplorerUnits`; accessors in `CvMilitaryAI.h` include `GetRecommendedMilitarySize()`, `GetRecommendLandArmySize()`, `GetRecommendNavySize()`, and `GetRecommendedExplorers()`.

generic unit production reads those counts inside `CvUnitProductionAI::CheckUnitBuildSanity(...)` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`. Examples include rejecting explorers when current explorers meet `GetRecommendedExplorers()`, rejecting supplied land/naval units once recommended army/navy counts are reached, and using `MilitaryAIHelpers::IsTestStrategy_NeedNavalUnits(...)`/`NeedNavalUnitsCritical(...)` for naval scoring.

composition strategies are hardcoded by XML strategy name in `CvMilitaryAI::UpdateMilitaryStrategies()`. Default active checks include `MILITARYAISTRATEGY_ENOUGH_MILITARY_UNITS`, `EMPIRE_DEFENSE`, `EMPIRE_DEFENSE_CRITICAL`, `ENOUGH_NAVAL_UNITS`, `NEED_NAVAL_UNITS`, `NEED_NAVAL_UNITS_CRITICAL`, `WAR_MOBILIZATION`, `AT_WAR`, barbarian strategies, winning/losing wars, default ranged need/enough when `!MOD_AI_UNIT_PRODUCTION`, mobile need/enough, air need/enough, nuke need, anti-air need/enough, and air-carrier need.

role-ratio helper functions compare current role counts to grand-strategy/personality flavors. Examples:

- `MilitaryAIHelpers::IsTestStrategy_NeedRangedUnits(...)` and `EnoughRangedUnits(...)` use `FLAVOR_RANGED`, ranged/melee counts, and underrepresented army-slot checks.

- `NeedMobileUnits(...)`/`EnoughMobileUnits(...)` use `FLAVOR_MOBILE`.

- `NeedAirUnits(...)`/`EnoughAirUnits(...)` use `FLAVOR_AIR`.

- `NeedAntiAirUnits(...)`/`EnoughAntiAirUnits(...)` inspect other players' air force presence and melee/AA ratios.

- MOD-gated role-split helpers use `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, and `FLAVOR_FIGHTER`.

`CvUnitProductionAI::CheckUnitBuildSanity(...)` reads `CvPlayerAI::IsUnderrepresentedUnitType(pkUnitEntry->GetDefaultUnitAIType())` and adds a bonus when the candidate default role is underrepresented. `MilitaryAIHelpers::IsTestStrategy_NeedRangedUnits(...)` also checks `IsUnderrepresentedUnitType(UNITAI_RANGED)` or `UNITAI_CITY_BOMBARD` when an army is being built.

operation skip pressure is maintained by `CvMilitaryAI::BumpNumberOfTimesOpsBuildSkippedOver()`, `GetNumberOfTimesOpsBuildSkippedOver()`, and `ResetNumberOfTimesOpsBuildSkippedOver()` and is used directly only in the `CITY_BUILDABLE_UNIT_FOR_OPERATION` candidate weight in `CvCityStrategyAI::ChooseProduction(...)`.

city unit flavor weights are cached in `CvUnitProductionAI::m_UnitAIWeights`. `CvCityStrategyAI::FlavorUpdate()` resets production AIs and calls `CvUnitProductionAI::AddFlavorWeights(...)` for every flavor using the city strategy's latest flavor values.

`CvFlavorManager::ChangeCityFlavors(...)` iterates registered city flavor recipients and calls `CvFlavorRecipient::ChangeFlavors(...)`, which updates recipient flavor state and calls `FlavorUpdate()`. `CvCity::init(...)` registers `m_pCityStrategyAI` as a city recipient; load serialization re-registers it according to the existing source map.

economic and military strategy start/end effects are concrete production inputs. `CvEconomicAI::UpdateEconomicStrategies()` and `CvMilitaryAI::UpdateMilitaryStrategies()` apply player flavor deltas through `ChangeActivePersonalityFlavors(...)` and city flavor deltas through `ChangeCityFlavors(...)`. City flavor deltas immediately rebuild `CvUnitProductionAI` weights through `CvCityStrategyAI::FlavorUpdate()`.

new cities receive already-active player strategy city flavors through `CvCityStrategyAI::UpdateFlavorsForNewCity()`, which loops active economic and military strategies and adds each strategy's `GetCityFlavorValue(...)` to `m_piLatestFlavorValues`.

city-local strategies and city specializations also affect unit production weights. `CvCityStrategyAI::DoTurn()` starts/ends `AICityStrategyTypes`, adds/removes `CvAICityStrategyEntry::GetFlavorValue(...)`, and calls `FlavorUpdate()`. `CvCityStrategyAI::SpecializationFlavorChange(...)` applies specialization flavors.

grand strategy is partly separate from cached city flavor weights. `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` is read directly in operation/army candidate weights and in many `CvMilitaryAI`/`CvUnitProductionAI` strategy/scoring checks.

operation-slot production chain:

```text
CvPlayer::doTurn()
-> CvPlayer::doTurnPostDiplomacy()
-> CvMilitaryAI::DoTurn()
-> CvMilitaryAI::UpdateOperations() / RequestCityAttack(...) / CheckLandDefenses(...) / CheckSeaDefenses(...)
-> CvPlayer::addAIOperation(...)
-> CreateAIOperation(...)
-> CvAIOperationMilitary::Init(...) or derived Init(...)
-> CvAIOperation::AddArmy(...)
-> CvArmyAI::SetFormation(...)
-> CvAIOperation::SetUpArmy(...)
-> CvAIOperation::GrabUnitsFromTheReserves(...)
-> m_viListOfUnitsWeStillNeedToBuild contains required OperationSlot
-> CvPlayer::doTurnPostDiplomacy() city loop
-> CvCity::doTurn() / CvCity::doProduction(...)
-> CvCityAI::AI_chooseProduction(...)
-> CvCityStrategyAI::ChooseProduction(...)
-> CvCity::GetUnitForOperation()
-> CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)
-> CvUnitProductionAI::RecommendUnit(primary/secondary UnitAI)
-> CITY_BUILDABLE_UNIT_FOR_OPERATION
-> CvUnitProductionAI::CheckUnitBuildSanity(..., bForOperation=true)
-> CvCity::pushOrder(ORDER_TRAIN, unit, default UnitAI, ...)
```

army-fill candidate chain:

```text
CvAIOperation::AddArmy(...) / CvArmyAI::SetFormation(...)
-> army has required open CvArmyFormationSlot entries
-> CvCityStrategyAI::ChooseProduction(...)
-> CvMilitaryAI::GetUnitTypeForArmy(CvCity*)
-> CvArmyAI::GetOpenSlots(true)
-> CvArmyAI::GetSlotInfo(slot)
-> CvUnitProductionAI::RecommendUnit(primary/secondary UnitAI)
-> CITY_BUILDABLE_UNIT_FOR_ARMY
-> normal production precheck shares the operation-slot same-muster check in CvCityStrategyAI::ChooseProduction(...)
-> CvUnitProductionAI::CheckUnitBuildSanity(...)
-> CvCity::pushOrder(ORDER_TRAIN, unit, default UnitAI, ...)
```

strategy/flavor generic unit chain:

```text
CvPlayer::doTurnPostDiplomacy()
-> CvEconomicAI::DoTurn() and CvMilitaryAI::DoTurn()
-> UpdateEconomicStrategies() / UpdateMilitaryStrategies()
-> CvFlavorManager::ChangeCityFlavors(...)
-> CvCityStrategyAI::ChangeFlavors(...) / CvCityStrategyAI::FlavorUpdate()
-> CvUnitProductionAI::Reset()
-> CvUnitProductionAI::AddFlavorWeights(flavor, city flavor value)
-> CvCityStrategyAI::ChooseProduction(...)
-> generic CITY_BUILDABLE_UNIT from CvUnitProductionAI::GetWeight(unit)
-> CvUnitProductionAI::CheckUnitBuildSanity(...)
-> CvCity::pushOrder(ORDER_TRAIN, unit, default UnitAI, ...)
```

emergency purchase chain:

```text
CvGame::updateMoves()
-> CvPlayerAI::AI_unitUpdate(...)
-> CvTacticalAI::Update()
-> CvTacticalAI::ProcessDominanceZones()
-> CvTacticalAI::PlotEmergencyPurchases(zone)
-> CvMilitaryAI::BuyEmergencyUnit(UnitAI, city)
-> CvUnitProductionAI::RecommendUnit(UnitAI, true)
-> CvUnitProductionAI::CheckUnitBuildSanity(..., bForPurchase=true)
-> CvEconomicAI::CanWithdrawMoneyForPurchase(PURCHASE_TYPE_UNIT, ...)
-> CvCity::PurchaseUnit(unit, YIELD_GOLD or YIELD_FAITH)
```

the existing report correctly identifies `CvCityAI::AI_chooseProduction(...)` and `CvCityStrategyAI::ChooseProduction(...)` as the final city production selection path.

it correctly identifies `CvCity::GetUnitForOperation()` and `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` as special unit sources injected before generic unit candidates.

it correctly documents that `CvUnitProductionAI::RecommendUnit(...)` is a helper for operation/army and purchase-like callers, while generic production candidates use cached flavor weights and then `CheckUnitBuildSanity(...)`.

it correctly states that economic and military strategies propagate city flavor deltas through `CvFlavorManager::ChangeCityFlavors(...)` and that `CvCityStrategyAI::FlavorUpdate()` rebuilds unit production weights.

it correctly separates direct purchase paths from normal production queue selection.

consider adding a short upstream section explaining that `CvPlayer::doTurnPostDiplomacy()` calls `CvEconomicAI::DoTurn()`, `CvMilitaryAI::DoTurn()`, and `CvCitySpecializationAI::DoTurn()` before city `doTurn()` production. This is the main ordering fact that explains why player-level demand is visible to city production in the same turn.

clarify that operation demand is stored in `CvAIOperation::m_viListOfUnitsWeStillNeedToBuild` as `OperationSlot` entries, created mostly by `CvAIOperation::GrabUnitsFromTheReserves(...)` after reserve-unit recruitment fails to fill required formation slots.

clarify the source quirk in normal production precheck: `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` share a `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` same-muster validation case in `CvCityStrategyAI::ChooseProduction(...)`, while purchase precheck handles operation and army candidates separately.

add `CvTacticalAI::PlotEmergencyPurchases(...)` and `CvAIOperation::BuyFinalUnit()` to the purchase appendix if the existing report wants all non-queue unit creation paths.

INFERRED: the existing report's phrase "army-fill candidates" may imply standing defensive armies. The inspected source supports operation-owned armies clearly; a future pass should verify whether any non-operation army creation exists before describing standing armies as a separate demand class.

`CvPlayer` owns armies and operations through `TContainer<CvArmyAI> m_armyAIs` and `std::vector<std::pair<int, CvAIOperation*> > m_AIOperations` in `CvPlayer.h`. It exposes army iteration/accessors (`firstArmyAI`, `nextArmyAI`, `getArmyAI`, `addArmyAI`, `deleteArmyAI`) and operation accessors (`getAIOperation`, `getNumAIOperations`, `getAIOperationByIndex`, `deleteAIOperation`). Source: `CvPlayer.h`; `CvPlayer.cpp`, army/operation accessor block.

`CvPlayer::addAIOperation(AIOperationTypes eOperationType, size_t iMaxMissingUnits, PlayerTypes eEnemy, CvCity* pTarget, CvCity* pMuster)` rejects human unit-AI players, creates the operation with `CreateAIOperation(...)`, checks `CvAIOperation::PreconditionsAreMet(...)`, pushes the operation into `m_AIOperations` before `Init(...)`, aborts if the operation needs more built units than `iMaxMissingUnits`, and deletes/returns null for invalid or aborted operations. Source: `CvPlayer.cpp`, `CvPlayer::addAIOperation`; `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`, `CreateAIOperation`.

`CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity*, bool&)` is the player-level bridge from operation demand to city production. It loops `m_AIOperations`, requires a muster plot, checks city access/same-muster conditions, calls `CvAIOperation::PeekAtNextUnitToBuild()`, verifies the army and free slot with `CvArmyAI::GetSlotStatus(...)`, and prefers required slots through `OperationalAIHelpers::IsSlotRequired(...)`. Source: `CvPlayerAI.cpp`, `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot`.

`CvCity::GetUnitForOperation()` converts an operation slot into a concrete trainable unit. It calls `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, requires same muster city, reads the slot's primary and secondary `UnitAITypes` through `CvArmyAI::GetSlotInfo(...)`, and asks `CvUnitProductionAI::RecommendUnit(...)` for a trainable unit. Source: `CvCity.cpp`, `CvCity::GetUnitForOperation`; `CvGameCoreDLLUtil/include/CvEnums.h`, `UnitAITypes`.

`CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_OPERATION` when `CvCity::GetUnitForOperation()` returns a unit. The weight uses `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, `FLAVOR_OFFENSE` via `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`, `CvMilitaryAI::GetNumberOfTimesOpsBuildSkippedOver()`, `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`, and `CvUnitProductionAI::GetWeight(...)`; it bumps skipped-operation pressure via `CvMilitaryAI::BumpNumberOfTimesOpsBuildSkippedOver()`. Source: `CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction`.

`CvCityStrategyAI::ChooseProduction(...)` also injects `CITY_BUILDABLE_UNIT_FOR_ARMY` from `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`. Final selected unit candidates are queued with `CvCity::pushOrder(ORDER_TRAIN, eUnitType, eUnitAI, ...)`; selected operation units reset skipped-operation pressure through `CvMilitaryAI::ResetNumberOfTimesOpsBuildSkippedOver()`. Source: `CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction`.

Operation build commitment functions live on `CvPlayerAI`: `CityCommitToBuildUnitForOperationSlot(...)` calls `CvAIOperation::CommitToBuildNextUnit(...)`; `CityUncommitToBuildUnitForOperationSlot(...)` calls `CvAIOperation::UncommitToBuildUnit(...)`; `CityFinishedBuildingUnitForOperationSlot(...)` adds the unit to the army with `CvArmyAI::AddUnit(...)` and calls `CvAIOperation::FinishedBuildingUnit(...)`. Source: `CvPlayerAI.cpp`, these functions.

`CvCity` has a second root-level operation-unit handling path that can purchase or queue a unit for an active operation, uses `CvEconomicAI::CanWithdrawMoneyForPurchase(...)`, `CvCity::PurchaseUnit(...)`, `CvCity::pushOrder(ORDER_TRAIN, ...)`, `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)`, and `CvPlayerAI::CityFinishedBuildingUnitForOperationSlot(...)`. Source: `CvCity.cpp`, operation unit purchase/queue block near `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`.

RISK: Operation and army state is serialized in `CvPlayer::Serialize` by operation type and operation object contents, plus `m_armyAIs`. Changes to `AIOperationTypes`, `OperationSlot`, formation slots, army IDs, or operation serialization can affect saves and city production demand. Source: `CvPlayer.cpp`, `CvPlayer::Serialize`; `CvGameCoreDLLUtil/include/CvEnums.h`, `AIOperationTypes`.

- queued operation/army unit candidates do not have a separate final selector. They compete in the same `CvWeightedVector<CvCityBuildable>` as units, buildings, projects, and processes inside `CvCityStrategyAI::ChooseProduction(...)`. After sanity checks, duration reweighting, current-production inertia, and special process handling, final production uses `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold(...)` with `CvHandicapInfo::getCityProductionChoiceCutoffThreshold()` and seed `CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID())`, as documented for city production in `docs-ai/atlas/decisions/city-production.md`.

- `CvCity::CheckForOperationUnits()` is a separate city-turn pressure path, not the normal combined city-production selector. It skips razing, puppet-annexable, minor, barbarian, human-automated, indebted, and already-training cities, then re-peeks a same-muster operation slot, recommends primary/secondary role units, and first tries purchase/investment logic using `CvUnitProductionAI::CheckUnitBuildSanity(..., bForOperation=true, bForPurchase=true)`, `CvEconomicAI::CanWithdrawMoneyForPurchase(PURCHASE_TYPE_UNIT, ...)`, `CvCity::IsCanPurchase(...)`, unit investment, or `CvCity::PurchaseUnit(...)`. If the purchase-sanity score is not positive, the unit is under seven turns away, and non-purchase operation sanity passes, it can `pushOrder(ORDER_TRAIN, ...)`; when the player is at war it inserts at the front and commits the operation slot.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U current operation bridge

Partial revalidation at `src-20261029-vp-c285a18`, pass `audit-20261029-task1u-01`. Exact current sources: `CvPlayerAI.cpp:1288–1363`, `CvCity.cpp:GetUnitForOperation` (34157), `CheckForOperationUnits` (15581), `CvCityStrategyAI.cpp:698–1148`, `CvAIOperation.cpp:310–373`, `CvMilitaryAI.cpp:2767`. The older whole lifecycle above has not been fully promoted to current verification.

`PeekAtNextUnitToBuildForOperationSlot` scans operations in stored order, ignores missing muster, computes same-city from muster owning city plus landmass/ocean access, skips inaccessible naval muster, reads only front needed slot, requires valid army/free slot and IsSlotRequired, then stops at first qualifying required slot. It can return a slot from a different muster; callers must inspect the same-city boolean. It does not search all slots for best unit score.

`GetUnitForOperation` requires valid same-muster slot, operation/army and access, then primary RecommendUnit, secondary only if primary yields NO_UNIT. `GetUnitTypeForArmy` scans every army's open required slots and chooses first recommendable slot from the army with strictly fewer missing slots than the current best. Equal open-slot counts preserve earlier choice. It has no own same-muster gate, but shared city sanity subsequently requires a valid same-muster operation slot **for both operation and army buildables**. The chosen army candidate need not identify the operation used by that second check. This is source-confirmed coupling, not proof of intended design.

`ChooseProduction` applies the same ORDER_TRAIN call using the unit's **default** UnitAI to generic/army/operation selections, and resets skipped-operation count for the operation tag. It does not call CityCommitToBuildUnitForOperationSlot or assign `m_unitBeingBuiltForOperation`. Therefore an operation buildable tag alone does not promise/reserve the requested slot.

`CheckForOperationUnits` is a distinct direct-acquisition path. Its nonpurchase short-build branch refuses turns>=7, reruns operation sanity and queues with requested role. When !bAppend it calls commit, stores the OperationSlot in `m_unitBeingBuiltForOperation`, resets skips. Purchase branch uses gold sanity/affordability/IsCanPurchase; investment option substitutes spending/investment/possible queue for unit creation. Successful direct PurchaseUnit cleans queue and, if stored slot valid, calls CityFinishedBuildingUnitForOperationSlot then invalidates; otherwise it tries recruiting into recruiting operations sorted by first-army required open slots. This is distinct from normal shared selection.

Commit moves matching needed token to committed vector; uncommit moves it back to needed front; finish erases committed token and checks stage transition when needed deque empty. Player finish first AddUnit(unit ID,slot,required), then operation FinishedBuildingUnit. `CvCity::popOrder` uncommits/invalidate stored token after order-specific handling; city destruction also releases. Exhaustive search of CityFinishedBuildingUnitForOperationSlot found only the direct purchase call in CvCity.cpp plus its definition: **no ordinary training completion call to this wrapper was found**. Existing reserve recruitment can later fill units, but exact ordinary completion→reserve→stage timing remains unresolved. Do not equate absence of this wrapper with proof no other indirect assignment path exists.

A source search for `addArmyAI()` found the generic definition and `CvAIOperation::AddArmy` caller; no independent non-operation standing-army creator was found in the searched C++ files. GetUnitTypeForArmy's broad scan does not itself prove standing armies exist. Formation loaders, all operation creation predicates, recruitment score terms, serialization of the city promise token and full completion/abort transitions still require current verification.

Observability: operation tag/default role/aggregate choice cannot correlate a normal queue to requested operation/army/slot. See [logging contract](../infrastructure/observability.md#task-2u-unit-logging-requirements).

### Current recruitment, promise and completion refinement

Additional current evidence: CvAIOperation.cpp446–547,730–738,1285–1346,1430–1475,3200 onward; CvArmyAI.cpp333; CvCity.cpp29855–29992 and30125–30230. Required slot generation now directly revalidated: rebuild needed deque, attempt every free required/optional formation slot, add a needed token only for failed required slot. SetFormation releases units and creates free entries from formation required flags when formation changes. Formation selection maps civilian operations to settler/concert/diplomat/merchant escort; pillage→FAST_PILLAGERS, nuke→NUKE_ATTACK, carrier→CARRIER_ESCORT, land/combined/naval attacks→current-best-formation helpers; naval superiority→NAVAL_SQUADRON, rapid response→RAPID_RESPONSE_FORCE, defense→CLOSE_CITY_DEFENSE. Dynamic best-formation helpers remain unresolved.

Reserve candidate score exactly `10000 + power + borderCitadelBonus - adjustedTurns*30`; borderCitadelBonus is **+42**, despite comment preferring units not guarding borders. adjustedTurns=path turns, +1 outside owner territory, +1 for land unit in other muster area. Recruitment needs suitability and path<=maximum recruit turns; rough reach range=maximum recruit turns*4, disallow ocean if player cannot cross. Suitability rejects cannot-use-for-operation, healing, target distance<TACTICAL_COMBAT_MAX_TARGET_DISTANCE, recent deployment, HP*100<maxHP*AI_OPERATIONAL_PERCENT_HEALTH_FOR_OPERATION, deep-ocean inability, active EXPLORE/EXPLORE_SEA/SETTLE roles, required embark inability and rough unreachable location. Role matching/native path/canUse/getPower remain dependencies. Candidate stable ordering and per-slot fit may suppress unit demand, but these are not city production weights.

SetUpArmy enters moving when no required open slots, recruiting otherwise; reserve pass may abort if no recruits and no filled slots, then stage-transition check twice. Waiting-for-reinforcement operation processing calls GrabUnitsFromTheReserves again (line737). This provides a current indirect route for newly trained unassigned units, subject to suitability; it is not an immediate promised-slot completion acknowledgment.

`m_unitBeingBuiltForOperation` has no occurrence in CvCity::Serialize in the symbol search; construction/init invalidate it, direct short-build assigns, purchase finishes and popOrder/destruction uncommit. This supports a missing persisted-token concern, not a runtime save/load finding. Ordinary ORDER_TRAIN completion calls produce(unit) then popOrder's unconditional token uncommit; produce/CreateUnit have no explicit promised-slot completion callback. The normal queue tag and direct promised short-build therefore remain distinct. Consequence: promise may return to needed at completion; later reserve rebuild can fill it. Exact inter-turn timing and stage outcomes remain PARTIAL.

CheckForOperationUnits entry gates: puppet/raze, minor/barbarian, human automation, treasury AverageIncome100(10)<0, **any** ORDER_TRAIN anywhere in queue suppresses. bAppend=false at major war, true otherwise. Its pre-sanity weight is CODE_CONSTANT100, unlike shared wrapper weights. Append short-build does not commit/store token. These direct-path conditions belong in logging, not an assumption all same-muster candidates follow normal selector.

## Task 1U-B lifecycle closure on VP 5.4.8

src-20261029-vp-548 / audit-20261029-task1u-b revalidates prior operation bridge and recruitment sections. Slot promises are not merged with candidate tags. The following feedback path is established in current source, not runtime:

1. Military/economic request supplies operation type/enemy/target/muster/maxMissing. Player::addAIOperation42185 rejects human-unit-AI control, creates typed operation/global ID, calls PreconditionsAreMet before registration, registers before Init, then aborts/deletes invalid initialization or needed>maxMissing. Operation target selection is an independent strategic-request interface (owner planning.md); its IDs/type/limits must be logged. Reserve availability preconditions use fake formation status and same suitability, not city flavor scores.
2. Dynamic formation helpers MilitaryAI4298–4331: combined/pure naval use game current era>MEDIEVAL to choose late invasion/pure naval; land era0 early rush;1/2 small;3/4 basic;5–7 bigger; default basic. This is global game era, not player's era. Formation cache/required flags feed AddArmy and SetFormation as previously mapped. Only located addArmyAI caller is operation AddArmy; no separately created standing armies in searched gameplay code.
3. Recruitment first matched role index accepts wildcard primary NO_UNITAI, current unit AI primary/secondary or entry multi-role primary/secondary. canUseForAIOperation(Unit7406) rejects assigned army, needed garrison, nonfriendly city zone, nearby enemy defender inside ring3 or contested border citadel; ordinary non-tactical civilian can pass early. Additional health/recent-deployment/path filters and score10000+power+42-border condition−30*adjustedTurns retain previous mapping. Power/path/native ownership are reserve-suitability interfaces, not city candidate flavor weights.
4. Required failed reserve slots enter needed deque; optional ones can recruit but do not independently enqueue demand. Normal shared city operation tag leaves needed token outstanding while training, stores no operation identity in ORDER_TRAIN and defaults role. Hence simultaneous/repeated requests can see same outstanding slot; no unique reservation is implied. Supply/counters/current queues constrain actual duplicate production. This is source behavior; describing reserve fulfillment as intended design remains INFERRED.
5. City::CheckForOperationUnits runs in city DoTurn2507 after strategy update/cooldown changes for !human-unit-AI. It suppresses when **any** train order exists; direct operation preweight100; war→prepend/commit token; peace→append without token; short-build turns<7 and sanity must pass. Normal selector remains separate. Shared normal army sanity still requires some current same-muster operation slot and uses operation=true; helper GetUnitTypeForArmy can choose a different army. This is confirmed coupling/likely design inconsistency, not repaired. Hurry army branch differs.
6. Promise token: city constructor/init invalidate; only nonappend direct short-build sets after commit; city popOrder/destruction uncommit then invalidate; purchase branch optionally finish/invalidate. Symbol search across whole selected source confirms no token in city serialization and no ordinary completion call to CityFinishedBuildingUnitForOperationSlot. Queued order stores unit/role, not demand token. Save/load promise continuity is therefore not represented by this city token; operation needed/committed lists themselves serialize. Runtime consequences not tested.
7. Ordinary completion: popOrder produce(unit) creates potentially replaced/upgraded unit, then uncommits stored token. No direct slot assignment here. On subsequent operation processing while army WAITING_FOR_UNITS_TO_REINFORCE, GrabUnitsFromTheReserves rebuilds needs and can recruit new suitable unassigned unit; current role reassignment (including explore) may prevent recruitment. It is actual reserve eligibility/slot fill that ends demand, not finishing any unit of that type. This explains future-demand feedback even without explicit completion correlation.
8. Purchase/investment: direct successful purchase may finish stored promise (PlayerAI adds to exact army slot then FinishedBuildingUnit); otherwise tries recruiting into recruiting operations by fewest required openings, erases matched needed slot. Investments spend/set class investment and may queue but create no unit and do not demonstrate fulfillment. Economic hurry/faith emergency purchases can create reserves/reset skipped counters without explicit exact-slot commitment. Native PurchaseUnit result and recruited unit/slot must be observed independently.
9. Stage completion: ArmyAI::HaveEnoughUnits809 counts free/present required and present optional; returns presentRequired>0 && freeRequired<=presentOptional/2+maxMissing. Thus optional recruits can substitute for missing required slots; empty needed deque is not the only readiness test. Military transition1530 recruiting→gathering on this predicate with maxMissing0; gathering→moving by furthest unit distance/tolerance; civilian variant also allows civilian already reaching target; nuke variant can act directly when sufficient. Operation::DoTurn793 calls transition after army processing. Abort/release/dropped/stuck units can recreate or end demand; terminal operation is not persistent unmet demand.

Source-confirmed risks: uncommitted ordinary production; nonserialized city promise; unconditional queue-pop uncommit; shared army/muster coupling; create/queue role differences; positive border-citadel recruitment42 despite contrary comment. No behavior fix, intended-design claim or runtime reproduction. Exact request-planner and pathfinding choices remain bounded independent outputs with identities/conditions specified above; unavailable loaded profile/runtime timing is UNRESOLVED_EXTERNAL, not a missing immediate lifecycle edge.

Army-fill direct tail refinement (City15768–15870): recommendation with >=10 turns returns; then requires some alive other player with IsWantsSneakAttack. Constant100 operation/purchase sanity precedes budget/IsCanPurchase; investment spends/sets class and may queue, actual purchase cleans queue/resets skips without exact promise assignment. If unaffordable, default-role ORDER_TRAIN is appended/prepended according to war flag, resets skips, and has no explicit commit. This is a materially different direct training candidate path from normal shared selection and operation short-build<7; both are included. Demand/planner boolean and price/budget output are bounded acquisition inputs.

Promise/rebuild interaction: CommitToBuildNextUnit changes needed/committed containers but does not occupy/reserve the physical army slot. GrabUnitsFromTheReserves clears/rebuilds needed from free physical required slots without excluding committed tokens; a later rebuild can therefore recreate a needed token while a city still has a promise. Stage readiness uses physical required/optional status. This is an additional source-confirmed reason to log both lists and physical slots, and to avoid inferring reservation from successful commit. CleanUpQueue removes invalid orders via popOrder and can itself clear a token before a subsequent purchase bookkeeping check. No runtime occurrence asserted.
