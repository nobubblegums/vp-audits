# Wonder desire and city assignment

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../source-snapshots.md#recovered-baseline>).

## Decision or contract question

Which wonder is desired, where is it assigned and when does it actually enter a queue?

Desired-wonder scoring, best-city assignment and final city competition are separate decisions. Object-specific race pressure, local feasibility and current-production rules cannot be replaced by one global wonder flavor.

## Reference synthesis and qualification

Desired-wonder selection, best-city assignment and normal city queue selection have separate owners and triggers. Desired-wonder weights use a recipient flavor cache, estimated duration and situational checks; final queue competition also sees local building sanity, production feasibility and opportunity cost. Race pressure appears through local object-specific paths, not only FLAVOR_WONDER.

The recovered improvement map classified `FindBestWonderCity` returning a null pointer as a high-confidence issue; its exact target/source state is not recoverable here. Team/limited wonder assignment and caller intent remain unresolved. Keep the original finding and the missing evidence together rather than silently declaring all wonder-city logic defective or correct.

Canonical related owners: [Building valuation and sanity](<../buildings/valuation.md>), [City production selection](<../production/city-selection.md>), [Strategic pressure and manager outputs](<../strategies/managers.md>), [Shared weighted selection and seeded draws](<../infrastructure/weighted-selection.md>), [Production queues and creation](<../../mechanics/production/queue-and-creation.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- City initialization calls `CvCity::AI_chooseProduction(false, false)` for non-human city-production controllers in `CvCity::init(...)` after `AI_init()` (`CvGameCoreDLL_Expansion2/CvCity.cpp`).

- Normal city production choice is `CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)` (`CvGameCoreDLL_Expansion2/CvCityAI.cpp`).

- If a city is already producing a spaceship part, `CvCityAI::AI_chooseProduction(...)` returns immediately.

- If a city is already producing a building, `CvCityAI::AI_chooseProduction(...)` checks whether it is a wonder using `CvWonderProductionAI::IsWonder(...)`. It preserves current wonder production unless `bInterruptWonders` is true, and preserves other building production unless `bInterruptBuildings` is true.

- If no direct forced wonder start happens, `CvCityAI::AI_chooseProduction(...)` delegates to `CvCityStrategyAI::ChooseProduction(NO_BUILDING, NO_UNIT, bInterruptBuildings, bInterruptWonders)`.

City specialization is the main player-level wonder update layer:

- `CvCitySpecializationAI::DoTurn()` skips humans using AI city production, minor civs, and turns before `AI_CITY_SPECIALIZATION_EARLIEST_TURN`.

- If `m_bChooseNewWonder` is true, `CvCitySpecializationAI::DoTurn()` calls `m_pPlayer->GetWonderProductionAI()->ChooseWonder(m_iNextWonderWeight)` and stores the result in `m_eNextWonderDesired`.

- `CvCitySpecializationAI::SetSpecializationsDirty(CitySpecializationUpdateType eUpdateType)` sets `m_bChooseNewWonder` for `SPECIALIZATION_UPDATE_WONDER_BUILT_BY_RIVAL`, `SPECIALIZATION_UPDATE_CITY_FOUNDED`, and `SPECIALIZATION_UPDATE_WONDER_BUILT_BY_US`.

- Tech unlocks can mark specialization dirty through `CvTeamTechs::SetHasTech(...)`, which calls `CvCitySpecializationAI::SetSpecializationsDirty(SPECIALIZATION_UPDATE_RESEARCH_COMPLETE)`.

- World-wonder tech unlock/built-by-rival handling also appears in `CvTeam::processTech(...)`, which calls `SetSpecializationsDirty(SPECIALIZATION_UPDATE_WONDER_BUILT_BY_RIVAL)`.

- New grand strategy, war state, city founding, city capture, and siege updates also affect specialization/interrupt flags through `CvCitySpecializationAI::SetSpecializationsDirty(...)`, `CvGrandStrategyAI`, `CvDiplomacyAI`, `CvTeam`, and `CvPlayer`.

INFERRED: For production decisions, the key ordering fact is that `CvEconomicAI::DoTurn`, `CvMilitaryAI::DoTurn`, and `CvCitySpecializationAI::DoTurn` run before city `CvCity::doTurn` in `CvPlayer::doTurnPostDiplomacy`, so city production can see same-turn player-level strategy state, operation demand, and desired-wonder state.

Decision role: `CvWonderProductionAI::ChooseWonder(int& iWonderWeight)` chooses the player-level desired wonder used by `CvCitySpecializationAI`, not the final city queue item. Final construction still goes through `CvCityStrategyAI::ChooseProduction` and `CvCity::pushOrder(ORDER_CONSTRUCT, ...)`.

- AI wonder choice has three separate layers, not one final "wonder AI" selector. The desired-wonder layer selects `CvCitySpecializationAI::m_eNextWonderDesired` through `CvWonderProductionAI::ChooseWonder(int&)`. The best-wonder-city assignment layer sets per-city `CvCity::m_abIsBestForWonder` flags through `CvPlayer::SetBestWonderCities()`. The final city queue layer treats wonders as `CITY_BUILDABLE_BUILDING` candidates inside `CvCityStrategyAI::ChooseProduction(...)` and queues a selected wonder with `CvCity::pushOrder(ORDER_CONSTRUCT, ...)`.

- desired-wonder selection is owned by `CvWonderProductionAI` in `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp` and triggered/stored by `CvCitySpecializationAI` in `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`. best-city flags are owned by `CvPlayer::SetBestWonderCities()` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp` and stored by `CvCity::SetBestForWonder(BuildingClassTypes, bool)` / `CvCity::IsBestForWonder(BuildingClassTypes)` in `CvGameCoreDLL_Expansion2/CvCity.cpp`. final queue selection is owned by `CvCityAI::AI_chooseProduction(...)` in `CvGameCoreDLL_Expansion2/CvCityAI.cpp` and `CvCityStrategyAI::ChooseProduction(...)` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`.

- `CvPlayer::doTurn()` calls `CvPlayer::SetBestWonderCities()` before `CvPlayer::doTurnPostDiplomacy()`. `CvPlayer::doTurnPostDiplomacy()` later calls `CvCitySpecializationAI::DoTurn()` before the city loop. `CvCitySpecializationAI::DoTurn()` calls `CvWonderProductionAI::ChooseWonder(...)` only when `m_bChooseNewWonder` is true; `CvCitySpecializationAI::SetSpecializationsDirty(...)` sets that flag for `SPECIALIZATION_UPDATE_WONDER_BUILT_BY_RIVAL`, `SPECIALIZATION_UPDATE_CITY_FOUNDED`, and `SPECIALIZATION_UPDATE_WONDER_BUILT_BY_US`. final normal production selection is reached from `CvCity::doProduction(bool)` when the city has no production, is producing a process, or `CvCityAI::AI_isChooseProductionDirty()` is true.

## Candidates and gates

Desired-wonder candidate generation happens in `CvWonderProductionAI::ChooseWonder(int& iWonderWeight)` (`CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`):

- Iterates each player city.

- Estimates city production as `pLoopCity->getRawProductionPerTurnTimes100() / 100`, clamped to at least 1.

- Iterates every building in `GC.GetGameBuildings()->GetNumBuildings()`.

- Skips null `CvBuildingEntry`.

- Requires `pLoopCity->IsBestForWonder(pkBuildingInfo->GetBuildingClassType())`.

- Computes `iTurnsRequired` from `CvBuildingEntry::GetProductionCost() / estimatedProduction`.

- Gives a 25x base wonder-weight multiplier if the city already has stored building production for that wonder via `CvCityBuildings::GetBuildingProduction(eBuilding)`.

- Reweights by duration with `CityStrategyAIHelpers::ReweightByTurnsLeft(...)`.

- Runs city building sanity with `CvBuildingProductionAI::CheckBuildingBuildSanity(eBuilding, iWeight, false, false, true)`.

- Adds candidates with positive final weight to `m_Buildables`.

Final city-production candidate generation happens in `CvCityStrategyAI::ChooseProduction(...)` (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`):

- Iterates all buildings.

- Applies puppet and automated-human filters. Automated human cities skip world, team, national, and limited wonders.

- Requires `CvCity::canConstruct(eLoopBuilding, vTotalBuildingCount, bContinueCurrentBuilding)`.

- Adds constructible buildings as `CITY_BUILDABLE_BUILDING` with initial weight from `CvBuildingProductionAI::GetWeight(eLoopBuilding)`.

- Later passes building candidates through `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`; only positive results survive into the final `m_Buildables`.

Best-wonder-city flags are maintained by `CvPlayer::SetBestWonderCities()` (`CvGameCoreDLL_Expansion2/CvPlayer.cpp`):

- Called during `CvPlayer::doTurn(...)` after happiness/uprising/revolt updates.

- Iterates buildings and keeps only world and national wonders.

- Skips civilization-unique wonders the current civilization cannot build.

- Skips puppet/annexable cities through `CityStrategyAIHelpers::IsTestCityStrategy_IsPuppetAndAnnexable(...)`.

- Requires `CvCity::canConstruct(eBuilding, allBuildingCount)`.

- Scores each city with `CvBuildingProductionAI::CheckBuildingBuildSanity(eBuilding, 1000, true, false, true)`.

- Multiplies score by 10 if the city is already producing or has stored production for that wonder.

- Subtracts `50 * pLoopCity->getProductionTurnsLeft(eBuilding, 0)`.

- Sets exactly one `CvCity::SetBestForWonder(eClass, true)` per evaluated class and clears other cities with `SetBestForWonder(eClass, false)`.

- Reduces the chosen city's score by 25% for other world wonders to spread world-wonder races across cities.

- desired-wonder candidates in `CvWonderProductionAI::ChooseWonder(...)` iterate every player city and every `CvBuildingEntry` from `GC.GetGameBuildings()->GetNumBuildings()`, skip null entries, and require `CvCity::IsBestForWonder(pkBuildingInfo->GetBuildingClassType())`. Desired-wonder generation does not call `CvCity::canConstruct(...)` directly; it relies on best-city flags already computed by `CvPlayer::SetBestWonderCities()`. `CvWonderProductionAI::IsWonder(const CvBuildingEntry&)` includes `isWorldWonderClass`, `isTeamWonderClass`, and `isNationalWonderClass`.

## Scoring and inputs

`CvWonderProductionAI` scoring is flavor-first and duration-aware:

- `CvWonderProductionAI::FlavorUpdate()` loops all flavors and calls `AddFlavorWeights(...)`.

- `CvWonderProductionAI::AddFlavorWeights(FlavorTypes eFlavor, int iWeight)` only applies to entries where `IsWonder(kBuilding)` is true, increasing `m_WonderAIWeights` by `CvBuildingEntry::GetFlavorValue(eFlavor) * iWeight`.

- `CvWonderProductionAI::ChooseWonder(...)` starts from `m_WonderAIWeights.GetWeight(eBuilding)`.

- Already-started wonders get 25x before duration reweighting.

- `CityStrategyAIHelpers::ReweightByTurnsLeft(...)` penalizes long construction time.

- `CvBuildingProductionAI::CheckBuildingBuildSanity(..., bIgnoreSituational=true)` adds/removes city/building value without situational war/rival-construction penalties in this desired-wonder pass.

- The desired-wonder chooser stable-sorts and uses `CvWeightedVector<BuildingTypes>::ChooseAbovePercentThreshold(...)` with handicap `CityProductionChoiceCutoffThreshold` and seed `CvSeeder::fromRaw(0x97d85ba6).mix(m_pPlayer->GetID()).mix(m_pPlayer->GetNumWonders())`.

Direct forced wonder start in `CvCityAI::AI_chooseProduction(...)` is narrow:

- If `bInterruptWonders` is true, the city is not already building a wonder, `GetNextWonderDesired()` is not `NO_BUILDING`, the city is `IsBestForWonder(...)`, and the player has at least one `UNITAI_ENGINEER`, the city pushes `ORDER_CONSTRUCT` for the desired wonder immediately.

- Desired-wonder flavor vector: `CvWonderProductionAI::FlavorUpdate()`, `CvWonderProductionAI::AddFlavorWeights(...)`, and `CvBuildingEntry::GetFlavorValue(...)`.

- Final city-production building flavor vector: `CvCityStrategyAI::FlavorUpdate()` and `CvBuildingProductionAI::AddFlavorWeights(...)`.

- Wonder specialization weight: `CvCitySpecializationAI::WeightProductionSubtypes()` reads `FLAVOR_WONDER` through `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`, multiplies by `AI_CITY_SPECIALIZATION_PRODUCTION_WEIGHT_FLAVOR_WONDER`, and caps it by a fraction of `m_iNextWonderWeight`.

- `CityStrategyAIHelpers::GetBuildingGrandStrategyValue(CvCity*, BuildingTypes, PlayerTypes)` reads grand-strategy priorities from `CvGrandStrategyAI::GetGrandStrategyPriority(...)` for `AIGRANDSTRATEGY_CONQUEST`, `AIGRANDSTRATEGY_CULTURE`, `AIGRANDSTRATEGY_UNITED_NATIONS`, and `AIGRANDSTRATEGY_SPACESHIP`.

- It maps building attributes to conquest, culture/tourism, diplomacy, and science values.

- It boosts or discounts those values using `CvDiplomacyAI::IsGoingForCultureVictory()`, `IsCloseToCultureVictory()`, `IsGoingForDiploVictory()`, `IsCloseToDiploVictory()`, `IsGoingForSpaceshipVictory()`, `IsCloseToSpaceshipVictory()`, `IsGoingForWorldConquest()`, and `IsCloseToWorldConquest()`.

- `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` adds `GetBuildingGrandStrategyValue(...) * (currentEra + 1)`.

- `CvCitySpecializationAI::WeightProductionSubtypes()` also considers active grand strategy specialization boosts for production, including offense and spaceship flavor checks.

- Religion gates for specific world wonders in `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`: if the player has no owned religion, world wonders that grant a free Great Prophet are rejected if no religion can still be founded and the player lacks `CvPlayerTraits::IsAlwaysReligion()`. World wonders that grant missionaries/inquisitors are rejected for non-founders, and non-prophet religion effects such as missionary strength/spreads or reformation are considered useless without a religion.

- Religion value is added through `CityStrategyAIHelpers::GetBuildingReligionValue(CvCity*, BuildingTypes, PlayerTypes)`.

- Policy value is added through `CityStrategyAIHelpers::GetBuildingPolicyValue(CvCity*, BuildingTypes)`.

- `GetBuildingPolicyValue(...)` reads policy and religion synergies including `CvPlayerPolicies::GetBuildingClassProductionModifier(...)`, `GetBuildingClassHappinessModifier(...)`, `GetNumericModifier(...)`, state religion beliefs, Golden Age belief yields, Great Person belief yields, and spy-pressure beliefs.

- `CheckBuildingBuildSanity(...)` also uses `CvPlayerPolicies::GetNumPoliciesOwned()` for happiness-per-policy building effects.

- Rival construction affects only the situational world-wonder branch in `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`: for met alive major civs with `getBuildingClassMaking(...) > 0`, `iBonus` is reduced by `150` each, plus another `100` each if the player has only one city.

- War and threat affect generic building scoring: if not a courthouse and not ignoring situational checks, the function applies a war penalty based on `CvMilitaryAI::GetNumberCivsAtWarWith(false)`, low military size, `CvCity::getThreatValue()`, and `CvDiplomacyAI::GetWarState(eLoopPlayer)`.

- Defense value is strongly boosted for border/coastal cities and during war/siege, using `CvPlayerAI::IsAtWarAnyMajor()`, `CvCity::isBorderCity()`, `CvCity::isCoastal(12)`, and `CvCity::isUnderSiege()`.

- Plot-stealing building effects use `CvDiplomacyAI::GetCivApproach(...)` and `CvDiplomacyAI::IsBadTheftTarget(...)` to reward harming enemies and avoid angering friends or bad theft targets.

- Wonder specialization reduces wonder production priority by half when `MILITARYAISTRATEGY_EMPIRE_DEFENSE_CRITICAL` is active.

- Flavor weights ultimately come from current flavor values propagated through `CvFlavorRecipient`; city specialization and building/wonder production use those flavor values.

- `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` reads trait modifiers including `CvPlayerTraits::GetWonderProductionModifier()`, `GetNumFreeResourceOnWorldWonderCompletion(...)`, `IsAlwaysReligion()`, trade-route range bonuses, specialist/great-person/golden-age synergies, religious trait checks, and unique-building overrides via `CvCivilizationInfo::isCivilizationBuildingOverridden(...)`.

- `CityStrategyAIHelpers::GetBuildingGrandStrategyValue(...)` multiplies strategic interests based on leader trait categories such as `IsExpansionist()`, `IsWarmonger()`, `IsNerd()`, `IsDiplomat()`, `IsSmaller()`, `IsTourism()`, and `IsReligious()`.

- Best-wonder-city scoring uses city production turns, already-started production, puppet/annexable status, `CvCity::canConstruct(...)`, and `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`.

- Generic building sanity reads city population, capital status, danger/falling/siege status, city plot stats, local resources, trade priorities, coastal/border state, city focus yield, specialists, Great Works, local happiness causes, local defense, current production, and existing buildings.

- City production final reweighting uses `CvCityStrategyAI::ReweightByDuration(...)`, which calls `CityStrategyAIHelpers::ReweightByTurnsLeft(...)` for all final candidates.

- Hard feasibility is `CvCity::canConstruct(...)` in both `CvPlayer::SetBestWonderCities()` and `CvCityStrategyAI::ChooseProduction(...)`.

- Desired-wonder selection does not call `canConstruct(...)` directly; it relies on `IsBestForWonder(...)` flags computed from `SetBestWonderCities()` plus `CheckBuildingBuildSanity(...)`.

- Duration/opportunity cost appears in three places: `SetBestWonderCities()` subtracts `50 * turnsLeft`, `CvWonderProductionAI::ChooseWonder(...)` reweights by estimated turns, and final city production calls `CvCityStrategyAI::ReweightByDuration(...)`.

- Current-build inertia in `CvCityStrategyAI::ChooseProduction(...)` keeps the current build if it remains within half of the top candidate's weight; this applies to building candidates when `bInterruptBuildings` is false.

- `CvCityAI::AI_chooseProduction(...)` separately preserves current wonder production unless `bInterruptWonders` is true.

- desired-wonder scoring starts from `CvWonderProductionAI::m_WonderAIWeights`, adds started-production weight, duration reweighting, and `CvBuildingProductionAI::CheckBuildingBuildSanity`. Final production starts from `CvBuildingProductionAI::GetWeight`, then applies `CheckBuildingBuildSanity`, `CvCityStrategyAI::ReweightByDuration`, and current-production inertia.

- desired wonder consumes player latest flavors through `CvWonderProductionAI::FlavorUpdate` and `CvBuildingEntry::GetFlavorValue`; final production consumes city latest flavors through `CvCityStrategyAI::FlavorUpdate` and `CvBuildingProductionAI::AddFlavorWeights`; `FLAVOR_WONDER` also affects specialization weighting.

Scoring shape: Desired-wonder scoring starts with cached wonder flavor weight, multiplies already-started wonders by `25`, reweights by estimated turns through `CityStrategyAIHelpers::ReweightByTurnsLeft`, then applies building sanity with `bIgnoreSituational=true`.

- desired-wonder scoring starts from `CvWonderProductionAI::m_WonderAIWeights`, which `CvWonderProductionAI::FlavorUpdate()` and `AddFlavorWeights(FlavorTypes, int)` build by multiplying latest player-recipient flavor values by `CvBuildingEntry::GetFlavorValue(eFlavor)` for entries where `CvWonderProductionAI::IsWonder(...)` is true. `ChooseWonder(...)` estimates turns from building production cost and raw city production, multiplies weight by `25` if the city already has stored production for the wonder, reweights with `CityStrategyAIHelpers::ReweightByTurnsLeft(...)`, then calls `CvBuildingProductionAI::CheckBuildingBuildSanity(eBuilding, iWeight, false, false, true)`.

- final city-production scoring starts from `CvBuildingProductionAI::GetWeight(eBuilding)`, where `CvBuildingProductionAI::AddFlavorWeights(...)` multiplies latest city-recipient flavors by `CvBuildingEntry::GetFlavorValue(eFlavor)`. `CvCityStrategyAI::ChooseProduction(...)` then calls `CvBuildingProductionAI::CheckBuildingBuildSanity(eBuilding, precheckWeight, plotStats, allExistingBuildings)`, reweights all surviving buildables with `CvCityStrategyAI::ReweightByDuration(...)`, and lets wonder candidates compete against units, ordinary buildings, projects, and processes.

- wonder choice consumes player flavor recipient state, city flavor recipient state, `Building_Flavors`, `CvBuildingEntry` and `CvBuildingClassInfo` wonder/class fields, production cost, city production rate, city population/capital/puppet/current-production state, `CvCity::canConstruct(...)`, stored building production, player traits, religion state, policy/religion/building synergies, known rival construction, military/diplomacy/war state through `CheckBuildingBuildSanity(...)`, and handicap `CityProductionChoiceCutoffThreshold`.

- desired-wonder base weights use player latest flavors through `CvWonderProductionAI::FlavorUpdate()` because `CvPlayer::init` registers `m_pWonderProductionAI` with `CvFlavorManager`. final building/wonder base weights use city latest flavors through `CvCityStrategyAI::FlavorUpdate()` and `CvBuildingProductionAI::AddFlavorWeights(...)`. `CvBuildingEntry::CacheResults(...)` loads `Building_Flavors` in `CvGameCoreDLL_Expansion2/CvBuildingClasses.cpp`; VP rewrites building flavor data in `(2) Vox Populi/Database Changes/AI/BuildingFlavorSweeps.sql`, with CP additions in `(1) Community Patch/Database Changes/AI/FlavorChanges.sql`.

- important non-flavor controls include best-wonder-city flags, `CvCity::canConstruct(...)`, duration reweighting, started-production inertia, current-production preservation in `CvCityAI::AI_chooseProduction(...)`, current-production inertia in `CvCityStrategyAI::ChooseProduction(...)`, Great Engineer direct-start logic, population/capital constraints, danger-of-falling rejection, religion usefulness gates, known rival construction penalty, and war/threat/geography/resource/happiness/defense/policy/religion/trait bonuses in `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`.

## Selection and control

- `CvCityStrategyAI::ChooseProduction(...)` stable-sorts `m_BuildablesPrecheck`, sanity-checks each candidate into `m_Buildables`, reweights by duration, stable-sorts again, and logs the post-check list.

- If no candidates survive, it falls back to `m_BuildablesPrecheck`.

- If current production remains competitive, it continues that current build.

- Otherwise it picks a new candidate with `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold(...)`, using handicap `CityProductionChoiceCutoffThreshold` and seed `CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID())`.

- For a selected building, including a wonder, it calls `CvCity::pushOrder(ORDER_CONSTRUCT, eBuildingType, -1, false, true, false, bRush)`.

- The direct Great Engineer/interrupt desired-wonder path in `CvCityAI::AI_chooseProduction(...)` calls `pushOrder(ORDER_CONSTRUCT, eNextWonder, -1, false, false, false, false)`.

## Application and state

- final city queue uses `CvCity::pushOrder(ORDER_CONSTRUCT, eBuildingType, ...)`; completion path includes `CvCity::CreateBuilding` and `CvCityBuildings::SetNumRealBuilding`.

## Data and configuration

| Folder / data area | Relevant tables and rows | Current-source consumer |
|---|---|---|
| `(2) Vox Populi/Database Changes/City/Buildings/` | modifies `Buildings`, `BuildingClasses`, building prerequisites, capture behavior, national/world wonder constraints, policy requirements, free units, faith-purchase buildings, and many building effect tables. | `CvBuildingEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvBuildingClasses.cpp`), `CvCity::canConstruct`, `CvBuildingProductionAI::CheckBuildingBuildSanity`, `CvWonderProductionAI::ChooseWonder`, `CvPlayer::SetBestWonderCities`, and policy/religion helper valuation. |

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvLeaderHeadInfo::CacheResults` (`CvGameCoreDLL_Expansion2/CvInfos.cpp`) | `Leader_Flavors` | Base personality for `CvFlavorManager::Init`; downstream policy, tech, wonder, production, diplomacy, religion direct personality reads. |
| `CvBuildingEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvBuildingClasses.cpp`) | `Buildings`, `Building_Flavors`, building-class/effect/prereq/wonder fields | Building and wonder production, desired-wonder choice, best-wonder-city scoring, policy/religion valuation. |

- source classes and entries include `CvWonderProductionAI`, `CvCitySpecializationAI`, `CvPlayer`, `CvCity`, `CvCityAI`, `CvCityStrategyAI`, `CvBuildingProductionAI`, `CvBuildingEntry`, `CvBuildingClassInfo`, `CvBuildingXMLEntries`, `CvWeightedVector<BuildingTypes>`, `CvWeightedVector<CvCityBuildable>`, `CvFlavorRecipient`, `CvFlavorManager`, `CvHandicapInfo`, `CvPlayerTraits`, `CvPlayerPolicies`, and `CvGameReligions`. enum/order-sensitive types include database-backed `BuildingTypes`, `BuildingClassTypes`, `FlavorTypes`, `CityBuildableType`, `ProductionSkipReason`, and `OrderTypes` such as `ORDER_CONSTRUCT`. database-sensitive inputs include `Building_Flavors`, `Buildings`, building-class max-instance fields used by `isWorldWonderClass`, `isTeamWonderClass`, `isNationalWonderClass`, and `isLimitedWonderClass`, city-specialization rows with `CvCitySpecializationXMLEntry::IsWonder()`, and `HandicapInfos.CityProductionChoiceCutoffThreshold`.

## Observability and validation obligations

- Desired-wonder candidates: `CvWonderProductionAI::LogPossibleWonders()` logs `"Possible Wonder"` lines through `CvCitySpecializationAI::LogMsg(...)`.

- Desired-wonder change: `CvCitySpecializationAI::DoTurn()` logs `"Next desired wonder is ..."` when `m_eNextWonderDesired` changes to a valid building.

- Best city for each wonder: `CvPlayer::SetBestWonderCities()` logs `"<city> is best to construct <wonder>"` through `CvCitySpecializationAI::LogMsg(...)`.

- Specialization updates: `CvCitySpecializationAI::LogSpecializationUpdate(...)` logs reasons such as new grand strategy, war, city founded, wonder built by rival, and wonder built by us.

- Specialization assignment: `CvCitySpecializationAI::LogSpecializationAssignment(...)` logs assigned city specialization and marks wonder-city assignments.

- Production candidate lists: `CvCityStrategyAI::LogPossibleBuilds(...)` logs `PRE` and `POST` candidate weights and turns.

- Final city choice: `CvCityStrategyAI::LogCityProduction(...)` labels limited building classes as `"Wonder"` when max global or max player instances are set.

- Direct forced wonder start: `CvCityAI::AI_chooseProduction(...)` logs a city-specialization message `"WONDER - Started ..."`.

- `CvWonderProductionAI::LogPossibleWonders`, `CvCitySpecializationAI::LogMsg`, `CvPlayer::SetBestWonderCities` specialization logs, `CvCityStrategyAI::LogPossibleBuilds`, and `LogCityProduction`.

1. Runtime wonder-city test: enable AI logging and compare `CvPlayer::SetBestWonderCities()` log lines with `CvCitySpecializationAI::DoTurn()` desired-wonder logs and observed `m_iWonderCityID` for the same `BuildingTypes`.

Logging: `CvWonderProductionAI::LogPossibleWonders` logs possible desired wonders through `CvCitySpecializationAI::LogMsg`. `CvCitySpecializationAI::DoTurn` logs desired-wonder changes.

- `CvWonderProductionAI::LogPossibleWonders()` logs desired-wonder candidates through `CvCitySpecializationAI::LogMsg(...)`. `CvCitySpecializationAI::DoTurn()` logs `"Next desired wonder is ..."` when the stored desired wonder changes. `CvPlayer::SetBestWonderCities()` logs `"<city> is best to construct <wonder>"` through `CvCitySpecializationAI::LogMsg(...)`. `CvCityStrategyAI::LogPossibleBuilds(...)` logs `PRE` and `POST` city production candidate lists, and `CvCityStrategyAI::LogCityProduction(...)` logs selected buildings with max global/player instances as `"Wonder"`. the direct desired-wonder bypass in `CvCityAI::AI_chooseProduction(...)` logs both normal city production and a city-specialization `"WONDER - Started ..."` message.

## Version differences, risks and unresolved questions

INFERRED: `CvCitySpecializationAI::FindBestWonderCity()` appears to intend to return a city whose `IsBestForWonder(...)` flag matches `m_eNextWonderDesired`, but the current function returns `pBestCity` without assigning it. As written, it returns `NULL` even when a matching city is found. Verify intent before changing this, because `m_iWonderCityID` assignment and wonder specialization assignment may be affected.

Team wonders are included by `CvWonderProductionAI::IsWonder(...)`, but `CvPlayer::SetBestWonderCities()` only evaluates world and national wonders. This means team wonders may not receive `IsBestForWonder(...)` flags from that path. The final generic city-production path can still consider constructible team wonders as buildings if their generic building weight and sanity score survive.

Limited wonders are filtered as wonder-like for automated human cities in `CvCityStrategyAI::ChooseProduction(...)`, but `CvWonderProductionAI::IsWonder(...)` does not include limited wonders. Document future limited-wonder behavior against source before editing.

INFERRED: `CvWonderProductionAI::HaveCityToBuild(BuildingTypes eBuilding) const` is currently unused in the traced path. It may be legacy or reserved helper code.

- Savegame-sensitive state: `CvWonderProductionAI::Serialize(...)` serializes `m_WonderAIWeights`; `CvCitySpecializationAI::Serialize(...)` serializes `m_bChooseNewWonder`, `m_eNextWonderDesired`, `m_iWonderCityID`, `m_iNextWonderWeight`, and related flags; `CvCity` serializes `m_abIsBestForWonder` through `SYNC_ARCHIVE_VAR`.

- Multiplayer/determinism-sensitive randomness: desired-wonder selection uses seed `0x97d85ba6` mixed with player ID and number of wonders; final city production uses seed `0x0e36d18b` mixed with city ID.

- Enum/database assumptions: loops over `GC.GetGameBuildings()->GetNumBuildings()`, `GC.getNumBuildingInfos()`, `GC.getNumBuildingClassInfos()`, `FlavorTypes`, `BuildingTypes`, and `BuildingClassTypes` assume stable database-backed enum/index ordering.

- XML/schema-sensitive fields: building flavor values, building-class wonder limits, production costs, prerequisites, free units, religion flags, policy/trait modifiers, and city-specialization XML fields such as `IsWonder` feed scoring directly.

- Behavioral risk: changing `CheckBuildingBuildSanity(...)` affects all buildings, not only wonders, because it is shared by city production, best-wonder-city scoring, and desired-wonder scoring.

- Behavioral risk: changing `SetBestWonderCities()` affects both desired-wonder candidate generation and final city-production filtering for world/national wonders through `CvCity::IsBestForWonder(...)`.

- Behavioral risk: changing interrupt flags in `CvCitySpecializationAI::SetSpecializationsDirty(...)` can cause cities to abandon current wonders/buildings or fail to react when a wonder is built by another player.

`CvCityStrategyAI::ChooseProduction` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp` is a shared production hub for units, buildings, wonders, projects, and processes. It also handles special operation/army unit candidates and city hurry selection.

RISK: Production score changes can change not only which unit or building is picked, but whether a city builds a unit, building, project, process, or wonder at all.

`CvUnitProductionAI::CheckUnitBuildSanity` and `CvBuildingProductionAI::CheckBuildingBuildSanity` are shared scoring hubs. They are used by normal production plus policy, religion, purchase, desired-wonder, and best-wonder-city logic.

- Risk notes: RISK `CheckBuildingBuildSanity` is shared by all buildings, `SetBestWonderCities` affects desired-wonder and final production paths, desired/final city production randomness is sync-sensitive, and serialized state includes `CvWonderProductionAI`, `CvCitySpecializationAI`, and `CvCity::m_abIsBestForWonder`.

- RISK `CvCityStrategyAI::ChooseProduction(...)` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`: shared normal production hub for all `CityBuildableType` values (`CITY_BUILDABLE_UNIT`, `CITY_BUILDABLE_UNIT_FOR_OPERATION`, `CITY_BUILDABLE_UNIT_FOR_ARMY`, `CITY_BUILDABLE_BUILDING`, `CITY_BUILDABLE_PROJECT`, `CITY_BUILDABLE_PROCESS`). A patch to candidate generation, precheck weights, sanity acceptance, duration reweighting, current-production inertia, or seed/cutoff affects units, buildings, wonders, projects, and processes together.

- RISK `CvUnitProductionAI::CheckUnitBuildSanity(...)` and `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`: shared scoring hubs used by normal production and helper paths. They also feed purchases, policy/religion valuation, desired-wonder scoring, and best-wonder-city scoring.

- OPEN_QUESTION `CvCitySpecializationAI::FindBestWonderCity()` appears, per the existing wonder decision docs, to return `NULL` in a branch that seems intended to return a matching best wonder city. This page does not infer intent; verify before any behavior edit.

- OPEN_QUESTION project/process completion and maintenance side effects (`ORDER_CREATE`, `ORDER_MAINTAIN`) remain less mapped than unit/building/wonder completion.

- Classification: HIGH_CONFIDENCE_BUG

- Affected decision: Wonder choice; specifically desired-wonder city specialization after `CvWonderProductionAI::ChooseWonder(...)`.

- Source functions: `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`, `CvCitySpecializationAI::FindBestWonderCity()` initializes `CvCity* pBestCity = NULL`, loops cities, and returns `pBestCity` when `pLoopCity->IsBestForWonder(...)` is true without assigning `pLoopCity` to `pBestCity`. Caller context: `CvCitySpecializationAI::DoTurn()` uses `FindBestWonderCity()` when setting `m_iWonderCityID`. Best-city flags are produced in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::SetBestWonderCities()`.

- Why current behavior may be suboptimal: best-wonder flags can be set by `CvPlayer::SetBestWonderCities()`, but `FindBestWonderCity()` currently cannot return the matching city. This can leave `m_iWonderCityID` unset even when `m_eNextWonderDesired` has an eligible best city. INFERRED: desired-wonder specialization assignment and logs can lose the intended city target.

- Lowest-risk investigation: Add or inspect existing `CvCitySpecializationAI::LogMsg(...)` output around `"Next desired wonder is"` and any wonder-city assignment line for turns where `CvPlayer::SetBestWonderCities()` logs a best city for the same wonder.

- Smallest possible patch point: `CvCitySpecializationAI::FindBestWonderCity()` only: return `pLoopCity` or assign `pBestCity = pLoopCity` before return. Do not alter `CvPlayer::SetBestWonderCities()` or wonder scoring first.

- Runtime/logging test: Enable AI logging and run an AI with constructible wonders. Compare `CvPlayer::SetBestWonderCities()` log lines (`"<city> is best to construct <wonder>"`) with `CvCitySpecializationAI::DoTurn()` desired-wonder logs and serialized/observed `m_iWonderCityID`.

- Risk of double-counting or unintended downstream effects: RISK fixing lookup can make desired-wonder specialization pressure actually land on a city, which can alter final `CvCityStrategyAI::ChooseProduction(...)` competition against units, ordinary buildings, projects, and processes. Save-sensitive fields include `CvCitySpecializationAI::Serialize(...)` members `m_eNextWonderDesired`, `m_iWonderCityID`, and `m_iNextWonderWeight`.

- Risk of double-counting or unintended downstream effects: RISK army candidates already add `AI_CITYSTRATEGY_ARMY_UNIT_BASE_WEIGHT` plus `FLAVOR_OFFENSE * AI_CITYSTRATEGY_OPERATION_UNIT_FLAVOR_MULTIPLIER`. Allowing them through could increase generic military production and crowd out buildings/wonders/projects. It may also overlap with `CITY_BUILDABLE_UNIT_FOR_OPERATION`, `MILITARYAISTRATEGY_NEED_*`, and `CvUnitProductionAI::CheckUnitBuildSanity(...)` recommended-force bonuses.

- Affected decision: City production, unit production, building/wonder production, projects, processes, and hurry purchases.

- Risk of double-counting or unintended downstream effects: RISK logging additions should avoid recomputing scoring functions with side effects or random calls. Behavior changes based on misunderstood logs are risky because the final vector mixes units, buildings, wonders, projects, and processes.

- Classification: DESIGN_HYPOTHESIS_ONLY

- Affected decision: Desired wonder and final city wonder construction.

- Source functions: `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`, `CvWonderProductionAI::ChooseWonder(...)` uses desired-wonder flavor weights, started-production multiplier, duration reweighting, and `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`. `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`, `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` contains local rival-construction and desired-wonder logic. Final selection is `CvCityStrategyAI::ChooseProduction(...)`.

- Why current behavior may be suboptimal: DESIGN_HYPOTHESIS "race or abandon" pressure for a specific wonder is not a natural fit for broad `FLAVOR_WONDER`; it is object-specific and timing-sensitive. Current behavior may be correct, but the pressure is difficult to audit as a named component.

- Lowest-risk investigation: Log a named score component for known rival construction and started-production/duration terms for world wonders only.

- Smallest possible patch point: `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` logging around the rival-construction penalty, not `CvFlavorManager` or `Building_Flavors`.

- Runtime/logging test: In a save where a known rival is building the same world wonder, compare desired-wonder logs, best-wonder-city logs, `PRE`/`POST` city production logs, and final selection.

- Risk of double-counting or unintended downstream effects: RISK adding a broad dynamic `FLAVOR_WONDER` for race pressure would affect all wonders and ordinary building competition if building flavors overlap. A named score/log component is safer than a new flavor producer.

- OPEN_QUESTION: `docs-ai/atlas/decisions/wonder-choice.md` correctly flags team wonders and limited wonders, but the atlas still lacks a separate source pass proving how team wonders behave when `CvWonderProductionAI::IsWonder(...)` includes team wonders while `CvPlayer::SetBestWonderCities()` documents world/national assignment.

Patch-risk notes: RISK: `CvWonderProductionAI::Serialize` stores `m_WonderAIWeights`. `CvCity::IsBestForWonder` flags are set elsewhere by `CvPlayer::SetBestWonderCities`, so changes to best-city logic alter desired-wonder candidate generation. `BuildingTypes`, `BuildingClassTypes`, `Building_Flavors`, `CityProductionChoiceCutoffThreshold`, and seed inputs are sensitive. OPEN_QUESTION: `docs-ai/ai-wonders.md` notes `CvCitySpecializationAI::FindBestWonderCity` appears to return `NULL` without assigning the matching city; verify intent before changing behavior.

- Open questions: OPEN_QUESTION `CvCitySpecializationAI::FindBestWonderCity()` appears to intend to return the city whose `IsBestForWonder(...)` flag matches `m_eNextWonderDesired`, but audited source returns `pBestCity` without assigning it, so it returns `NULL` even when a matching city is found. Verify design intent before any behavior change because this affects `m_iWonderCityID` and wonder specialization assignment. OPEN_QUESTION team wonders are included by `CvWonderProductionAI::IsWonder(...)`, but `CvPlayer::SetBestWonderCities()` only assigns best-city flags for world and national wonders; team wonder behavior should be traced separately before changing that mismatch. OPEN_QUESTION limited wonders are blocked for automated humans and logged as wonder-like by max-instance checks, but they are not included by `CvWonderProductionAI::IsWonder(...)`.

- Risk notes: RISK desired-wonder and final queue selection are multiplayer/replay sensitive because they use seeded threshold-random selection with seeds `0x97d85ba6` and `0x0e36d18b`. RISK savegame-sensitive state includes `CvWonderProductionAI::Serialize(...)` for `m_WonderAIWeights`, `CvCitySpecializationAI::Serialize(...)` for `m_bChooseNewWonder`, `m_eNextWonderDesired`, `m_iWonderCityID`, and `m_iNextWonderWeight`, and `CvCity` serialization of `m_abIsBestForWonder` through `SYNC_ARCHIVE_VAR`. RISK `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` is shared by ordinary building production, final wonder production, desired-wonder scoring, and best-wonder-city scoring; changing it is not wonder-local. RISK database enum/order changes for buildings, building classes, and flavors affect cached vectors, saved state, and seeded candidate order.

## Source and dependency lookup

INFERRED:
- `CvWonderProductionAI::ChooseWonder(...)` appears to choose the player-level desired wonder for specialization, while `CvCityStrategyAI::ChooseProduction(...)` makes the final city build order. A focused pass should verify all handoff state through `CvCitySpecializationAI::m_eNextWonderDesired`, `m_iWonderCityID`, and city specialization assignment.

Unresolved questions for focused pass:
- Separate ordinary buildings, national wonders, team wonders, world wonders, and limited wonders in `CheckBuildingBuildSanity(...)`.
- Trace when `CvCitySpecializationAI::m_bChooseNewWonder` is set and how often desired wonders are recomputed.
- Audit `CvPlayer::SetBestWonderCities()` scoring inputs and whether only one city can be best for each wonder class.
- Trace building completion side effects in `CvCity::processBuilding(...)`, `CvCity::CreateBuilding(...)`, and player/team building processing.

- `CvCitySpecializationAI` in `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`: desired wonder, city specialization, and city production dirty/interrupt state.

Units and buildings are not selected in isolation. `CvUnitProductionAI::CheckUnitBuildSanity` and `CvBuildingProductionAI::CheckBuildingBuildSanity` are shared scoring hubs used by normal production and helper paths such as policy/religion valuation, purchases, best-wonder-city selection, and desired-wonder selection.

- City queue decisions live in `CvCityAI`, `CvCityStrategyAI`, `CvUnitProductionAI`, `CvBuildingProductionAI`, `CvWonderProductionAI`, `CvProjectProductionAI`, and `CvProcessProductionAI`.

| Class / manager | Decisions affected |
|---|---|
| `CvCity` (`CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvCity.h`) | city turn, production trigger, queue mutation, purchase application, production completion, operation-unit city path, best-wonder flags. Affects city production, unit production, wonder construction, operation unit demand, and emergency purchases. |
| `CvCityAI` (`CvGameCoreDLL_Expansion2/CvCityAI.cpp`) | immediate city production AI entry through `AI_chooseProduction(...)`, current production preservation, desired-wonder Great Engineer bypass. Affects city production and wonder choice. |
| `CvCityStrategyAI` (`CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI.h`) | combined final queue selector for units/buildings/wonders/projects/processes, city flavor recipient, production AI cache coordinator, hurry selector, production logging. Affects city production, unit production, wonder choice, operation unit demand, emergency purchases, and flavor propagation. |
| `CvMilitaryAI` (`CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `.h`) | military strategies, recommended force sizes, operations, army slot demand, emergency purchases, skip counters. Affects unit production, building/wonder sanity, city specialization, operation demand, emergency purchases, and flavor propagation. |
| `CvFlavorManager` / `CvFlavorRecipient` (`CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`, `.h`) | base personality, active player flavor, city flavor propagation, recipient update hooks. Affects policy weights, desired-wonder weights, city unit/building/project/process weights, and direct personality-flavor consumers. |

| Function / enum / table | Decisions affected |
|---|---|
| `CvWonderProductionAI::ChooseWonder(...)`, `FlavorUpdate()`, `AddFlavorWeights(...)`, `IsWonder(...)` (`CvWonderProductionAI.cpp`) | desired-wonder candidate scoring and selection. |
| `CvCitySpecializationAI::DoTurn()`, `SetSpecializationsDirty(...)`, `GetNextWonderDesired()`, `WeightSpecializations()`, `WeightProductionSubtypes()` (`CvCitySpecializationAI.cpp`) | desired-wonder trigger/state, specialization assignment, production interruption, and city production flavor pressure. |
| `CvCityAI::AI_chooseProduction(...)`, `AI_isChooseProductionDirty()`, `AI_setChooseProductionDirty(...)` (`CvCityAI.cpp`) | immediate production entry, current-production preservation, desired-wonder bypass, and dirty trigger state. |
| `CvCity::pushOrder(...)`, `popOrder(...)`, `produce(...)`, `CreateUnit(...)`, `CreateBuilding(...)`, `PurchaseUnit(...)`, `PurchaseBuilding(...)` (`CvCity.cpp`) | queue/purchase application, completion, and side effects for unit, building/wonder, project, process, and purchase decisions. |
| `CvCity::GetUnitForOperation()`, `CheckForOperationUnits()`, `SetBestForWonder(...)`, `IsBestForWonder(...)` (`CvCity.cpp`) | operation unit conversion, direct operation-unit purchase/queue pressure, and wonder best-city gating. |
| `CvBuildingProductionAI::GetWeight(...)`, `AddFlavorWeights(...)`, `CheckBuildingBuildSanity(...)` (`CvBuildingProductionAI.cpp`) | building/wonder base weights, final building/wonder sanity, desired-wonder scoring, best-wonder-city scoring, policy/religion helper valuation, purchase/hurry sanity. |

- best-city layer: `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::SetBestWonderCities`; `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvCity::SetBestForWonder` / `IsBestForWonder`.

- desired-wonder layer: `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`, `CvCitySpecializationAI::DoTurn`, `GetNextWonderDesired`; `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`, `CvWonderProductionAI::ChooseWonder`, `FlavorUpdate`, `AddFlavorWeights`, `IsWonder`.

- final queue layer: `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, `CvCityAI::AI_chooseProduction`; `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction`; building/wonder sanity in `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`, `CvBuildingProductionAI::CheckBuildingBuildSanity`.

- application: `CvCity::pushOrder(ORDER_CONSTRUCT, ...)`, `CvCity::CreateBuilding`, `CvCityBuildings::SetNumRealBuilding`.

- data: `CvBuildingEntry`, `CvBuildingClassInfo`, `Building_Flavors`, `CitySpecialization_Flavors`, VP building flavor data in `(2) Vox Populi/Database Changes/AI/BuildingFlavorSweeps.sql` and building data under `(2) Vox Populi/Database Changes/City/Buildings/`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvCityStrategyAI::ChooseProduction(...)` | generic units, operation units, army units, buildings, wonders, projects, processes. | RISK one candidate class's score changes alter all other classes' selection frequency. |
| `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` | ordinary building production, final wonder production, desired-wonder scoring, best-wonder-city scoring, policy branch building valuation, religion/policy helper valuation, hurry selection. | RISK not building-local or wonder-local; it feeds many score estimators. |
| `CvCity::pushOrder(...)` | unit/building/wonder/project/process final queue application. | RISK legality rechecks, making counts, order insertion mode, and queue state can diverge from selector intent. |
| `CvPlayer::SetBestWonderCities()` | desired-wonder candidate generation and final wonder sanity. | RISK best-city flags are serialized and constrain both player-level desired wonder and city-level construction. |

1. Start with HIGH_CONFIDENCE_BUG candidates that have tiny source surfaces: `CvCitySpecializationAI::FindBestWonderCity()` and `CvTacticalAI::PlotEmergencyPurchases(...)`.

5. RISK: Avoid adding broad dynamic flavors where the pressure is object-specific (`FLAVOR_WONDER` race pressure) or role-specific without data calibration (`FLAVOR_ARCHER`, `FLAVOR_NAVAL_MELEE`, etc.).

- Keep the atlas' central boundary that normal city production is one shared selector, not separate final unit/wonder/building selectors. `CvCity::doProduction(...)` enters `CvCityAI::AI_chooseProduction(...)`, which normally delegates to `CvCityStrategyAI::ChooseProduction(...)`; selected items are applied through `CvCity::pushOrder(...)` in `CvGameCoreDLL_Expansion2/CvCity.cpp`, `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, and `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`.

- Reject/defer broad `FLAVOR_WONDER` race-pressure changes. object-specific wonder pressure already appears locally in `CvWonderProductionAI::ChooseWonder(...)`, `CvPlayer::SetBestWonderCities()`, and `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`. DESIGN_HYPOTHESIS logging named components is safer than adding broad flavor deltas.

3. Recheck `CvCitySpecializationAI::FindBestWonderCity()` against caller expectations in `CvCitySpecializationAI::DoTurn()`. Source currently initializes `pBestCity = NULL` and returns it when a matching `CvCity::IsBestForWonder(...)` city is found.

`CvWonderProductionAI` is a `CvFlavorRecipient` declared in `CvGameCoreDLL_Expansion2/CvWonderProductionAI.h` and implemented in `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`.

Inputs consumed: `FlavorUpdate` reads recipient latest flavors with `GetLatestFlavorValue` and calls `AddFlavorWeights`. `AddFlavorWeights` accumulates `CvBuildingEntry::GetFlavorValue(eFlavor) * iWeight` only for `CvWonderProductionAI::IsWonder(kBuilding)`. `ChooseWonder` iterates player cities and buildings, requires `CvCity::IsBestForWonder(building class)`, reads stored building production, production cost, estimated city production, and delegates situational value to the city's `CvBuildingProductionAI::CheckBuildingBuildSanity(eBuilding, iWeight, false, false, true)`.

Selection mechanism: `ChooseWonder` stable-sorts `m_Buildables`, logs candidates, sets `iWonderWeight` to total weight, and uses `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getCityProductionChoiceCutoffThreshold(), CvSeeder::fromRaw(0x97d85ba6).mix(player ID).mix(GetNumWonders()))`.

| Function / class member | Decisions affected |
|---|---|
| `CvWonderProductionAI::AddFlavorWeights` | Desired-wonder flavor score cache |
| `CvWonderProductionAI::IsWonder` | Desired-wonder candidate class; current wonder interruption checks |
| `CvFlavorManager::AddFlavorRecipient` | Registers policy/tech/wonder/city strategy recipients for downstream scoring caches |
| `CvFlavorManager::ChangeActivePersonalityFlavors` | Player-level recipient caches: policy, tech, desired wonder |
| `CvFlavorRecipient::GetLatestFlavorValue` | Cached recipient weights for policies, wonders, and city production |

Production is primarily city-local, but the player layer supplies demand and state. `CvEconomicAI::DoTurn()` and `CvMilitaryAI::DoTurn()` can change player/city flavor state; `CvMilitaryAI::DoTurn()` can create/update operations; `CvCitySpecializationAI::DoTurn()` can choose desired wonders and mark production dirty; `CvPlayer::SetBestWonderCities()` runs earlier in `CvPlayer::doTurn()`. Source: `CvPlayer.cpp`, `doTurn`, `doTurnPostDiplomacy`; `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `CvEconomicAI::DoTurn`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::DoTurn`; `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`, `CvCitySpecializationAI::DoTurn`.

`CvPlayer::GetCitySpecializationAI()` -> desired wonder, city specialization flavor changes, production dirty/interrupt behavior.

`CvPlayer::GetWonderProductionAI()` -> desired-wonder scoring cache and player-level wonder choice.

`CvPlayer::GetFlavorManager()` -> player flavor propagation into policies, techs, desired wonders, and city flavor recipients.

- Best-wonder-city assignment: `CvPlayer::SetBestWonderCities()` loops `GC.getNumBuildingInfos()`, keeps only world and national wonders, skips civilization-unique wonders the civilization cannot build, skips puppet/annexable cities through `CityStrategyAIHelpers::IsTestCityStrategy_IsPuppetAndAnnexable(...)`, and requires `CvCity::canConstruct(eBuilding, allBuildingCount)`. It scores each candidate city with `CvBuildingProductionAI::CheckBuildingBuildSanity(eBuilding, 1000, true, false, true)`, multiplies by `10` for current/stored production, subtracts `50 * CvCity::getProductionTurnsLeft(eBuilding, 0)`, sets exactly one best city flag per building class with `CvCity::SetBestForWonder(...)`, clears other cities' flags, and reduces the chosen city's score by 25% for other world wonders to spread world-wonder races across cities.

- Final production candidates: `CvCityStrategyAI::ChooseProduction(...)` loops all buildings, skips null `CvBuildingEntry`, applies puppet and automated-human filters, requires `CvCity::canConstruct(eLoopBuilding, vTotalBuildingCount, bContinueCurrentBuilding)`, and pushes constructible buildings and wonders as `CvCityBuildable` with `m_eBuildableType = CITY_BUILDABLE_BUILDING` when `CvBuildingProductionAI::GetWeight(eLoopBuilding)` is positive. Automated human cities skip `isWorldWonderClass`, `isTeamWonderClass`, `isNationalWonderClass`, and `isLimitedWonderClass`; normal AI cities do not use that automated-human wonder skip.

- Wonder sanity gates and bonuses: `CvBuildingProductionAI::CheckBuildingBuildSanity(...)` rejects buildings when the city is in danger of falling, converts raw flavor weight with `sqrti(10 * iValue)`, blocks world/team/national/limited wonders in non-capital cities with population `<= 6`, and for national/world wonders enforces `CvCity::IsBestForWonder(...)` unless `bNoBestWonderCityCheck` is true. Best wonder city adds `+500`; non-best city returns `SR_STRATEGY`. The same function adds a large desired-wonder bonus when `CvCitySpecializationAI::GetNextWonderDesired() == eBuilding`, handles religion usefulness gates for some world wonders, applies trait wonder-production and defense-per-wonder terms, and penalizes known rival major civs already making the same world-wonder class when situational checks are not ignored.

- desired-wonder selection stable-sorts `CvWeightedVector<BuildingTypes> m_Buildables`, stores `iWonderWeight = m_Buildables.GetTotalWeight()`, and calls `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getCityProductionChoiceCutoffThreshold(), CvSeeder::fromRaw(0x97d85ba6).mix(m_pPlayer->GetID()).mix(m_pPlayer->GetNumWonders()))`. final city queue selection stable-sorts all surviving `CvCityBuildable` candidates, may keep current production if it is at least half as good as the top candidate, then usually calls `ChooseAbovePercentThreshold(...)` with `CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID())`.

- Direct desired-wonder start: `CvCityAI::AI_chooseProduction(...)` has a narrow bypass before generic `ChooseProduction(...)`: if `bInterruptWonders` is true, the city is not already building a wonder, `CvCitySpecializationAI::GetNextWonderDesired()` is valid, the city is `IsBestForWonder(...)`, and the player has at least one `UNITAI_ENGINEER`, it directly pushes `ORDER_CONSTRUCT` for the desired wonder. Otherwise, final wonder construction is just the normal building candidate path in `CvCityStrategyAI::ChooseProduction(...)`.

- Relationship to existing docs: GUIDE_PATTERN `docs-ai/vp-ai-quickref.md` was used only as a routing pattern for candidate generation, scoring, selection, application, logging, and risk categories. current behavior above was checked against `CvWonderProductionAI.cpp`, `CvPlayer.cpp`, `CvCitySpecializationAI.cpp`, `CvCityAI.cpp`, `CvCityStrategyAI.cpp`, `CvBuildingProductionAI.cpp`, `CvCity.cpp`, and `CvBuildingClasses.cpp`. See `docs-ai/atlas/01-turn-lifecycle.md`, `docs-ai/atlas/02-decision-catalog.md`, `docs-ai/ai-wonders.md`, and `docs-ai/ai-flavors.md` as maps, not proof.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
