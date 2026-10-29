# AI purchasing and emergency acquisition

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../source-snapshots.md#recovered-baseline>).

## Decision or contract question

When does AI acquire a unit or building through purchase, investment or emergency action?

Economic, military, tactical, religion and operation paths have different gates, scoring parameters and application semantics. A shared recommendation/sanity helper does not imply the same acquisition decision or side effects.

Canonical related owners: [Unit production and recommendation](<../units/production.md>), [City production selection](<city-selection.md>), [Production queues and creation](<../../mechanics/production/queue-and-creation.md>), [Puppet faith purchasing](<../../mechanics/production/puppet-faith-purchasing.md>), [Military event log and Lua contract](<../../architecture/military-log.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- Immediate AI purchase/investment choices that bypass the normal city queue selector. This record covers tactical emergency defenders, military emergency units/buildings, operation final-unit purchases, economic hurry purchases/investments, and AI faith purchases where audited source directly confirms them. Normal queued production is separate: `CvCityStrategyAI::ChooseProduction(...)` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` applies choices with `CvCity::pushOrder(...)`, while the purchase paths here call `CvCity::PurchaseUnit(...)`, `CvCity::PurchaseBuilding(...)`, `CvCity::CreateBuilding(...)`, or investment setters directly.

- tactical city-defense purchase triggers are in `CvTacticalAI::PlotEmergencyPurchases(CvTacticalDominanceZone*)` in `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`. Military purchase execution is in `CvMilitaryAI::BuyEmergencyUnit(UnitAITypes, CvCity*)`, `CvMilitaryAI::BuyEmergencyBuilding(CvCity*)`, and `CvMilitaryAI::MakeEmergencyPurchases()` in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`. Operation acceleration is in `CvAIOperation::BuyFinalUnit()` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`. Gold hurry/investment selection is in `CvEconomicAI::DoHurry()` and `CvEconomicAI::CanWithdrawMoneyForPurchase(PurchaseType, int, int)` in `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`. Faith purchase AI is in `CvReligionAI::DoTurn()`, `CvReligionAI::DoFaithPurchases()`, `CvReligionAI::DoFaithPurchasesInCities(CvCity*)`, and related buy helpers in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`.

- `CvMilitaryAI::DoTurn()` calls `CvMilitaryAI::MakeEmergencyPurchases()` before normal city turns; `CvAIOperation::BuyFinalUnit()` can be reached from that path. `CvEconomicAI::DoTurn()` calls `CvEconomicAI::DoHurry()` before normal city turns. `CvReligionAI::DoTurn()` calls `DoFaithPurchases()` and later `DoFaithPurchasesInCities(...)` before normal city turns. tactical emergency purchases happen later from `CvTacticalAI::ProcessDominanceZones()` through `CvTacticalAI::PlotEmergencyPurchases(...)`, during tactical dominance-zone processing described in `docs-ai/atlas/01-turn-lifecycle.md`.

## Candidates and gates

- Tactical emergency defenders: `CvTacticalAI::PlotEmergencyPurchases(...)` only continues for an owned non-water dominance zone city, rejects cities in danger of falling, disables unit desire when `MOD_BALANCE_PURCHASED_UNIT_DAMAGE` and city damage is above half max HP, and triggers only when dominance is worse than friendly or the city is under siege. It asks `CvMilitaryAI::BuyEmergencyBuilding(...)` when building investments are not enabled and asks `CvMilitaryAI::BuyEmergencyUnit(...)` for `UNITAI_RANGED`, `UNITAI_ATTACK_SEA`, `UNITAI_COUNTER`, or `UNITAI_DEFENSE` depending on garrison, water/coastal checks, `bWantUnits`, and `MOD_AI_UNIT_PRODUCTION`.

- STALE_OR_MISMATCH: `CvTacticalAI::PlotEmergencyPurchases(...)` returns immediately when `pZone->IsWater()`, but a later branch checks `pZone->IsWater() && pCity->isCoastal()` to buy `UNITAI_ATTACK_SEA`. Under the current visible control flow, that naval emergency-unit branch appears unreachable unless `IsWater()` semantics are not what the branch names imply.

- Military emergency unit: `CvMilitaryAI::BuyEmergencyUnit(...)` rejects puppet-and-annexable and human-automated cities, calls `CvUnitProductionAI::RecommendUnit(UnitAITypes, true)` for the requested role, verifies `CvUnitEntry::GetUnitAIType(eUnitType)`, then runs `CvUnitProductionAI::CheckUnitBuildSanity(eType, false, 100, true)` before trying to buy.

- Military emergency building: `CvMilitaryAI::BuyEmergencyBuilding(...)` loops `GC.GetGameBuildings()->GetNumBuildings()`, null-checks `CvBuildingEntry`, requires `CvCity::canConstruct(...)`, requires `CvBuildingEntry::GetDefenseModifier() > 0`, and requires `CvCity::IsCanPurchase(..., YIELD_GOLD)`.

- Operation final unit: `CvAIOperation::BuyFinalUnit()` only runs when `m_viListOfUnitsWeStillNeedToBuild.size() == 1` and `m_eCurrentState == AI_OPERATION_STATE_RECRUITING_UNITS`. It chooses a city near the muster plot, reads the remaining `OperationSlot`, obtains `CvFormationSlotEntry` through `CvArmyAI::GetSlotInfo(...)`, and asks `CvMilitaryAI::BuyEmergencyUnit(thisSlotEntry.m_primaryUnitType, pCity)`.

- Economic hurry: `CvEconomicAI::DoHurry()` builds a cross-city `CvWeightedVector<CvCityBuildable>`. It skips puppet-and-annexable cities, avoids the most threatened non-capital city, calls each city's `CvCityStrategyAI::ChooseHurry()`, and separately adds diplomatic `UNITAI_MESSENGER` candidates when `MOD_BALANCE_VP`, a valid diplomatic victory, met minors, and `RESOURCE_PAPER` allow it.

- Faith purchases: `CvReligionAI::DoFaithPurchasesInCities(...)` first targets faith buildings with `FaithBuildingAvailable(...)`, then other faith-cost buildings, then non-faith military units if all core cities are converted, net gold is positive, and the player is not out of supply. Its non-faith unit branch calls `CvCityStrategyAI::ChooseHurry(true, true)`. `CvReligionAI::DoFaithPurchases()` separately buys prophets, great people, missionaries, and inquisitors through `BuyGreatPerson(...)`, `BuyMissionaryOrInquisitor(...)`, `BuyMissionary(...)`, and `BuyInquisitor(...)`.

## Scoring and inputs

- Purchase legality: `CvCity::IsCanPurchase(...)` gates both gold and faith purchases. Shared checks include exactly one requested item, puppet exceptions, resistance/razing, unit placement, `CvCity::canTrain(...)` or `CvCity::canConstruct(...)`, costs, cooldowns, and purchase-specific requirements. Gold purchases use `CvCity::GetPurchaseCost(...)`, treasury gold, building/unit investment checks, and purchase cooldowns. Faith purchases use `CvCity::GetFaithPurchaseCost(...)`, faith balance, faith cooldowns, religion majority/owned religion, belief unlocks, `CvBelief` faith-buying gates, `CvUnitEntry::IsRequiresFaithPurchaseEnabled()`, `CvUnitEntry::IsRequiresEnhancedReligion()`, `CvBuildingEntry::IsUnlockedByBelief()`, and domain restrictions for belief-enabled faith unit purchase.

- purchase paths often inherit cached unit/building weights from the same city production AIs used by normal production. `CvCityStrategyAI::ChooseHurry(...)` uses `CvUnitProductionAI::GetWeight(...)`, `CvBuildingProductionAI::GetWeight(...)`, operation/army wrapper constants, and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)`. `CvUnitProductionAI::RecommendUnit(...)` also uses cached unit weights or unit flavor fallback. Tactical role choice itself is direct heuristic logic rather than flavor scoring.

- important direct inputs include tactical dominance flag, city siege/damage/garrison/falling state, puppet/automation state, operation recruitment state, army formation primary `UnitAITypes`, treasury buffer, purchase-request priority, threatened city avoidance, diplomatic victory and `RESOURCE_PAPER`, religion conversion state, net gold, supply, faith balance, faith cooldowns, belief unlocks, and city purchase legality.

- Data tables / entries / enums: source classes and entries include `CvTacticalAI`, `CvMilitaryAI`, `CvAIOperation`, `CvArmyAI`, `CvFormationSlotEntry`, `CvEconomicAI`, `CvReligionAI`, `CvCityStrategyAI`, `CvUnitProductionAI`, `CvBuildingProductionAI`, `CvCity`, `CvUnitEntry`, `CvBuildingEntry`, `CvBeliefEntry`, `CvReligion`, `CvWeightedVector<CvCityBuildable>`, and `CvPurchaseRequest`. enum/order-sensitive types include `UnitAITypes` (`UNITAI_RANGED`, `UNITAI_ATTACK_SEA`, `UNITAI_COUNTER`, `UNITAI_DEFENSE`, `UNITAI_MESSENGER`), `CityBuildableType` (`CITY_BUILDABLE_UNIT`, `CITY_BUILDABLE_UNIT_FOR_OPERATION`, `CITY_BUILDABLE_UNIT_FOR_ARMY`, `CITY_BUILDABLE_BUILDING`), `PurchaseType`, `YieldTypes` (`YIELD_GOLD`, `YIELD_FAITH`), `UnitCreationReason` (`REASON_BUY`, `REASON_FAITH_BUY`), `OrderTypes`, `UnitTypes`, `BuildingTypes`, `BuildingClassTypes`, `BeliefTypes`, and `ReligionTypes`.

- Relevant source-data tables: purchase behavior depends on unit/building costs and fields cached into `CvUnitEntry` and `CvBuildingEntry`, unit/building flavor rows (`Unit_Flavors`, `Building_Flavors`), faith unlock data consumed through belief/religion methods, and strategy/flavor data feeding cached city production weights. RISK schema-sensitive areas include unit/building faith cost fields, belief faith-purchase tables, purchase cooldown fields, `BuildingClassPurchaseRequireds`, resource requirements, and `HandicapInfos.CityProductionChoiceCutoffThreshold`.

## Selection and control

- Emergency unit role selection: tactical role choice is mostly direct heuristic selection in `CvTacticalAI::PlotEmergencyPurchases(...)`; the concrete unit type for a requested `UnitAITypes` is selected by `CvUnitProductionAI::RecommendUnit(...)`, which stable-sorts candidate unit weights and returns the top unit. This is not the normal `CvCityStrategyAI::ChooseProduction(...)` threshold selection.

- Emergency unit sanity: `CvMilitaryAI::BuyEmergencyUnit(...)`, `CvEconomicAI::DoHurry()`, `CvReligionAI::DoFaithPurchasesInCities(...)`, and `CvCityStrategyAI::ChooseHurry(...)` all use `CvUnitProductionAI::CheckUnitBuildSanity(..., bForPurchase=true)` for purchase-facing unit validation or scoring. RISK this helper is shared with normal city production, so changing purchase sanity can affect queued unit scoring and vice versa.

- Emergency building selection: `CvMilitaryAI::BuyEmergencyBuilding(...)` does not call `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`; it directly buys the first currently constructible, gold-purchasable building with `CvBuildingEntry::GetDefenseModifier() > 0` that passes `CanWithdrawMoneyForPurchase(...)`.

- Hurry selection: `CvCityStrategyAI::ChooseHurry(bool bUnitOnly, bool bFaithPurchase)` creates purchase candidates, stable-sorts precheck candidates, reweights by duration, runs purchase-facing unit sanity and building sanity, stable-sorts final candidates, logs `PRE` and `POST`, then selects with `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold(...)` using `CvHandicapInfo::getCityProductionChoiceCutoffThreshold()` and seed `CvSeeder::fromRaw(0xe362f42a).mix(m_pCity->GetID())`.

- Gold reservation: `CvEconomicAI::CanWithdrawMoneyForPurchase(...)` copies `m_RequestedSavings`, stable-sorts `CvPurchaseRequest` entries by priority, subtracts higher-priority requested savings from current treasury balance, and returns whether enough balance remains for the purchase type being tested. Purchase types use `PurchaseType` values such as `PURCHASE_TYPE_UNIT`, `PURCHASE_TYPE_BUILDING`, and `PURCHASE_TYPE_TILE`.

- Faith selection: faith purchase priority is ordered procedurally in `CvReligionAI::DoFaithPurchases()` and `CvReligionAI::DoFaithPurchasesInCities(...)`, not by one shared weighted vector. Non-faith military faith purchases use `ChooseHurry(true, true)` and then prefer units equal to `CvCity::GetUnitForOperation()` or `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`, otherwise cap purchases by existing unit AI count.

## Application and state

- Unit purchase application: `CvCity::PurchaseUnit(UnitTypes, YieldTypes)` in `CvGameCoreDLL_Expansion2/CvCity.cpp` first rechecks `CvCity::IsCanPurchase(...)`. For `YIELD_GOLD`, it either sets unit investment and may push an urgent `ORDER_TRAIN` when investment mode applies, or creates the unit immediately with `CvCity::CreateUnit(eUnitType, NO_UNITAI, REASON_BUY)`, logs treasury expenditure, sets unit purchase cooldown, changes gold, and fires purchase instant yields/events. For `YIELD_FAITH`, it creates the unit with `REASON_FAITH_BUY`, applies faith cooldowns, sets religion data, deducts faith, updates faith-purchased great-person counters, logs `FAITH UNIT PURCHASE`, and fires faith purchase instant yields/events.

- Building purchase application: `CvCity::PurchaseBuilding(BuildingTypes, YieldTypes)` rechecks `IsCanPurchase(...)`. For gold, it either sets building investment and may push `ORDER_CONSTRUCT`, or calls `CvCity::CreateBuilding(...)`, cleans the queue, fires city constructed events, logs expenditure, changes gold, and fires purchase instant yields. For faith, it creates the building, cleans the queue, fires events, deducts faith, logs `FAITH BUILDING PURCHASE`, and fires faith purchase instant yields.

- Emergency defensive building application: `CvMilitaryAI::BuyEmergencyBuilding(...)` bypasses `CvCity::PurchaseBuilding(...)` and directly logs treasury expenditure, subtracts gold with `CvTreasury::ChangeGold(-iGoldCost)`, and calls `CvCity::CreateBuilding(eBldg)` for a defensive building. RISK this path does not use the same cooldown/event/instant-yield handling as `CvCity::PurchaseBuilding(...)` in the inspected code.

- Operation final-unit side effect: after `CvAIOperation::BuyFinalUnit()` receives a purchased unit, it calls `CvArmyAI::AddUnit(pUnit->GetID(), slot, true)`, removes the slot from `m_viListOfUnitsWeStillNeedToBuild`, and calls `CheckTransitionToNextStage()` if no needed or committed units remain.

- Post-purchase cleanup: `CvEconomicAI::DoHurry()` calls `CvCity::CleanUpQueue()` after successful direct unit purchase and after direct non-investment building purchase. It also resets operation skipped pressure for `CITY_BUILDABLE_UNIT_FOR_OPERATION` and settler skipped pressure for founding units.

## Observability and validation obligations

- `CvMilitaryAI::BuyEmergencyUnit(...)` and `CvMilitaryAI::BuyEmergencyBuilding(...)` log tactical purchase messages through `CvTacticalAI::LogTacticalMessage(...)`.

- `CvCityStrategyAI::ChooseHurry(...)` logs hurry candidates through `CvCityStrategyAI::LogPossibleHurries(...)`.

- `CvEconomicAI::DoHurry()` logs cross-city hurry candidates with `CvEconomicAI::LogPossibleHurries(...)` and logs buy/investment actions through `CvHomelandAI::LogHomelandMessage(...)`.

- `CvCity::PurchaseUnit(...)` logs faith unit purchases through `CvGameReligions::LogReligionMessage(...)`; `CvCity::PurchaseBuilding(...)` logs faith building purchases similarly. Gold purchases log treasury expenditures with `CvTreasury::LogExpenditure(...)`.

- `CvReligionAI::DoFaithPurchases(...)`, `DoFaithPurchasesInCities(...)`, `BuyMissionary(...)`, and `BuyInquisitor(...)` add religion and homeland log messages for faith purchase decisions when logging is enabled.

## Version differences, risks and unresolved questions

- RISK emergency and direct purchases bypass normal buildable competition. They can create units/buildings immediately or mark investments before or after normal city production, so changing these paths can alter tactical outcomes, operation readiness, treasury availability, faith spending, and subsequent city queue legality.

- RISK `CvCity::PurchaseUnit(...)` may create immediate units with `NO_UNITAI` and relies on `CvCity::CreateUnit(...)`/unit initialization for final role behavior, while normal queue production passes `CvUnitEntry::GetDefaultUnitAIType()` through `CvCity::pushOrder(...)`.

- RISK gold investment mode (`MOD_BALANCE_UNIT_INVESTMENTS`, `MOD_BALANCE_BUILDING_INVESTMENTS`, and spaceship-project special cases) changes a "purchase" from immediate creation to investment plus possible urgent queue insertion.

- RISK deterministic behavior depends on stable sorting, the hurry seed `0xe362f42a` mixed with city ID, tactical random selection between `UNITAI_COUNTER` and `UNITAI_DEFENSE` using `CvSeeder(pCity->plot()->GetPseudoRandomSeed())`, and database-backed enum ordering.

- RISK savegame-sensitive state includes purchase cooldowns, faith purchase cooldowns, city queues, investment flags, operation slot lists, army membership, treasury requested savings, and serialized production AI caches.

- OPEN_QUESTION the exact call site name for every possible human/UI/network purchase path is outside this AI decision record. This record only claims AI purchase behavior verified in `CvTacticalAI`, `CvMilitaryAI`, `CvAIOperation`, `CvEconomicAI`, `CvReligionAI`, `CvCityStrategyAI`, and `CvCity`.

- DESIGN_HYPOTHESIS future purchase docs should split "immediate creation", "investment plus queue insertion", and "direct `CreateBuilding` emergency construction" into separate application subtypes because they have different cooldown/event/instant-yield side effects.

## Source and dependency lookup

`CvMilitaryAI::BuyEmergencyUnit(UnitAITypes, CvCity*)` bypasses the production queue. It rejects puppet/automated cities, asks `CvUnitProductionAI::RecommendUnit(...)` for the requested `UnitAITypes`, checks the returned unit's role, calls `CvUnitProductionAI::CheckUnitBuildSanity(..., bForPurchase=true)`, uses `CvEconomicAI::CanWithdrawMoneyForPurchase(PURCHASE_TYPE_UNIT, ...)`, then calls `CvCity::PurchaseUnit(...)` with gold or faith. It explicitly promotes the purchased AI unit if needed.

`CvTacticalAI::PlotEmergencyPurchases(CvTacticalDominanceZone*)` calls `BuyEmergencyBuilding(...)` and `BuyEmergencyUnit(...)` for threatened/under-siege city zones. It may request `UNITAI_RANGED`, `UNITAI_ATTACK_SEA`, `UNITAI_COUNTER`, or `UNITAI_DEFENSE`; with `MOD_AI_UNIT_PRODUCTION`, it avoids counter for land defense because comments treat counter as anti-air.

`CvAIOperation::BuyFinalUnit()` bypasses queued production when an operation in `AI_OPERATION_STATE_RECRUITING_UNITS` needs exactly one remaining unit. It chooses a nearby city, reads the unfilled slot's primary `UnitAITypes`, calls `CvMilitaryAI::BuyEmergencyUnit(...)`, and adds the purchased unit to the army slot.

`CvMilitaryAI::MakeEmergencyPurchases()` calls `CvAIOperation::BuyFinalUnit()` when the player is not in a winning-at-war state according to its condition. The source comment says defensive spending is handled in tactical dominance-zone processing and this path speeds operations.

`CvEconomicAI` hurry/purchase logic builds a weighted list from `CvCityStrategyAI::ChooseHurry(...)`, adds diplomatic `UNITAI_MESSENGER` candidates, and for unit selections calls `CvCity::PurchaseUnit(...)` or unit investment. It checks resource availability and `CvUnitProductionAI::CheckUnitBuildSanity(..., bForPurchase=true)`. This bypasses `ChooseProduction(...)` queue selection but shares the city buildable and unit sanity infrastructure.

faith purchase logic in `CvReligionAI` includes military/non-faith unit purchases. Around the non-faith unit branch, it calls `CvCityStrategyAI::ChooseHurry(true, true)`, checks `CheckUnitBuildSanity(...)`, prefers units equal to `pCity->GetUnitForOperation()` or `GetMilitaryAI()->GetUnitTypeForArmy(pCity)`, and calls `CvCity::PurchaseUnit(eUnit, YIELD_FAITH)`.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
