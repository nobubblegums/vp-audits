# Production queues and creation

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../../ai/source-snapshots.md).

## Decision or contract question

How does a selected order become queued work, completion and created objects?

This owns the shared player-facing queue/application mechanics. AI decisions link here for pushOrder, legality rechecks, making counts, produce/CreateUnit/CreateBuilding and related state. Source coverage of full native legality remains incomplete.

## Reference synthesis

AI selects a buildable; shared queue/application code decides whether the order can be accepted. `pushOrder` revalidates train/construct/create/maintain conditions and updates queue/making/UI state. Later production completion reaches creation or progress/process application, with additional state effects. Human/network and AI entry points may share application while differing in selection.

The old reports do not complete every native legality, Lua veto or cancellation branch. In particular, queueing an operation buildable does not prove its city-to-operation commitment linkage. Keep that unresolved interface visible in the demand and operation-state owners.

Canonical related owners: [City production selection](<../../ai/production/city-selection.md>), [Operation and army demand reaching unit production](<../../ai/units/operation-demand.md>), [Operation and army state contracts](<../../architecture/operations-armies.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Application and state

`CvCityStrategyAI::ChooseProduction(...)` applies the selected item through `CvCity::pushOrder(...)`: units use `ORDER_TRAIN`, buildings and wonders use `ORDER_CONSTRUCT`, projects use `ORDER_CREATE`, and processes use `ORDER_MAINTAIN`. `OrderTypes` are defined in `CvGameCoreDLLUtil/include/CvEnums.h`.

`CvCity::pushOrder(...)` rechecks legality, increments owner/team making counts for units/buildings/projects, clears the queue when `bPop` is true, inserts the order, starts the head order, updates city strength for defense processes, and dirties city UI state.

`CvCity::popOrder(...)` decrements unit/building/project making counts and calls completion paths when `bFinish` is true. Unit completion reaches `CvCity::produce(UnitTypes, UnitAITypes, bool)` and `CvCity::CreateUnit(...)`; building completion reaches `CvCity::produce(BuildingTypes, bool)` and `CvCity::CreateBuilding(...)`; project completion reaches `CvCity::produce(ProjectTypes, bool)` and `CvCity::CreateProject(...)`.

Unit completion can trigger traits, Lua/game events (`GAMEEVENT_CityTrained` or `CityTrained`), instant yields, overflow handling, and for AI settlers can call `EconomicAIHelpers::IsTestStrategy_FoundCity(...)` to immediately reconsider founding-city strategy. Building and project completion similarly trigger game events/Lua hooks and overflow handling.

RISK: `CvCity::pushOrder(...)`, `CvCity::popOrder(...)`, and `CvCity::produce(...)` are side-effect-heavy application/completion paths. A selector may choose one buildable, but legality rechecks, making counts, queue insertion mode, production overflow, traits, game events, and completion hooks determine final game-state effects.

Policy application:

- `CvPlayer::setHasPolicy(PolicyTypes, bool, bool)` mutates policy ownership and calls `CvPlayer::processPolicies(...)`.

- UI/network responses enter these paths through `CvDllNetMessageHandler::ResponseAdoptPolicy` and `ResponseAdoptPolicyBranch`.

Religion application:

- UI/network responses for religion founding/enhancement are in `CvDllNetMessageHandler.cpp`, `ResponseFoundReligion` and `ResponseEnhanceReligion`.

Production application:

- `CvCity::pushOrder(...)` in `CvCity.cpp` is the shared queue application for normal city production, including AI-selected `ORDER_TRAIN` / `ORDER_CONSTRUCT`.

- `CvCity::PurchaseUnit(...)` in `CvCity.cpp` is the shared immediate unit purchase path used by emergency military and faith-purchase decisions.

- `CvCityStrategyAI::ChooseProduction(...)` is the normal city queue selector but final production completion effects occur later through city production/creation functions such as `CvCity::produce(...)`, `CvCity::CreateUnit(...)`, and `CvCity::CreateBuilding(...)`.

- `CvCityStrategyAI::ChooseProduction(...)` applies the chosen `CvCityBuildable` with `CvCity::pushOrder(...)` in `CvGameCoreDLL_Expansion2/CvCity.cpp`: units use `ORDER_TRAIN` and `CvUnitEntry::GetDefaultUnitAIType()`, buildings and wonders use `ORDER_CONSTRUCT`, projects use `ORDER_CREATE`, and processes use `ORDER_MAINTAIN`. `CvCity::pushOrder(...)` revalidates legality with `canTrain`, `canConstruct`, `canCreate`, or `canMaintain`; updates owner/team making counts for units, buildings, and projects; clears or inserts into the order queue; starts the head order; updates defense strength for defense processes; and dirties UI city-production state. selected operation units reset `CvMilitaryAI::ResetNumberOfTimesOpsBuildSkippedOver()`, and settler candidate outcomes update `ResetNumberOfTimesSettlerBuildSkippedOver()` or `BumpNumberOfTimesSettlerBuildSkippedOver()`.

- queue choice changes unit/building/project/process production progress and making counts through `CvCity::pushOrder(...)`. Later completion paths are outside this selector but include unit creation through `CvCity::produce(...)` and `CvCity::CreateUnit(...)`, building creation through `CvCity::CreateBuilding(...)`, project progress/completion through project production handling, and process effects while maintained. INFERRED because the selector is a single shared vector, raising one class's precheck or sanity weight changes not only that class's internal ranking but also the frequency of all other buildable classes.

- Queue application: selected units of any unit buildable class apply with `CvCity::pushOrder(ORDER_TRAIN, eUnitType, CvUnitEntry::GetDefaultUnitAIType(), false, true, false, bRush)` in `CvCityStrategyAI::ChooseProduction(...)`. `CvCity::pushOrder(...)` in `CvGameCoreDLL_Expansion2/CvCity.cpp` revalidates `ORDER_TRAIN` with `CvCity::canTrain(...)`, fills/uses the `UnitAITypes`, updates the owner's unit-class making counts, and stores the order. Production completion later reaches `CvCity::produce(...)` and `CvCity::CreateUnit(UnitTypes, UnitAITypes, UnitCreationReason)`.

- final queue application for a selected wonder uses `CvCity::pushOrder(ORDER_CONSTRUCT, eBuildingType, -1, false, true, false, bRush)` in `CvCityStrategyAI::ChooseProduction(...)`; the direct desired-wonder bypass uses `pushOrder(ORDER_CONSTRUCT, eNextWonder, -1, false, false, false, false)` in `CvCityAI::AI_chooseProduction(...)`. later building completion and effect application are outside this selector; the relevant completion-side functions include `CvCity::CreateBuilding(...)` and `CvCityBuildings::SetNumRealBuilding(...)` in the city/building code.

- best-city flags constrain both desired-wonder generation and final national/world wonder sanity checks. desired-wonder selection biases final building sanity through `CvCitySpecializationAI::GetNextWonderDesired()`. final city production changes the order queue through `CvCity::pushOrder(...)`, causing normal production progress and later construction completion effects. INFERRED because final production uses one shared vector, increasing a wonder's building weight or sanity bonus changes its frequency relative to units, ordinary buildings, projects, and processes, not only relative to other wonders.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U current unit queue and creation

Partial current verification `src-20261029-vp-c285a18`, `audit-20261029-task1u-01` ([identity](../../ai/source-snapshots.md#task-1u-selected-source-pass)); only this section current. Source CvCity.cpp pushOrder29312, popOrder29455, produce(UnitTypes)29855, CreateUnit30125. Other above-baseline mechanics remain historical.

Normal AI call uses bPop=true: clears existing queue first, then canTrain with default parameters (no continue-current flag); valid unit increments class-making, missing role becomes default. This void application can leave a cleared queue if legality fails. Selected-choice log occurs earlier, so it proves intent only. Human application shares queue mechanics but not automatic scoring/selection.

popOrder ORDER_TRAIN decrements class-making; bFinish calls `produce(eTrainUnit)` **without forwarding extracted eTrainAIUnit**. produce uses its declaration/default role parameter contract, not necessarily the role stored in queue. It creates unit, clears investment and may replace/upgrade it through trait effects, emits CityTrained via MOD_EVENTS_CITY or Lua hook, applies completion yields/production overflow; failed creation retains near-complete production. The tail accesses pUnit->isFound after the null-creation branch: source-confirmed null-risk expression, no observed crash or fix claimed.

CreateUnit chooses GetPlotForNewUnit or stacks city if no plot, initUnit, purchased damage option, production experience, moves restored except purchases lacking move-after-purchase, settler population option, rally mission or secondary explorer role reassignment, built count/completion yields. None of these inspected bodies calls CityFinishedBuildingUnitForOperationSlot. popOrder then uncommits/invalidates a stored operation promise regardless of finish. See [operation bridge](../../ai/units/operation-demand.md#task-1u-current-operation-bridge) for indirect reserve recruitment, which remains subject to role/health/path suitability. Selected role, stored queue role, created role and recruited slot are separate observable states.

## Task 1U-B unit application revalidation

src-20261029-vp-548 / audit-20261029-task1u-b. CvCity.cpp pushOrder29312, popOrder29455, produce29855, CreateUnit30125 remain materially unchanged in the task paths. Prior queue/creation findings revalidated. Header1600 default produce role=NO_UNITAI; popOrder extracts stored role but calls produce(unit) without forwarding it. Created unit's init/default role or explore reassignment may therefore differ from queue/requested role. The city promise accessor IsBuildingUnitForOperation (header1645) only checks token validity; no gameplay caller found, so it supplies no hidden normal-queue linkage.

Queue application clears when bPop before checking native canTrain; void return/logged choice is not acceptance. Making counts update on valid enqueue/pop; actual train completion creates unit before pop's promise uncommit. Investment is not creation. Stored city promise token has no serialization occurrence; operation lists serialize independently. Successful direct-purchase exact assignment and later reserve recruitment are separate paths; see [operation lifecycle](../../ai/units/operation-demand.md#task-1u-b-lifecycle-closure-on-vp-548). No new logging or behavior change. Null creation handling and tail pUnit dereference remain a source risk, not a reproduced runtime defect.
