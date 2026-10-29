# Shared weighted selection and seeded draws

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

How do candidate order, cutoffs, weights and RNG context produce a selection?

The weighted-vector helpers are shared by multiple decisions. Threshold filtering, zero-weight fallback, stable order and seeded draw context are part of the decision contract, not incidental tie behavior.

## Reference synthesis

The reported shared threshold chooser finds the highest weight, uses a relative percentage cutoff, then performs weighted choice over the retained ordered set. Highest/total zero fallbacks differ from deterministic top selection and can select among candidates the score author considered valueless. Sorting stability, database enumeration and seed mix context therefore belong to provenance.

Do not conflate helper recommendation ranking, a threshold gate and the final stochastic draw. The schema calls RNG consumption `random-draw`; tie-breaking is one possible purpose. The code-specific semantics and constants recorded below remain qualified to the recovered source baseline.

Canonical related owners: [City production selection](<../production/city-selection.md>), [Belief selection](<../beliefs/selection.md>), [Policies, branches, tenets and ideology](<../policies/selection.md>), [Wonder desire and city assignment](<../wonders/selection.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

- Likely source files:
  - `CvGameCoreDLL_Expansion2/CvFlavorManager.h`
  - `CvGameCoreDLL_Expansion2/CvFlavorManager.cpp`
  - `CvGameCoreDLL_Expansion2/CvPlayer.cpp`
  - `CvGameCoreDLL_Expansion2/CvCity.cpp`
  - `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`
  - `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`
- Likely classes:
  - `CvFlavorManager`
  - `CvFlavorRecipient`
  - `CvCityStrategyAI`
  - `CvPlayerPolicies`
  - `CvWonderProductionAI`
  - `CvBuildingProductionAI`
  - `CvUnitProductionAI`
  - `CvPolicyAI`
- Likely entry/update functions:
  - `CvFlavorManager::Init(CvPlayer* pPlayer)`
  - `CvFlavorManager::ResetToBasePersonality()`
  - `CvFlavorManager::RandomizeWeights()`
  - `CvFlavorManager::BroadcastBaseFlavors()`
  - `CvFlavorManager::AddFlavorRecipient(CvFlavorRecipient* pTargetObject, bool bPropegateFlavorValues)`
  - `CvFlavorRecipient::SetFlavors(...)`
  - `CvFlavorRecipient::ChangeFlavors(...)`
  - `CvFlavorRecipient::GetLatestFlavorValue(FlavorTypes eFlavor, bool bAllowNegative)`
- Verified recipient wiring:
  - `CvPlayer::init(...)` initializes `m_pFlavorManager` and adds player-level recipients `m_pPlayerTechs`, `m_pPlayerPolicies`, and `m_pWonderProductionAI`.
  - `CvCity::init(...)` adds `m_pCityStrategyAI` as a city-level recipient.
  - Load path in `CvPlayer::Serialize(...)` re-adds player-level flavor recipients without propagating values; `CvCity::Serialize(...)` re-adds city strategy without propagation.
- Likely consumers:
  - `CvCityStrategyAI::FlavorUpdate()` resets building/unit/project/process production AIs and calls their `AddFlavorWeights(...)`.
  - `CvPlayerPolicies::FlavorUpdate()` calls `AddFlavorAsStrategies(...)`, which feeds `CvPolicyAI::AddFlavorWeights(...)`.
  - `CvWonderProductionAI::FlavorUpdate()` calls `CvWonderProductionAI::AddFlavorWeights(...)`.
  - `CvPolicyAI::WeighPolicy(...)` also reads dynamic flavor/personality through `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`.
  - `CvUnitProductionAI::CheckUnitBuildSanity(...)` and `CvCityStrategyAI::ChooseProduction(...)` read `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` for specific production/military choices.
- Likely scoring infrastructure:
  - Database-backed entries expose `GetFlavorValue(...)` on entries such as `CvPolicyEntry`, `CvBuildingEntry`, and `CvUnitEntry`.
  - Weighted candidates use `CvWeightedVector<T>` from `CvGameCoreDLLUtil/include/CvWeightedVector.h`, especially `StableSortItems()`, `ChooseAbovePercentThreshold(...)`, and direct `GetElement(0)` in some paths.

## Selection and control

- chooser functions use `CvWeightedVector<BeliefTypes>::StableSortItems` and `ChooseAbovePercentThreshold` with `BeliefChoiceCutoffThreshold` and category-specific seeds in `CvReligionClasses.cpp`.

- one `CvWeightedVector<int>` stores branch IDs directly and policy IDs offset by `GC.getNumPolicyBranchInfos()`, then selects with `ChooseAbovePercentThreshold` using `PolicyChoiceCutoffThreshold` and seed `0x5fef0474` mixed with player ID and owned policy count.

- selection: `CvReligionAI::ChoosePantheonBelief`, `ChooseFounderBelief`, `ChooseFollowerBelief`, `ChooseEnhancerBelief`, `ChooseBonusBelief`, `ChooseReformationBelief`; shared `CvWeightedVector<BeliefTypes>::ChooseAbovePercentThreshold`.

`CvWeightedVector<T>::StableSortItems` and `CvWeightedVector<T>::ChooseAbovePercentThreshold` in `CvGameCoreDLLUtil/include/CvWeightedVector.h` are shared by belief choice, normal policy/branch/tenet choice, desired-wonder choice, and final city production. `ChooseAbovePercentThreshold(int iPercent, CvSeeder seed)` clamps the percentage to `0..100`, finds the highest weight, chooses random if all weights are zero, chooses by full weight if the cutoff is zero, otherwise builds a temporary vector of candidates whose weight is at least `highest * percent / 100` and chooses by weight from that filtered vector.

RISK: Candidate order, zero-weight pools, score scale, threshold globals such as `BeliefChoiceCutoffThreshold`, `PolicyChoiceCutoffThreshold`, and `CityProductionChoiceCutoffThreshold`, and seed inputs are multiplayer/determinism-sensitive.

GUIDE_PATTERN: The guide says random choices should use seeded randomness so results are reproducible. Main function: `CvGame::randRangeInclusive(int min, int max, const CvSeeder& extraSeed)`. Seeds are built with `CvSeeder::fromRaw(...)` and `.mix(...)` calls. Reused seed literals should be avoided. Mixed values often include IDs and current turn number.

GUIDE_PATTERN: For AI choice docs, record exact seed usage. This matters for reproducibility and multiplayer.

GUIDE_PATTERN: Random choices should use seeded randomness so outcomes are reproducible. The guide highlights `CvGame::randRangeInclusive(int min, int max, const CvSeeder& extraSeed)`, with seeds built using `CvSeeder::fromRaw(...).mix(...)`. Common `mix` inputs include player IDs, object IDs, candidate counts, and current turn numbers.

GUIDE_PATTERN: When a decision uses randomness, document:

GUIDE_PATTERN: - Seed literal.

GUIDE_PATTERN: - Values mixed into the seed.

GUIDE_PATTERN: - Whether selection is pure random, thresholded weighted choice, or tie/random among high scorers.

GUIDE_PATTERN: - Whether multiplayer determinism might be affected.

## Version differences, risks and unresolved questions

- Savegame/sync-sensitive areas appear in audited source:
  - `CvBuildingProductionAI::Serialize(...)`, `CvUnitProductionAI::Serialize(...)`, `CvWonderProductionAI::Serialize(...)`, `CvFlavorManager::Serialize(...)`, and large `CvPlayer`/`CvCity` serialization blocks.
  - `CvPlayer.h` contains `SYNC_ARCHIVE_VAR(...)` fields for several wonder-related counters and modifiers.
- Randomness-sensitive choices use `CvSeeder::fromRaw(...).mix(...)` in belief, policy, city production, and wonder selection.
- Enum-sensitive areas include open enum types in `CvGameCoreDLLUtil/include/CvEnums.h`, especially `FlavorTypes`, `PolicyTypes`, `PolicyBranchTypes`, `BeliefTypes`, `BuildingTypes`, `UnitTypes`, and `UnitAITypes`.

`CvWeightedVector<T>::ChooseAbovePercentThreshold` in `CvGameCoreDLLUtil/include/CvWeightedVector.h` is a shared selection primitive. It computes the highest weight, chooses randomly if all weights are zero, otherwise filters by a percentage cutoff and chooses by weight. Decisions using it include belief choice, policy/branch/tenet choice, desired-wonder choice, and final city production.

- Known improvement hooks: OPEN_QUESTION document `CvWeightedVector::ChooseAbovePercentThreshold` once in a shared infrastructure doc; enumerate `CvBeliefEntry::CacheResults` fields and database schema separately.

- RISK `CvWeightedVector<T>::ChooseAbovePercentThreshold(...)`: shared threshold-random primitive. Candidate ordering, score scale, all-zero pools, `BeliefChoiceCutoffThreshold`, `PolicyChoiceCutoffThreshold`, `CityProductionChoiceCutoffThreshold`, and seed literals (`0x0af7fe29`, `0x9db23f3c`, `0x93c8983d`, `0x3a862bb8`, `0xc0809801`, `0x894eaafe`, `0x5fef0474`, `0x97d85ba6`, `0x0e36d18b`, `0xe362f42a`) are determinism-sensitive.

## Source and dependency lookup

- Likely source files:
  - `CvGameCoreDLL_Expansion2/CvReligionClasses.h`
  - `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`
  - `CvGameCoreDLL_Expansion2/CvPlayer.cpp`
- Likely classes:
  - `CvReligionAI`
  - `CvGameReligions`
  - `CvPlayerReligions`
  - `CvBeliefEntry`
- Likely entry functions:
  - `CvReligionAI::ChoosePantheonBelief(PlayerTypes ePlayer)`
  - `CvReligionAI::ChooseFounderBelief(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvReligionAI::ChooseFollowerBelief(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvReligionAI::ChooseEnhancerBelief(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvReligionAI::ChooseBonusBelief(PlayerTypes ePlayer, ReligionTypes eReligion, int iExcludeBelief1, int iExcludeBelief2, int iExcludeBelief3)`
  - `CvReligionAI::ChooseReformationBelief(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvGameReligions::DoPlayerTurn(CvPlayer& kPlayer)` calls `ChoosePantheonBelief(...)`/`FoundPantheon(...)` and `ChooseReformationBelief(...)`/`AddReformationBelief(...)`.
  - `CvPlayer::doTurn(...)` includes a pantheon call to `ChoosePantheonBelief(...)`/`CvGameReligions::FoundPantheon(...)`.
  - `CvPlayer::doAdoptPolicy(...)` includes a reformation call to `ChooseReformationBelief(...)`/`CvGameReligions::AddReformationBelief(...)`.
- Likely candidate generation functions:
  - `CvGameReligions::GetAvailablePantheonBeliefs(PlayerTypes ePlayer)`
  - `CvGameReligions::GetAvailableFounderBeliefs(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvGameReligions::GetAvailableFollowerBeliefs(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvGameReligions::GetAvailableEnhancerBeliefs(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvGameReligions::GetAvailableBonusBeliefs(PlayerTypes ePlayer, ReligionTypes eReligion)`
  - `CvGameReligions::GetAvailableReformationBeliefs(PlayerTypes ePlayer, ReligionTypes eReligion)`
- Likely scoring functions:
  - `CvReligionAI::ScoreBelief(CvBeliefEntry* pEntry, CvWeightedVector<int> viPlotWeights, bool bForBonus, bool bConsiderFutureTech) const`
  - `CvReligionAI::ScoreBeliefAtPlot(CvBeliefEntry* pEntry, CvPlot* pPlot, bool bConsiderFutureTech) const`
  - `CvReligionAI::ScoreBeliefAtCity(CvBeliefEntry* pEntry, CvCity* pCity) const`
  - `CvReligionAI::ScoreBeliefForPlayer(CvBeliefEntry* pEntry, bool bReturnConquest, bool bReturnCulture, bool bReturnScience, bool bReturnDiplo) const`
  - Also used by `ScoreBelief`: `ScorePantheonBeliefAtCity(...)` and `ScorePantheonBeliefForPlayer(...)`.
- Likely selection functions:
  - Each `Choose*Belief` builds a `CvWeightedVector<BeliefTypes>`, calls `StableSortItems()`, then selects with `ChooseAbovePercentThreshold(...)`.
  - Seed literals verified in the chooser functions include `0x0af7fe29`, `0x9db23f3c`, `0x93c8983d`, `0x3a862bb8`, `0xc0809801`, and `0x894eaafe`.
- Likely final application functions:
  - `CvGameReligions::FoundPantheon(PlayerTypes ePlayer, BeliefTypes eBelief)`
  - `CvGameReligions::FoundReligion(PlayerTypes ePlayer, ReligionTypes eReligion, const char* szCustomName, BeliefTypes eBelief1, BeliefTypes eBelief2, BeliefTypes eBelief3, BeliefTypes eBelief4, CvCity* pkHolyCity)`
  - `CvGameReligions::EnhanceReligion(PlayerTypes ePlayer, ReligionTypes eReligion, BeliefTypes eBelief1, BeliefTypes eBelief2, bool bNotify, bool bSetAsEnhanced)`
  - `CvGameReligions::AddReformationBelief(PlayerTypes ePlayer, ReligionTypes eReligion, BeliefTypes eBelief1)`
- Logging:
  - `CvReligionAI::LogBeliefChoices(...)` is called after belief selection.

- Likely source files:
  - `CvGameCoreDLL_Expansion2/CvPolicyAI.h`
  - `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvPolicyClasses.h`
  - `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`
  - `CvGameCoreDLL_Expansion2/CvPlayer.cpp`
- Likely classes:
  - `CvPolicyAI`
  - `CvPlayerPolicies`
  - `CvPolicyEntry`
  - `CvPolicyBranchEntry`
- Likely entry functions:
  - `CvPlayerPolicies::DoChooseNextPolicy()`
  - `CvPolicyAI::ChooseNextPolicy(CvPlayer* pPlayer)`
  - `CvPlayerPolicies::DoChooseIdeology()`
  - `CvPolicyAI::DoChooseIdeology(CvPlayer* pPlayer)`
  - `CvPlayerPolicies::ScorePolicy(PolicyTypes ePolicy) const`
  - `CvPlayerPolicies::ScorePolicyBranch(PolicyBranchTypes ePolicyBranch) const`
- Likely candidate generation functions:
  - `CvPolicyAI::ChooseNextPolicy(...)` loops all policies and checks `CvPlayerPolicies::CanAdoptPolicy(PolicyTypes eIndex, bool bIgnoreCost) const`.
  - `CvPolicyAI::ChooseNextPolicy(...)` also loops policy branches and checks `CvPlayerPolicies::CanUnlockPolicyBranch(...)`, `CvPlayerPolicies::IsPolicyBranchUnlocked(...)`, and `CvPolicyAI::IsBranchEffectiveInGame(...)`.
  - Tenet constraint is in `CvPolicyAI::ChooseNextPolicy(...)` through `pPlayer->GetNumFreeTenets()` and `CvPolicyEntry::GetLevel()`.
- Likely scoring functions:
  - `CvPolicyAI::WeighPolicy(CvPlayer* pPlayer, PolicyTypes ePolicy)`
  - `CvPolicyAI::WeighBranch(CvPlayer* pPlayer, PolicyBranchTypes eBranch)`
  - `CvPolicyAI::WeightPolicyAttributes(CvPlayer* pPlayer, PolicyTypes ePolicy)`
  - `CvPolicyAI::AddFlavorWeights(FlavorTypes eFlavor, int iWeight, int iPropagationPercent)`
  - `CvPolicyAI::PropagateWeights(int iPolicy, int iWeight, int iPropagationPercent, int iPropagationLevel)`
- Likely selection functions:
  - `CvPolicyAI::ChooseNextPolicy(...)` uses `m_AdoptablePolicies.StableSortItems()` and `ChooseAbovePercentThreshold(...)`.
  - Verified policy seed: `CvSeeder::fromRaw(0x5fef0474).mix(pPlayer->GetID()).mix(pPlayer->GetPlayerPolicies()->GetNumPoliciesOwned(false, false, true))`.
  - `CvPolicyAI::DoChooseIdeology(...)` directly sets ideology branches in some cases, including vassal/master ideology mirroring.
- Likely final application functions:
  - `CvPlayerPolicies::DoUnlockPolicyBranch(PolicyBranchTypes eBranch)`
  - `CvPlayer::doAdoptPolicy(PolicyTypes ePolicy)`
  - `CvPlayerPolicies::SetPolicyBranchUnlocked(...)` in ideology choice paths.
- Logging:
  - `CvPolicyAI::LogPossiblePolicies()`
  - `CvPolicyAI::LogPolicyChoice(PolicyTypes ePolicy)`
  - `CvPolicyAI::LogBranchChoice(PolicyBranchTypes eBranch)`

- Likely source files:
  - `CvGameCoreDLL_Expansion2/CvWonderProductionAI.h`
  - `CvGameCoreDLL_Expansion2/CvWonderProductionAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.h`
  - `CvGameCoreDLL_Expansion2/CvBuildingProductionAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`
  - `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.h`
  - `CvGameCoreDLL_Expansion2/CvCitySpecializationAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCityAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCity.cpp`
  - `CvGameCoreDLL_Expansion2/CvPlayer.cpp`
- Likely classes:
  - `CvWonderProductionAI`
  - `CvBuildingProductionAI`
  - `CvCityStrategyAI`
  - `CvCitySpecializationAI`
  - `CvCityAI`
  - `CvCity`
  - `CvBuildingEntry`
- Likely entry functions:
  - `CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)`
  - `CvCityStrategyAI::ChooseProduction(BuildingTypes eIgnoreBldg, UnitTypes eIgnoreUnit, bool bInterruptBuildings, bool bInterruptWonders)`
  - `CvCitySpecializationAI::DoTurn()`-area update that calls `CvWonderProductionAI::ChooseWonder(int& iWonderWeight)`
  - `CvWonderProductionAI::ChooseWonder(int& iWonderWeight)`
- Likely candidate generation functions:
  - City production loop: `CvCityStrategyAI::ChooseProduction(...)` loops `GC.GetGameBuildings()->GetNumBuildings()`, skips null `CvBuildingEntry`, and checks `CvCity::canConstruct(...)`.
  - Wonder preselection loop: `CvWonderProductionAI::ChooseWonder(...)` loops cities and buildings, requires `CvCity::IsBestForWonder(...)`, reweights by estimated turns, and calls city `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`.
  - Best-city support: `CvPlayer::SetBestWonderCities()`, `CvCity::SetBestForWonder(...)`, and `CvCity::IsBestForWonder(...)`.
  - Wonder classification: `CvWonderProductionAI::IsWonder(const CvBuildingEntry& kBuilding)` checks world, team, and national wonder classes.
- Likely scoring functions:
  - `CvBuildingProductionAI::AddFlavorWeights(FlavorTypes eFlavor, int iWeight)`
  - `CvBuildingProductionAI::GetWeight(BuildingTypes eBuilding)`
  - `CvBuildingProductionAI::CheckBuildingBuildSanity(BuildingTypes eBuilding, int iValue, ...)`
  - `CvWonderProductionAI::AddFlavorWeights(FlavorTypes eFlavor, int iWeight)`
  - `CvWonderProductionAI::GetWeight(BuildingTypes eBldg)`
  - `CityStrategyAIHelpers::ReweightByTurnsLeft(...)`
  - `CvCityStrategyAI::ReweightByDuration(CvWeightedVector<CvCityBuildable>& options)`
- Likely selection functions:
  - General production: `CvCityStrategyAI::ChooseProduction(...)` builds `m_BuildablesPrecheck`, sanity-checks into `m_Buildables`, calls `StableSortItems()`, may continue current production by inertia, otherwise calls `ChooseAbovePercentThreshold(...)`.
  - Verified general production seed: `CvSeeder::fromRaw(0x0e36d18b).mix(m_pCity->GetID())`.
  - Wonder preselection: `CvWonderProductionAI::ChooseWonder(...)` calls `StableSortItems()` and `ChooseAbovePercentThreshold(...)`.
  - Verified wonder seed: `CvSeeder::fromRaw(0x97d85ba6).mix(m_pPlayer->GetID()).mix(m_pPlayer->GetNumWonders())`.
- Likely final application functions:
  - `CvCityStrategyAI::ChooseProduction(...)` applies building choice with `CvCity::pushOrder(ORDER_CONSTRUCT, eBuildingType, -1, false, true, false, bRush)`.
  - Production completion path includes `CvCity::CreateBuilding(BuildingTypes eBuildingType)` and `CvCityBuildings::SetNumRealBuilding(...)` calls from `CvCity.cpp`.
- Logging:
  - `CvCityStrategyAI::LogPossibleBuilds(...)`
  - `CvCityStrategyAI::LogCityProduction(CvCityBuildable buildable, bool bRush)`
  - `CvBuildingProductionAI::LogPossibleBuilds()`
  - `CvWonderProductionAI::LogPossibleWonders()`

- Likely source files:
  - `CvGameCoreDLL_Expansion2/CvUnitProductionAI.h`
  - `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCityStrategyAI.h`
  - `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCityAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvCity.cpp`
  - `CvGameCoreDLL_Expansion2/CvMilitaryAI.h`
  - `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`
  - `CvGameCoreDLL_Expansion2/CvPlayer.cpp`
- Likely classes:
  - `CvUnitProductionAI`
  - `CvCityStrategyAI`
  - `CvCityAI`
  - `CvCity`
  - `CvMilitaryAI`
  - `CvUnitEntry`
- Likely entry functions:
  - `CvCityAI::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)`
  - `CvCityStrategyAI::ChooseProduction(BuildingTypes eIgnoreBldg, UnitTypes eIgnoreUnit, bool bInterruptBuildings, bool bInterruptWonders)`
  - `CvUnitProductionAI::RecommendUnit(UnitAITypes eUnitAIType, bool bAllowStrategicResource)`
  - Operation/army unit sources: `CvCity::GetUnitForOperation()` and `CvMilitaryAI::GetUnitTypeForArmy(CvCity* pCity)`.
- Likely candidate generation functions:
  - General city production loop: `CvCityStrategyAI::ChooseProduction(...)` loops `GC.GetGameUnits()->GetNumUnits()` and checks `CvCity::canTrain(...)`.
  - Operation and army candidates are injected before general unit loop as `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY`.
  - `CvUnitProductionAI::RecommendUnit(...)` loops all units, filters by `CvUnitEntry::GetUnitAIType(eUnitAIType)`, strategic resource allowance, and `CvCity::canTrain(...)`.
- Likely scoring functions:
  - `CvUnitProductionAI::AddFlavorWeights(FlavorTypes eFlavor, int iWeight)`
  - `CvUnitProductionAI::GetWeight(UnitTypes eUnit)`
  - `CvUnitProductionAI::RecommendUnit(...)` uses `m_UnitAIWeights`, fallback sum of unit flavors, and `CityStrategyAIHelpers::ReweightByTurnsLeft(...)`.
  - `CvUnitProductionAI::CheckUnitBuildSanity(UnitTypes eUnit, bool bForOperation, int iFlavorWeight, bool bForPurchase, bool bFree)`
  - `CvCityStrategyAI::ReweightByDuration(CvWeightedVector<CvCityBuildable>& options)`
- Likely selection functions:
  - General production selection is shared with buildings in `CvCityStrategyAI::ChooseProduction(...)`: sorted `CvWeightedVector<CvCityBuildable>`, current-build inertia, and threshold selection with seed `0x0e36d18b`.
  - `CvUnitProductionAI::RecommendUnit(...)` sorts and returns the top unit with `m_Buildables.GetElement(0)` rather than threshold random selection.
- Likely final application functions:
  - `CvCityStrategyAI::ChooseProduction(...)` applies unit choice with `CvCity::pushOrder(ORDER_TRAIN, eUnitType, eUnitAI, false, true, false, bRush)`.
  - Production completion path includes `CvCity::produce(UnitTypes eTrainUnit, UnitAITypes eTrainAIUnit, bool bCanOverflow)` and `CvCity::CreateUnit(UnitTypes eUnitType, UnitAITypes eAIType, UnitCreationReason eReason)`.
- Logging:
  - `CvUnitProductionAI::LogPossibleBuilds(UnitAITypes eUnitAIType = NO_UNITAI)`
  - `CvCityStrategyAI::LogPossibleBuilds(...)`
  - `CvCityStrategyAI::LogCityProduction(...)`

The normal city production decision is `CvCity::doProduction` -> `CvCityAI::AI_chooseProduction` -> `CvCityStrategyAI::ChooseProduction`. `ChooseProduction` builds one `CvWeightedVector<CvCityBuildable>` containing units, buildings, projects, and processes, then applies type-specific sanity checks, duration reweighting, current-production inertia, threshold selection, and finally `CvCity::pushOrder`.

- final city production selection through `CvWeightedVector<CvCityBuildable>::ChooseAbovePercentThreshold`; `CvUnitProductionAI::RecommendUnit` is a helper that stable-sorts and returns the top unit for requested `UnitAITypes`.

- candidate generation/scoring/selection: `CvCityStrategyAI::ChooseProduction`; candidate type enum `CityBuildableType`; sanity code enum `ProductionSkipReason`; shared `CvWeightedVector<CvCityBuildable>`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvWeightedVector<T>::ChooseAbovePercentThreshold(...)` | beliefs, normal policies/branches/tenets, desired wonders, final city production, hurry selection. | RISK seed inputs, candidate ordering, zero-weight behavior, cutoff thresholds, and database enum ordering affect deterministic AI outcomes. |

RISK: Most candidates below touch shared hubs: `CvCityStrategyAI::ChooseProduction(...)`, `CvUnitProductionAI::CheckUnitBuildSanity(...)`, `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, `CvWeightedVector<T>::ChooseAbovePercentThreshold(...)`, `CvFlavorManager` propagation, operation-slot queues, and policy/religion application paths. Treat "smallest patch point" as a future edit location, not a recommendation to edit now.

- Keep the risk treatment for `CvWeightedVector<T>::ChooseAbovePercentThreshold(...)` in `CvGameCoreDLLUtil/include/CvWeightedVector.h`. It is shared by belief choice, policy/branch/tenet choice, desired-wonder choice, final city production, and hurry selection.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U bounded current helper revalidation

`src-20261029-vp-c285a18`, `audit-20261029-task1u-01`: CvWeightedVector.h162–234 and CvGame.cpp10142–10184 verified for city unit competition. [Current shared selection](../production/city-selection.md#task-1u-current-unit-selection) owns call-specific cutoff/seed/pool/order/fallback. This is partial revalidation of this document, not all helper consumers. randCore mixes mapRandomSeed+gameTurn+extra CvSeeder, modulo range; CvSeeder mixing internals remain unresolved. Recommendation is stable deterministic first maximum; shared city selector is score-weighted sampling above relative cutoff unless continuation/override applies. Random sampling is not synonymous with tie-break.

## Task 1U-B CvSeeder closure

Current authority src-20261029-vp-548 / audit-20261029-task1u-b. CvRandom.h20–88 closes previously unresolved mixing. All following arithmetic is uint32 modulo2^32 (overflow intentional). `H(x)`: state=x*747796405+2891336453; word=((state>>((state>>28)+4)) XOR state)*277803737; result=(word>>22) XOR word. CvSeeder(x)=H(uint(x)); fromRaw(x)=x. `M(v,x)=v XOR (x+0x9e3779b9+(v<<6)+(v>>2))`. mix(integer x)=M(v,H(x)); mix(CvSeeder x)=M(v,x.value), so it does not hash a seeder again.

Normal city extra=`M(0x0e36d18b,H(cityID))`; hurry extra uses0xe362f42a. Game::randCore10126= `M(M(mapRandomSeed,H(gameTurn)),extra.value)`, because map uses fromRaw and gameTurn uses hashed constructor. Exclusive draw=result%weightTotal, no mutable RNG-stream advance in this path. Weighted mapping retains cutoff order and subtracts weights until <0; zero total uses result%poolSize. Modulo bias is not ruled out. Stable sorted equal weights preserve insertion order, including duplicate unit identities with different buildable tags. Continuation/victory/defense overrides draw nothing.

Other included draw purpose: TacticalAI::PlotEmergencyPurchases1784, !AI_UNIT_PRODUCTION chooses COUNTER if draw[0,5)<2 else DEFENSE using city plot pseudo-random seed wrapped as CvSeeder (copy preserves existing seed), combined with map/turn; ON always DEFENSE. Personality initialization draw (raw0xe655df8f, player,flavor) belongs to flavor lifecycle. Neither is merely a final-choice tie-break. Plot::GetPseudoRandomSeed16444 returns CvSeeder(uint(x)*17+uint(y)*23), so actual value is H(x*17+y*23); the outer CvSeeder copies it. Capture plot coordinates/value. Underlying map/turn seed, constants, city/plot IDs, eligible order, total and draw suffice to reproduce included selections.
