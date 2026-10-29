# Task 1U: unit demand through city training

**Original Task 0 revalidation boundary; the partial Task 1U pass is recorded below.** Begin with the maintained [unit production](production.md), [operation demand](operation-demand.md) and [role-strategy/profile](role-strategies.md) references and their `legacy-vp526` provenance. Select/register the actual target in [source snapshots](../source-snapshots.md) before making modern verification claims.

The first vertical slice asks: **Why does the AI decide that it wants a particular unit, and why does a city ultimately choose/train that unit?** It does not mean "all unit AI." Use the [decision template](../templates/decision.md), [term schema](../score-provenance-schema.md), [flavor registry](../flavor-signal-registry.md) and [logging matrix](../logging-coverage.md), with the planning IDs in the [decision registry](../decision-registry.md).

## Included questions

1. **Demand producers.** Trace strategic, military, economic and relevant diplomacy/strategy pressure that is actually consumed by unit demand or production: strategy conditions, recommended counts/composition, resource/supply/economic constraints, operation creation/continuation pressure and skipped-demand state where applicable. For each consumed manager output, trace its calculation, inputs and update timing. Record independent upstream decision owners where encountered; do not silently absorb all war planning or every manager method.
2. **Operation/formation bridge.** Trace formation/role data, required/open slots, reserve recruitment insofar as it determines unmet demand, primary/secondary role fallback, pending/committed demand, city/muster/reachability filters and army-fill interfaces that actually feed production. Establish demand creation, exposure, commitment, cancellation and fulfillment; historical ambiguity about normal queue commitment is a question, not a confirmed defect.
3. **Candidates and legality.** Map generic and specially requested unit candidates, enumeration/order, duplicates by role/context, city legality and automation filters, fallback weights and helper recommendations as applicable. Include all unit types encountered as training candidates, including civilian, naval and air units; their subsequent behavior is a separate audit. Trace relevant native gate inputs/contracts instead of treating a getter name as complete provenance.
4. **Flavor weights.** Trace object rows, loading/caches, personality/player/city recipient layers, strategy/specialization deltas and parallel grand-strategy/direct reads actually consumed. Establish which getter/layer each term uses and when cached weights rebuild. Shared infrastructure ownership is limited to these consumer paths; it is not a complete cross-system flavor audit.
5. **Direct scoring and transforms.** Decompose every applicable non-flavor gate, bonus, penalty, overwrite, multiplier, Define, constant, derived metric and direct trait/policy/belief/religion input. Investigate historical `CvUnitProductionAI` recommendation, cached weight and sanity paths, their parameter variants, root/duration transforms and returned skip codes without presuming they still work as described.
6. **City-production integration and selection.** Map triggers/interrupts, candidate injection, shared competition, sanity/duration stages, inertia/overrides, threshold, ordering, weighted/random behavior, empty/all-rejected fallbacks and no-choice outcomes. Historical leads include `CvCityAI` and `CvCityStrategyAI`; do not mistake a unit recommendation helper for the final queue selector.
7. **Application and feedback.** Trace selected-unit queue application, parameters, legality recheck/failure, rush/continuation as applicable, making counts and relevant side effects. Follow training completion far enough to establish unit creation and demand/army commitment fulfillment or release, including cancellation/reset/persistence where they affect later demand. Stop before independently auditing created units' missions or tactical actions.
8. **Observability.** Assess every reconstruction dimension for the demand-to-training chain, including terms, input layers, rejected candidates, selection controls, applied outcome and state feedback. Record missing information without adding logging or launching a game unless a later task explicitly authorizes runtime work.

These questions establish revalidation and completeness obligations for the existing maintained reference, not a requirement to rediscover it from scratch. Inspect the relevant old baseline and dependency surface, re-establish precise source identity where the old commit is unknown, and fill the exhaustive score schema while updating canonical owners. Register newly discovered decisions while retaining stable IDs/redirects.

## Coupled paths and stopping rules

**Mixed purchase/training operation paths belong at the boundary.** If an operation-demand city check can buy/invest instead of queueing a unit, include the branch predicates, shared scoring parameter differences, queue-entry/suppression conditions and demand-state transitions needed to explain training. Index the purchase handoff and its success/failure contract. Detailed purchasing policy, affordability allocation, purchase-only scoring and full purchase side effects remain the deferred purchase area, unless current source proves a narrowly specified part indispensable to this chain. Any expansion must name that part and its owner in the registry.

**Shared city competition must be visible.** Include non-unit candidate identities, entry points, competing final weights/output contracts and common transforms/selector behavior needed to explain why a unit wins or loses. Detailed building/wonder/project/process valuation is adjacent work, not silently part of unit scoring. If those outputs' contributing provenance is untraced, record the dependencies and keep the overall city-comparison provenance partial. A unit-only calculation may be complete for its stated sub-scope; it cannot certify every input to the shared city decision or full runtime reconstruction.

**Producer contracts are real dependencies.** Follow the calculations producing consumed metrics/state, with source identity, scale, conditions and lifetime. Where an independent strategic decision or other untraced owner remains upstream, index it and state the completeness limit. A label such as "military pressure" cannot terminate the term graph. Excluding unrelated outputs does not excuse omitting an input to the included calculation.

## Adjacent area index

All entries below remain `AREA` buckets deferred from Task 1U. Whole-area exhaustive coverage is `UNMAPPED` and modern-target source verification is `UNCHECKED`; this does not erase partial older-baseline knowledge now held by canonical references. This section owns the slice deferrals while the registry indexes existing knowledge and later discovered bounded decisions.

| Area ID / intended domain | Deferred work | Task 1U interface, if encountered |
|---|---|---|
| `units.purchasing` / acquisition | Emergency/normal/hurry/gold/faith purchases and acquisition-only policy | Mixed operation purchase/training guards and demand-state handoff above; broader purchase behavior deferred |
| `units.upgrades` / acquisition | Upgrade candidate, funding, timing and application choices | Existing unit/composition state consumed by production only |
| `units.promotions` / unit development | Promotion selection and valuation | Direct already-held state only if consumed by demand/scoring |
| `units.tactical-actions` / tactical | Movement, attacks and tactical action selection | Consumed danger/tactical state or purchase-request handoff only |
| `units.homeland` / homeland | Existing unit recruitment, assignment and movement | Any proven unit-demand producer/consumer interface only |
| `units.worker-improvements` / improvements | Worker task, tile/improvement and route decisions | Worker demand/count/production inputs only |
| `units.settler-destinations` / expansion | Settlement-site and destination choice/mission execution | Settler demand and training inputs; upstream site output is a named dependency |
| `units.great-person-actions` / great people | Action, mission and consumption choices | Production/demand interface only if actually present |
| `units.other-missions-roles` / unit roles | Other mission/role selection, operational target planning and movement | Formation role requirements and reserve suitability only as they determine production demand |
| `production.nonunit-valuation` / city production | Detailed building, wonder, project and process scoring | Candidate/final-score contracts and shared selector; untraced provenance remains explicit |

This index is deliberately not exhaustive. Discovering another adjacent path requires an explicit registry entry or addition here, not silent inclusion or omission. No historical modmod/UPAI activation/default is presumed upstream.

## Task 1U reporting boundary

Report evidence and coverage independently for demand, unit candidate scoring, shared selection/application and unit-related flavor infrastructure. Preserve unknowns and deferred dependencies. The permitted conclusion is a scoped demand-to-training audit with stated coverage; **"unit AI complete" is not justified by this slice**. Do not claim full city-choice score provenance while non-unit or independent upstream inputs remain unresolved.

Start from [unit production](production.md), [operation demand](operation-demand.md), [flavors](../infrastructure/flavors.md), [city selection](../production/city-selection.md) and [operation/army state](../../architecture/operations-armies.md). Reuse the maintained understanding, revalidate changed or uncertain paths and preserve unresolved findings until evidence resolves them. Task 1U has not begun during migration.

## Task 1U current pass coverage

Pass `audit-20261029-task1u-01` has begun; the opening migration-only statement describes the prior reference, not this added pass. Selected sources are registered in [snapshots](../source-snapshots.md#task-1u-selected-source-pass). **Task 1U remains incomplete.** Base VP current and recovered UPAI are distinct; no source/data/runtime/config changes, stage or commit authorized or performed.

| Axis | Current bounded result / remaining work |
|---|---|
| Source identity | Clean VP c285a18 selected, separate UPAI file hashes and separate reference DB hash registered; exact UPAI intended VP revision unknown |
| Incremental revalidation | Unit scorer release diff/current bodies inspected; city/military changed, therefore historical whole-document claims not promoted |
| Direct unit scorer | All direct sanity blocks inventoried with ordered formulas/branches; transitive getter/loader/DB origins unresolved |
| Flavor | Exact city latest/cache versus GS/direct personality consumers mapped; full producer initialization/delta timing still partial |
| Operations | Normal shared queue lacks commit; direct short-build commits; purchase callback differs; ordinary finish/recruit/stage chain unresolved |
| Shared selector | Unit wrapper/sanity/duration/fallback/continuation/cutoff/draw/application mapped; non-unit and process-scoring dependencies partial |
| Configuration | Bounded matrix covers directly inspected gates; transitive legality/strategy/queue compile/runtime gates not exhaustive |
| UPAI | Data-only upstream-gate enablement and role/strategy/flavor changes identified; manifest mismatch, fixed-ID compatibility, full formation delta/effective DB unresolved |
| Logging | Source capability matrix and provisional Task 2U fields; no implementation; lifecycle coverage incomplete |

Completeness search: current C++ RecommendUnit/CheckUnitBuildSanity callers indexed in production owner; additional advisor/policy/religion/economic/military/process consumers discovered. Direct scorer MOD search, military gate/role helpers, operation token call sites, UPAI source files inspected. A broader cross-source inventory search was then performed (details below); relevance/call-graph closure and complete new-equivalent-symbol coverage remain **not yet completed**. Relevant paths remain unmapped; no lifecycle/score/logging axis is certified COMPLETE. Runtime NOT_TESTED; DLL compilation NOT_TESTED; no gameplay launched.

### Broad inventory and validation checkpoint

Current base search across DLL, DLLUtil, CP and VP C++/headers/SQL/XML for UnitProductionAI, UNITAI_, Unit_Flavors, AI_UNIT_PRODUCTION, RecommendUnit, CheckUnitBuildSanity, CITY_BUILDABLE_UNIT, formation-slot table and commitment token matched **70 files**. The same package search matched five UPAI SQL/XML files (NewStrategies, NewUnitFlavors, Triggers, Formations, NewUnitAITypes); leader/tech flavor files were separately located through the manifest. This is discovery coverage, not proof every hit is a relevant decision or every renamed equivalent was found.

Unclosed hits include CvBarbarians direct acquisitions, CvMinorCivAI unit gifts, CvTradeClasses route supply/priority, CvTraitClasses effects, CvUnit/CvAStar native suitability/path/creation, CvTacticalAI emergency requests, CvAdvisorRecommender presentation, and policy/religion/process consumers. Ordinary tactical/homeland/mission execution remains deferred, but acquisition or producer interfaces within these files cannot all be dismissed by filename. Their materiality and transitive gates remain explicit unresolved dependencies. Thus the current pass must not be handed to Task 2U as an exhaustive certified graph.

Checkpoint checks: all 27 added-section local Markdown links/anchors resolved; no added-section trailing whitespace. Source Git HEAD unchanged c285a18, porcelain status empty, submodule status empty; UPAI file hashes still match registered manifest and reference DB hash unchanged. Fifteen maintained Markdown owners updated (fourteen AI owners plus queue/creation). No runtime launched; runtime NOT_TESTED, compilation NOT_TESTED. Static/source inspection and read-only SQLite queries only. Repository-wide git diff checks do not apply to the non-Git maintained docs directory; the target source repository has no diff. Documentation table pipe escapes corrected. No stage or commit.

## Task 1U-B coverage and validation

Authoritative target **src-20261029-vp-548**, pass **audit-20261029-task1u-b**. Prior partial pass/c285a18 and recovered~5.2.6 evidence remain separate historical layers. This is incremental closure, not a replacement report. All43 direct scorer records revalidated unchanged; unit-side flavor routing, operation bridge, process proxy, shared selection/RNG, caller relevance and legality contracts extended. Independent strategic request/planner and native rules-accounting interfaces are explicitly bounded at named outputs, inputs, scale and update generation; unavailable effective mod stack/runtime is UNRESOLVED_EXTERNAL.

The following ledger records **all discovered paths from the scoped broad search**, including helpers subsequently followed. Search roots: task-selected5.4.8 DLL/DLLUtil/CP/VP cpp,h,sql,xml; patterns UnitProductionAI, UnitAI/UNITAI_, Unit_Flavors, AI_UNIT_PRODUCTION, RecommendUnit, CheckUnitBuildSanity, CITY_BUILDABLE_UNIT, ORDER_TRAIN, MultiUnitFormation_SlotEntries, canTrain and production logging; adaptive CustomMods/configuration searches follow actual consumers.98 primary matched files plus9 followed helpers; UPAI eight files all individually classified in role-strategies. A source-hit count is not global proof of all CivV AI coverage. Relevant snippets/callers have dispositions; ordinary unit actions remain excluded. Headers inherit consuming contract classification.

| Source path | Disposition | Boundary / rationale |
|---|---|---|
| `(1) Community Patch/Database Changes/AI/FlavorChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(1) Community Patch/Database Changes/NewCustomModOptions.xml` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(1) Community Patch/Database Changes/Tech/EraTableChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(1) Community Patch/Database Changes/Units/CoreNewUnitFormations.xml` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(1) Community Patch/Database Changes/Units/CoreUnitAIChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/AI/UnitFlavorSweeps.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/America.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Assyria.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Brazil.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Celts.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Greece.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Korea.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Morocco.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Ottoman.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Rome.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Spain.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Sweden.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Civilizations/Venice.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/CityStateUnitChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/NewUnitFormations.xml` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/NewUnits.xml` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/PolicyUnitChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/PreUnitChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/UnitAIChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `(2) Vox Populi/Database Changes/Units/UnitChanges.sql` | IN_SCOPE_INTERFACE | Data/option/formation/flavor/role or civilization effect rows; values enter identified entry/rules contracts |
| `CvGameCoreDLLUtil/include/CvEnumSerialization.h` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLLUtil/include/CvEnums.h` | IN_SCOPE_INTERFACE | Feature macro/default/cache plumbing; relevant gates have matrix dispositions |
| `CvGameCoreDLLUtil/include/CvWeightedVector.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CustomMods.cpp` | IN_SCOPE_INTERFACE | Feature macro/default/cache plumbing; relevant gates have matrix dispositions |
| `CvGameCoreDLL_Expansion2/CustomMods.h` | IN_SCOPE_INTERFACE | Feature macro/default/cache plumbing; relevant gates have matrix dispositions |
| `CvGameCoreDLL_Expansion2/CvAIOperation.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvAIOperation.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvAStar.cpp` | IN_SCOPE_INTERFACE | Existing-unit role/suitability/placement/path query output consumed; ordinary actions deferred |
| `CvGameCoreDLL_Expansion2/CvAdvisorRecommender.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/CvArmyAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvBarbarians.cpp` | ADJACENT_DEFERRED | Spawning/gifting/contracts alter existing unit ledger; not an additional shared scorer/recommender caller or ordinary ORDER_TRAIN selector |
| `CvGameCoreDLL_Expansion2/CvBarbarians.h` | ADJACENT_DEFERRED | Spawning/gifting/contracts alter existing unit ledger; not an additional shared scorer/recommender caller or ordinary ORDER_TRAIN selector |
| `CvGameCoreDLL_Expansion2/CvBeliefClasses.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp` | ADJACENT_DEFERRED | Nonunit valuation or cultural acquisition; existing state/final competitor output boundary retained |
| `CvGameCoreDLL_Expansion2/CvCity.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvCity.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvCityAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvContractClasses.cpp` | ADJACENT_DEFERRED | Spawning/gifting/contracts alter existing unit ledger; not an additional shared scorer/recommender caller or ordinary ORDER_TRAIN selector |
| `CvGameCoreDLL_Expansion2/CvCultureClasses.cpp` | ADJACENT_DEFERRED | Nonunit valuation or cultural acquisition; existing state/final competitor output boundary retained |
| `CvGameCoreDLL_Expansion2/CvDatabaseUtility.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvDiplomacyAI.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvDllDatabaseUtility.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvEconomicAI.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvFlavorManager.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvGame.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvGame.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvGameCoreDLLPCH.h` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/CvGameCoreUtils.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvGameCoreUtils.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvGlobals.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvGlobals.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp` | IN_SCOPE_INTERFACE | Emergency role request/reserve staging/explorer state; normal movement and sentry execution deferred |
| `CvGameCoreDLL_Expansion2/CvHomelandAI.h` | IN_SCOPE_INTERFACE | Emergency role request/reserve staging/explorer state; normal movement and sentry execution deferred |
| `CvGameCoreDLL_Expansion2/CvInfos.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvInfos.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvInfosSerializationHelper.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/CvInfosSerializationHelper.h` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/CvMap.cpp` | ADJACENT_DEFERRED | Unit action/mission/map setup or resulting state, no additional scorer/recommender caller |
| `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvMilitaryAI.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvMinorCivAI.cpp` | ADJACENT_DEFERRED | Spawning/gifting/contracts alter existing unit ledger; not an additional shared scorer/recommender caller or ordinary ORDER_TRAIN selector |
| `CvGameCoreDLL_Expansion2/CvPlayer.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvPlayer.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvPlot.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvPlot.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp` | IN_SCOPE_INTERFACE | All shared scorer/recommender calls separately classified; other policy/religion choices deferred |
| `CvGameCoreDLL_Expansion2/CvProcessProductionAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvRandom.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` | IN_SCOPE_INTERFACE | All shared scorer/recommender calls separately classified; other policy/religion choices deferred |
| `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp` | IN_SCOPE_INTERFACE | Emergency role request/reserve staging/explorer state; normal movement and sentry execution deferred |
| `CvGameCoreDLL_Expansion2/CvTacticalAnalysisMap.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTacticalAnalysisMap.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTeam.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTechClasses.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTradeClasses.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTraitClasses.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTreasury.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvTreasury.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvUnit.cpp` | IN_SCOPE_INTERFACE | Existing-unit role/suitability/placement/path query output consumed; ordinary actions deferred |
| `CvGameCoreDLL_Expansion2/CvUnit.h` | IN_SCOPE_INTERFACE | Existing-unit role/suitability/placement/path query output consumed; ordinary actions deferred |
| `CvGameCoreDLL_Expansion2/CvUnitClasses.cpp` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvUnitClasses.h` | IN_SCOPE_INTERFACE | Named native state/data/strategic producer contract; broader independent decision internals bounded |
| `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp` | ADJACENT_DEFERRED | Unit action/mission/map setup or resulting state, no additional scorer/recommender caller |
| `CvGameCoreDLL_Expansion2/CvUnitCycler.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/CvUnitMission.cpp` | ADJACENT_DEFERRED | Unit action/mission/map setup or resulting state, no additional scorer/recommender caller |
| `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvUnitProductionAI.h` | IN_SCOPE_CORE | Direct demand/candidate/scorer/selector/queue/recruitment or exact random/flavor helper |
| `CvGameCoreDLL_Expansion2/CvWorldBuilderMapLoader.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaCity.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaCity.h` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaEnums.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaGame.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaUnit.cpp` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaUnit.h` | NON_AI/PRESENTATION_ONLY | UI/export/serialization/initial scenario adapter; actions reuse native legality/application; no hidden native automatic scorer caller |

### Explicit external limits

- **UNRESOLVED_EXTERNAL effective profile:** no certified5.4.8 base or UPAI effective DB/build/content observed. Registered reference snapshot and temporary transformation results are scope-qualified. Missing evidence: loaded mod order, DB after successful activation, cached IDs/rows/flags. This blocks effective-package compatibility/runtime claims, not symbolic source/control logging. Task2U can safely record actual loaded values and package status.
- **UNRESOLVED_EXTERNAL package origin/loader:** recovered manifest names missing file. Source proves mismatch in artifact, not original distribution provenance or executable error/transaction policy. Task2U can proceed with explicit failed/missing-action/profile fields; it must not assume UPAI successfully activated.
- **Independent decision interfaces:** attack target/request, diplomacy pursuit/progress, settlement ranking, ranked trade routes and tactical analysis provide time-indexed outputs. Their exact unit-side contracts are documented; internal optimizer provenance remains owned outside this vertical slice. Logging can snapshot outputs/IDs/generation. Whole-city provenance remains PARTIAL because nonunit scores intentionally stop at final competitor output, except included unit reward proxy.

Completion status is recorded after final verification; historical 'Task1U incomplete' paragraphs above describe prior pass only and do not substitute for that decision. No source/config/data/runtime modifications authorized or performed; temporary fixtures only.

### Final closure decision

**TASK_1U_COMPLETE**, bounded to the unit demand-to-training source interfaces and participating configuration/acquisition feedback. All43 ordered direct records retained/revalidated;107 discovered/followed source paths classified; all current scorer/recommender caller families classified; process unit proxy, normal/hurry selection and exact CvSeeder mapped; known operation token/queue/recruitment contradictions documented. The current native/strategic producer contracts specify their outputs, inputs, source, type/scale, lifecycle and consumers. No immediate scorer/selector/lifecycle gap is knowingly left open. Full independent planner/native rules internals and nonunit city-score provenance remain explicitly outside the bounded source closure, with registry axes qualified accordingly.

UNRESOLVED_EXTERNAL limits above remain real. They prevent asserting effective5.4.8+UPAI compatibility, original-package authorship of the mismatch, actual listener/loaded values or runtime reproduction; they do not prevent Task2U from logging the bounded chain safely. Task2U's conceptual logging contract is finalized; no instrumentation or implementation design performed.

Validation record: target Git HEAD/subject7307aef.../5.4.8 Release verified directly, detached clean tree and source git diff --check clean; no alternate repository used as target. Comparison source read only. Original gameplay DB SHA-256 unchanged, UPAI eight files match registered hashes, no source/config/runtime edits. Temporary transformation and trigger savepoint test only; original reference files remain read-only. Maintained documentation preimages and review diffs prepared outside source repository; no stage/commit. Static Markdown/link/whitespace and final artifact checks are recorded after application. Compilation NOT_TESTED; runtime NOT_TESTED; no game launched.

Final applied checks: all205 local links/anchors across the15 touched owners resolved before the final producer-registry link (which resolves to the existing producer anchor);43 unique direct term anchors preserved. Added-line whitespace/table checks passed; source git diff --check0, HEAD unchanged7307aef..., porcelain empty. Eight UPAI hashes and gameplay DB SHA-256429d65... unchanged. Temporary fixtures/scripts are not distributed; none are runtime evidence or durable dependencies. No staging/commit or compiler/runtime execution.
