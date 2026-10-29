# Building valuation and sanity

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../source-snapshots.md#recovered-baseline>).

## Decision or contract question

How is a building or wonder valued as a city-production candidate?

This owns the recovered building-specific valuation/sanity contract, including shared policy/religion/purchase estimators. Detailed exhaustive field-level scoring remains incomplete. The common selector and application path have separate owners.

## Reference synthesis

Building valuation is a shared estimator used beyond ordinary construction. The recovered notes identify cached object/latest-flavor weights, root conversion, direct local/strategic usefulness, wonder-city/race rules and context flags such as ignoring situational checks. Shared policy, religion and purchase consumers make the parameter contract important.

This is a partial building-scoring reference. The migration has not supplied the missing field-by-field `CvBuildingEntry` provenance or established current effective DB values. Shared city selection and shared queue application belong to their linked owners.

Canonical related owners: [City production selection](<../production/city-selection.md>), [Wonder desire and city assignment](<../wonders/selection.md>), [Flavor production, propagation and consumption](<../infrastructure/flavors.md>), [Production queues and creation](<../../mechanics/production/queue-and-creation.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

Decision role: `CvBuildingProductionAI` is a building/wonder weight and sanity specialist. It does not choose the final queue by itself in the normal path; `CvCityStrategyAI::ChooseProduction` uses its `GetWeight` and `CheckBuildingBuildSanity` outputs while competing buildings against units, projects, and processes.

## Scoring and inputs

Scoring shape: `CheckBuildingBuildSanity` rejects impossible or useless cases with negative `ProductionSkipReason` values, converts raw flavor weight with `sqrti(10 * iValue)`, applies hard wonder-city checks, desired-wonder bonuses, and many local/strategic bonuses, and returns a positive score only when enough bonus value remains.

## Data and configuration

Building data path is `Buildings` / `BuildingClasses` / `Building_Flavors` / prerequisite and effect fields -> `CvBuildingEntry::CacheResults` -> `CvBuildingProductionAI::AddFlavorWeights` for final city production and `CvWonderProductionAI::AddFlavorWeights` for desired wonders -> `CvCity::canConstruct`, `CvPlayer::SetBestWonderCities`, and `CvBuildingProductionAI::CheckBuildingBuildSanity` -> `CvCity::pushOrder(ORDER_CONSTRUCT, ...)` and completion through `CvCity::CreateBuilding` / `CvCityBuildings::SetNumRealBuilding`.

Desired-wonder selection is a separate player-level cache/choice: `CvWonderProductionAI::FlavorUpdate`, `ChooseWonder`, and `CvCitySpecializationAI::DoTurn` select/track desired wonders before final city production competes buildables in `CvCityStrategyAI::ChooseProduction`.

## Observability and validation obligations

Logging: `CvBuildingProductionAI::LogPossibleBuilds` can log building weights. Normal production logs are usually through `CvCityStrategyAI::LogPossibleBuilds`, `LogInvalidItem`, and `LogCityProduction`; final city production labels limited buildings as `"Wonder"` when max global or max player instances are set.

## Version differences, risks and unresolved questions

Patch-risk notes: RISK: `CheckBuildingBuildSanity` is shared by normal production, desired-wonder choice, best-wonder-city scoring in `CvPlayer::SetBestWonderCities`, policy branch valuation through `CvPolicyAI::WeighBranch`, and other helper valuations. Changes are likely to affect wonders and ordinary buildings, and can shift the city buildable mix away from units/projects/processes. `CvBuildingProductionAI::Serialize` stores `m_BuildingAIWeights`.

## Source and dependency lookup

Generic building flavor weights come from `CvBuildingProductionAI`:

- `CvCityStrategyAI::FlavorUpdate()` resets `CvBuildingProductionAI`, `CvUnitProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`, then broadcasts each latest city strategy flavor to `CvBuildingProductionAI::AddFlavorWeights(...)`.

- `CvBuildingProductionAI::AddFlavorWeights(...)` loops every building and increases `m_BuildingAIWeights` by `CvBuildingEntry::GetFlavorValue(eFlavor) * iWeight`.

- `CvCityStrategyAI::ChooseProduction(...)` uses `CvBuildingProductionAI::GetWeight(eLoopBuilding)` as the initial weight for any constructible building, including wonders.

`CvBuildingProductionAI::CheckBuildingBuildSanity(...)` is the main shared value/sanity function:

- Rejects impossible/null/low-value inputs and cities in danger of falling.

- Converts raw flavor weight to a smaller range with `sqrti(10 * iValue)`.

- Blocks world/team/national/limited wonders in non-capital cities with population `<= 6`.

- For national/world wonders, enforces best-wonder-city unless `bNoBestWonderCityCheck` is true: best city gets `+500`, non-best city returns `SR_STRATEGY`.

- Adds a large bonus if `CvCitySpecializationAI::GetNextWonderDesired() == eBuilding`: up to `min(1000, population * 50)` for national wonders and `min(1500, population * 100)` otherwise.

- For world wonders when not free and not ignoring situational checks, adds player/wonder production modifiers and city defense-per-wonder, then subtracts for met rival major civs currently constructing the same building class.

- Adds quest, geography, resource, happiness, defense, military, base-building, grand-strategy, policy, yield, religion, maintenance, war, era, prerequisite-chain, and unique-building terms.

- Returns `max(1, iValue + iBonus)` only if `iBonus >= 1`; otherwise returns a negative sanity result such as `SR_UNKNOWN`.

`CvBuildingProductionAI` is declared in `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.h` and implemented in `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`. It is owned by `CvCityStrategyAI`.

Inputs consumed: `CvCityStrategyAI::FlavorUpdate` broadcasts latest city flavors into `CvBuildingProductionAI::AddFlavorWeights`. `AddFlavorWeights` multiplies `CvBuildingEntry::GetFlavorValue(eFlavor)` by the city flavor value and stores `m_BuildingAIWeights`. `CheckBuildingBuildSanity` consumes `CvBuildingEntry`, city plot stats, existing building counts, city danger/siege/population/capital/coastal/border state, best-wonder-city flags, desired wonder from `CvCitySpecializationAI`, rival construction, policies, beliefs/religion, traits, resources, yields, maintenance, war state, and grand-strategy value helpers.

Selection mechanism: For normal city production, `CvCityStrategyAI::ChooseProduction` loops constructible buildings via `CvCity::canConstruct`, creates `CITY_BUILDABLE_BUILDING` precheck entries with `CvBuildingProductionAI::GetWeight`, calls `CheckBuildingBuildSanity`, reweights by duration, and then chooses among all buildable types through `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold` with seed `0x0e36d18b` mixed with city ID.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
