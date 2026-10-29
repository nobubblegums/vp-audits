# AI logs, reconstruction limits and validation scenarios

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

What did the source-audit reports establish about logging, and what remains unobservable?

This consolidates reported emitters, fields, gaps and proposed scenarios. It is the technical reference behind the normalized logging-coverage matrix; proposed instrumentation and scenarios are not implemented or executed by this migration.

## Reference synthesis

The recovered log inventory gives useful emitter names and candidate/selection traces; it does not provide exhaustive per-term reconstruction. Production PRE/POST vectors, invalid-item reasons, category choice logs, aggregate belief components and flavor deltas expose different stages. A later random seed or final total does not identify all consumed inputs.

The scenarios below are validation obligations or proposals from the recovered reports, not runtime tests run by this migration. The normalized coverage matrix records the difference between an older source-backed capability reference and an unchecked current-target assessment.

Canonical related owners: [City production selection](<../production/city-selection.md>), [Unit production and recommendation](<../units/production.md>), [Flavor production, propagation and consumption](<flavors.md>), [Military event log and Lua contract](<../../architecture/military-log.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Observability and validation obligations

1. What logs already exist for city production, unit scoring, operations, formations, military strategies, and purchases?

2. Which logs can reconstruct final production decisions today?

3. Which fields are missing?

4. Where should missing fields be logged with minimal behavior risk?

5. How can log volume be controlled during autoplay?

6. Which sample scenarios should be used for first validation?

### Which fields are missing for operation/formation/UPAI validation?

DESIGN_HYPOTHESIS: The closed source/data pass identifies required logging fields, but does not prove current log coverage. The minimum missing fields to validate UPAI operation behavior are:

- operation ID/type and selected formation enum/name;

- army ID, slot index, required flag, position type, primary UnitAI, secondary UnitAI;

- reserve recruitment result and no-fill reason per slot;

- `m_viListOfUnitsWeStillNeedToBuild` queue additions/removals;

- city/muster access and selected operation slot in `PeekAtNextUnitToBuildForOperationSlot(...)`;

- primary and secondary `RecommendUnit(...)` candidates and weights;

- operation/army buildable precheck weight, sanity result, and final city-production selection;

- tactical/homeland assignment with unit default/current UnitAI and originating operation/army/slot if available.

INFERRED: Broader logging questions remain open until a source log inventory and controlled run logs are reviewed.

DESIGN_HYPOTHESIS: Keep additions behind existing `GC.getLogging() && GC.getAILogging()` gates, use existing per-player/per-city log naming helpers, and avoid changing candidate order, score values, seed inputs, or application behavior.

1. target: `CvUnitProductionAI::CheckUnitBuildSanity` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`.
   DESIGN_HYPOTHESIS: Add an optional component summary line to `CityStrategyAIProductionLog...csv` or a new `UnitProductionAIComponents...csv`: unit, city, `bForOperation`, `bForPurchase`, base flavor after `sqrti`, major gates, major additive buckets, final score or `ProductionSkipReason`.

2. target: `CvBuildingProductionAI::CheckBuildingBuildSanity` in `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`.
   DESIGN_HYPOTHESIS: Add a component summary for building/wonder candidates: base flavor, best-wonder-city result, desired-wonder bonus, rival penalty, war/threat penalty, religion/policy/grand-strategy/yield/geography/maintenance buckets, final sanity result.

3. target: `CvPolicyAI::WeighPolicy` and `CvPolicyAI::WeighBranch` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`.
   DESIGN_HYPOTHESIS: Add one compact row per final candidate in `PolicyAILog...csv` with base cached weight, attribute-yield total, victory buckets, tenet multiplier, branch continuation/finisher modifiers, and final weight.

4. target: `CvReligionAI::ScoreBelief` and helper scoring functions in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`.
   DESIGN_HYPOTHESIS: Add optional top-N or chosen-only detail rows for plot, city, player, and pantheon-special subcomponents to avoid exploding log size.

5. target: `CvAIOperation::CommitToBuildNextUnit`, `UncommitToBuildUnit`, `FinishedBuildingUnit`, `PeekAtNextUnitToBuild`, and `CvCity::GetUnitForOperation`.
   DESIGN_HYPOTHESIS: Log operation ID, army ID, slot ID, primary/secondary `UnitAITypes`, city, chosen `UnitTypes`, and whether the slot moved from needed to committed to finished.

6. target: `CvMilitaryAI::BuyEmergencyUnit`, `BuyEmergencyBuilding`, `CvEconomicAI::CanWithdrawMoneyForPurchase`, and `CvCityStrategyAI::ChooseHurry`.
   DESIGN_HYPOTHESIS: Log purchase failure reasons and affordability decisions with `PurchaseType`, `YieldTypes`, unit/building, city, cost, priority, and `ProductionSkipReason`.

7. target: `CvWeightedVector<T>::ChooseAbovePercentThreshold` call sites rather than the template itself.
   DESIGN_HYPOTHESIS: At each decision log, include cutoff threshold, top score, eligible threshold score, number of eligible candidates, seed literal/call-site seed inputs, and selected index/weight. Avoid logging inside the generic template unless all call sites can tolerate the format.

- Belief regression test: enable AI logging, run fixed seed/map to first pantheon/founding/enhancement/reformation, and compare `ReligionAILog...csv`, `TotalBeliefScoringReligionLog.csv`, and `PlayerBeliefReligionLog.csv` for candidate list, broad component totals, and selected `BeliefTypes`. Breakpoint anchors: `CvReligionAI::ChoosePantheonBelief`, `ChooseFounderBelief`, `ChooseFollowerBelief`, `ChooseEnhancerBelief`, `ChooseReformationBelief`, `CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, `AddReformationBelief`.

- Policy/ideology regression test: compare `PolicyAILog...csv` and `PolicyAIYieldAttributesLog...csv` across a fixed save before policy spending. Validate `CvPolicyAI::ChooseNextPolicy`, `DoChooseIdeology`, `DoConsiderIdeologySwitch`, `CvPlayerPolicies::DoPolicyAI`, and application through `CvPlayer::doAdoptPolicy` / `CvPlayerPolicies::DoUnlockPolicyBranch`.

- City production regression test: compare `CityStrategyAIProductionLog...csv` before and after a candidate tuning patch. Validate precheck candidate generation, post-sanity filtering, skip reasons, final choice, `CvCityStrategyAI::ChooseProduction`, `CvUnitProductionAI::CheckUnitBuildSanity`, `CvBuildingProductionAI::CheckBuildingBuildSanity`, `CvCityStrategyAI::ReweightByDuration`, and `CvCity::pushOrder`.

- Unit role request test: force or observe operation/army/emergency role demand and compare `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)` plus production/tactical logs. Validate `CvUnitProductionAI::RecommendUnit`, `CvCity::GetUnitForOperation`, `CvMilitaryAI::GetUnitTypeForArmy`, `CvMilitaryAI::BuyEmergencyUnit`, and `CvTacticalAI::PlotEmergencyPurchases`.

- Wonder selection test: compare city specialization messages and production logs after tech unlock, city founding, wonder built by rival/us, or grand-strategy change. Validate `CvPlayer::SetBestWonderCities`, `CvWonderProductionAI::ChooseWonder`, `CvCitySpecializationAI::DoTurn`, `CvCityAI::AI_chooseProduction`, and final `CvCityStrategyAI::ChooseProduction`.

- Flavor propagation test: compare `FlavorAILog...csv`, `CityStrategyAILog...csv`, `PolicyAILog...csv`, and `CityStrategyAIProductionLog...csv` after economic/military/city strategy starts/ends. Validate `CvFlavorManager::ChangeActivePersonalityFlavors`, `ChangeCityFlavors`, `CvFlavorRecipient::ChangeFlavors`, `CvPlayerPolicies::FlavorUpdate`, `CvWonderProductionAI::FlavorUpdate`, and `CvCityStrategyAI::FlavorUpdate`.

- Operation lifecycle test: compare `OperationalAILog...csv`, `MilitarySummary.csv`, and city production logs from operation creation through unit demand and movement. Validate `CvPlayer::addAIOperation`, `CvAIOperation::SetUpArmy`, `GrabUnitsFromTheReserves`, `PeekAtNextUnitToBuild`, `CommitToBuildNextUnit`, `FinishedBuildingUnit`, `CvArmyAI::SetFormation`, and `CvCity::GetUnitForOperation`.

- Emergency purchase test: compare `PlayerTacticalAILog...csv`, `CityStrategyAIHurryLog...csv`, and economic/military logs around threatened cities. Validate `CvTacticalAI::PlotEmergencyPurchases`, `CvMilitaryAI::MakeEmergencyPurchases`, `BuyEmergencyUnit`, `BuyEmergencyBuilding`, `CvEconomicAI::CanWithdrawMoneyForPurchase`, and `CvCity::PurchaseUnit` / `PurchaseBuilding`.

- Player turn ordering: `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`; city production trigger: `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvCity::doProduction`; AI movement trigger: `CvGameCoreDLL_Expansion2/CvGame.cpp`, `CvGame::updateMoves`; unit AI update: `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`, `CvPlayerAI::AI_unitUpdate`.

- Shared threshold selector: `CvGameCoreDLLUtil/include/CvWeightedVector.h`, `CvWeightedVector<T>::ChooseAbovePercentThreshold`. RISK: break here only with a conditional/call-stack filter because belief, policy, desired-wonder, production, and hurry decisions all use it.

- `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvGameReligions::GetAvailablePantheonBeliefs`, `GetAvailableFounderBeliefs`, `GetAvailableFollowerBeliefs`, `GetAvailableEnhancerBeliefs`, `GetAvailableBonusBeliefs`, `GetAvailableReformationBeliefs`.

- Scoring and selection: `CvReligionAI::ScoreBelief`, `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, `ScoreBeliefForPlayer`, `ScorePantheonBeliefForPlayer`, and chooser functions `Choose*Belief`.

- `CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, `AddReformationBelief`; prophet triggers in `CvGameCoreDLL_Expansion2/CvUnit.cpp`, `CvUnit::DoFoundReligion`, `CvUnit::DoEnhanceReligion`.

- `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::doTurn` policy section; `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, `CvPlayerPolicies::DoPolicyAI`, `DoChooseIdeology`, `CanAdoptPolicy`, `CanUnlockPolicyBranch`.

- Scoring/selection: `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvPolicyAI::ChooseNextPolicy`, `WeighPolicy`, `WeightPolicyAttributes`, `WeighBranch`, `AddFlavorWeights`, `DoChooseIdeology`, `DoConsiderIdeologySwitch`.

- `CvPlayer::doAdoptPolicy`, `CvPlayer::setHasPolicy`, `CvPlayer::processPolicies`, `CvPlayerPolicies::DoUnlockPolicyBranch`, `SetPolicyBranchUnlocked`, `DoSwitchIdeologies`.

- Desired wonder and best city: `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::SetBestWonderCities`; `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`, `CvWonderProductionAI::ChooseWonder`; `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`, `CvCitySpecializationAI::DoTurn`, `SetSpecializationsDirty`, `WeightProductionSubtypes`.

- Final city production: `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, `CvCityAI::AI_chooseProduction`; `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction`, `ChooseHurry`, `ReweightByDuration`, `LogInvalidItem`, `LogCityProduction`.

- Type-specific sanity: `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, `CvUnitProductionAI::RecommendUnit`, `CheckUnitBuildSanity`; `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`, `CvBuildingProductionAI::CheckBuildingBuildSanity`; `CvProjectProductionAI::CheckProjectBuildSanity`; `CvProcessProductionAI::CheckProcessBuildSanity`.

- `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvCity::pushOrder`, `popOrder`, `produce`, `CreateUnit`, `CreateBuilding`, `PurchaseUnit`, `PurchaseBuilding`.

- Operation creation and demand: `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::addAIOperation`; `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`, `CreateAIOperation`, `CvAIOperation::SetUpArmy`, `GrabUnitsFromTheReserves`, `PeekAtNextUnitToBuild`, `CommitToBuildNextUnit`, `UncommitToBuildUnit`, `FinishedBuildingUnit`, `BuyFinalUnit`.

- Army formation: `CvGameCoreDLL_Expansion2/CvArmyAI.cpp`, `CvArmyAI::SetFormation`, `GetOpenSlots`, `GetSlotInfo`, `AddUnit`, `RemoveUnit`.

- City bridge for operation/army units: `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`, `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot`, `CityCommitToBuildUnitForOperationSlot`, `CityUncommitToBuildUnitForOperationSlot`, `CityFinishedBuildingUnitForOperationSlot`; `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvCity::GetUnitForOperation`, `CheckForOperationUnits`.

- Tactical/homeland movement and purchases: `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, `CvTacticalAI::Update`, `ProcessDominanceZones`, `PlotEmergencyPurchases`; `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp`, `CvHomelandAI::Update`, `ExecuteProphetMoves`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::MakeEmergencyPurchases`, `BuyEmergencyUnit`, `BuyEmergencyBuilding`.

- Flavor propagation: `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`, `CvFlavorManager::Init`, `BroadcastBaseFlavors`, `ChangeActivePersonalityFlavors`, `ChangeCityFlavors`, `GetPersonalityIndividualFlavor`; `CvFlavorRecipient::SetFlavors`, `ChangeFlavors`, `GetLatestFlavorValue`.

- Strategy producers: `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `CvEconomicAI::DoTurn`, `UpdateEconomicStrategies`, `LogStrategy`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::DoTurn`, `UpdateMilitaryStrategies`, `LogStrategy`, `LogMilitaryStatus`; `CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp`, `CvGrandStrategyAI::DoTurn`, `GetPersonalityAndGrandStrategy`, `LogGrandStrategies`; `CvGameCoreDLL_Expansion2/CvDiplomacyAI.cpp`, `CvDiplomacyAI::DoTurn`, `LogStatus`, `LogWarStatus`, `LogMajorCivApproachUpdate`.

## Version differences, risks and unresolved questions

- RISK: `CvUnitProductionAI::CheckUnitBuildSanity` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp` is a major shared scoring hub, but current logs mainly show final positive/negative result through city production and role recommendation lists. Missing component-level output blocks precise tuning of supply, maintenance, recommended army/navy/explorer caps, resource pressure, domain-role bonuses, war/diplomacy terms, religious combat terms, and underrepresented-unit bonuses.

- RISK: `CvBuildingProductionAI::CheckBuildingBuildSanity` in `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp` is shared by buildings, wonders, desired-wonder scoring, best-wonder-city scoring, purchases, and policy/religion valuation. Current logs show candidate weights and rejections but not component-level building/wonder score contributors such as best-wonder-city bonus, desired-wonder bonus, rival construction penalty, war/threat penalty, religion value, policy value, geography, maintenance, or yield terms.

- RISK: `CvPolicyAI::WeightPolicyAttributes` has a yield log, but `CvPolicyAI::WeighPolicy` lacks a compact component breakdown for base flavor weight, conquest/diplomacy/science/culture/generic buckets, tenet multiplier, branch continuation, finisher bonus, and branch-age penalties.

- RISK: `CvReligionAI::ScoreBelief` logs broad totals, but missing subcomponent logs inside `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, and `ScorePantheonBeliefForPlayer` make it hard to explain why specific terrain/resource/city/religion clauses dominate.

- RISK: Operation production-demand logging is incomplete for city conversion. `CvAIOperation::CommitToBuildNextUnit`, `UncommitToBuildUnit`, and `FinishedBuildingUnit` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp` currently maintain deques/vectors but do not log slot movement between needed/committed/finished states.

- RISK: Normal production operation-unit selection has an unmapped lifecycle gap: `CvCityStrategyAI::ChooseProduction` logs `CITY_BUILDABLE_UNIT_FOR_OPERATION`, but current docs do not verify a visible call to `CvPlayerAI::CityCommitToBuildUnitForOperationSlot` on the normal queue path. Logging at queue application or operation-slot commit would block fewer archaeology passes.

- RISK: Emergency purchases are visible only after success in tactical logs from `CvMilitaryAI::BuyEmergencyUnit` and `BuyEmergencyBuilding`. Missing logs for failed candidate, failed purchase affordability, `CvEconomicAI::CanWithdrawMoneyForPurchase`, faith fallback, and sanity rejection make emergency-buy tuning difficult.

- RISK: `CvWeightedVector<T>::ChooseAbovePercentThreshold` in `CvGameCoreDLLUtil/include/CvWeightedVector.h` is shared by belief, policy, desired-wonder, city production, and hurry selection, but selection logs usually show only the selected item plus current random seed in city production. Missing threshold pool and random draw details make replaying threshold choices harder.

- OPEN_QUESTION: Purchase-specific logging outside military emergency buys needs a focused source pass. Known purchase helpers include `CvCityStrategyAI::ChooseHurry`, `CvEconomicAI::DoHurry`, `CvReligionAI::DoFaithPurchases` / `DoFaithPurchasesInCities`, `CvAIOperation::BuyFinalUnit`, `CvCity::PurchaseUnit`, and `CvCity::PurchaseBuilding`.

- RISK: Logging code that evaluates extra getters can accidentally change performance or expose null-entry assumptions. Keep additions read-only and null-guarded like existing candidate loops.

- RISK: Do not alter `CvWeightedVector` ordering, `StableSortItems`, cutoff thresholds, seed literals, or seed mix values while adding logs.

- RISK: Do not add logging that changes savegame serialization, `SYNC_ARCHIVE_*` variables, enum ordering, or database schemas.

- RISK: In multiplayer or deterministic replay work, log-only changes should avoid adding random calls or changing timing-sensitive control flow.

- Classification: HIGH_CONFIDENCE_BUG

- Affected decision: Emergency and direct purchases; tactical city-defense purchases.

- Source functions: `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, `CvTacticalAI::PlotEmergencyPurchases(CvTacticalDominanceZone*)` immediately returns when `!pZone || pZone->IsWater()`. Later in the same function, inside the unit-purchase branch, it checks `if (pZone->IsWater() && pCity->isCoastal())` to buy `UNITAI_ATTACK_SEA`.

- Why current behavior may be suboptimal: the later water-zone naval purchase condition cannot be reached if `pZone->IsWater()` has the same meaning in both checks. INFERRED coastal cities in water dominance zones may never request emergency naval melee from this tactical path.

- Lowest-risk investigation: Log dominance-zone type and city/coastal state at entry and before land defensive purchases to confirm whether a non-water zone can still represent a coastal naval threat.

- Smallest possible patch point: `CvTacticalAI::PlotEmergencyPurchases(...)` entry guard or the later naval branch, depending on intended `CvTacticalDominanceZone::IsWater()` semantics.

- Classification: LIKELY_BAD_HEURISTIC

- Affected decision: Emergency and direct purchases; tactical defensive building purchase.

- Why current behavior may be suboptimal: the emergency path does not call `CvCity::PurchaseBuilding(...)`, so it may bypass purchase cooldowns, investment behavior, purchase instant-yield/event handling, and queue cleanup that the shared purchase function handles. INFERRED this can make defensive emergency buildings mechanically different from other gold purchases even after `IsCanPurchase(...)` passes.

- Runtime/logging test: Trigger emergency building purchase with AI logging, then inspect treasury expenditure logs, city building state, purchase cooldown state, and any city-constructed event logs.

- Risk of double-counting or unintended downstream effects: RISK switching to `PurchaseBuilding(...)` may enable investment mode instead of immediate defense, or fire additional events/instant yields. Direct `CreateBuilding(...)` may have been intentional for emergency immediacy.

- Classification: UNFINISHED_INTEGRATION

- Affected decision: Unit production, military role strategies, policy/tech flavor bucketing.

- Risk of double-counting or unintended downstream effects: RISK role deficits already influence unit sanity through default `MILITARYAISTRATEGY_NEED_RANGED`, recommended army/navy counts, broad `FLAVOR_RANGED`, `FLAVOR_NAVAL`, and operation demand. Adding role flavor rows without recalibration can overproduce specific combat classes and alter policy/tech priorities.

- Classification: TRACEABILITY_PROBLEM

- Why current behavior may be suboptimal: policy scores depend on flavor recomputation, but the flavor input layer is not logged through the normal flavor-recipient hook. INFERRED this makes it hard to diagnose whether a policy was selected due to personality, grand strategy, situational boosts, propagated prereq weights, or `WeightPolicyAttributes(...)`.

- Lowest-risk investigation: Documentation-only log audit: compare `PolicyAILog` and `PolicyAIYieldAttributesLog` with `FlavorAILog` for the same player/turn and list missing inputs.

- Runtime/logging test: Enable AI logging and choose policies across Ancient/Medieval boundary. Verify whether logs reveal the switch from `GetPersonalityIndividualFlavor(...)` to `GetPersonalityAndGrandStrategy(...)` and situational `+5/+3` boosts.

- Lowest-risk investigation: Build an external parser for existing `PRE`, `POST`, invalid-item, flavor-change, and final production logs before adding DLL logging.

- Runtime/logging test: Run the same save with AI logging and inspect one city's production choice across several turns. Confirm whether candidate rank changes can be attributed without source stepping.

- Classification: TESTING_GAP

- Affected decision: Emergency/direct purchases, city hurry, faith military purchase, normal unit/building production.

- Why current behavior may be suboptimal: INFERRED the same sanity functions influence both queue selection and immediate purchases, but purchase application can create units/buildings immediately, invest, insert urgent orders, or bypass shared purchase functions. Without tests, a small scoring change intended for normal production can alter emergency treasury/faith spending and tactical defense.

- Lowest-risk investigation: Create a scenario matrix, not a code patch: normal queue, hurry gold, tactical emergency, operation final-unit purchase, and faith military purchase for the same city and candidate type.

- Runtime/logging test: Enable `CityStrategyAI` hurry logs, tactical logs, religion logs, and treasury logs. Verify that each purchase path has one clear candidate log, one affordability decision, and one application log.

- Classification: DESIGN_HYPOTHESIS_ONLY

- Affected decision: Policy selection, building production, tech selection, and possibly diplomacy/economic strategy.

- Why current behavior may be suboptimal: DESIGN_HYPOTHESIS a leading civ without espionage traits may need stronger defensive espionage pressure, but current atlas has not fully traced spy pressure, counter-intelligence buildings, tech, and policy paths. This is not source-confirmed as a bug.

- Smallest possible patch point: Unknown until source pass. Likely candidates are a logged economic/diplomacy strategy flavor producer or building/policy-specific score component, not a broad flavor change first.

- Runtime/logging test: Compare high-tech/culture leader games with and without espionage pressure: policy logs, building production logs for espionage buildings, tech logs, and diplomacy/spy logs if available.

- Risk of double-counting or unintended downstream effects: RISK `FLAVOR_DIPLOMACY`, `FLAVOR_ESPIONAGE`, policy direct attributes, building flavors, and tech flavors may already overlap. A generic counter-intelligence boost could distort diplomatic victory, spy offense, and defensive building production at the same time.

5. OPEN_QUESTION: Build a merged runtime DB inventory for `Flavors`, `Unit_Flavors`, `Policy_Flavors`, `Technology_Flavors`, `AIMilitaryStrategy_*_Flavors`, and `AI_UNIT_PRODUCTION` before documenting role flavors as active or inactive current behavior.

## Source and dependency lookup

Make AI decisions reconstructable before changing behavior.

Source tracing answers what can happen.
Compiled DB snapshots answer what data exists.
Logs answer what did happen.

1. Inventory existing logs.

2. Run controlled sample play/autoplay.

3. Review logs with ChatGPT brain.

4. Identify missing fields.

5. Ask Codex to trace exact source functions.

6. Add logging-only patch.

7. Re-run sample.

8. Evaluate behavior.

Start small:

- baseline VP, no UPAI;

- VP + UPAI;

- same civ;

- same map seed;

- same game speed;

- same difficulty;

- autoplay 50-100 turns.

Then add:

- land war scenario;

- naval map;

- air-era scenario;

- settler/escort operation scenario.

- Why did the city choose this item?

- Was the unit generic, operation-driven, army-fill, purchase, or faith purchase?

- Which active military strategies existed?

- Which role was requested?

- Which candidates were skipped and why?

- Did UPAI need/enough logic add score or suppress candidate?

- Did the selected unit beat a building/wonder due to strategy pressure?

- Did an operation stall because a formation slot could not be filled?

Production:
- turn/player/city;
- final selected item;
- top unit/building/wonder/project/process candidates;
- current-production inertia;
- duration reweight;
- threshold/seed.

Unit candidate:
- unit type/class;
- UnitAI roles;
- candidate source;
- base flavor weight;
- transformed score;
- active relevant strategies;
- UPAI need/enough effect;
- skip reason;
- final score.

Operation/formation:
- operation type;
- formation type;
- army ID;
- slot ID;
- required/optional;
- primary/secondary UnitAI;
- reserve unit found;
- missing slot queued;
- city selected;
- recommended unit;
- failure/abort reason.

Most logs below are gated by `GC.getLogging()` and `GC.getAILogging()`. Many also split by player or city through `GC.getPlayerAndCityAILogSplit()`. Source locations are exact current-source anchors; existing docs are maps only.

| Subsystem | Log file(s) | Source anchors | Main contents |
|---|---|---|---|
| Belief and religion choice | `ReligionAILog.csv` or split name from `CvGameReligions::GetLogFileName`; `TotalBeliefScoringReligionLog.csv`; `PlayerBeliefReligionLog.csv` | `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvGameReligions::GetLogFileName`, `CvGameReligions::LogReligionMessage`, `CvReligionAI::LogBeliefChoices`, `CvReligionAI::ScoreBelief`, `CvReligionAI::ScoreBeliefForPlayer` | Belief candidates and chosen belief; total plot/owned-city/potential-city/player score components; player-level belief category components; religion application messages. |
| Policy, branch, tenet, ideology | `PolicyAILog.csv` / `PolicyAILog_<player>.csv`; `PolicyAIYieldAttributesLog.csv` / split file | `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvPolicyAI::LogPossiblePolicies`, `LogPolicyChoice`, `LogBranchChoice`, `LogIdeologyChoice`, `LogPolicyAttributeYields`, `GetLogFileName` | Adoptable policy/branch weights; chosen policy or branch; ideology stage weights; per-yield output from `CvPolicyAI::WeightPolicyAttributes`. |
| City strategy and production | `CityStrategyAILog.csv`; `CityStrategyAIProductionLog.csv`; `CityStrategyAIHurryLog.csv`; split city variants | `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::LogFlavorChange`, `LogStrategy`, `LogSpecializationChange`, `LogPossibleBuilds`, `LogInvalidItem`, `LogCityProduction`, `LogPossibleHurries`, `LogHurry`, `LogHurryMessage`, `GetLogFileName`, `GetProductionLogFileName`, `GetHurryLogFileName` | City strategy/flavor deltas; `PRE` and `POST` production candidates with weights and turns; rejected buildables with `ProductionSkipReason`; final production choice, random seed, type, era, turns, GPT; hurry candidates/messages. |
| Unit, building, project, process base production weights | City strategy log files | `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, `CvUnitProductionAI::LogPossibleBuilds`; `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`, `CvBuildingProductionAI::LogPossibleBuilds`; `CvGameCoreDLL_Expansion2/CvProjectProductionAI.cpp`, `CvProjectProductionAI::LogPossibleBuilds`; `CvGameCoreDLL_Expansion2/CvProcessProductionAI.cpp`, `CvProcessProductionAI::LogPossibleBuilds` | Base weighted lists used by production sub-AIs; `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)` also logs `RecommendUnit` special-request lists. |
| Desired wonders and city specialization | City specialization messages, routed through city strategy/specialization logging | `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`, `CvWonderProductionAI::LogPossibleWonders`; `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`, `CvCitySpecializationAI::LogMsg`, `LogSpecializationUpdate`, `LogSpecializationAssignment`; `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::SetBestWonderCities` | Desired-wonder candidates, desired-wonder changes, best city per wonder, specialization update reasons, specialization assignments, direct forced wonder-start messages from `CvCityAI::AI_chooseProduction`. |
| Flavor manager | `FlavorAILog.csv` / `FlavorAILog_<player>.csv`; city flavor deltas also in `CityStrategyAILog...csv` | `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`, `CvFlavorManager::LogActivePersonalityChange`; `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::LogFlavorChange`; `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `CvEconomicAI::LogStrategy`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::LogStrategy` | Active personality flavor changes; economic and military strategy starts/ends; city-local strategy/specialization flavor deltas. |
| Economic AI | `PlayerEconomyAILog.csv` / split file; `EconomicMonitorLog.csv` / split file | `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `CvEconomicAI::GetLogFileName`, `LogEconomyMessage`, `LogMonitor`, `LogCityMonitor`, `LogStrategy`, `DoTurn` | Strategy test messages, active strategy changes, empire/city economic monitor rows, explorer creation/retirement messages. |
| Military AI | `MilitaryAILog.csv` / split file; `MilitarySummary.csv`; flavor log side writes | `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::LogMilitaryStatus`, `LogStrategy`, `LogDeclarationOfWar`, `LogPeace`, `GetLogFileName`, `BuyEmergencyUnit`, `BuyEmergencyBuilding` | Military summary counts and recommended force sizes; military strategy changes; war/peace messages; emergency purchases are logged through tactical messages after purchase. |
| Operations and armies | `OperationalAILog.csv` / `OperationalAILog_<player>.csv` | `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`, `CvAIOperation::LogOperationStart`, `LogOperationStatus`, `LogOperationSpecialMessage`, `LogOperationEnd`, `GetLogFileName`; `CvArmyAI` state is included through operation status output | Operation start/end, state transitions, recruiting/gathering/moving status, army center/target/progress, slot fill state, abort reason, target changes, recruitment rejection messages from `OperationalAIHelpers::IsUnitSuitableForRecruitment`. |
| Tactical AI | `PlayerTacticalAILog.csv` / split file | `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, `CvTacticalAI::LogTacticalMessage`, `GetLogFileName`, `PlotEmergencyPurchases`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::BuyEmergencyUnit`, `BuyEmergencyBuilding` | Tactical movement/combat messages, tactical simulations, emergency purchase messages such as purchased defensive unit/building. |
| Homeland AI | `PlayerHomelandAILog.csv` / split file | `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp`, `CvHomelandAI::LogHomelandMessage`, `GetLogFileName` | Homeland move messages, great-person mission execution, exploration targets, civilian and worker movement decisions. |
| Grand strategy | `GrandStrategyAI_Log.csv` / split file; `GrandStrategyAI_Guess_Log.csv` / split file | `CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp`, `CvGrandStrategyAI::LogGrandStrategies`, `LogGuessOtherPlayerGrandStrategy` | Active grand-strategy priorities and modified priorities; guesses about other players' grand strategies. |
| Diplomacy AI | `DiplomacyAI_Log.csv`, `DiplomacyAI_War_Log.csv`, `DiplomacyAI_Messages_Log.csv`, `DiplomacyAI_Approach_Log.csv`, `DiplomacyAI_Approach_Deltas_Log.csv`, `DiplomacyAI_WarmongerStatus_Log.csv`, `DiplomacyAI_MinorCiv_Log.csv`, `AI_Personality_Log.csv`, with split variants | `CvGameCoreDLL_Expansion2/CvDiplomacyAI.cpp`, `CvDiplomacyAI::LogStatus`, `LogWarStatus`, `LogMajorCivApproachUpdate`, `LogMinorCivApproachUpdate`, `LogApproachValueDeltas`, `LogMajorCivWarmongerUpdate`, `LogPersonality`, message log functions | Diplomacy state that feeds AI scoring: approaches, opinions, proximity, war state, target value, strength, disputes, warmonger state, messages. |

- Belief logs can validate candidate availability after source filters, `CvReligionAI::ScoreBelief` component totals, `CvReligionAI::ScoreBeliefForPlayer` category totals, and final threshold-selected belief through `CvReligionAI::LogBeliefChoices`. They do not by themselves validate every plot/city micro-term inside `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity`, or `ScoreBeliefAtCity`.

- `PolicyAILog...csv` can validate the final adoptable branch/policy/tenet pool, weights after `CvPolicyAI::WeighPolicy` and `WeighBranch`, selected result, and ideology stage totals from `CvPolicyAI::DoChooseIdeology`. `PolicyAIYieldAttributesLog...csv` can validate nonzero yield estimates from `CvPolicyAI::WeightPolicyAttributes`, but not every intermediate policy field contribution.

- `CityStrategyAIProductionLog...csv` can validate the combined candidate competition in `CvCityStrategyAI::ChooseProduction`: precheck weights, post-sanity weights, turns, negative skip reasons through `CvCityStrategyAI::LogInvalidItem`, and the final chosen buildable from `LogCityProduction`.

- `CityStrategyAIHurryLog...csv` can validate `CvCityStrategyAI::ChooseHurry` candidate pools and hurry/purchase messages. It does not cover all purchase paths; `CvMilitaryAI::BuyEmergencyUnit` and `BuyEmergencyBuilding` write successful emergency purchases to `CvTacticalAI::LogTacticalMessage`.

- `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes)` can validate `RecommendUnit(...)` candidate ordering for operation, army, and emergency role requests. It does not log every rejection in `CvUnitProductionAI::CheckUnitBuildSanity`; final city production rejections are visible only when candidates enter `CvCityStrategyAI::ChooseProduction`.

- Desired-wonder and specialization logs can validate `CvWonderProductionAI::ChooseWonder` candidate weights, `CvCitySpecializationAI::DoTurn` desired-wonder changes, and `CvPlayer::SetBestWonderCities` best-city assignments. Final wonder queue choice still must be validated in `CityStrategyAIProductionLog...csv`.

- Flavor logs can validate strategy-driven flavor deltas and recipient-visible city flavor changes. They do not prove how a flavor changed final behavior; production, policy, wonder, and belief scoring logs must be correlated with the same turn/player/city.

- `MilitarySummary.csv` validates aggregate counts used by force-size pressure: land/naval/air counts, recommended army/navy values, supply, war count, and endangered city. It does not expose the full branch-by-branch scoring inside `CvUnitProductionAI::CheckUnitBuildSanity`.

- `OperationalAILog...csv` validates operation lifecycle and slot/army status, including recruiting/gathering/moving states and abort reasons. It does not directly validate which city converted an operation slot into a `CITY_BUILDABLE_UNIT_FOR_OPERATION` candidate; correlate with `CityStrategyAIProductionLog...csv` and `CvCity::GetUnitForOperation`.

- Tactical and homeland logs validate movement-side execution and emergency purchase effects. They do not validate normal city production choice except where tactical purchases bypass the queue.

- Diplomacy and grand-strategy logs validate upstream state used by policy, ideology, belief, building, unit, and specialization scoring, including `CvDiplomacyAI` approach/opinion/dispute state and `CvGrandStrategyAI` priorities.

3. TRACEABILITY_PROBLEM: Improve or externally parse logs before recalibrating major score hubs. Production and policy choices need component-level auditability.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 2U unit logging requirements

Derived from partial current pass `audit-20261029-task1u-01`; capability evidence in [obs007](../logging-coverage.md#obs007). This is a requirements contract, not a chosen C++ logging architecture, and was provisional in the initial pass; Task 1U-B finalization below supersedes that status. No implementation performed.

1. **Decision context:** source/build/content/profile identity, effective feature gates, turn/player/city and invocation identity, trigger/interrupt/control mode, current queue, caller parameter variants (operation/purchase/free), cache generation/read time.
2. **Demand and candidate:** demand kind; operation/army/formation/slot/required/muster IDs; requested primary/secondary roles and fallback result; role deficits and producer snapshot; enumeration identity/order; distinct generic/operation/army duplicates; cache-zero fallback use.
3. **Legality:** canTrain and purchase-query parameter set/results; specific decisive native/Lua rejection where available; silent same-muster/access gate; resource/supply/current-production counts and subtract-current adjustments. Positive eligibility must be distinguishable from absence of an error message.
4. **Flavor cache:** exact consumed city latest stored and clamped values, object coefficient per flavor, product and sum, prior reset/cache version; personality/active/city delta producer and baseline distinction; direct GS/personality bypass values and mode. UPAI role strategy/data/profile keys remain explicit.
5. **Ordered sanity trace:** input, root result, every executed branch/guard and operand with provenance ID; bonus before/delta/after and overwrites/floors; rejected predicate/code; final occupation division/floor. Include skipped role branches and enough-role operation rejection. Constants/Defines/DB keys must be resolvable to content snapshot; runtime getters need actual values and producer age.
6. **Shared competition:** meaningful complete candidate identities/types and PRE/sanity/duration/final scores; actual turns and divisor/Defines; early-expansion ×3; non-unit final outputs without pretending their internal provenance is audited. Explicit empty-sanity restoration event and effective fallback pool after POST.
7. **Selection control:** stable ordering/ties, continuation band and matched item, valid-victory project/defense-process override, effective handicap cutoff/comparator/eligible pool, weight sum, actual draw range/value and mapSeed/gameTurn/city/constant seed context. Keep draw separate from cumulative weighted mapping; log no-draw deterministic outcomes explicitly.
8. **Application and fulfillment:** chosen score/version, requested queue operation/role/rush, queue mutation result and resulting head; investment versus creation; promised slot commit result/token, uncommit/cancellation, actual created unit ID, slot assignment/recruit result and operation stage changes. Never infer fulfillment from selected operation buildable or purchase-intent message.

Minimum correlation must join producer state → cache → invocation/candidate/term → shared selection → application → later unit/slot transition. Snapshot baselines plus reversible deltas may replace repetitive static fields only if reconstruction is deterministic and cache invalidation/load/reset boundaries are recorded. Required unresolved work before finalizing Task 2U scope: complete native legality/profile gate inventory, shared producers, operation lifecycle and additional sanity consumers including process competition. No instrumentation placement or implementation API is prescribed here.

## Task 1U-B Task 2U contract disposition

src-20261029-vp-548 / audit-20261029-task1u-b. The eight Task2U requirements above are retained and refined by current closure evidence, not implemented. Required additional distinctions: latest vector versus consumed cache generation (specialization may mutate without rebuild); normal versus hurry duration ordering; process reward's capital-city unit proxy1500; operation/army candidates with different same-muster semantics; investment versus created unit; promised versus uncommitted production; queue/requested/default/created/current AI roles; reserve assignment versus exact promised-slot completion; optional-unit substitution in HaveEnoughUnits; nonserialized city promise; precheck restoration after empty POST; current source/gate/profile and missing package actions.

Required conceptual records remain decision context, producer snapshots/transitions, demand/request/formation slot, candidate/legality, cache coefficient/product/reduction, ordered sanity trace (t001–t043 plus input p001–p016), common score stages/competitor outputs, deterministic selection or threshold/draw/cumulative mapping, and application/acquisition/creation/recruitment/stage outcomes. Correlation: build/content/profile ID; turn/player/city; invocation+caller and producer/cache generation; candidate **including buildable kind**; operation/army/formation/slot/token; queue action and created unit ID; ordered term IDs and before/after scores. A chosen unit/type is insufficient correlation.

Static constants/DB rows can be deduplicated into a content manifest only if record IDs resolve the exact version actually consumed. Dynamic independent-planner outputs (victory pursuit/progress, trade-ranked list, settlement score/plot, tactical zone, treasury/supply components, trait classification witness) require contemporaneous output snapshots and producing generation. Native legality record needs true/false and decisive gate for nonenumeration/application recheck, not only sanity skip code. Extension listener existence/results and missing-action/load status remain profile/runtime fields; compile/source defaults never substitute for them.

Current output cannot supply a reconstructed full decision without these records. No final C++ logger/API/class design is selected. The final nonprovisional disposition is governed by current coverage status in [Task1U-B checkpoint](../units/task-1u-scope.md#task-1u-b-coverage-and-validation), not by renaming unresolved source dependencies external. No log emitted or modified.

### Final Task 2U contract

The conceptual record contract is **FINAL for the bounded Task1U source interfaces**. It is no longer provisional because of a known unmapped immediate unit-scoring/lifecycle/selection path. UNRESOLVED_EXTERNAL effective profiles, executable missing-file semantics and independent-planner internals are retained as explicit boundaries requiring consumed-output/context records. Task2U may safely implement these observations without enabling UPAI or altering behavior. Independent-source state snapshots do not turn the requested logger into an audit of every upstream optimizer. Require coverage for every t001–t043/p001–p016/f001–f006 branch, including no-candidate/no-order/no-created-unit outcomes and the process proxy. Detailed implementation architecture and enabled default verbosity remain for Task2U review.

Promise rebuild logging also requires committed-list and needed-list membership independently of physical slot occupancy; a committed token can reappear in needed after reserve rebuild.
