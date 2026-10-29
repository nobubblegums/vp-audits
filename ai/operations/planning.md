# Operation creation and recruitment planning

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

When is an operation requested, accepted, recruited or aborted?

Operation preconditions, missing-unit budgets, reserve suitability and formation choice influence unit demand. Generic operation/army state and city-facing production demand have separate canonical owners.

Canonical related owners: [Operation and army state contracts](<../../architecture/operations-armies.md>), [Operation and army demand reaching unit production](<../units/operation-demand.md>), [Strategic pressure and manager outputs](<../strategies/managers.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

`CvMilitaryAI::DoTurn()` in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp` is the main upstream military producer. It calls:

- `ScanForBarbarians()`

- `UpdateBaseData()`

- `UpdateDefenseState()`

- `UpdateMilitaryStrategies()`

- `UpdateWarType()`

- `UpdateAttackTargets()`

- for AI-controlled units: `UpdateOperations()`, `MakeEmergencyPurchases()`, and `DisbandObsoleteUnits()`

`UpdateBaseData()` counts existing military composition into member variables such as `m_iNumLandUnits`, `m_iNumNavalUnits`, `m_iNumLandUnitsInArmies`, `m_iNumNavalUnitsInArmies`, `m_iNumRangedLandUnits`, `m_iNumMeleeLandUnits`, `m_iNumAntiAirUnits`, `m_iNumAirUnits`, and role-split counters such as `m_iNumArcherLandUnits`, `m_iNumSiegeLandUnits`, `m_iNumSkirmisherLandUnits`, `m_iNumMeleeNavalUnits`, `m_iNumRangedNavalUnits`, `m_iNumSubmarineNavalUnits`, `m_iNumBomberAirUnits`, and `m_iNumFighterAirUnits`. It finishes by calling `SetRecommendedArmyNavySize()`.

`CvMilitaryAI::SetRecommendedArmyNavySize()` computes `m_iRecLandUnits`, `m_iRecNavalUnits`, and `m_iRecExplorerUnits` from `CvEconomicAI::GetSoftSupplyCap()`, number of cities/coastal cities, settler count, exposed cities, preferred attack targets, conquest diplomacy state, naval flavor, defense/offense grand-strategy flavors, explorer demand from `CvEconomicAI::GetExplorersNeeded()`, and current land/naval/explorer counts.

`CvMilitaryAI::UpdateDefenseState()` converts actual vs recommended counts into `m_eLandDefenseState` and `m_eNavalDefenseState`, with under-siege cities forcing critical land defense. `MilitaryAIHelpers::IsTestStrategy_EmpireDefense(...)`, `IsTestStrategy_EmpireDefenseCritical(...)`, `IsTestStrategy_NeedNavalUnits(...)`, and `IsTestStrategy_NeedNavalUnitsCritical(...)` read those states.

`CvMilitaryAI::UpdateMilitaryStrategies()` starts/ends hardcoded XML-named military strategies. On start/end it calls `CvFlavorManager::ChangeActivePersonalityFlavors(...)` and `CvFlavorManager::ChangeCityFlavors(...)` with `CvMilitaryAIStrategyXMLEntry::GetPlayerFlavorValue(...)` and `GetCityFlavorValue(...)`.

`CvEconomicAI::DoTurn()` updates economic strategies and starts/ends them with the same flavor propagation pattern using `CvEconomicAIStrategyXMLEntry::GetPlayerFlavorValue(...)`, `GetCityFlavorValue(...)`, `CvFlavorManager::ChangeActivePersonalityFlavors(...)`, and `ChangeCityFlavors(...)`.

`CvEconomicAI` also produces unit-relevant economic demand: explorer counts are stored by `SetExplorersNeeded(...)` and `SetNavalExplorersNeeded(...)`; settler operations are created by `EconomicAIHelpers::IsTestStrategy_EnoughExpansion(...)`/related expansion logic around `AI_OPERATION_FOUND_CITY`; and messenger/diplomatic unit purchases are considered in economic hurry logic.

`CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FlavorTypes, bool)` in `CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp` combines active grand strategy flavor modifiers with base personality flavor. Unit production consumers read it directly for offense, defense, expansion, naval, ranged, mobile, air, nuke, and role-split flavors.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U current acquisition interface

Partial verification at `src-20261029-vp-c285a18`, `audit-20261029-task1u-01`: needed/committed slot state operations and city bridges now have current evidence in [operation demand](../units/operation-demand.md#task-1u-current-operation-bridge). Current searches identify CvAIOperation::AddArmy as the only located addArmyAI caller. This does not promote all operation-type creation/target/precondition/recruitment formulas above to current verification. Required versus optional formation slots and operation stage/abort transitions still require complete current-source revalidation. Normal shared selection does not establish slot commitment; direct short-build and purchase branches differ.

## Task 1U-B operation-request boundary

src-20261029-vp-548 / audit-20261029-task1u-b. Current source contracts: MilitaryAI::DoTurn/UpdateOperations2401, military RequestCityAttack/RequestPillageAttack/RequestNukeAttack/defense/carrier paths and EconomicAI found-city/merchant/musician/diplomat requests produce typed operation requests. `CvPlayer::addAIOperation42185` consumes type, enemy, target, muster and maxMissing; [unit lifecycle](../units/operation-demand.md#task-1u-b-lifecycle-closure-on-vp-548) traces registration, formation, reserve needs and fulfillment. These are independent strategic operation decisions; their output contract includes request reason, selected target/muster, allowed missing count, boolean precondition/init result and resulting operation/global IDs. Choosing which foreign city to attack is not silently absorbed into unit scoring. Preferred-attack/exposed-city arrays read by force sizing are cached outputs of UpdateAttackTargets, with prior/current generation distinguished by DoTurn order.

5.4.8 delta IsPossibleMusterCity ownership fix is consumed by HomelandAI7304 reserve staging. It is an indirect reserve-position/availability influence, not a changed numeric unit-scoring term or normal same-muster gate. No independent standing-army creation found outside operation AddArmy. Non-unit mission target execution stays deferred; its direct request interface and resulting required-slot demand are included.
