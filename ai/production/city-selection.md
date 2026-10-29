# City production selection

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

Why does a city choose one unit, building, project or process over the others?

Normal queue production is described as one mixed buildable competition. Candidate generation, type-specific sanity, duration reweighting, inertia, special overrides and thresholded selection must be kept distinct from queue application.

## Reference synthesis

Unit/building/project/process candidates participate in one mixed city-production vector in the recovered model. Type-specific base values and special operation/army wrappers precede sanity. Duration reweighting, current-production inertia, victory/project or defense-process exceptions, all-rejected fallback and final threshold/weighted selection are separate control stages. Record which score version each consumes.

The recovered description of continuation uses a comparison against half the top weight, while final randomized selection uses the handicap city-production cutoff. Those are different controls with different origins. A logged PRE/POST vector does not decompose each intervening term. The selected buildable is only an intent until queue application rechecks legality.

Canonical related owners: [Unit production and recommendation](<../units/production.md>), [Building valuation and sanity](<../buildings/valuation.md>), [Wonder desire and city assignment](<../wonders/selection.md>), [Shared weighted selection and seeded draws](<../infrastructure/weighted-selection.md>), [Production queues and creation](<../../mechanics/production/queue-and-creation.md>), [AI logs, reconstruction limits and validation scenarios](<../infrastructure/observability.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- city init, `CvCity::doProduction`, queue cleanup, automation, and specialization/dirty flags can call `AI_chooseProduction`.

`CvCity` in `CvGameCoreDLL_Expansion2/CvCity.cpp` owns city turn progression, queue state, legality rechecks, production completion, best-wonder-city flags, and operation-unit lookup. Relevant functions include `CvCity::doTurn`, `CvCity::doProduction`, `CvCity::pushOrder`, `CvCity::popOrder`, `CvCity::produce(UnitTypes, UnitAITypes, bool)`, `CvCity::produce(BuildingTypes, bool)`, `CvCity::produce(ProjectTypes, bool)`, `CvCity::GetUnitForOperation`, `CvCity::SetBestForWonder`, and `CvCity::IsBestForWonder`.

`CvCityAI` in `CvGameCoreDLL_Expansion2/CvCityAI.cpp` is the immediate AI production entry point. `CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)` preserves spaceship-part production, usually preserves current building/wonder production unless interrupt flags permit changing it, handles a narrow desired-wonder Great Engineer path, then delegates to `CvCityStrategyAI::ChooseProduction(...)`. `CvCityAI::AI_isChooseProductionDirty` and `CvCityAI::AI_setChooseProductionDirty` gate forced reselection.

`CvCityStrategyAI` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` owns city strategy state, city-local flavor state, production AI caches, combined production candidate selection, hurry selection, and city-production logging. It serializes `m_piLatestFlavorValues`, active city strategies, specialization state, and the four production AI subobjects through `CvCityStrategyAI::Serialize(...)`.

During `CvCity::doTurn()`, city-local strategy updates run before production. The ordering includes citizen/work-plot validation, `CvCityStrategyAI::DoTurn()`, `CvCityAI::AI_doTurn()`, cooldown and operation checks, razing/resistance/growth handling, and then `CvCity::doProduction(!doCheckProduction())`.

`CvCity::doProduction(bool bAllowNoProduction)` calls `CvCityAI::AI_chooseProduction(false, false)` for AI-controlled production or automated production when the city has no production, is maintaining a process, or `CvCityAI::AI_isChooseProductionDirty()` is true.

`CvCity::setProductionAutomated(bool bNewValue, bool bClear)` can clear the queue when automation is enabled and calls `AI_chooseProduction(false, false)` immediately if no production is present.

`CvCityStrategyAI::SetSpecialization(CitySpecializationTypes)` changes specialization flavor state through `SpecializationFlavorChange(...)` and marks city production dirty with `m_pCity->AI_setChooseProductionDirty(true)`.

INFERRED: Because `CvCityStrategyAI::DoTurn()` runs before `CvCity::doProduction(...)`, city-local strategy flavor changes in the same city turn can affect production weights before the queue selector runs.

- Combined city queue choice among units, buildings, wonders, projects, and processes. Wonders are represented as `CITY_BUILDABLE_BUILDING`, so the final wonder-vs-building-vs-unit-vs-project-vs-process competition happens in one city buildable selector, not in a separate final wonder selector.

- `CvCityStrategyAI::ChooseProduction(BuildingTypes eIgnoreBldg, UnitTypes eIgnoreUnit, bool bInterruptBuildings, bool bInterruptWonders)` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` is the shared selector. `CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)` in `CvGameCoreDLL_Expansion2/CvCityAI.cpp` is the immediate city AI entry point and can preserve/force some current production before delegating. Candidate wrappers and cached base weights are owned by `CvUnitProductionAI`, `CvBuildingProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`.

- `CvCity::doProduction(bool bAllowNoProduction)` in `CvGameCoreDLL_Expansion2/CvCity.cpp` calls `CvCityAI::AI_chooseProduction(false, false)` for AI-controlled or production-automated cities when the city has no production, is maintaining a process, or `CvCityAI::AI_isChooseProductionDirty()` is true. `CvCityAI::AI_chooseProduction(...)` returns early for spaceship-part production, normally preserves existing building/wonder production unless interrupt flags allow a change, may directly start `CvCitySpecializationAI::GetNextWonderDesired()` when the city is best for that wonder and the player has a `UNITAI_ENGINEER`, and otherwise calls `CvCityStrategyAI::ChooseProduction(NO_BUILDING, NO_UNIT, bInterruptBuildings, bInterruptWonders)`. upstream timing is documented in `docs-ai/atlas/01-turn-lifecycle.md`: `CvEconomicAI::DoTurn()`, `CvMilitaryAI::DoTurn()`, `CvReligionAI::DoTurn()`, `CvCitySpecializationAI::DoTurn()`, danger/threat updates, and city-local `CvCityStrategyAI::DoTurn()` run before normal city production.

## Candidates and gates

- `CvCityBuildable` and `CityBuildableType` are defined in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h` with `CITY_BUILDABLE_BUILDING`, `CITY_BUILDABLE_UNIT`, `CITY_BUILDABLE_PROJECT`, `CITY_BUILDABLE_PROCESS`, `CITY_BUILDABLE_UNIT_FOR_OPERATION`, and `CITY_BUILDABLE_UNIT_FOR_ARMY`. `CvCityStrategyAI::ChooseProduction(...)` first clears `m_Buildables` and `m_BuildablesPrecheck`, then pushes all candidate classes into the single `CvWeightedVector<CvCityBuildable> m_BuildablesPrecheck`:
  - operation unit: `CvCity::GetUnitForOperation()` returns a unit, then `ChooseProduction(...)` pushes `CITY_BUILDABLE_UNIT_FOR_OPERATION` with `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)`, `CvMilitaryAI::GetNumberOfTimesOpsBuildSkippedOver()`, `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`, and `CvUnitProductionAI::GetWeight(eUnitForOperation)`.
  - army unit: `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` returns a unit, then `ChooseProduction(...)` pushes `CITY_BUILDABLE_UNIT_FOR_ARMY` with `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT` plus `FLAVOR_OFFENSE * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`.
  - generic units: loop `GC.GetGameUnits()->GetNumUnits()`, require `CvCity::canTrain(UnitTypes, bContinueCurrent)`, apply automated-human filters, then push `CITY_BUILDABLE_UNIT` when `CvUnitProductionAI::GetWeight(UnitTypes)` is positive.
  - buildings and wonders: loop `GC.GetGameBuildings()->GetNumBuildings()`, skip null `CvBuildingEntry`, apply puppet and automated-human filters, require `CvCity::canConstruct(BuildingTypes, totalBuildingCount, bContinueCurrent)`, then push `CITY_BUILDABLE_BUILDING` when `CvBuildingProductionAI::GetWeight(BuildingTypes)` is positive. Automated human cities skip `isWorldWonderClass`, `isTeamWonderClass`, `isNationalWonderClass`, and `isLimitedWonderClass`; AI cities do not use that automated-human wonder skip.
  - projects: loop `GC.GetGameProjects()->GetNumProjects()`, require `CvCity::canCreate(ProjectTypes, bContinueCurrent)`, skip limited projects in puppets, then push `CITY_BUILDABLE_PROJECT` when `CvProjectProductionAI::GetWeight(ProjectTypes)` is positive.
  - processes: loop `GC.getNumProcessInfos()` only when raw city production is at least `500` hundredths or no other precheck candidates exist, require `CvCity::canMaintain(ProcessTypes, bContinueCurrent)`, use `CvProcessProductionAI::GetWeight(ProcessTypes)`, force process weight to `100` for `CvProcessInfo::getDefenseValue() > 0`, then push `CITY_BUILDABLE_PROCESS` with one turn.

## Scoring and inputs

- `RecommendUnit` filters by `CvUnitEntry::GetUnitAIType`, `CvCity::canTrain`, optional strategic-resource allowance, cached unit weight or fallback flavor sum, and duration. `ChooseProduction` wraps the selected unit with operation base weight, offense flavor, skipped-operation count, and unit weight.

- precheck scores are type-specific base weights from `CvUnitProductionAI::GetWeight`, `CvBuildingProductionAI::GetWeight`, `CvProjectProductionAI::GetWeight`, and `CvProcessProductionAI::GetWeight`, plus special operation/army wrapper constants. `CvCityStrategyAI::FlavorUpdate()` resets all four production AI subobjects and broadcasts each latest city flavor from `CvFlavorRecipient::GetLatestFlavorValue(FlavorTypes)` into `CvBuildingProductionAI::AddFlavorWeights`, `CvUnitProductionAI::AddFlavorWeights`, `CvProjectProductionAI::AddFlavorWeights`, and `CvProcessProductionAI::AddFlavorWeights`; those methods multiply each candidate entry's `GetFlavorValue(eFlavor)` by the current city flavor and accumulate cached weights. after `m_BuildablesPrecheck.StableSortItems()`, `ChooseProduction(...)` transforms candidates into `m_Buildables` with type-specific sanity/value checks: `CvUnitProductionAI::CheckUnitBuildSanity(...)`, `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, `CvProjectProductionAI::CheckProjectBuildSanity(...)`, and `CvProcessProductionAI::CheckProcessBuildSanity(...)`. Negative sanity codes are `ProductionSkipReason` values in `CvCityStrategyAI.h`: `SR_IMPOSSIBLE`, `SR_UNITSUPPLY`, `SR_MAINTENANCE`, `SR_STRATEGY`, `SR_USELESS`, and `SR_BALANCE`. `CvCityStrategyAI::ReweightByDuration(CvWeightedVector<CvCityBuildable>&)` then calls `CityStrategyAIHelpers::ReweightByTurnsLeft(...)` on every surviving candidate, regardless of buildable type.

- city state and local legality gates from `CvCity` (`canTrain`, `canConstruct`, `canCreate`, `canMaintain`, production turns, current queue, raw production, puppet/automation flags, siege/threat state through sanity helpers); player state from `CvPlayerAI`, `CvEconomicAI`, `CvMilitaryAI`, `CvGrandStrategyAI`, `CvDiplomacyAI`, `CvReligionAI`, policies, traits, resources, operations, armies, city specializations, danger/threat updates, and current/future war state. operation and army candidate inputs are mapped in `docs-ai/ai-unit-production-upstream.md`; desired-wonder and best-wonder-city inputs are mapped in `docs-ai/ai-wonders.md`; flavor propagation is mapped in `docs-ai/ai-flavors.md`.

- city-production base weights use recipient latest city flavors, not raw leader personality alone. `CvCityStrategyAI::FlavorUpdate()` is the common bridge from city flavor state to all four production AI caches. object flavor rows are loaded by `CvUnitEntry::CacheResults` from `Unit_Flavors`, `CvBuildingEntry::CacheResults` from `Building_Flavors`, `CvProjectEntry::CacheResults` from `Project_Flavors`, and `CvProcessInfo::CacheResults` from `Process_Flavors` via `CvDatabaseUtility::SetFlavors(...)`. audited repository data includes VP rewrites/additions in `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql`, `BuildingFlavorSweeps.sql`, `ProjectFlavorChanges.sql`, and `ProcessFlavorSweeps.sql`, plus strategy flavor deltas in `(1) Community Patch/Database Changes/AI/CoreStrategyChanges.sql`, `(2) Vox Populi/Database Changes/AI/StrategyChanges.sql`, and `(2) Vox Populi/Database Changes/AI/StrategyFlavorSweeps.sql`. operation and army wrapper candidates additionally read `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FLAVOR_OFFENSE)` directly.

- the selector is heavily shaped by non-flavor logic: current production interruption rules in `CvCityAI::AI_chooseProduction(...)`, no-interrupt world-wonder gate in `CvCityStrategyAI::ChooseProduction(...)`, operation and army slot injection, `CvCity::canTrain/canConstruct/canCreate/canMaintain`, puppet and automated-human filters, process availability only for high production or empty precheck, defense process override, type-specific sanity functions, duration reweighting, current-production inertia, victory-project continuation, rush threshold `AI_ATTEMPT_RUSH_OVER_X_TURNS_TO_BUILD`, operation skipped-count pressure, and settler skipped-count bookkeeping.

## Selection and control

`CvCityStrategyAI::ChooseProduction(...)`:
1. Builds `m_BuildablesPrecheck` with operation units, army units, generic units, buildings, projects, and processes.
2. Stable-sorts precheck candidates and logs `PRE`.
3. Runs type-specific sanity checks. Unit entries call `CvUnitProductionAI::CheckUnitBuildSanity(...)`; operation/army items also verify current operation slot/muster city before accepting.
4. Calls `ReweightByDuration(m_Buildables)`.
5. Stable-sorts final candidates and logs `POST`.
6. If sanity removed everything, falls back to `m_BuildablesPrecheck`.
7. Keeps current production if it appears among candidates whose weight is at least half the top weight.
8. Otherwise chooses the defense process if it is the top process, or calls `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getCityProductionChoiceCutoffThreshold(), CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID()))`.
9. Logs the chosen item with `LogCityProduction(...)`.
10. For units, calls `CvCity::pushOrder(ORDER_TRAIN, eUnitType, pkUnitInfo->GetDefaultUnitAIType(), false, true, false, bRush)`.

Downstream queue/application path:
- `CvCity::pushOrder(...)` validates `ORDER_TRAIN` with `CvCity::canTrain(...)`, fills missing `iData2` with `CvUnitEntry::GetDefaultUnitAIType()`, increments the owner's unit-class-making count, and stores the order (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- When production completes, `CvCity::produce(UnitTypes eTrainUnit, UnitAITypes eTrainAIUnit, bool bCanOverflow)` calls `CvCity::CreateUnit(eTrainUnit, eTrainAIUnit, REASON_TRAIN)`, handles some post-create trait/event/overflow logic, and clears or reduces unit production (`CvGameCoreDLL_Expansion2/CvCity.cpp`).
- `CvCity::CreateUnit(UnitTypes eUnitType, UnitAITypes eAIType, UnitCreationReason eReason)` places the unit, calls `CvPlayer::initUnit(...)`, applies production experience and movement state, handles trade-unit logging, settler population rules, rally plots, and some AI role reassignment such as explore-capable units (`CvGameCoreDLL_Expansion2/CvCity.cpp`).

`CvCityBuildable` and `CityBuildableType` are defined in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`. The buildable types are `CITY_BUILDABLE_BUILDING`, `CITY_BUILDABLE_UNIT`, `CITY_BUILDABLE_PROJECT`, `CITY_BUILDABLE_PROCESS`, `CITY_BUILDABLE_UNIT_FOR_OPERATION`, and `CITY_BUILDABLE_UNIT_FOR_ARMY`. Skip codes are `ProductionSkipReason`: `SR_IMPOSSIBLE`, `SR_UNITSUPPLY`, `SR_MAINTENANCE`, `SR_STRATEGY`, `SR_USELESS`, and `SR_BALANCE`.

`CvCityStrategyAI::ChooseProduction(...)` creates one shared `CvWeightedVector<CvCityBuildable>` competition for units, buildings, wonders, projects, and processes. Wonders are ordinary `CITY_BUILDABLE_BUILDING` candidates at this final queue layer.

Candidate generation in `CvCityStrategyAI::ChooseProduction(...)` pushes:

- Operation units from `CvCity::GetUnitForOperation()` as `CITY_BUILDABLE_UNIT_FOR_OPERATION`.

- Army-fill units from `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` as `CITY_BUILDABLE_UNIT_FOR_ARMY`.

- Generic trainable units after `CvCity::canTrain(...)`, with human-automation filters.

- Buildings and wonders after null checks, puppet/automation filters, and `CvCity::canConstruct(...)`.

- Projects after `CvCity::canCreate(...)`, with limited-project puppet filtering.

- Processes after `CvCity::canMaintain(...)`, only when raw city production is at least `500` hundredths or no other candidates exist; defense processes are forced to precheck weight `100`.

Precheck weights come from `CvUnitProductionAI::GetWeight`, `CvBuildingProductionAI::GetWeight`, `CvProjectProductionAI::GetWeight`, and `CvProcessProductionAI::GetWeight`, plus operation and army wrapper constants `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT`, and `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER` from `CvGameCoreDLL_Expansion2/CvGlobals.cpp`.

After `m_BuildablesPrecheck.StableSortItems()`, `ChooseProduction(...)` calls type-specific sanity functions: `CvUnitProductionAI::CheckUnitBuildSanity(...)`, `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, `CvProjectProductionAI::CheckProjectBuildSanity(...)`, and `CvProcessProductionAI::CheckProcessBuildSanity(...)`. Rejected candidates are logged with `CvCityStrategyAI::LogInvalidItem(...)`.

`CvCityStrategyAI::ReweightByDuration(CvWeightedVector<CvCityBuildable>&)` applies `CityStrategyAIHelpers::ReweightByTurnsLeft(...)` to every surviving candidate before final sorting.

Final selection preserves current unit/building/project production if it remains within half the top candidate weight; processes are explicitly not continued for inertia. A valid victory-condition project can force continuation. If not continuing, a top-ranked defense process is deterministic under `MOD_BALANCE_VP`; otherwise selection uses `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold(...)` with `CvHandicapInfo::getCityProductionChoiceCutoffThreshold()` and seed `CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID())`.

RISK: This is a shared competition hub. Any change to one class's candidate list, cached flavor weight, sanity function, duration reweighting, or selection threshold changes the opportunity cost and frequency of all other buildable classes.

## Application and state

- Final selection: final queue selection is the shared city-production selection, not a unit-only selection. `CvCityStrategyAI::ChooseProduction(...)` stable-sorts surviving `m_Buildables`, falls back to `m_BuildablesPrecheck` if sanity removed everything, can keep current unit production if it is within half the top candidate weight, can deterministically pick a top defense process, and otherwise calls `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getCityProductionChoiceCutoffThreshold(), CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID()))`.

## Data and configuration

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvAICityStrategyEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`) | `AICityStrategy_Flavors`, `AICityStrategy_PersonalityFlavorThresholdMods` | City-local strategies and city flavor deltas consumed by city production. |

- source classes and entries include `CvCityStrategyAI`, `CvCityAI`, `CvCity`, `CvCityBuildable`, `CvWeightedVector<CvCityBuildable>`, `CvUnitProductionAI`, `CvBuildingProductionAI`, `CvProjectProductionAI`, `CvProcessProductionAI`, `CvUnitEntry`, `CvBuildingEntry`, `CvProjectEntry`, `CvProcessInfo`, `CvBuildingClassInfo`, `CvHandicapInfo`, `CvFlavorRecipient`, `CvFlavorManager`, `CvGrandStrategyAI`, and `CvMilitaryAI`. enum/order-sensitive types include `CityBuildableType` and `ProductionSkipReason` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`, `OrderTypes` (`ORDER_TRAIN`, `ORDER_CONSTRUCT`, `ORDER_CREATE`, `ORDER_MAINTAIN`) in `CvGameCoreDLLUtil/include/CvEnums.h`, and database-backed `UnitTypes`, `BuildingTypes`, `ProjectTypes`, `ProcessTypes`, `FlavorTypes`, `UnitAITypes`, and `BuildingClassTypes`. database tables include `Unit_Flavors`, `Building_Flavors`, `Project_Flavors`, `Process_Flavors`, `AICityStrategy_Flavors`, `AIEconomicStrategy_Player_Flavors`, `AIEconomicStrategy_City_Flavors`, `AIMilitaryStrategy_Player_Flavors`, `AIMilitaryStrategy_City_Flavors`, `CitySpecialization_Flavors`, and `HandicapInfos.CityProductionChoiceCutoffThreshold`; define-backed constants include `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT`, `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT`, `AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`, and `AI_ATTEMPT_RUSH_OVER_X_TURNS_TO_BUILD` in `CvGameCoreDLL_Expansion2/CvGlobals.cpp` and `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql`.

## Observability and validation obligations

- `CvCityStrategyAI::LogPossibleBuilds`, `LogCityProduction`, `LogInvalidItem`, and `LogFlavorChange`.

- `CvCityStrategyAI::LogFlavorChange` logs city flavor changes; other flavor logs need a focused pass.

`CvCityStrategyAI::LogPossibleBuilds(...)` writes `PRE` and `POST` buildable lists to the production log from `CvCityStrategyAI::GetProductionLogFileName(...)`.

`CvCityStrategyAI::LogInvalidItem(...)` records rejected candidates and maps `ProductionSkipReason` values to text such as `impossible`, `nosupply`, `tooexpensive`, and `badtimeorplace`.

`CvCityStrategyAI::LogCityProduction(...)` records selected type, description, rush/no-rush state, era, turns, GPT, and the current JonRand seed. Buildings with max global or player instances are logged as `"Wonder"`.

`CvCityAI::AI_chooseProduction(...)` logs the direct desired-wonder start through both `CvCityStrategyAI::LogCityProduction(...)` and `CvCitySpecializationAI::LogMsg(...)`. `CvCity::produce(BuildingTypes, bool)` logs wonder completion through `CvCitySpecializationAI::LogMsg(...)`.

- `CvCityStrategyAI::LogPossibleBuilds(...)` logs `PRE` and `POST` weighted buildable lists to `CityStrategyAIProductionLog...csv` via `CvCityStrategyAI::GetProductionLogFileName(...)`. `CvCityStrategyAI::LogInvalidItem(...)` logs rejected candidates and maps `ProductionSkipReason` codes. `CvCityStrategyAI::LogCityProduction(...)` logs selected type, description, rush/no-rush, era, turns, GPT, and current random seed; buildings with max global/player instances are logged as `"Wonder"`. `CvCityStrategyAI::LogFlavorChange(...)` logs city flavor changes that feed later production weights. direct desired-wonder starts in `CvCityAI::AI_chooseProduction(...)` also call `LogCityProduction(...)` and write a city-specialization message.

## Version differences, risks and unresolved questions

- Smallest possible patch point: `CvCityStrategyAI::ChooseProduction(...)` around the transition from `m_BuildablesPrecheck` to `m_Buildables`, recording component columns only under `GC.getAILogging()`.

OPEN_QUESTION: `CITY_BUILDABLE_UNIT_FOR_ARMY` is inserted separately, but the post-precheck branch for both operation and army unit buildables requires a valid same-city `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` before sanity scoring. The source behavior is verified; the design intent for pure army-fill candidates needs separate tracing.

RISK: Units, buildings, wonders, projects, and processes compete in one `CvWeightedVector<CvCityBuildable>`. Production scoring is not class-local at the final decision point.

RISK: `CvUnitProductionAI::CheckUnitBuildSanity(...)`, `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, `CvProjectProductionAI::CheckProjectBuildSanity(...)`, and `CvProcessProductionAI::CheckProcessBuildSanity(...)` are shared hubs. In particular, building sanity covers ordinary buildings and wonders, while unit sanity covers generic, operation, and army candidates.

RISK: Determinism depends on stable candidate ordering, `StableSortItems()`, database-backed enum ordering, and seeded threshold choice with `0x0e36d18b` mixed with city ID. Handicap table `HandicapInfos.CityProductionChoiceCutoffThreshold` is defined by `(1) Community Patch/Database Changes/Difficulty/DifficultyTableChanges.sql`.

RISK: Best-wonder flags (`CvCity::m_abIsBestForWonder`), current-production inertia, desired-wonder state from `CvCitySpecializationAI::GetNextWonderDesired()`, and operation demand from `CvPlayerAI` / `CvAIOperation` / `CvArmyAI` all feed the same final selector.

RISK: Source comments and docs may describe unit, wonder, or process choice separately, but audited source applies them through the shared city queue path unless a path explicitly bypasses it, such as the desired-wonder Great Engineer start in `CvCityAI::AI_chooseProduction(...)` or purchase/hurry selection in `CvCityStrategyAI::ChooseHurry(...)`.

- Risk notes: RISK this is the central shared competition hub for normal city queue choices. Changing candidate generation, base weights, sanity checks, duration weighting, current-production inertia, or threshold selection for one buildable class changes the opportunity cost and selection frequency of all other classes. RISK deterministic multiplayer/save behavior depends on seeded selection `0x0e36d18b` mixed with city ID and on stable database enum ordering for unit/building/project/process/flavor IDs. RISK savegame-sensitive serialized state includes `CvCityStrategyAI::Serialize(...)` and production AI cache serialization for `CvUnitProductionAI::Serialize(...)`, `CvBuildingProductionAI::Serialize(...)`, `CvProjectProductionAI::Serialize(...)`, and `CvProcessProductionAI::Serialize(...)`. RISK `CvCity::pushOrder(...)` updates making counts and queue state, so failed or changed legality rechecks can desynchronize score intent from applied production. RISK `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` is shared by ordinary buildings and wonders, while `CvUnitProductionAI::CheckUnitBuildSanity(...)` is shared by generic units, operation units, army units, and some purchase helpers.

- Known improvement hooks: DESIGN_HYPOTHESIS a reusable production-score index should split `CvCityStrategyAI::ChooseProduction(...)` into candidate generation, precheck weight, sanity score, duration score, selection, and application tables for each `CityBuildableType`. OPEN_QUESTION trace `CvCity::canTrain`, `CvCity::canConstruct`, `CvCity::canCreate`, and `CvCity::canMaintain` if a future record needs complete legality gates, Lua/game-event vetoes, resource constraints, or schema dependencies. OPEN_QUESTION operation/army candidate validation deserves a focused follow-up: audited source injects `CITY_BUILDABLE_UNIT_FOR_ARMY`, but the normal post-precheck path handles army and operation unit buildables in the same branch requiring `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` and same muster city. OPEN_QUESTION completion side effects for `ORDER_CREATE` projects and `ORDER_MAINTAIN` processes should be documented separately before changing project/process scoring.

## Source and dependency lookup

| Class / manager | Decisions affected |
|---|---|
| `CvProjectProductionAI` / `CvProcessProductionAI` (`CvGameCoreDLL_Expansion2/CvProjectProductionAI.cpp`, `CvProcessProductionAI.cpp`) | project/process base weights and sanity checks used by `CvCityStrategyAI::ChooseProduction(...)`; process and project production decisions. Affects city production and flavor propagation. |

| Function / enum / table | Decisions affected |
|---|---|
| `CvCityStrategyAI::FlavorUpdate()`, `SetSpecialization(...)`, `SpecializationFlavorChange(...)`, `DoTurn()` (`CvCityStrategyAI.cpp`) | city flavor propagation and production cache refresh. |
| `CvCityStrategyAI::ReweightByDuration(...)`, `LogPossibleBuilds(...)`, `LogInvalidItem(...)`, `LogCityProduction(...)`, `LogPossibleHurries(...)` (`CvCityStrategyAI.cpp`) | production/hurry duration scoring and logs. |

- trigger: `CvPlayer::doTurnPostDiplomacy` -> `CvCity::doTurn` -> `CvCity::doProduction` -> `CvCityAI::AI_chooseProduction`.

- producers: `CvEconomicAI::UpdateEconomicStrategies`, `CvMilitaryAI::UpdateMilitaryStrategies`, `CvCityStrategyAI::DoTurn`, `CvCityStrategyAI::SetSpecialization`, `CvGrandStrategyAI::GetPersonalityAndGrandStrategy`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvCityStrategyAI::ChooseHurry(...)` | economic hurry, faith military purchase, operation/army purchase candidates. | RISK purchase behavior bypasses or modifies normal queue competition. |

`CvCity`, `CvCityAI`, and `CvCityStrategyAI` are the city-local turn, production-entry, and production-selection classes for normal queue decisions. The final city queue selector is `CvCityStrategyAI::ChooseProduction(BuildingTypes eIgnoreBldg, UnitTypes eIgnoreUnit, bool bInterruptBuildings, bool bInterruptWonders)` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`.

`CvCity::canTrain`, `CvCity::canConstruct`, `CvCity::canCreate`, and `CvCity::canMaintain` are the first queue-level legality gates in `CvCityStrategyAI::ChooseProduction(...)`; `CvCity::pushOrder(...)` revalidates the chosen order before mutating queue state.

`CvCityAI::AI_chooseProduction(...)` refuses to alter spaceship-part production with `isProductionSpaceshipPart()`. It also returns early when already building a wonder and `bInterruptWonders` is false, or when already building a building and `bInterruptBuildings` is false.

`CvCityStrategyAI::ChooseProduction(...)` also returns early for current combat-unit production when `bInterruptBuildings` is true, and returns early for current world-wonder production when `bInterruptWonders` is false.

Wonder constraints are split. `CvCity::SetBestForWonder(BuildingClassTypes, bool)` / `CvCity::IsBestForWonder(BuildingClassTypes)` store city-local best-wonder flags in `m_abIsBestForWonder`, and that vector is serialized in `CvCity::Serialize(...)`. Final wonder candidates still enter as buildings, but `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` consumes best-wonder-city state.

`CvCity::GetUnitForOperation()` reads `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity*, bool&)`, requires the same muster city, checks `CvAIOperation::GetMusterPlot()` landmass/ocean access with `CvCity::HasAccessToLandmassOrOcean(...)`, reads `CvArmyAI::GetSlotInfo(...)`, and calls `CvUnitProductionAI::RecommendUnit(...)` for primary and secondary `UnitAITypes`.

- `CvCity::doProduction` (`CvCity.cpp`): normal AI/automated production reselection trigger; process replacement; dirty production response.

- `CvCity::setProductionAutomated` (`CvCity.cpp`): automated human city production trigger; optional queue clearing; immediate AI choice when empty.

- `CvCity::pushOrder` (`CvCity.cpp`): final application for unit, building/wonder, project, and process decisions; legality recheck; making-count mutation.

- `CvCity::popOrder` (`CvCity.cpp`): completion/removal application; making-count decrement; calls unit/building/project production completion.

- `CvCity::produce(UnitTypes, UnitAITypes, bool)` (`CvCity.cpp`): unit completion side effects; trained-unit events; overflow; AI settler/found-city strategy hook.

- `CvCity::produce(BuildingTypes, bool)` (`CvCity.cpp`): building and wonder completion; constructed-building events; wonder completion logging; overflow.

- `CvCity::produce(ProjectTypes, bool)` (`CvCity.cpp`): project completion; project events; overflow.

- `CvCity::GetUnitForOperation` (`CvCity.cpp`): operation unit candidate generation; formation-slot `UnitAITypes`; muster-city and landmass/ocean constraints.

- `CvCity::SetBestForWonder` / `CvCity::IsBestForWonder` (`CvCity.cpp`): desired-wonder candidate generation and final wonder sanity constraints.

- `CvCityAI::AI_isChooseProductionDirty` / `CvCityAI::AI_setChooseProductionDirty` (`CvCityAI.cpp`): production reselection trigger state.

- `CvCityStrategyAI::SetSpecialization` / `CvCityStrategyAI::SpecializationFlavorChange` (`CvCityStrategyAI.cpp`): city specialization flavor changes; dirty production trigger.

- `CvCityStrategyAI::FlavorUpdate` (`CvCityStrategyAI.cpp`): rebuilds unit/building/project/process cached production weights from city-local latest flavors.

- `CvCityStrategyAI::ReweightByDuration` (`CvCityStrategyAI.cpp`): production duration pressure for every surviving buildable class.

- `CvCityStrategyAI::ChooseHurry` (`CvCityStrategyAI.cpp`): purchase/hurry selection using similar buildable machinery, but not normal queue selection.

- `CvCityStrategyAI::LogInvalidItem` (`CvCityStrategyAI.cpp`): sanity-rejection audit for all buildable classes.

- `CvCityStrategyAI::LogCityProduction` (`CvCityStrategyAI.cpp`): final selected production audit.

- `CvCityStrategyAI::Serialize` (`CvCityStrategyAI.cpp`): savegame-sensitive city flavor, strategy, specialization, and production AI cache state.

- final selection is shared across all entries in `m_Buildables`, after `m_Buildables.StableSortItems()`. If all sanity checks reject all candidates, `ChooseProduction(...)` falls back to `m_BuildablesPrecheck`. It then keeps the current unit, building, or project if that current item appears among candidates with weight at least half the top candidate; processes are explicitly not continued for inertia. A valid victory-condition project can also force continuation. If not continuing, `MOD_BALANCE_VP` makes a top-ranked defense process deterministic. Otherwise `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold(...)` selects from the top scoring candidates using `CvHandicapInfo::getCityProductionChoiceCutoffThreshold()` and seed `CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID())`. this is the concrete current weighted-vector behavior in `CvCityStrategyAI::ChooseProduction(...)`; `docs-ai/vp-ai-quickref.md` was used only as a routing aid.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U current unit selection

VERIFIED_IN_SOURCE, `src-20261029-vp-c285a18`, `audit-20261029-task1u-01`, `CvCityStrategyAI.cpp:698–1148,1748–1760,2261–2267`; helper `CvGameCoreDLLUtil/include/CvWeightedVector.h:162–234`; RNG `CvGame.cpp:10142–10184`. This section partially revalidates unit integration only; detailed non-unit provenance remains deferred.

Ordered graph:

1. Interrupt-buildings request preserves an existing combat unit; uninterruptible world wonder returns. Clear both vectors.
2. Operation preweight = `AI_CITYSTRATEGY_OPERATION_UNIT_BASE_WEIGHT + (GS(OFFENSE)+opsSkipped)*AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER + unitCache`; positive insertion bumps skipped count **before sanity**. Army preweight = `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT + GS(OFFENSE)*multiplier`, with no cache term. Generic weight=GetWeight, require positive/cache and canTrain(continue current). Human automation permits only VP noncombat units with work rate. Same unit may appear multiple times with distinct buildable types/weights.
3. Buildings/wonders use building weight and legality; projects use project weight and legality (puppet limited-project filter); processes enumerated when rawProductionTimes100>=500 or no precheck options, defense process preweight overwritten100. Puppet building limitation and human automation alter competitor pool. Detailed competitor formulas are unresolved inputs to why a unit won.
4. Stable-sort PRE; log. Operation **and army** require another valid same-muster operation slot before CheckUnitBuildSanity(operation=true); otherwise silently absent from checked pool. Positive sanity gets ×3 in early expansion for either tag. Generic sanity operation=false. Positive results enter final pool; rejected ones log skip reason. Non-unit sanity runs independently.
5. Duration transform for every final item: `int(double(score)/pow(double(turns), AI_PRODUCTION_WEIGHT_BASE_MOD + turns*AI_PRODUCTION_WEIGHT_MOD_PER_TURN_LEFT))`. No local zero-turn clamp in helper. Unit recommendation earlier has its own duration transform; wrapper preweight is independently constructed, not the recommendation's transformed score.
6. Stable-sort POST; log. If pool empty, restore PRE vector **after** duration/logging. Fallback thus bypasses sanity rejections and duration; POST can be empty while a real choice follows.
7. Scan ordered items until weight<topWeight/2 (integer). First current matching unit/building/project continues (building needs !interrupt); valid-victory project in this band can force continuation even if not current. Processes never continue. This is deterministic precedence, not random tie-breaking.
8. Otherwise VP defense process at top is deterministic override; otherwise cutoff selection below. Rush is turns>`AI_ATTEMPT_RUSH_OVER_X_TURNS_TO_BUILD*TrainPercent/100`. Selection is logged before application.
9. All unit tags pushOrder(ORDER_TRAIN,type,defaultRole,false,true,false,rush); only operation tag resets operation skips. Settler counter reset if chosen founder or no settler in final pool, else bumped. No positive queue-result assertion or operation commit in selector. Shared queue/application owner remains a transitive dependency.

Cutoff (`threshold`, then `weighted-select`): clamp handicap CityProductionChoiceCutoffThreshold to [0,100]; highest=max(0,weights). If highest==0, choose uniformly by vector index. If percent==0 use whole weighted vector. Otherwise cutoff=highest*percent/100 (integer); retain weight>=cutoff including equality, in vector order. ChooseByWeight sums weights; zero total chooses uniformly; else draw in [0,total) and subtract successive weights until negative. Equal-weight items preserve enumeration via stable sort. This samples by score across the eligible band; it is not merely a tie-break.

`random-draw` is separate: normal selector extra seed=`CvSeeder::fromRaw(0x0e36d18b).mix(cityID)`; randCore combines mapRandomSeed, game turn, extra seed, then `% total` (or `% size` uniform fallback). Modulo mapping is source fact; ideal unbiased uniformity is not claimed. Exact CvSeeder hash implementation still needs its own term record. No persistent RNG stream draw is shown by these helpers. ChooseHurry has a separate seed `0xe362f42a` and remains partially mapped.

Logging exposes PRE and POST pool totals plus selected buildable, but not component versions, cutoff membership, draw or continuation/override reason. Reference DB confirms five wrapper/duration Defines only for its own hash; selected-source effective handicap/cost profile is not established.

## Task 1U-B selector and process-interface closure

src-20261029-vp-548 / audit-20261029-task1u-b. ChooseProduction698–1148, duration1748/2261 and weighted helpers are unchanged and directly rechecked on5.4.8; prior ordered graph remains applicable. Add this current distinction: **ChooseHurry is not ChooseProduction with a different currency**. Hurry1154–1401 omits operation/army injections for faith; applies military-support actual supply gate; generic gold/faith uses IsCanPurchase and excludes missionary/inquisitor/nonfaith/special faith candidates as coded. It sorts PRE then duration-reweights **before** sanity; operation sanity needs same-muster and operation=true, army sanity operation=false with no shared operation muster test. Normal production's army+operation common gate remains source behavior. Hurry uses seed0xe362f42a.mix(cityID), samples only when final total>0, otherwise returns NOT_A_CITY_BUILDABLE, with no normal PRE restoration. Faith caller can subsequently rerun sanity's gold-purchase predicate; preserve currency versus sanity parameter mismatch.

`production.process-unit-proxy:t001`: ProcessProductionAI::CheckProcessBuildSanity336–493 iterates LeagueProjects whose Process matches. Sets bIsProject=true, adds current-process continuation10000, requires CanContributeToLeagueProject to value rewards, loops tier3,2,1 reward entries. Each reward FreeUnitClass resolves GetSpecificUnitType for civilization; starts proxy1500, capital exists→capital CheckUnitBuildSanity(unit,false,1500,false,false), no capital retains1500; positive proxy added to iModifier, negative/zero ignored. No RecommendUnit or unit flavor cache in this proxy. Process final score for league project=max(1,inputWeight+iModifier); nonproject uses inputWeight*(100+iModifier)/100. The proxy can change whether process defeats a unit; it does not create immediate formation demand or buy the reward. Reward fields from LeagueProjectRewards and league eligibility are DB/native interfaces. Other process bonuses/building reward scores remain deliberately non-unit valuation dependencies; this prevents whole-city score provenance COMPLETE but the unit proxy is resolved.

No current-production counter/empty-pool behavior is inferred from selected item alone. Normal PRE fallback bypasses earlier sanity and duration; ordinary queue acceptance can still fail after clearing current queue. Common record must include effective pool generation and applied result.
