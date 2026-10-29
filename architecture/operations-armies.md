# Operation and army state contracts

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How do formations, slots, recruitment and commitments persist and transition?

This owns operation/army bookkeeping independently of the score assigned to a requested unit. Pending/committed queues, city association and fulfillment must not be inferred merely from a selected buildable.

## Reference synthesis and unresolved ownership

Operation IDs, army IDs and formation slot indices connect reserve recruitment, pending requests, city commitments and completion. Required versus optional slots control whether unfilled roles become build demand. These persistent/temporal transitions are not implied by a high production score.

The recovered normal city selector and the separate `CheckForOperationUnits` path do not establish identical commitment behavior. The pending-to-committed-to-fulfilled contract is documented below, but the normal queued buildable's connection to it remains unresolved. Non-operation standing-army ownership was also not proved. AI planning and city-facing wrappers link here rather than owning duplicate state descriptions.

Canonical related owners: [Operation and army demand reaching unit production](<../ai/units/operation-demand.md>), [Operation creation and recruitment planning](<../ai/operations/planning.md>), [Production queues and creation](<../mechanics/production/queue-and-creation.md>), [Role strategies and the VP+UPAI profile](<../ai/units/role-strategies.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- demand is owned by `CvAIOperation`/`CvArmyAI`; city conversion to a buildable is in `CvCity::GetUnitForOperation` and `CvCityStrategyAI::ChooseProduction`.

- operations created by `CvPlayer::addAIOperation`; `CvAIOperation::SetUpArmy` calls `GrabUnitsFromTheReserves`, which may populate `m_viListOfUnitsWeStillNeedToBuild`.

`AIOperationTypes` is a closed enum in `CvAIOperation.h` with values including `AI_OPERATION_FOUND_CITY`, `AI_OPERATION_PILLAGE_ENEMY`, `AI_OPERATION_CITY_ATTACK_LAND`, `AI_OPERATION_CITY_ATTACK_NAVAL`, `AI_OPERATION_CITY_ATTACK_COMBINED`, `AI_OPERATION_CITY_DEFENSE`, `AI_OPERATION_RAPID_RESPONSE`, `AI_OPERATION_NAVAL_SUPERIORITY`, `AI_OPERATION_NUKE_ATTACK`, `AI_OPERATION_CARRIER_GROUP`, `AI_OPERATION_MUSICIAN_CONCERT_TOUR`, `AI_OPERATION_MERCHANT_DELEGATION`, and `AI_OPERATION_DIPLOMAT_DELEGATION`.

`AIOperationState` in `CvAIOperation.h` tracks `AI_OPERATION_STATE_RECRUITING_UNITS`, `AI_OPERATION_STATE_GATHERING_FORCES`, `AI_OPERATION_STATE_MOVING_TO_TARGET`, `AI_OPERATION_STATE_AT_TARGET`, and terminal/invalid states. Operation unit demand is primarily produced while recruiting, but unit removal can also abort later stages.

`OperationSlot` in `CvAIOperation.h` is the demand token: `m_iOperationID`, `m_iArmyID`, and `m_iSlotID`, with `IsValid()`, `Invalidate()`, and equality by all three IDs.

`MultiunitFormationTypes` is a closed enum in `CvGameCoreDLLUtil/include/CvEnums.h`; formation rows and slot entries are database-backed through `MultiUnitFormations` and `MultiUnitFormation_SlotEntries`. `CvFormationSlotEntry` in `CvGameCoreDLL_Expansion2/CvInfos.h` contains `m_primaryUnitType`, `m_secondaryUnitType`, `m_ePositionType`, and `m_requiredSlot`.

VP/CP formation source data is in `(1) Community Patch/Database Changes/Units/CoreNewUnitFormations.xml`, `(1) Community Patch/Database Changes/Units/CoreUnitFormationChanges.sql`, and `(2) Vox Populi/Database Changes/Units/NewUnitFormations.xml`. Example: `(2) Vox Populi/Database Changes/Units/NewUnitFormations.xml` adds `MUFORMATION_DIPLOMAT_ESCORT` with a required `UNITAI_DIPLOMAT` slot and an optional defensive escort slot.

RISK: `AIOperationTypes`, `AIOperationState`, `MultiunitFormationTypes`, `OperationSlot`, `CvArmyFormationSlot`, and formation database row/slot ordering are savegame/schema-sensitive. `CvAIOperation::Serialize(...)` serializes operation type/state and both demand queues; `CvArmyAI::Serialize(...)` serializes formation, operation ID, army state, and `m_FormationEntries`.

Operations are created through `CvPlayer::addAIOperation(AIOperationTypes eOperationType, size_t iMaxMissingUnits, PlayerTypes eEnemy, CvCity* pTarget, CvCity* pMuster)` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`. It rejects human unit-AI players, creates a derived operation with `CreateAIOperation(...)`, checks `CvAIOperation::PreconditionsAreMet(...)`, pushes the operation into `m_AIOperations` before initialization so lookup works during `Init(...)`, then aborts/deletes if initialization fails or `GetNumUnitsNeededToBeBuilt()` exceeds `iMaxMissingUnits`.

`CreateAIOperation(AIOperationTypes, int, PlayerTypes, PlayerTypes)` maps operation enum values to concrete classes such as `CvAIOperationCityAttackLand`, `CvAIOperationCityAttackNaval`, `CvAIOperationCityAttackCombined`, `CvAIOperationPillageEnemy`, `CvAIOperationNavalSuperiority`, `CvAIOperationDefendCity`, `CvAIOperationDefenseRapidResponse`, `CvAIOperationNukeAttack`, `CvAIOperationCarrierGroup`, and civilian operation classes in `CvAIOperation.cpp`.

Operation triggers are upstream manager decisions. `CvMilitaryAI::DoTurn()` calls `UpdateOperations()` and `MakeEmergencyPurchases()` for non-human unit-AI players; military requests include `CvMilitaryAI::RequestCityAttack(...)`, `RequestPillageAttack(...)`, `RequestNukeAttack(...)`, `SetupInstantDefenses(...)`, `CheckLandDefenses(...)`, `CheckSeaDefenses(...)`, and carrier-group creation in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`. Civilian operations are requested by `CvEconomicAI` for settlers and diplomatic/great-person missions, as mapped in `docs-ai/ai-unit-production-upstream.md`.

## Candidates and gates

`CvAIOperation::GrabUnitsFromTheReserves(CvPlot* pMusterPlot, CvPlot* pTargetPlot, CvArmyAI* pArmy)` is the main existing-unit recruitment and demand-generation function. It gets open slots with `CvArmyAI::GetOpenSlots(false)`, reads slot roles with `CvArmyAI::GetSlotInfo(...)`, builds reachable plots from the muster plot with `GC.GetStepFinder().GetPlotsInReach(...)`, then scans the owner player's units.

Existing units are filtered by `OperationalAIHelpers::IsUnitSuitableForRecruitment(...)`. The source checks `CvUnit::canUseForAIOperation()`, healing state, proximity to target, recent deployment, health threshold `AI_OPERATIONAL_PERCENT_HEALTH_FOR_OPERATION`, ocean/deep-water constraints, and slot-role suitability against the available `CvFormationSlotEntry` list.

`GrabUnitsFromTheReserves(...)` scores recruitable units as `10000 + pLoopUnit->GetPower() + iExtraScore - iTurnsToReachCheckpoint * 30`, after pathing to muster with `CvUnit::TurnsToReachTarget(...)`. It penalizes units outside friendly territory or outside the muster area for land units, and adds an extra score for land units guarding border citadels near unfriendly majors. It stable-sorts `vector<OptionWithScore<int>> choices`.

After scoring, `GrabUnitsFromTheReserves(...)` clears `m_viListOfUnitsWeStillNeedToBuild`. For each free army slot, it calls `CvAIOperation::FindBestFitReserveUnit(...)`; if no reserve unit is assigned and `CvArmyAI::GetSlotInfo(i).m_requiredSlot` is true, it pushes `OperationSlot(m_iID, pArmy->GetID(), i)` into `m_viListOfUnitsWeStillNeedToBuild`.

`CvAIOperation::RecruitUnit(CvUnit*)` is a single-unit fill path used after direct purchases or late recruitment. It recomputes free slot info, calls `OperationalAIHelpers::IsUnitSuitableForRecruitment(...)`, adds the unit to `CvArmyAI` with `CvArmyAI::AddUnit(...)`, then removes the matching slot ID from `m_viListOfUnitsWeStillNeedToBuild`.

INFERRED: Reserve recruitment is a slot-fill optimizer, not a city-production scorer. Its score uses power, pathing time, territory/chokepoint adjustments, and border-citadel handling; this pass found no direct flavor use inside `GrabUnitsFromTheReserves(...)`.

`CvAIOperation::m_viListOfUnitsWeStillNeedToBuild` is a `std::deque<OperationSlot>` of required formation slots that reserve recruitment did not fill. `CvAIOperation::PeekAtNextUnitToBuild()` returns only the front slot or an invalid `OperationSlot`.

`CvAIOperation::m_viListOfUnitsCitiesHaveCommittedToBuild` tracks slots promised by cities. `CvAIOperation::CommitToBuildNextUnit(OperationSlot)` moves a matching slot from `m_viListOfUnitsWeStillNeedToBuild` to committed. `UncommitToBuildUnit(...)` moves a committed slot back to the front of the needed deque. `FinishedBuildingUnit(...)` removes a committed slot and calls `CheckTransitionToNextStage()` if no unbuilt slots remain.

`CvArmyAI::GetOpenSlots(bool bRequiredOnly)` returns free formation-slot indexes, optionally restricted to slots whose stored `CvArmyFormationSlot::IsRequired()` is true. `CvArmyAI::GetSlotInfo(size_t)` returns the original `CvFormationSlotEntry` from `CvMultiUnitFormationInfo`, including primary/secondary `UnitAITypes`.

`CvArmyAI::AddUnit(int iUnitID, int iSlotNum, bool bIsRequired)` removes the unit from any previous army, may upgrade it, stores it in the requested formation slot, sets `CvUnit::setArmyID(GetID())`, assigns tactical move `AI_TACTICAL_OPERATION`, and estimates turns to the current army checkpoint.

`CvArmyAI::RemoveUnit(int iUnitToRemoveID, bool bTempOnly)` clears the slot, resets the unit's army ID and UnitAI to its default, and if `bTempOnly` is false calls `CvAIOperation::UnitWasRemoved(GetID(), slotIndex)`. `CvAIOperation::UnitWasRemoved(...)` pushes the slot back into `m_viListOfUnitsWeStillNeedToBuild` only when the operation is still `AI_OPERATION_STATE_RECRUITING_UNITS`; in later active states it may abort if strength falls too far.

RISK: Slot "required" status exists in both database formation slot info and live `CvArmyFormationSlot` state. `CvAIOperation::UnitWasRemoved(...)` comments note that later strength checks consult the original formation because army slots can be cleared/set non-required. Future behavior edits must distinguish original `CvFormationSlotEntry::m_requiredSlot` from mutable slot state.

## Data and configuration

- `CvMultiUnitFormationInfo`, `CvFormationSlotEntry`, `CvUnitEntry`, `UnitAITypes`, `AIOperationTypes`, `ArmyType`, `MultiunitFormationTypes`.

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvFormationSlotEntry` / `CvMultiUnitFormationInfo` | `MultiUnitFormations`, `MultiUnitFormation_SlotEntries` | Operation/army slot demand consumed by `CvAIOperation`, `CvArmyAI`, and city unit production bridges. |

## Observability and validation obligations

2. Runtime army-candidate test: capture `CityStrategyAIProductionLog...csv` `PRE`, invalid-item, and `POST` rows for cities where `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` returns a unit but `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` is invalid or not same muster.

3. Runtime operation-commit test: force an operation missing-slot demand, let normal `CvCityStrategyAI::ChooseProduction(...)` select `CITY_BUILDABLE_UNIT_FOR_OPERATION`, and inspect `CvAIOperation::m_viListOfUnitsWeStillNeedToBuild`, `m_viListOfUnitsCitiesHaveCommittedToBuild`, and `CvCity::m_unitBeingBuiltForOperation` before and after completion.

Operation logs include `CvAIOperation::LogOperationStart()`, `LogOperationStatus(...)`, `LogOperationEnd()`, and `LogOperationSpecialMessage(...)` in `CvAIOperation.cpp`. `GrabUnitsFromTheReserves(...)` logs potential reserve counts when `GC.getLogging()` and `GC.getAILogging()` are enabled.

City/production logs include `CvCityStrategyAI::LogPossibleBuilds(...)`, `LogInvalidItem(...)`, `LogCityProduction(...)`, `CvCityStrategyAI::LogHurryMessage(...)`, and `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)`.

## Version differences, risks and unresolved questions

OPEN_QUESTION: `docs-ai/ai-unit-production-upstream.md` marks the full lifecycle of `CvAIOperation::CommitToBuildNextUnit`, `UncommitToBuildUnit`, and `FinishedBuildingUnit` as not fully traced.

`CvAIOperation`, `CvArmyAI`, operation slot queues, and formation slot data are shared between strategic operation planning, army movement, city production demand, and emergency purchase helpers.

- OPEN_QUESTION operation demand logging should be mapped with `CvAIOperation` logs; production side logs through `CvCityStrategyAI`.

- RISK operation/army demand (`CvAIOperation`, `CvArmyAI`, `OperationSlot`, `CvPlayerAI` slot bridge): split consumers include normal production, `CvCity::CheckForOperationUnits()`, hurry selection, emergency purchase, and faith purchase. Serialized queues and formation slot IDs make changes savegame- and schema-sensitive.

- OPEN_QUESTION `CITY_BUILDABLE_UNIT_FOR_ARMY` is injected from `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`, but current normal `CvCityStrategyAI::ChooseProduction(...)` sanity handles it in the same branch as operation units and requires `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`. Purchase sanity treats army candidates differently. Design intent remains unmapped.

- OPEN_QUESTION Normal `CvCityStrategyAI::ChooseProduction(...)` queues selected `CITY_BUILDABLE_UNIT_FOR_OPERATION` with `CvCity::pushOrder(...)`, but the explicit `CityCommitToBuildUnitForOperationSlot(...)` path is verified in `CvCity::CheckForOperationUnits()`, not visibly in the normal final selector. Trace `CvCity::pushOrder`, `CvCity::produce`, and `CvCity::CreateUnit` before claiming full normal-queue operation-slot lifecycle.

- OPEN_QUESTION Non-operation standing armies: current docs verify operation-owned armies through `CvAIOperation::AddArmy(...)` and generic `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` iteration, but do not prove whether all relevant armies are operation-owned.

- Classification: HIGH_CONFIDENCE_BUG

- Affected decision: Unit production; `CITY_BUILDABLE_UNIT_FOR_ARMY` in normal city production.

- Why current behavior may be suboptimal: `CvMilitaryAI::GetUnitTypeForArmy(...)` scans open army slots broadly, but normal production later filters army candidates through an operation-slot bridge. STALE_OR_MISMATCH: the source comment says `CITY_BUILDABLE_UNIT_FOR_ARMY` is a "useful unit"; current normal-production validation makes it dependent on an operation missing-slot queue. This can make the injected army candidate inert outside same-muster operation demand.

- Lowest-risk investigation: Use existing `CvCityStrategyAI::LogPossibleBuilds(..., "PRE")`, `LogInvalidItem(...)`, and `"Army unit"` logging to count how often army candidates enter precheck but disappear before `"POST"`.

- Runtime/logging test: Create/observe an AI with armies that have required open slots but no front operation slot for the city. Confirm whether `"Army unit"` appears in `PRE`, is absent from `POST`, and no operation-slot invalid log explains the loss.

- Classification: UNFINISHED_INTEGRATION

- Affected decision: Operation unit demand and normal city unit production.

- Source functions: `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction(...)` queues selected `CITY_BUILDABLE_UNIT_FOR_OPERATION` through `CvCity::pushOrder(ORDER_TRAIN, ...)` and resets skipped-operation pressure. explicit operation slot bookkeeping lives in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`, `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)`, `CityUncommitToBuildUnitForOperationSlot(...)`, and `CityFinishedBuildingUnitForOperationSlot(...)`. the explicit commit path is visible in `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvCity::CheckForOperationUnits()` when it pushes a front order during war and stores `m_unitBeingBuiltForOperation`.

- Why current behavior may be suboptimal: OPEN_QUESTION the normal final selector can choose an operation unit but no direct `CityCommitToBuildUnitForOperationSlot(...)` call was verified on that path. If completion is not associated with `m_unitBeingBuiltForOperation`, operation demand may remain in `m_viListOfUnitsWeStillNeedToBuild` while a city builds a matching unit as ordinary production.

- Lowest-risk investigation: Trace all callers and finish paths for `m_unitBeingBuiltForOperation`, `CityCommitToBuildUnitForOperationSlot(...)`, and `CityFinishedBuildingUnitForOperationSlot(...)` while comparing normal `LogCityProduction(...)` operation-unit selections with operation logs from `CvAIOperation::LogOperationSpecialMessage(...)`.

- Smallest possible patch point: If confirmed, the selected `CITY_BUILDABLE_UNIT_FOR_OPERATION` application branch in `CvCityStrategyAI::ChooseProduction(...)`, but only after identifying the exact `OperationSlot` selected by the same validation pass.

- Runtime/logging test: Force an operation missing-slot demand, let normal city production pick `CITY_BUILDABLE_UNIT_FOR_OPERATION`, and verify whether the slot moves from `CvAIOperation::m_viListOfUnitsWeStillNeedToBuild` to `m_viListOfUnitsCitiesHaveCommittedToBuild` before completion.

- Risk of double-counting or unintended downstream effects: RISK duplicate commitment or wrong-slot commitment can deadlock operation queues, remove demand prematurely, or assign a completed unit to the wrong `CvArmyAI` slot. Serialized state includes both operation deques and `CvCity::m_unitBeingBuiltForOperation`.

- Why current behavior may be suboptimal: INFERRED the logs show candidate lists before and after major transformations, but not a normalized component table of base flavor product, operation/army wrapper, sanity score, duration reweight, current-production inertia, and final selected weight. This obscures whether an apparent bad build came from data flavors, local sanity, duration, randomness threshold, or final queue preservation.

2. OPEN_QUESTION: Trace operation-slot lifecycle for normal `CITY_BUILDABLE_UNIT_FOR_OPERATION` queue selection before changing operation or army production.

- / OPEN_QUESTION: Revise normal operation-unit queue claims to stay explicitly unresolved. `CvCityStrategyAI::ChooseProduction(...)` queues selected operation units with `CvCity::pushOrder(ORDER_TRAIN, ...)` and resets skipped-operation pressure, but the visible commit path is `CvCity::CheckForOperationUnits()` calling `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)` and storing `CvCity::m_unitBeingBuiltForOperation`. The atlas should not imply completed normal-selector operation units are committed or uncommitted until `CvCity::produce(...)` / `CvCity::CreateUnit(...)` is traced for that path.

- OPEN_QUESTION: The atlas lacks a completed normal-selector operation-unit lifecycle. Required evidence: `CITY_BUILDABLE_UNIT_FOR_OPERATION` selected in `CvCityStrategyAI::ChooseProduction(...)`, order stored by `CvCity::pushOrder(...)`, completion through `CvCity::produce(UnitTypes, UnitAITypes, bool)` / `CvCity::CreateUnit(...)`, and whether `CvPlayerAI::CityFinishedBuildingUnitForOperationSlot(...)` is reached without `CvCity::m_unitBeingBuiltForOperation`.

- OPEN_QUESTION: The atlas does not prove whether non-operation standing armies exist in audited source and materially feed `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`. `CvAIOperation::AddArmy(...)` is mapped, but generic `CvPlayer` army ownership needs a focused pass.

OPEN_QUESTION: In the inspected normal `CvCityStrategyAI::ChooseProduction(...)` final application branch, selected operation/army units are queued with `CvCity::pushOrder(ORDER_TRAIN, eUnitType, default UnitAI, ...)`, and selected operation units reset skipped-operation pressure, but no direct call to `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)` was visible there. Explicit slot commit/finish bookkeeping is verified in `CvCity::CheckForOperationUnits()`.

OPEN_QUESTION: Verify whether any audited source path creates non-operation standing armies that materially affect `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`. This pass verified operation-owned army creation through `CvAIOperation::AddArmy(...)` and generic player army iteration, but did not prove all armies are operation-owned.

OPEN_QUESTION: Trace completed normal `CITY_BUILDABLE_UNIT_FOR_OPERATION` orders from `CvCityStrategyAI::ChooseProduction(...)` through `CvCity::produce(...)` / `CvCity::CreateUnit(...)` to confirm whether a queued normal-selector operation unit is ever associated with an `OperationSlot`. The explicit commit/finish path is verified in `CvCity::CheckForOperationUnits()`.

RISK: City-facing operation demand is split across normal production, hurry/purchase selection, city root operation-unit checks, military emergency purchase, and religion faith purchase. Changes to one consumer can leave stale missing slots or duplicate unit pressure if operation bookkeeping is not updated consistently.

RISK: Direct purchases bypass the normal weighted city production competition. `CvCity::CheckForOperationUnits()`, `CvAIOperation::BuyFinalUnit()`, `CvMilitaryAI::BuyEmergencyUnit(...)`, and religion purchase logic can consume treasury/faith and satisfy slots before or outside `CvCityStrategyAI::ChooseProduction(...)`.

RISK: Determinism and save compatibility depend on stable sort/order in reserve recruitment, serialized operation/army queues, operation IDs, army IDs, and enum/database alignment for operation and formation types.

## Source and dependency lookup

Operations are created through `CvPlayer::addAIOperation` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp` and concrete operation classes from `CreateAIOperation` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`. `CvAIOperation::AddArmy` creates `CvArmyAI` objects, and `CvArmyAI::SetFormation` in `CvGameCoreDLL_Expansion2/CvArmyAI.cpp` loads formation slots from `CvMultiUnitFormationInfo`.

- Movement and immediate tactical reactions live in `CvTacticalAI`, `CvHomelandAI`, `CvAIOperation`, and `CvArmyAI`.

| Class / manager | Decisions affected |
|---|---|
| `CvPlayerAI` (`CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`, `CvPlayerAI.h`) | AI turn hooks, tactical/homeland update entry, operation-slot production bridge, operation-slot commit/uncommit/finish bookkeeping. Affects operation unit demand, unit production, emergency purchases, and movement timing. |

| Function / enum / table | Decisions affected |
|---|---|
| `CreateAIOperation(...)`, `OperationalAIHelpers::GetArmyFormationForOpType(...)`, `CvAIOperation::AddArmy(...)`, `SetUpArmy(...)`, `GrabUnitsFromTheReserves(...)`, `PeekAtNextUnitToBuild()`, `CommitToBuildNextUnit(...)`, `UncommitToBuildUnit(...)`, `FinishedBuildingUnit(...)`, `BuyFinalUnit()` (`CvAIOperation.cpp`) | operation lifecycle, formation demand, missing/committed slot bookkeeping, final-unit purchase. |
| `CvArmyAI::SetFormation(...)`, `GetOpenSlots(...)`, `GetSlotInfo(...)`, `AddUnit(...)`, `RemoveUnit(...)` (`CvArmyAI.cpp`) | live army slot creation, role lookup, slot fill/removal, operation demand recreation. |
| `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, `CityCommitToBuildUnitForOperationSlot(...)`, `CityUncommitToBuildUnitForOperationSlot(...)`, `CityFinishedBuildingUnitForOperationSlot(...)` (`CvPlayerAI.cpp`) | city-visible operation demand and bookkeeping. |
| `FlavorTypes`, `OrderTypes`, `UnitAITypes`, `MultiunitFormationTypes` (`CvGameCoreDLLUtil/include/CvEnums.h`) and `AIOperationTypes` (`CvGameCoreDLL_Expansion2/CvAIOperation.h`) | enum/schema-sensitive identifiers used by flavor propagation, queue application, unit role selection, formations, and operations. |

- army units: `CvMilitaryAI::GetUnitTypeForArmy`, `CvArmyAI::GetOpenSlots`, `CvArmyAI::GetSlotInfo`, `CITY_BUILDABLE_UNIT_FOR_ARMY`.

- formation/demand creation: `OperationalAIHelpers::GetArmyFormationForOpType`, `CvAIOperation::AddArmy`, `CvArmyAI::SetFormation`, `CvAIOperation::SetUpArmy`, `CvAIOperation::GrabUnitsFromTheReserves`, `CvAIOperation::UnitWasRemoved`.

- city exposure: `CvAIOperation::PeekAtNextUnitToBuild`, `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot`, `CvCity::GetUnitForOperation`, `CvMilitaryAI::GetUnitTypeForArmy`.

- bookkeeping: `CvPlayerAI::CityCommitToBuildUnitForOperationSlot`, `CityUncommitToBuildUnitForOperationSlot`, `CityFinishedBuildingUnitForOperationSlot`; `CvAIOperation::CommitToBuildNextUnit`, `UncommitToBuildUnit`, `FinishedBuildingUnit`; `CvArmyAI::AddUnit`.

- data: `AIOperationTypes`, `AIOperationState`, `OperationSlot`, `MultiunitFormationTypes`, `CvFormationSlotEntry`, `MultiUnitFormations`, `MultiUnitFormation_SlotEntries`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` | operation unit production, operation purchase/queue path, operation/army sanity branch in normal production. | RISK same-muster/access checks determine whether operation demand reaches cities at all. |
| `CvAIOperation::CommitToBuildNextUnit(...)`, `UncommitToBuildUnit(...)`, `FinishedBuildingUnit(...)` | operation slot bookkeeping for direct city operation path. | RISK stale or duplicate demand if queue/purchase paths do not update committed/needed deques consistently. |

- / RISK: Revise `docs-ai/atlas/06-ai-improvement-map.md` item 2 from `HIGH_CONFIDENCE_BUG` to a source-confirmed contradiction with unconfirmed design intent. Source confirms `CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_ARMY` from `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`, then normal sanity handles `CITY_BUILDABLE_UNIT_FOR_ARMY` and `CITY_BUILDABLE_UNIT_FOR_OPERATION` together by requiring `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` and same muster city. However, runtime logs are needed before asserting the army path is functionally broken.

1. Trace `CvCityStrategyAI::ChooseProduction(...)` normal operation-unit application against `CvCity::CheckForOperationUnits()` in `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, and `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`. Target functions: `CityCommitToBuildUnitForOperationSlot(...)`, `CityUncommitToBuildUnitForOperationSlot(...)`, `CityFinishedBuildingUnitForOperationSlot(...)`, `CvCity::produce(...)`, and `CvCity::CreateUnit(...)`.

2. Audit army ownership producers beyond operation creation. Start with `CvPlayer::addArmyAI(...)`, `CvAIOperation::AddArmy(...)`, `CvArmyAI::SetOperationID(...)`, and all callers of `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`.

`CvPlayer::addAIOperation(...)` -> military/civilian operation creation, army demand, future city unit production for operation slots.

`CvPlayerAI::CityUncommitToBuildUnitForOperationSlot(...)` -> operation-slot queue cancellation/cleanup.

These classes are operation/formation demand classes, not final city production selectors. They produce and maintain `OperationSlot` demand; city-facing production and purchase consumers convert those slots into concrete `UnitTypes` through `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, `CvCity::GetUnitForOperation()`, `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`, `CvCity::CheckForOperationUnits()`, `CvCityStrategyAI::ChooseProduction(...)`, and `CvCityStrategyAI::ChooseHurry(...)`.

GUIDE_PATTERN: The relevant decision pattern is upstream demand generation -> role/slot conversion -> city trainability/scoring -> production/purchase selection -> operation bookkeeping. Do not treat `CvAIOperation::AddArmy(...)` or `CvArmyAI::SetFormation(...)` as city production selection.

`CvAIOperationMilitary::Init(...)` and civilian operation init paths call `CvAIOperation::AddArmy(OperationalAIHelpers::GetArmyFormationForOpType(m_eType))`, then `CvAIOperation::SetUpArmy(...)` after choosing target and muster plots.

`OperationalAIHelpers::GetArmyFormationForOpType(AIOperationTypes)` maps operation types to formations. Examples in `CvAIOperation.cpp`: `AI_OPERATION_FOUND_CITY` -> `MUFORMATION_SETTLER_ESCORT`, `AI_OPERATION_MUSICIAN_CONCERT_TOUR` -> `MUFORMATION_CONCERT_TOUR`, `AI_OPERATION_DIPLOMAT_DELEGATION` -> `MUFORMATION_DIPLOMAT_ESCORT`, `AI_OPERATION_MERCHANT_DELEGATION` -> `MUFORMATION_MERCHANT_ESCORT`, `AI_OPERATION_PILLAGE_ENEMY` -> `MUFORMATION_FAST_PILLAGERS`, `AI_OPERATION_NUKE_ATTACK` -> `MUFORMATION_NUKE_ATTACK`, `AI_OPERATION_CARRIER_GROUP` -> `MUFORMATION_CARRIER_ESCORT`, `AI_OPERATION_NAVAL_SUPERIORITY` -> `MUFORMATION_NAVAL_SQUADRON`, `AI_OPERATION_RAPID_RESPONSE` -> `MUFORMATION_RAPID_RESPONSE_FORCE`, and `AI_OPERATION_CITY_DEFENSE` -> `MUFORMATION_CLOSE_CITY_DEFENSE`. City-attack operations delegate to `MilitaryAIHelpers::GetCurrentBestFormationTypeForLandAttack()`, `GetCurrentBestFormationTypeForCombinedAttack()`, or `GetCurrentBestFormationTypeForPureNavalAttack()`.

`CvAIOperation::AddArmy(...)` creates a player army with `CvPlayer::addArmyAI()`, stores its ID in `m_viArmyIDs`, assigns the operation ID with `CvArmyAI::SetOperationID(m_iID)`, sets `ARMYAISTATE_WAITING_FOR_UNITS_TO_REINFORCE`, sets `ArmyType`, and calls `CvArmyAI::SetFormation(...)`.

`CvArmyAI::SetFormation(MultiunitFormationTypes)` releases existing units when formation changes, loads `CvMultiUnitFormationInfo` through `GC.getMultiUnitFormationInfo(eFormation)`, then creates one `CvArmyFormationSlot(-1, required)` per `CvFormationSlotEntry`, preserving the database slot's `m_requiredSlot`.

`CvAIOperation::SetUpArmy(...)` sets operation turn, target, muster, and army goal plot. If `CvArmyAI::GetOpenSlots(true)` has no required open slots, it sets `ARMYAISTATE_MOVING_TO_DESTINATION` and `AI_OPERATION_STATE_MOVING_TO_TARGET`; otherwise it sets `ARMYAISTATE_WAITING_FOR_UNITS_TO_REINFORCE` and `AI_OPERATION_STATE_RECRUITING_UNITS`. It then calls `GrabUnitsFromTheReserves(...)`, aborting with `AI_ABORT_NO_UNITS` if no suitable unit fills any slot and the army has zero filled slots.

`CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity* pCity, bool& bCitySameAsMuster)` is the bridge from operation demand to cities. It scans `m_AIOperations`, requires a muster plot, marks `bCitySameAsMuster` only if the city is the muster plot's owning city and has access to the muster landmass/ocean, skips naval operations when the city lacks muster access, peeks the operation's front missing slot, verifies the army and that the slot is free, and returns the first required slot via `OperationalAIHelpers::IsSlotRequired(...)`.

`CvCity::GetUnitForOperation()` requires the operation slot to be valid and same-muster, verifies city access to the operation's muster landmass/ocean, reads the slot's primary then secondary `UnitAITypes` from `CvArmyAI::GetSlotInfo(...)`, and asks `CvUnitProductionAI::RecommendUnit(UnitAITypes, true)` for a trainable `UnitTypes`.

`CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` is a broader army-slot query. It scans all player armies, gets required open slots with `CvArmyAI::GetOpenSlots(true)`, tries primary then secondary slot `UnitAITypes` through the city's `CvUnitProductionAI::RecommendUnit(...)`, and prefers the army with the fewest open required slots.

`CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_OPERATION` when `CvCity::GetUnitForOperation()` returns a unit. Its precheck weight uses `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, `FLAVOR_OFFENSE` from `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`, `CvMilitaryAI::GetNumberOfTimesOpsBuildSkippedOver()`, `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`, and `CvUnitProductionAI::GetWeight(eUnitForOperation)`. Source defaults in `CvGameCoreDLL_Expansion2/CvGlobals.cpp` are `5000`, `250`, and army base `750`; `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql` also defines `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`.

`CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_ARMY` from `CvMilitaryAI::GetUnitTypeForArmy(GetCity())`, with `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT + FLAVOR_OFFENSE * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`. In normal production sanity, both `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` share a branch that re-peeks `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, requires a same-muster slot, and calls `CvUnitProductionAI::CheckUnitBuildSanity(eUnitType, true, weight, false, false)`.

STALE_OR_MISMATCH: The `CITY_BUILDABLE_UNIT_FOR_ARMY` normal-production comment says "useful unit", and `CvMilitaryAI::GetUnitTypeForArmy(...)` is broader than operation peeking, but current `CvCityStrategyAI::ChooseProduction(...)` normal sanity still requires a valid same-muster operation slot for both operation and army buildables. Purchase sanity treats army buildables differently.

`CvCityStrategyAI::ChooseHurry(...)` also adds operation and army unit buildables unless faith-purchase mode is requested. In purchase sanity, `CITY_BUILDABLE_UNIT_FOR_OPERATION` requires a valid same-muster operation slot and calls `CheckUnitBuildSanity(..., bForOperation=true, bForPurchase=true)`, while `CITY_BUILDABLE_UNIT_FOR_ARMY` calls `CheckUnitBuildSanity(..., bForOperation=false, bForPurchase=true)` without operation-slot validation.

`CvCity::CheckForOperationUnits()` is a city-turn pressure path separate from normal combined production selection. It skips razing, puppet-annexable, minor, barbarian, human-automated, indebted, and already-training cities; chooses append vs front insertion based on war state; then uses `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, slot primary/secondary `UnitAITypes`, and `CvUnitProductionAI::RecommendUnit(...)`.

In `CvCity::CheckForOperationUnits()`, a positive purchase sanity result from `CvUnitProductionAI::CheckUnitBuildSanity(eBestUnit, true, weight, true)` can lead to unit investment or `CvCity::PurchaseUnit(eBestUnit, YIELD_GOLD)` if `CvEconomicAI::CanWithdrawMoneyForPurchase(PURCHASE_TYPE_UNIT, ...)` and `CvCity::IsCanPurchase(...)` pass. A purchased unit completes bookkeeping through `CvPlayerAI::CityFinishedBuildingUnitForOperationSlot(...)` if the city had `m_unitBeingBuiltForOperation`; otherwise the city tries to recruit it into recruiting operations via `CvAIOperation::RecruitUnit(...)`, prioritizing operations with fewer required open slots.

If purchase sanity fails but the unit can be built soon enough, `CvCity::CheckForOperationUnits()` can queue it with `CvCity::pushOrder(ORDER_TRAIN, eBestUnit, eUnitAI, ...)`. When inserted at the front during war (`bAppend == false`), it calls `CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)`, stores `CvCity::m_unitBeingBuiltForOperation`, and resets operation skipped-build pressure.

Cleanup paths uncommit promised operation units. `CvCity::kill(...)` and order-removal/cancellation code in `CvCity.cpp` call `CvPlayerAI::CityUncommitToBuildUnitForOperationSlot(m_unitBeingBuiltForOperation)` and invalidate the city member when `m_unitBeingBuiltForOperation.IsValid()`.

`CvPlayerAI::CityCommitToBuildUnitForOperationSlot(...)`, `CityUncommitToBuildUnitForOperationSlot(...)`, and `CityFinishedBuildingUnitForOperationSlot(...)` are thin wrappers over operation bookkeeping. The finish path also calls `CvArmyAI::AddUnit(pThisUnit->GetID(), slotId, true)`.

`CvAIOperation::BuyFinalUnit()` is an operation-owned emergency purchase consumer. If the operation is recruiting and exactly one missing slot remains, it chooses a nearby friendly city, reads the missing slot's primary `UnitAITypes`, calls `CvMilitaryAI::BuyEmergencyUnit(...)`, adds the purchased unit to the army slot, pops the needed deque, and calls `CheckTransitionToNextStage()` when no missing or committed units remain.

`CvMilitaryAI::MakeEmergencyPurchases()` can call `CvAIOperation::BuyFinalUnit()` for operations waiting on one last unit. `CvMilitaryAI::BuyEmergencyUnit(UnitAITypes, CvCity*)` uses `CvUnitProductionAI::RecommendUnit(...)`, verifies the unit has the requested `UnitAITypes`, checks `CvUnitProductionAI::CheckUnitBuildSanity(..., bForPurchase=true)`, uses `CvEconomicAI::CanWithdrawMoneyForPurchase(PURCHASE_TYPE_UNIT, ...)`, then calls `CvCity::PurchaseUnit(...)` with gold or faith and explicitly promotes AI units.

Religion faith-purchase logic can prefer operation/army units. In `CvReligionClasses.cpp`, the non-faith unit purchase branch compares the candidate from `CvCityStrategyAI::ChooseHurry(true, true)` against `CvCity::GetUnitForOperation()` and `CvMilitaryAI::GetUnitTypeForArmy(pCity)` before calling `CvCity::PurchaseUnit(eUnit, YIELD_FAITH)`.

| Function / state | Decisions affected |
|---|---|
| `CvPlayer::addAIOperation(...)` | Operation creation; rejects/accepts target/muster operation plans; limits missing required units with `iMaxMissingUnits`; seeds operation demand visible to later city production. |
| `CvAIOperation::GrabUnitsFromTheReserves(...)` | Fills formation slots from existing units; creates `m_viListOfUnitsWeStillNeedToBuild` for required slots not filled by reserves. |
| `CvPlayerAI::CityUncommitToBuildUnitForOperationSlot(...)` | Restores a promised slot to the front of the needed deque after queue cancellation/city loss. |
| `CvArmyAI::RemoveUnit(...)` | Clears a formation slot and notifies the operation unless temporary; can recreate demand or abort an operation. |
| `CvReligionAI` non-faith unit purchase branch | Faith-purchase path can prefer units matching `CvCity::GetUnitForOperation()` or `CvMilitaryAI::GetUnitTypeForArmy(...)`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
