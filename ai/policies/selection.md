# Policies, branches, tenets and ideology

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../source-snapshots.md#recovered-baseline>).

## Decision or contract question

Which policy/branch/tenet is chosen, which ideology is adopted, and when is it switched?

Ordinary policy/branch/tenet competition, initial ideology selection and ideology switching are separate decisions. Human vassal ideology updates and forced choices require their own control-flow treatment.

## Reference synthesis

Treat ordinary policy/branch/tenet choice, initial ideology adoption and ideology switching as separate decisions. Normal candidates are filtered by native legality, free-tenet constraints, branch continuation and game effectiveness. Base flavor propagation and direct policy-attribute valuation are separate contributions. Ideology adoption/switching additionally uses team/vassal, happiness/public-opinion and other special paths described below.

A flavor recipient update does not mean policy scoring consumes that recipient's latest vector. The recovered recomputation reads long-term personality or grand-strategy values with direct situational boosts. Its empty flavor logger is an observability limit, not evidence that recomputation never happens. Human vassals can also receive forced ideology changes even though normal AI policy choice is skipped.

Canonical related owners: [Policy adoption and ideology application](<../../mechanics/policies/adoption-and-ideology.md>), [Flavor production, propagation and consumption](<../infrastructure/flavors.md>), [Shared weighted selection and seeded draws](<../infrastructure/weighted-selection.md>), [Strategic pressure and manager outputs](<../strategies/managers.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

Turn trigger is `CvPlayer::doTurn` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`. After culture accumulation and `CvPlayer::DoUpdateNextPolicyCost`, major civs with cities enter the policy section. Non-human policy-choice players check `CvPlayerPolicies::IsTimeToChooseIdeology` and `GetLateGamePolicyTree`; if ideology is due, they may resolve an adoption-free-tech trait with `CvPlayerTraits::IsAdoptionFreeTech`, then call `CvPlayerPolicies::DoChooseIdeology`. They then call `CvPlayerPolicies::DoPolicyAI`.

Human policy-choice players use notification/UI flow in `CvPlayer::doTurn`, but the same function still calls `CvPlayerPolicies::DoPolicyAI` afterward to force ideology-update handling for human vassals. `CvPlayerPolicies::DoPolicyAI` immediately calls `CvPolicyAI::DoConsiderIdeologySwitch`, then returns for `m_pPlayer->isHuman(ISHUMAN_AI_POLICY_CHOICE)`.

Spending trigger is `CvPlayerPolicies::DoPolicyAI` in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`. It loops while the player has enough culture, free policies, or free tenets, calls `CvPolicyAI::ChooseNextPolicy`, then applies branch results with `CvPlayerPolicies::DoUnlockPolicyBranch` or policy/tenet results with `CvPlayer::doAdoptPolicy`.

Ideology switch trigger is also `CvPlayerPolicies::DoPolicyAI`. `CvPolicyAI::DoConsiderIdeologySwitch` runs before the human early return, so vassal ideology forcing and some public-opinion switch logic can run independently of normal AI policy spending.

Lua/network/UI paths share final application functions but not AI selection. `CvDllNetMessageHandler::ResponseAdoptPolicy` and `ResponseAdoptPolicyBranch` in `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp` call `CvPlayer::doAdoptPolicy` or `CvPlayerPolicies::DoUnlockPolicyBranch`/`SetPolicyBranchUnlocked`.

- `CvFlavorManager`, `CvFlavorRecipient`, and recipient classes such as `CvPlayerPolicies`, `CvPlayerTechs`, `CvWonderProductionAI`, and `CvCityStrategyAI`.

Decision role: `CvPolicyAI::ChooseNextPolicy` selects normal policies, branch openings, and ideology tenets. `CvPolicyAI::DoChooseIdeology` separately selects late-game ideology branches. `CvPolicyAI::DoConsiderIdeologySwitch` evaluates ideology switching.

- AI choice of normal social policies, normal policy branch openings, ideology tenets, first ideology branch, and ideology switching. Normal policies, normal branches, and tenets share `CvPolicyAI::ChooseNextPolicy(CvPlayer*)` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`. First ideology branch choice is separate in `CvPolicyAI::DoChooseIdeology(CvPlayer*)`. Later ideology switching is separate again in `CvPolicyAI::DoConsiderIdeologySwitch(CvPlayer*)`.

- selection is owned by `CvPolicyAI` and policy state/application is owned by `CvPlayerPolicies` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, and `CvGameCoreDLL_Expansion2/CvPolicyClasses.h`. final policy application uses `CvPlayer::doAdoptPolicy(PolicyTypes)`, `CvPlayer::setHasPolicy(PolicyTypes, bool, bool)`, and `CvPlayer::processPolicies(PolicyTypes, int)` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`.

- policy and ideology processing happens in `CvPlayer::doTurnPostDiplomacy()` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, after city turns, culture accumulation, `CvPlayer::DoUpdateNextPolicyCost()`, and anarchy decrement. For non-human policy-choice players, `CvPlayer::doTurnPostDiplomacy()` calls `CvPlayerPolicies::IsTimeToChooseIdeology()`, then `CvPlayerPolicies::DoChooseIdeology()` if no late-game tree exists, then `CvPlayerPolicies::DoPolicyAI()`. Human policy-choice players use notification/UI flow for ordinary choices, but `CvPlayer::doTurnPostDiplomacy()` still calls `CvPlayerPolicies::DoPolicyAI()` afterward so `CvPolicyAI::DoConsiderIdeologySwitch(...)` can force ideology updates for human vassals.

### Ideology Switching

- `CvPlayerPolicies::DoPolicyAI()` always calls `CvPolicyAI::DoConsiderIdeologySwitch(CvPlayer*)` before returning for human policy-choice players and before normal AI policy spending. This means ideology-switch forcing is evaluated even when no ordinary policy is bought.

## Candidates and gates

`CvPolicyAI::ChooseNextPolicy` first loops all `CvPolicyEntry` rows from `CvPolicyXMLEntries` and keeps policies where `CvPlayerPolicies::CanAdoptPolicy` is true. If `CvPlayer::GetNumFreeTenets() > 0`, it further requires `CvPolicyEntry::GetLevel() > 0`, so free-tenet spending only considers tenets.

`CvPlayerPolicies::CanAdoptPolicy` rejects null entries, already-owned policies, insufficient culture/free-policy/free-tenet state, policies with `NO_POLICY_BRANCH_TYPE`, locked branches, invalid purchase-by-level tenet tier counts, unmet OR/AND policy prerequisites, disabled policies, unmet tech prerequisites, `PlayerCanAdoptPolicy` Lua vetoes, and `GAMEEVENT_PlayerCanAdoptTenet` vetoes under `MOD_EVENTS_IDEOLOGIES`.

Tenet candidate rules are implemented through ordinary policy gates. `CvPolicyBranchEntry::IsPurchaseByLevel` marks ideology-style branches; `CvPolicyEntry::GetLevel` marks tenet tier. `CvPlayerPolicies::CanAdoptPolicy` enforces that level 2/3 tenets require enough previous-level tenets and enforces `MAX_NUM_TENETS_LEVEL_1`, `MAX_NUM_TENETS_LEVEL_2`, and `MAX_NUM_TENETS_LEVEL_3`.

Branch candidates are added by `CvPolicyAI::ChooseNextPolicy` only when the player is not forced to spend a free tenet and is not trying to finish an already-unlocked branch. It loops `GC.getNumPolicyBranchInfos()`, skips null `CvPolicyBranchEntry`, skips mutually exclusive branches if another mutually exclusive branch has already been unlocked, requires `CvPlayerPolicies::CanUnlockPolicyBranch`, requires the branch not already be unlocked, and uses `CvPolicyAI::IsBranchEffectiveInGame` to zero branches disabled by game options.

`CvPlayerPolicies::CanUnlockPolicyBranch` rejects insufficient culture/free-policy state, purchase-by-level ideology branches, branches locked without religion when `GAMEOPTION_NO_RELIGION` is active, unmet era or policy-count unlock requirements, and `PlayerCanAdoptPolicyBranch` Lua vetoes. Ideology branches are explicitly not unlocked through this normal branch path.

Ideology branch candidates in `CvPolicyAI::DoChooseIdeology` are the hard-coded `POLICY_BRANCH_FREEDOM`, `POLICY_BRANCH_AUTOCRACY`, and `POLICY_BRANCH_ORDER` from `GD_INT_GET`; if any is `NO_POLICY_BRANCH_TYPE`, the function returns. Under `MOD_EVENTS_IDEOLOGIES`, `CvPlayerPolicies::CanAdoptIdeology` can zero any of the three through `GAMEEVENT_PlayerCanAdoptIdeology`.

`CvPlayerPolicies::IsTimeToChooseIdeology` excludes minors and barbarians; requires Freedom, Autocracy, and Order branch constants; then allows ideology by current era plus ideology points/policy count, by era later than `IDEOLOGY_START_ERA`, or by owning enough of a building whose `CvBuildingEntry::GetXBuiltTriggersIdeologyChoice` threshold is met.

- `CvPolicyAI::ChooseNextPolicy(CvPlayer*)` first loops all policy rows from `CvPolicyXMLEntries::GetNumPolicies()` / `CvPolicyXMLEntries::GetPolicyEntry(...)`. It keeps policies where `CvPlayerPolicies::CanAdoptPolicy(PolicyTypes, bool)` is true. If `CvPlayer::GetNumFreeTenets() > 0`, it additionally requires `CvPolicyEntry::GetLevel() > 0`, so free-tenet spending is restricted to tenets.

- Candidate generation, policy legality: `CvPlayerPolicies::CanAdoptPolicy(...)` rejects null `CvPolicyEntry`, already-owned policies, insufficient culture/free policy/free tenet state, `NO_POLICY_BRANCH_TYPE`, locked branches, unmet tenet level constraints for `CvPolicyBranchEntry::IsPurchaseByLevel()`, unmet OR/AND prerequisites from `CvPolicyEntry::GetPrereqOrPolicies(...)` and `GetPrereqAndPolicies(...)`, policies disabled by owned policies through `CvPolicyEntry::GetPolicyDisables()`, unmet `CvPolicyEntry::GetTechPrereq()`, Lua `PlayerCanAdoptPolicy`, and `GAMEEVENT_PlayerCanAdoptTenet` under `MOD_EVENTS_IDEOLOGIES`.

- Candidate generation, tenet legality: tenets are ordinary `Policies` rows whose `CvPolicyEntry::GetLevel() > 0` and whose branch uses `CvPolicyBranchEntry::IsPurchaseByLevel()`. `CvPlayerPolicies::CanAdoptPolicy(...)` enforces the previous-level requirement for level 2/3 tenets and the define-backed caps `MAX_NUM_TENETS_LEVEL_1`, `MAX_NUM_TENETS_LEVEL_2`, and `MAX_NUM_TENETS_LEVEL_3`.

- Candidate generation, normal branch legality: `CvPolicyAI::ChooseNextPolicy(...)` considers branch opening only when the player is not forced to spend a free tenet and does not need to finish an already-started branch. It loops `GC.getNumPolicyBranchInfos()`, null-checks `CvPolicyBranchEntry`, skips mutually exclusive branches after one mutually exclusive branch is unlocked, requires `CvPlayerPolicies::CanUnlockPolicyBranch(...)`, requires the branch not already be unlocked, and scores the branch only if `CvPolicyAI::IsBranchEffectiveInGame(PolicyBranchTypes)` says game options do not make it ineffective.

- Candidate generation, branch legality: `CvPlayerPolicies::CanUnlockPolicyBranch(...)` rejects insufficient culture/free-policy state, `CvPolicyBranchEntry::IsPurchaseByLevel()` ideology branches, `CvPolicyBranchEntry::IsLockedWithoutReligion()` under `GAMEOPTION_NO_RELIGION`, unmet era or `CvPolicyBranchEntry::GetNumPolicyRequirement()` gates, and Lua `PlayerCanAdoptPolicyBranch`. STALE_OR_MISMATCH any guide pattern implying ideology branches open through normal branch selection is stale; audited source comments and code route ideology branches through direct `SetPolicyBranchUnlocked(...)`.

### First Ideology Branch Choice

- `CvPlayerPolicies::IsTimeToChooseIdeology()` excludes minor civs and barbarians, requires define-backed branches `POLICY_BRANCH_FREEDOM`, `POLICY_BRANCH_AUTOCRACY`, and `POLICY_BRANCH_ORDER`, and returns true by current era plus `IDEOLOGY_PREREQ_ERA` / `IDEOLOGY_UNLOCK_NUM_POLICY_BRANCHES_NEEDED` / `IDEOLOGY_UNLOCK_NUM_POLICIES_NEEDED`, by being later than `IDEOLOGY_START_ERA`, or by owning enough of a building whose `CvBuildingEntry::GetXBuiltTriggersIdeologyChoice()` threshold is met.

- `CvPolicyAI::DoChooseIdeology(...)` uses hard-coded global define IDs from `GD_INT_GET(POLICY_BRANCH_FREEDOM)`, `GD_INT_GET(POLICY_BRANCH_AUTOCRACY)`, and `GD_INT_GET(POLICY_BRANCH_ORDER)`. Under `MOD_EVENTS_IDEOLOGIES`, `CvPlayerPolicies::CanAdoptIdeology(PolicyBranchTypes)` can zero a branch through `GAMEEVENT_PlayerCanAdoptIdeology`. INFERRED `POLICY_BRANCH_HERITAGE` is an optional source path, not one of the three required branches, because `CvPolicyAI::DoChooseIdeology(...)` reads it and can select it only under `MOD_ISKA_HERITAGE`.

### Ideology Switching

- `CvPolicyAI::DoConsiderIdeologySwitch(...)` starts from the current ideology returned by `CvPlayerPolicies::GetLateGamePolicyTree()`. If the player has no late-game policy tree, it returns. Vassals can switch to their master's ideology. Otherwise human AI-diplomacy players return. AI players consider `CvPlayerCulture::GetPublicOpinionPreferredIdeology()` only when `CvPlayerCulture::GetPublicOpinionUnhappiness()` is nonzero, the preferred ideology is valid, and it differs from the current ideology.

## Scoring and inputs

Base policy weights are stored in `CvPolicyAI::m_PolicyAIWeights`. `CvPlayerPolicies::FlavorUpdate` calls `CvPlayerPolicies::AddFlavorAsStrategies`, which calls `CvPolicyAI::Reset` and `CvPolicyAI::AddFlavorWeights` for each positive flavor. `CvPolicyAI::AddFlavorWeights` multiplies each policy's `CvPolicyEntry::GetFlavorValue(eFlavor)` by the flavor weight, propagates some weight backward through prerequisites via `CvPolicyAI::WeightPrereqs` and `PropagateWeights`, then adds it to `m_PolicyAIWeights`.

`CvPolicyAI::WeighPolicy` starts with `m_PolicyAIWeights.GetWeight(ePolicy)`, then reads grand-strategy priorities for `AIGRANDSTRATEGY_CONQUEST`, `AIGRANDSTRATEGY_CULTURE`, `AIGRANDSTRATEGY_UNITED_NATIONS`, and `AIGRANDSTRATEGY_SPACESHIP`. It modifies those interests by `CvPlayerTraits` flags such as `IsWarmonger`, `IsExpansionist`, `IsNerd`, `IsDiplomat`, `IsSmaller`, `IsTourism`, and `IsReligious`.

`CvPolicyAI::WeighPolicy` loops all flavors on the policy and routes them into conquest, diplomacy, science, culture, or generic weight buckets by flavor type string. Military, air, naval, nuke, and similar flavors add conquest value; diplomacy, gold, trade, water connection, espionage, and related flavors add diplomacy value; science/spaceship add science value; great people, wonder, archaeology, and tourism-oriented effects add culture value; growth, happiness, infrastructure, religion, expansion, production, and tile-improvement flavors add generic weight.

`CvPolicyAI::WeightPolicyAttributes` is the large effect-to-yield estimator for individual policies and tenets. It returns `Firaxis::Array<int, NUM_YIELD_TYPES>` and evaluates many `CvPolicyEntry` fields against current player state, city state, map counts, unit/building availability, traits, corporations, religion-related policy fields, specialists, great works, improvements, resources, terrain, features, promotions, trade routes, wonders, unhappiness categories, and military/city-state state.

`CvPolicyAI::WeighPolicy` adds all estimated yields from `WeightPolicyAttributes` to the generic weight, then maps `YIELD_GREAT_GENERAL_POINTS` and `YIELD_GREAT_ADMIRAL_POINTS` into conquest value, `YIELD_TOURISM` into culture value, `YIELD_SCIENCE` into science value, and half of `YIELD_GOLD` plus half of `YIELD_PRODUCTION` into diplomacy value. Those category values are multiplied by grand-strategy interest and then added at full or half strength depending on current victory focus from `CvDiplomacyAI`.

`CvPolicyAI::WeighPolicy` gives tenets extra priority by multiplying policies with `CvPolicyEntry::GetLevel() > 0` by `level + 1`. It zeros corporation-specific policies if the player has not founded a corporation. It multiplies policies in an already-started/unlocked branch by the number of policies owned in that branch, penalizes older unopened branch policies, suppresses Patronage when there are no minor civs or few city-state friends, adds finisher weight when the policy would finish a branch, and returns `max(iWeight, 0)`.

`CvPolicyAI::WeighBranch` scores a branch by summing `WeighPolicy` for unowned policies in that branch, the branch's `CvPolicyBranchEntry::GetFreePolicy`, the branch's `GetFreeFinishingPolicy`, and sanity-checked buildings whose `CvBuildingEntry::GetPolicyBranchType` matches the branch. Building scoring delegates to the capital city's `CvBuildingProductionAI::CheckBuildingBuildSanity`.

`CvPolicyAI::DoChooseIdeology` scores ideology branches in stages: grand strategies, first/second adopter free tenets via `PolicyHelpers::GetNumFreePolicies`, relations to other major civs and their ideologies, tenet/branch weights from `WeighBranch / 25`, building-happiness potential from `GetBranchBuildingHappiness` and `GetNumHappinessPolicies`, public-opinion happiness if the empire is unhappy and another ideology already exists, and final multiplier gates.

Flavor input for the policy AI is wired at player init in `CvPlayer::init`, which adds `m_pPlayerPolicies` to `CvFlavorManager` with `CvFlavorManager::AddFlavorRecipient`. `CvPlayerPolicies::FlavorUpdate` rebuilds policy AI weights when flavor recipients receive updated flavor values.

`CvPlayerPolicies::AddFlavorAsStrategies` uses `CvFlavorManager::GetPersonalityIndividualFlavor` before Medieval era, and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy` from Medieval onward. It adds situational boosts: `FLAVOR_GOLD` if `ECONOMICAISTRATEGY_LOSING_MONEY`, `FLAVOR_HAPPINESS` if happiness is below current unhappiness, and `FLAVOR_DEFENSE`/`FLAVOR_CITY_DEFENSE` when the team is at war.

Victory-strategy inputs enter `CvPolicyAI::WeighPolicy` through `CvGrandStrategyAI::GetGrandStrategyPriority`, `CvDiplomacyAI::IsSeriousAboutVictory`, `IsGoingForWorldConquest`, `IsGoingForDiploVictory`, `IsGoingForSpaceshipVictory`, `IsGoingForCultureVictory`, and the corresponding `IsCloseTo*Victory` helpers. They determine whether conquest, diplomacy, science, and culture category values are added at full or half strength.

Ideology choice uses the same broad victory concepts but with different branch mapping. `CvPolicyAI::DoChooseIdeology` treats Freedom as diplomacy + technology + culture, Autocracy as diplomacy + conquest + culture, and Order as technology + conquest + culture. If one victory priority is clearly ahead by `IDEOLOGY_PERCENT_CLEAR_VICTORY_PREF`, it can zero the ideology least compatible with that victory path.

Civilization/leader trait inputs are pervasive in `CvPolicyAI::WeightPolicyAttributes` and `WeighPolicy`. Verified trait gates include `CvPlayerTraits::IsWarmonger`, `IsExpansionist`, `IsNerd`, `IsDiplomat`, `IsSmaller`, `IsTourism`, `IsReligious`, `IsNoAnnexing`, `GetExtraFoundedCityTerritoryClaimRange`, `GetExtraTenetsFirstAdoption`, `GetPolicyGEorGM`, and `IsAdoptionFreeTech`.

Prior belief choice is not directly inspected by normal policy scoring as individual `BeliefTypes`. Religion-related policy scoring mostly uses leader religious traits, policy religion fields, and broad religion state. Direct religion-state input appears in `CvPolicyAI::DoChooseIdeology`, where `MOD_ISKA_HERITAGE` checks `CvPlayerReligions::GetOwnedReligion` and `HasReligionInMostCities`; in `CvPlayerPolicies::CanUnlockPolicyBranch`, which reacts to `CvPolicyBranchEntry::IsLockedWithoutReligion` under no-religion game option; and in final policy effects such as `CvPlayer::doAdoptPolicy` triggering reformation belief choice through `CvPlayer::processPolicies`.

Diplomacy state affects normal policy scoring through `CvPolicyAI::WeighPolicy`, which checks minor-civ approaches when scoring `FLAVOR_DIPLOMACY`, and through victory-focus helpers on `CvDiplomacyAI`. Ideology choice is strongly diplomacy-aware: `DoChooseIdeology` considers vassal master ideology, team leader ideology and score, `GetBiggestCompetitor`, `GetMostValuableAlly`, `GetMostValuableFriend`, `GetCivApproach` values including `CIV_APPROACH_HOSTILE`, `CIV_APPROACH_GUARDED`, `CIV_APPROACH_AFRAID`, and `CIV_APPROACH_FRIENDLY`, and public-opinion pressure through `CvPlayerCulture`.

Game-state inputs include current era, policy count, ideology points, free policies, free tenets, culture, team war count, economic deficit strategy, happiness/unhappiness categories, city count, population, capital population, wonders, technologies known, corporations, city-state friends/allies, vassals, trade routes, great works, specialists, buildings, units, improvements, resources, plots, features, terrain, promotions, city revolt counters, and game options such as no religion/no policies/no science/no city-states.

Object flavor rows are cached through `CvDatabaseUtility::SetFlavors` in `CvGameCoreDLL_Expansion2/CvDatabaseUtility.cpp` and consumed by entry classes such as `CvUnitEntry`, `CvBuildingEntry`, `CvPolicyEntry`, `CvTechEntry`, `CvProjectEntry`, and `CvProcessInfo`.

- flavor interpretation is often hardcoded by string comparisons in consumers such as `CvPolicyAI::WeighPolicy`, `CvTechEntry::GetGrandStrategyPriority`, and `CvMilitaryAI` role helpers.

Scoring shape: `CvPolicyAI::AddFlavorWeights` multiplies `CvPolicyEntry::GetFlavorValue(eFlavor)` by current flavor weights and propagates some value backward through prerequisites using `WeightPrereqs` and `PropagateWeights`. At choice time, `WeighPolicy` starts from cached policy weight, adds hardcoded flavor-name buckets, grand-strategy/victory adjustments, `WeightPolicyAttributes` estimated yields, tenet level multipliers, branch continuation, finisher and branch-age logic, and returns `max(iWeight, 0)`. `WeighBranch` sums unowned branch policy weights, branch free policies, finisher policy value, and branch building value through `CvBuildingProductionAI::CheckBuildingBuildSanity`.

- base policy weights are cached in `CvPolicyAI::m_PolicyAIWeights`. `CvPlayerPolicies::FlavorUpdate()` calls `CvPlayerPolicies::AddFlavorAsStrategies(int)`, which resets `CvPolicyAI`, reads flavors, applies situational boosts, and calls `CvPolicyAI::AddFlavorWeights(FlavorTypes, int, int)`. `CvPolicyAI::AddFlavorWeights(...)` multiplies each policy's `CvPolicyEntry::GetFlavorValue(eFlavor)` by the active flavor weight, propagates weight backward through prerequisites via `CvPolicyAI::WeightPrereqs(...)` and `CvPolicyAI::PropagateWeights(...)`, then increases `m_PolicyAIWeights`.

- `CvPlayerPolicies::AddFlavorAsStrategies(...)` uses `CvFlavorManager::GetPersonalityIndividualFlavor(FlavorTypes)` before `ERA_MEDIEVAL` and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(FlavorTypes)` from Medieval onward. It adds direct boosts for `ECONOMICAISTRATEGY_LOSING_MONEY` into `FLAVOR_GOLD`, low happiness into `FLAVOR_HAPPINESS`, and war into `FLAVOR_DEFENSE` / `FLAVOR_CITY_DEFENSE`. `CvPolicyAI::WeighPolicy(...)` also rereads `CvPolicyEntry::GetFlavorValue(...)` and classifies flavor names into conquest, diplomacy, science, culture, and generic buckets with hardcoded string comparisons such as `FLAVOR_OFFENSE`, `FLAVOR_DIPLOMACY`, `FLAVOR_SCIENCE`, `FLAVOR_SPACESHIP`, `FLAVOR_WONDER`, and `FLAVOR_HAPPINESS`.

- Scoring components: `CvPolicyAI::WeighPolicy(CvPlayer*, PolicyTypes)` starts from `m_PolicyAIWeights.GetWeight(ePolicy)`, adds grand-strategy priorities for `AIGRANDSTRATEGY_CONQUEST`, `AIGRANDSTRATEGY_CULTURE`, `AIGRANDSTRATEGY_UNITED_NATIONS`, and `AIGRANDSTRATEGY_SPACESHIP`, modifies those interests with `CvPlayerTraits` flags such as `IsWarmonger()`, `IsExpansionist()`, `IsNerd()`, `IsDiplomat()`, `IsSmaller()`, `IsTourism()`, and `IsReligious()`, then adds policy flavor buckets, `CvPolicyAI::WeightPolicyAttributes(...)` yield estimates, victory-focus modifiers from `CvDiplomacyAI`, tenet-level multipliers, corporation gates, branch-continuation/old-branch modifiers, Patronage suppression, and finisher-policy weight.

### First Ideology Branch Choice

- `CvPolicyAI::DoChooseIdeology(...)` builds separate integer priorities for Freedom, Autocracy, and Order. It maps grand-strategy priorities so Freedom receives diplomacy + technology + culture, Autocracy receives diplomacy + conquest + culture, and Order receives technology + conquest + culture; normalizes them against `IDEOLOGY_SCORE_GRAND_STRATS`; adds first/second adopter free tenets through `PolicyHelpers::GetNumFreePolicies(...) * IDEOLOGY_SCORE_PER_FREE_TENET`; adjusts for other major civs' ideologies using `CvDiplomacyAI::GetBiggestCompetitor()`, `GetMostValuableAlly()`, `GetMostValuableFriend()`, and `GetCivApproach(...)`; adds `CvPolicyAI::WeighBranch(...) / 25` for purchase-by-level branches; adds branch building-happiness potential via `GetBranchBuildingHappiness(...)` and `GetNumHappinessPolicies(...)`; and, if the empire is unhappy and another ideology exists, adds public-opinion happiness terms from `CvPlayerCulture::ComputeHypotheticalPublicOpinionUnhappiness(...)`.

## Selection and control

`CvPolicyAI::ChooseNextPolicy` sorts `m_AdoptablePolicies` with `StableSortItems`, logs the possible choices, then selects with `CvWeightedVector<int>::ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getPolicyChoiceCutoffThreshold(), seed)`.

The normal policy/branch/tenet selection seed is `CvSeeder::fromRaw(0x5fef0474).mix(pPlayer->GetID()).mix(pPlayer->GetPlayerPolicies()->GetNumPoliciesOwned(false, false, true))` in `CvPolicyAI::ChooseNextPolicy`. This is threshold-random among high-scoring candidates, not guaranteed top-score selection.

`CvPolicyAI::DoChooseIdeology` does not use `CvWeightedVector` randomness. After applying overrides and scores, it chooses Freedom if its priority is at least both others, else Autocracy if it is at least both others, else Order. The tie order is therefore Freedom, then Autocracy, then Order.

`CvPolicyAI::DoConsiderIdeologySwitch` is conditional and deterministic. It can switch vassals to the master's ideology, or AI players to the public-opinion preferred ideology under super-unhappy, very-unhappy, or team-leader-pressure cases after happiness, city revolt, anti-flip-flop, and victory-path checks.

- `CvPolicyAI::WeighPolicy`, `WeighBranch`, `WeightPolicyAttributes`, `AddFlavorWeights`, `WeightPrereqs`, and `PropagateWeights`.

- `CvPlayerPolicies::AddFlavorAsStrategies` reads personality flavors before Medieval and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy` from Medieval onward; `CvPolicyAI::AddFlavorWeights` consumes `CvPolicyEntry::GetFlavorValue`; `CvPolicyAI::WeighPolicy` also hardcodes flavor-name buckets.

- `CvPolicyEntry`, `CvPolicyBranchEntry`, `Policy_Flavors`, policy prerequisites, branch metadata, `CvBuildingEntry`, `CvUnitEntry`, `CvImprovementEntry`, `CvResourceInfo`, and many effect fields cached by `CvPolicyEntry::CacheResults`.

- trigger: `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`.

- normal policy/branch/tenet candidate generation and selection: `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvPolicyAI::ChooseNextPolicy`; legality in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, `CvPlayerPolicies::CanAdoptPolicy` and `CanUnlockPolicyBranch`.

- scoring: `CvPolicyAI::AddFlavorWeights`, `WeighPolicy`, `WeightPolicyAttributes`, `WeighBranch`, `WeightPrereqs`, `PropagateWeights`; policy flavor recomputation in `CvPlayerPolicies::FlavorUpdate` and `AddFlavorAsStrategies`.

- ideology choice/switching: `CvPolicyAI::DoChooseIdeology`, `DoConsiderIdeologySwitch`; `CvPlayerPolicies::IsTimeToChooseIdeology`, `DoChooseIdeology`, `DoPolicyAI`, `DoSwitchIdeologies`.

- application: `CvPlayerPolicies::DoUnlockPolicyBranch`, `SetPolicyBranchUnlocked`; `CvPlayer::doAdoptPolicy`, `setHasPolicy`, `processPolicies`.

- data: `CvPolicyEntry`, `CvPolicyBranchEntry`, `Policy_Flavors`, `PolicyBranchTypes`, `Policies`, VP data under `(2) Vox Populi/Database Changes/Policies/` and `(2) Vox Populi/Database Changes/AI/PolicyFlavorSweeps.sql`.

## Application and state

- Application usually lives outside the selector: `CvGameReligions::FoundPantheon`, `CvGameReligions::FoundReligion`, `CvPlayer::doAdoptPolicy`, `CvPlayerPolicies::DoUnlockPolicyBranch`, `CvPlayerPolicies::SetPolicyBranchUnlocked`, `CvCity::pushOrder`, `CvCity::produce`, `CvCity::CreateUnit`, and `CvCity::CreateBuilding`.

- Application, branch opening: `CvPlayerPolicies::DoUnlockPolicyBranch(...)` validates `CanUnlockPolicyBranch(...)`, calls `SetPolicyBranchUnlocked(eBranchType, true, false)`, calls `DoSwitchToPolicyBranch(...)`, spends culture or a free policy, updates next policy cost, dirties UI, triggers `CvPlayerTraits::GetPolicyGEorGM()` handling, grants instant yield and great-person progress through `doInstantYield(...)` and `doInstantGreatPersonProgress(...)`, dirties city citizens, and fires Lua hook `PlayerAdoptPolicyBranch`.

- Application, policy/tenet: `CvPlayer::doAdoptPolicy(...)` validates `CvPlayer::canAdoptPolicy(...)`, spends a free tenet, free policy, or culture, calls `CvPlayer::setHasPolicy(...)`, updates next policy cost, ensures the branch is unlocked with `CvPlayerPolicies::SetPolicyBranchUnlocked(...)`, dirties UI, triggers trait great engineer/merchant handling, and continues into instant yields, great-person progress, Lua hook `PlayerAdoptPolicy`, and yield updates later in the function.

- Application side effects: `CvPlayer::setHasPolicy(...)` calls `CvPlayerPolicies::SetPolicy(...)` and `CvPlayer::processPolicies(...)`. `CvPlayerPolicies::SetPolicy(...)` updates owned/free policy arrays, policy counts, modifier cache, branch-finished state, finisher grants/removals, difficulty bonus, and branch-pick history. `CvPlayer::processPolicies(...)` applies or removes the `CvPolicyEntry` effects across player/city/unit/resource/trade/religion/corporation systems. reformation is a policy side effect: `CvPlayer::processPolicies(...)` checks `CvPolicyEntry::IsAddReformationBelief()` and can call `CvReligionAI::ChooseReformationBelief(...)` and `CvGameReligions::AddReformationBelief(...)`.

### First Ideology Branch Choice

- Application side effects: `CvPlayerPolicies::SetPolicyBranchUnlocked(...)` recognizes purchase-by-level branches through `CvPolicyBranchEntry::IsPurchaseByLevel()`. On ideology adoption it grants free tenets through `CvPlayer::ChangeNumFreeTenets(...)`, includes `CvPlayerTraits::GetExtraTenetsFirstAdoption()` on first ideology adoption, notifies other major civs, grants branch free policy through `CvPlayer::setHasPolicy(...)`, fires `GAMEEVENT_IdeologyAdopted` when not a revolution, records `CvPlayerCulture::SetTurnIdeologyAdopted(...)`, and completes ideology accomplishments for Freedom, Autocracy, or Order.

### Ideology Switching

- Application side effects: `CvPlayerPolicies::DoSwitchIdeologies(...)` fires `GAMEEVENT_IdeologySwitched`, computes retained free tenets from old tenets minus `SWITCH_POLICY_BRANCHES_TENETS_LOST`, clears the old branch with `ClearPolicyBranch(...)`, toggles old and new branches with `SetPolicyBranchUnlocked(...)`, recomputes public opinion with `CvPlayerCulture::DoPublicOpinion()`, records the switch turn, zeros culture through `CvPlayer::setJONSCultureTimes100(0)`, grants retained free tenets, fires Lua hook `PlayerAdoptPolicyBranch`, destroys buildings enabled by the old ideology branch or old-branch policies, and dirties the policies UI.

## Data and configuration

| Folder / data area | Relevant tables and rows | Current-source consumer |
|---|---|---|
| `(2) Vox Populi/Database Changes/AI/` | high-value AI data sweeps: `LeaderFlavorSweeps.sql`, `UnitFlavorSweeps.sql`, `BuildingFlavorSweeps.sql`, `PolicyFlavorSweeps.sql`, `TechFlavorSweeps.sql`, `ProjectFlavorChanges.sql`, `ProcessFlavorSweeps.sql`, `StrategyFlavorSweeps.sql`, `StrategyChanges.sql`, `CitySpecializationFlavorChanges.sql`, plus event flavor files. | Same flavor loaders plus object loaders: `CvUnitEntry::CacheResults`, `CvBuildingEntry::CacheResults`, `CvPolicyEntry::CacheResults`, `CvTechEntry::CacheResults`, `CvProjectEntry::CacheResults`, `CvProcessInfo::CacheResults` / process production cache. |
| `(2) Vox Populi/Database Changes/Policies/` | modifies `Policies`, `PolicyBranchTypes`, `Policy_PrereqPolicies`, and branch/tenet/policy effect fields in branch files such as `Authority.sql`, `Progress.sql`, `Fealty.sql`, `Statecraft.sql`, `Artistry.sql`, `Industry.sql`, `Imperialism.sql`, `Rationalism.sql`, `Freedom.sql`, `Order.sql`, `Autocracy.sql`, `PolicyTreeChanges.sql`, and `NewPolicies.xml`. | `CvPolicyEntry::CacheResults`, `CvPolicyBranchEntry::CacheResults`, `CvPlayerPolicies::CanAdoptPolicy`, `CanUnlockPolicyBranch`, `CvPolicyAI::ChooseNextPolicy`, `WeighPolicy`, `WeightPolicyAttributes`, and `WeighBranch` (`CvPolicyClasses.cpp`, `CvPolicyAI.cpp`). |

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvPolicyEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`) | `Policies`, `Policy_Flavors`, `Policy_PrereqPolicies`, many policy-effect tables/fields | Policy/tenet scoring, branch scoring, application side effects. |
| `CvPolicyBranchEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`) | `PolicyBranchTypes`, `PolicyBranch_Disables`, branch flags (`PurchaseByLevel`, mutual exclusion, delay/lock fields, free policies) | Branch selection, ideology branch state, branch unlock legality/application. |

Policy path is `Policies` / `Policy_Flavors` / `Policy_PrereqPolicies` / `PolicyBranchTypes` -> `CvPolicyEntry::CacheResults` and `CvPolicyBranchEntry::CacheResults` -> `CvPlayerPolicies::FlavorUpdate` / `AddFlavorAsStrategies` -> `CvPolicyAI::AddFlavorWeights` and `CvPolicyAI::WeighPolicy` / `WeighBranch` -> `CvPolicyAI::ChooseNextPolicy` -> `CvPlayerPolicies::DoUnlockPolicyBranch` or `CvPlayer::doAdoptPolicy`.

Ideology path reuses `PolicyBranchTypes` and `Policies` but has a separate scoring/selection function: `CvPolicyAI::DoChooseIdeology` uses global define IDs such as `POLICY_BRANCH_FREEDOM`, `POLICY_BRANCH_AUTOCRACY`, `POLICY_BRANCH_ORDER`, and under `MOD_ISKA_HERITAGE` `POLICY_BRANCH_HERITAGE`, then applies with `CvPlayerPolicies::SetPolicyBranchUnlocked`.

| Option / macro | Default in repo | Verified AI relevance |
|---|---:|---|
| `DIPLOAI_LIMIT_VICTORY_PURSUIT_RANDOMIZATION`, `DIPLOAI_PASSIVE_MODE`, `DIPLOAI_AGGRESSIVE_MODE`, `DIPLOAI_ENABLE_NUCLEAR_GANDHI`, `DIPLOAI_HONEST_OPINION_MODIFIERS` | mixed defaults in `NewCustomModOptions.xml` | RISK: diplomacy AI state feeds policy, ideology, belief, military, and production scoring through `CvDiplomacyAI`; verify each consumer before making current-behavior claims. |
| `MOD_EVENTS_*`, `MOD_EVENTS_IDEOLOGIES` | varies | policy and belief legality paths include Lua/game-event vetoes such as `PlayerCanAdoptPolicy`, `PlayerCanAdoptPolicyBranch`, `PlayerCanHaveBelief`, and `ReligionCanHaveBelief`; event option gates are sync/mod-hook sensitive. |

## Observability and validation obligations

`CvPolicyAI::LogPossiblePolicies`, `LogPolicyChoice`, `LogBranchChoice`, and `LogIdeologyChoice` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp` log to `PolicyAILog.csv` or `PolicyAILog_<player>.csv` when `GC.getLogging()` and `GC.getAILogging()` are enabled.

`CvPolicyAI::LogPolicyAttributeYields` logs per-yield components from `CvPolicyAI::WeightPolicyAttributes` to `PolicyAIYieldAttributesLog.csv` or a per-player split file under the same logging gates.

Script hooks and game events around policy choice/application include `PlayerCanAdoptPolicy`, `PlayerCanAdoptPolicyBranch`, `PlayerCanAdoptTenet`, `PlayerCanAdoptIdeology`, `PlayerAdoptPolicy`, `PlayerAdoptPolicyBranch`, `GAMEEVENT_IdeologyAdopted`, and `GAMEEVENT_IdeologySwitched`.

- `CvPolicyAI::LogIdeologyChoice` logs ideology scoring.

Logging: `CvPolicyAI::LogPossiblePolicies`, `LogPolicyChoice`, `LogBranchChoice`, and `LogIdeologyChoice` write policy/branch/ideology scoring logs. `LogPolicyAttributeYields` logs the per-yield output of `WeightPolicyAttributes`.

## Version differences, risks and unresolved questions

INFERRED: The exact threshold mechanics of `CvWeightedVector<int>::ChooseAbovePercentThreshold` should be documented in a shared scoring-infrastructure doc from `CvGameCoreDLLUtil/include/CvWeightedVector.h`. This file verifies usage and seed inputs but does not restate the template internals.

INFERRED: `CvPolicyAI::WeightPolicyAttributes` is broad enough that a future low-level policy-scoring appendix should enumerate every `CvPolicyEntry` field family and table source from `CvPolicyEntry::CacheResults` in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`. This pass summarizes the verified categories rather than reproducing every getter.

INFERRED: The `MOD_ISKA_HERITAGE` ideology path in `CvPolicyAI::DoChooseIdeology` chooses `POLICY_BRANCH_HERITAGE` from source when religion is dominant unless another ideology's priority is at least double the other two combined. This is verified as behavior, but whether Heritage should be treated as a normal ideology by all UI/diplomacy systems needs a separate pass.

Policy AI randomness is synchronization-sensitive. Changing candidate ordering, branch/policy ID offsets, scores, `PolicyChoiceCutoffThreshold`, `GetNumPoliciesOwned(false, false, true)`, or seed literal `0x5fef0474` in `CvPolicyAI::ChooseNextPolicy` can change deterministic AI choices.

Policy state is savegame-sensitive. `CvPlayerPolicies::Serialize` stores free/owned/one-shot policy arrays, branch unlocked/blocked/finished arrays, recent branch picks, latest flavor values, `CvPolicyAI`, and policy modifiers; `CvPlayer::Serialize` also stores free tenets, ideology points, and many policy effect counters.

Open enum and database assumptions are high risk. Candidate and scoring loops cast integer indices to `PolicyTypes`, `PolicyBranchTypes`, `FlavorTypes`, `YieldTypes`, `BuildingClassTypes`, `UnitClassTypes`, `ImprovementTypes`, `ResourceTypes`, `PlotTypes`, `FeatureTypes`, `TerrainTypes`, `DomainTypes`, `SpecialistTypes`, `GreatPersonTypes`, `PromotionTypes`, and `AIGrandStrategyTypes`.

`CvPolicyAI::ChooseNextPolicy` encodes branches and policies in the same integer vector by adding `GC.getNumPolicyBranchInfos()` to policy IDs. Any change to branch count, enum ordering, or interpretation of returned values must preserve this split.

Ideology application and switching have broad side effects: free tenets, public-opinion turn state, notifications to other players, Lua/game events, culture reset on switch, old-branch building deletion, city citizen dirtying, trait-triggered great people/techs, and reformation belief selection from policy effects.

Flavor updates are cached and serialized. Changes to `CvPlayerPolicies::AddFlavorAsStrategies`, `CvPolicyAI::AddFlavorWeights`, propagation constants such as `POLICY_WEIGHT_PROPAGATION_PERCENT`, `POLICY_WEIGHT_PROPAGATION_LEVELS`, and `POLICY_WEIGHT_PERCENT_DROP_NEW_BRANCH` can affect long-term AI policy priorities across saves.

`CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, and `AddReformationBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`; `CvPlayer::doAdoptPolicy` and `CvPlayer::processPolicies` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`; and `CvPlayerPolicies::SetPolicyBranchUnlocked`, `DoUnlockPolicyBranch`, and `DoSwitchIdeologies` in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp` are broad side-effect hubs.

RISK: These functions touch notifications, Lua/game events, traits, city/player state, caches, free policies/tenets/techs, religion state, and serialized data.

- RISK `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`: many systems read this directly instead of recipient latest flavors. Scale or blending changes affect policy, ideology, specialization, operations, military/economic helpers, tech/building/religion-adjacent valuation, and diplomacy-adjacent scoring.

- RISK policy application functions (`CvPlayer::doAdoptPolicy`, `CvPlayer::setHasPolicy`, `CvPlayer::processPolicies`, `CvPlayerPolicies::SetPolicyBranchUnlocked`, `CvPlayerPolicies::DoSwitchIdeologies`): side effects include free tenets, culture spending/reset, public opinion, buildings, instant yields, Lua/game events, policy modifier caches, branch state, and reformation belief selection.

- Source functions: `CvGameCoreDLL_Expansion2/CustomMods.h` defines `MOD_AI_UNIT_PRODUCTION`; `(1) Community Patch/Database Changes/NewCustomModOptions.xml` sets `AI_UNIT_PRODUCTION = 0`. C++ references role flavors in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp` helpers such as `MilitaryAIHelpers::IsTestStrategy_NeedArcherUnits(...)`, `IsTestStrategy_NeedSiegeUnits(...)`, `IsTestStrategy_NeedSkirmisherUnits(...)`, and naval role helpers; `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp`, `CvUnitProductionAI::CheckUnitBuildSanity(...)`; `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvPolicyAI::WeighPolicy(...)`; and `CvGameCoreDLL_Expansion2/CvTechClasses.cpp`, `CvTechEntry::GetGrandStrategyPriority(...)`. repository search under `(1) Community Patch/Database Changes/` and `(2) Vox Populi/Database Changes/` found no rows for `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, or `FLAVOR_FIGHTER`.

- Affected decision: Policy, branch, and tenet selection.

- Source functions: `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, `CvPlayerPolicies::AddFlavorAsStrategies(...)` resets `CvPolicyAI`, uses personality before Medieval and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` from Medieval onward, adds direct gold/happiness/war boosts, and calls `CvPolicyAI::AddFlavorWeights(...)`. `CvPlayerPolicies::LogFlavorChange(...)` is an empty function with comment "Now using personality flavors, so this is unnecessary (or is it?)". `CvPolicyAI::LogPossiblePolicies(...)` and `LogPolicyAttributeYields(...)` log later candidate scores/components.

- Smallest possible patch point: `CvPlayerPolicies::LogFlavorChange(...)` or a new diagnostic-only section in `CvPlayerPolicies::AddFlavorAsStrategies(...)` that logs final `iFlavorValue` and source category without changing weights.

- Risk of double-counting or unintended downstream effects: RISK logging-only changes have low behavior risk but can be noisy. Behavior changes in `AddFlavorAsStrategies(...)` are high risk because `CvPolicyAI::AddFlavorWeights(...)`, `WeightPrereqs(...)`, `PropagateWeights(...)`, and `WeighPolicy(...)` already stack flavor-derived and direct strategy-derived terms.

- Source functions: atlas maps show `CvPolicyAI::WeighPolicy(...)` buckets `FLAVOR_ESPIONAGE` with diplomacy-adjacent policy value; `CvCityStrategyAI.cpp` and `CvEconomicAI.cpp` contain espionage-flavor threshold references; object flavor consumers include `Policy_Flavors`, `Building_Flavors`, and `Technology_Flavors`.

- Lowest-risk investigation: Focused source pass over espionage AI, spy pressure metrics, espionage building scoring in `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, policy espionage fields in `CvPolicyAI::WeightPolicyAttributes(...)`, and `Technology_Flavors` consumers.

Patch-risk notes: RISK: The branch/policy integer packing in `ChooseNextPolicy`, `PolicyTypes` and `PolicyBranchTypes` ordering, policy prerequisite propagation constants, seeded threshold randomness, `CvPlayerPolicies::Serialize`, ideology switch side effects, and broad `CvPlayer::processPolicies` effects are high risk. STALE_OR_MISMATCH: `CvPolicyAI::WeighPolicy` contains always-compiled checks for role flavor names associated with `MOD_AI_UNIT_PRODUCTION`; repository data does not define all of those flavor rows, so those checks can be inert without external data.

- source classes and entries include `CvPolicyAI`, `CvPlayerPolicies`, `CvPolicyEntry`, `CvPolicyBranchEntry`, `CvPolicyXMLEntries`, `CvPlayer`, `CvPlayerTraits`, `CvPlayerCulture`, `CvGrandStrategyAI`, `CvDiplomacyAI`, `CvBuildingEntry`, `CvBuildingClassInfo`, and `CvWeightedVector<int>`. `CvPolicyEntry::CacheResults(...)` reads `Policies` columns including `Level`, `PolicyBranchType`, `TechPrereq`, `IdeologyPoint`, `AddReformationBelief`, many effect fields used by `WeightPolicyAttributes(...)`, and `Policy_Flavors` via `CvDatabaseUtility::SetFlavors(...)`. `CvPolicyBranchEntry::CacheResults(...)` reads `PolicyBranchTypes` fields including `EraPrereq`, `FreePolicy`, `FreeFinishingPolicy`, `FirstAdopterFreePolicies`, `SecondAdopterFreePolicies`, `PurchaseByLevel`, `LockedWithoutReligion`, `AIMutuallyExclusive`, `AIDelayNoReligion`, `AIDelayNoCulture`, `AIDelayNoCityStates`, `AIDelayNoScience`, `NumPolicyRequirement`, and `PolicyBranch_Disables`.

- Database/source-data references: VP policy content and tenet levels are updated under `(2) Vox Populi/Database Changes/Policies/`, including `Freedom.sql`, `Order.sql`, `Autocracy.sql`, `NewPolicies.xml`, and `PolicyTreeChanges.sql`. VP policy flavors are rewritten in `(2) Vox Populi/Database Changes/AI/PolicyFlavorSweeps.sql`; CP also touches policy flavors in `(1) Community Patch/Database Changes/AI/FlavorChanges.sql`. define-backed constants used by this decision are declared/cached in `CvGameCoreDLL_Expansion2/CvGlobals.cpp` and configured in `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql`, including `POLICY_WEIGHT_PROPAGATION_PERCENT`, `POLICY_WEIGHT_PROPAGATION_LEVELS`, `MAX_NUM_TENETS_LEVEL_1`, `MAX_NUM_TENETS_LEVEL_2`, `MAX_NUM_TENETS_LEVEL_3`, `IDEOLOGY_START_ERA`, `IDEOLOGY_PREREQ_ERA`, `IDEOLOGY_UNLOCK_NUM_POLICY_BRANCHES_NEEDED`, `IDEOLOGY_UNLOCK_NUM_POLICIES_NEEDED`, `IDEOLOGY_PERCENT_CLEAR_VICTORY_PREF`, ideology score constants, `SWITCH_POLICY_BRANCHES_ANARCHY_TURNS`, `SWITCH_POLICY_BRANCHES_TENETS_LOST`, and branch define IDs `POLICY_BRANCH_FREEDOM`, `POLICY_BRANCH_AUTOCRACY`, `POLICY_BRANCH_ORDER`, and `POLICY_BRANCH_HERITAGE`.

- Enums and ID assumptions: `PolicyTypes`, `PolicyBranchTypes`, and `FlavorTypes` are open enums in `CvGameCoreDLLUtil/include/CvEnums.h`; `NO_POLICY`, `NO_POLICY_BRANCH_TYPE`, and `NO_FLAVOR` are the only compiled enum values shown there. RISK policy, branch, flavor, building, yield, improvement, resource, specialist, promotion, and grand-strategy IDs are database-backed and array-indexed, so enum ordering and database row changes are savegame- and determinism-sensitive.

- `CvPolicyAI::LogPossiblePolicies()`, `LogPolicyChoice(PolicyTypes)`, `LogBranchChoice(PolicyBranchTypes)`, `LogIdeologyChoice(CvString&, int, int, int)`, and `LogPolicyAttributeYields(...)` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp` log under `GC.getLogging()` and `GC.getAILogging()`. Policy and ideology choices go to `PolicyAILog.csv` or `PolicyAILog_<player>.csv` through `CvPolicyAI::GetLogFileName(...)`; attribute-yield components go to `PolicyAIYieldAttributesLog.csv` or a per-player split file.

- Risk notes: RISK normal policy selection is synchronization-sensitive because it uses threshold-random selection with seed `0x5fef0474`, player ID, and owned policy count; changes to candidate ordering, branch/policy integer packing, weights, `PolicyChoiceCutoffThreshold`, or database ordering can alter deterministic AI outcomes. RISK policy state is savegame-sensitive: `CvPlayerPolicies::Serialize(...)` stores free/owned/one-shot policy arrays, branch unlocked/blocked/finished arrays, recent branch picks, latest flavor values, `CvPolicyAI`, and policy modifier caches; `CvPlayer` also serializes free tenets, ideology points, culture, anarchy, and policy-derived counters. RISK ideology adoption/switching has broad side effects: free tenets, public-opinion turn state, notifications, Lua/game events, culture reset on switch, old-branch building deletion, trait-triggered great people/free techs, and reformation belief selection through policy effects.

- Known improvement hooks: DESIGN_HYPOTHESIS split a future low-level scoring appendix around `CvPolicyAI::WeightPolicyAttributes(...)`, grouped by `CvPolicyEntry::CacheResults(...)` field families and source tables. OPEN_QUESTION document `CvWeightedVector<int>::ChooseAbovePercentThreshold(...)` once in a shared scoring infrastructure page so policy, belief, production, and wonder records can cite the common selection semantics. OPEN_QUESTION verify whether `POLICY_BRANCH_HERITAGE` should be treated as a full ideology in diplomacy, UI, public opinion, and save/migration docs; this record only verifies the `MOD_ISKA_HERITAGE` branch selection path in `CvPolicyAI::DoChooseIdeology(...)`.

## Source and dependency lookup

GUIDE_PATTERN: The guide's candidate generation -> scoring -> weighted/thresholded selection -> application pattern matches `CvPolicyAI::ChooseNextPolicy` for normal policies, branches, and tenets. Ideology branch choice is not threshold-random; it is a direct max-priority choice after explicit score stages and overrides.

Unresolved questions for focused pass:
- Fully trace `CvPlayerPolicies::DoChooseNextPolicy()` trigger timing and human/AI split.
- Map `DoChooseIdeology(...)` branch priorities after the initial vassal/team-leader cases.
- Separate normal policy branches from ideology branches and tenets by `CvPolicyEntry::GetLevel()` and branch metadata.
- Audit `CvPlayer::doAdoptPolicy(...)` side effects and save/sync-sensitive policy state.

- `CvPolicyAI` and `CvPlayerPolicies` in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp` and `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`: policy, branch, tenet, ideology choice and application state.

- Durable player intent lives in player managers such as `CvEconomicAI`, `CvMilitaryAI`, `CvGrandStrategyAI`, `CvDiplomacyAI`, `CvReligionAI`, `CvPolicyAI`, and `CvCitySpecializationAI`.

| Class / manager | Decisions affected |
|---|---|
| `CvPlayer` (`CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer.h`) | turn orchestration, best-wonder-city flags, policy application, first-city pantheon trigger, operation ownership, army ownership, manager accessors, serialization. Affects belief selection, policy/ideology selection, wonder choice, city production, operation unit demand, emergency purchases, and flavor propagation. |
| `CvPolicyAI` (`CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvPolicyAI.h`) | normal policy/branch/tenet selection, ideology choice, ideology switching, policy scoring and logging. Affects policy selection and ideology selection. |
| `CvPlayerPolicies` (`CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, `CvPolicyClasses.h`) | policy trigger/state/application routing, branch unlocks, ideology switching, flavor recipient for policy weights, serialization. Affects policy selection, ideology selection, reformation side effects, and flavor propagation. |
| `CvGrandStrategyAI` (`CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp`, `.h`) | grand-strategy priorities and `GetPersonalityAndGrandStrategy(...)`. Affects belief scoring, policy/ideology scoring, city specialization, operation/army unit weights, unit/building scoring helpers, and flavor-adjacent valuation. |
| `CvDiplomacyAI` (`CvGameCoreDLL_Expansion2/CvDiplomacyAI.cpp`, `.h`) | victory pursuit, public opinion, war/approach/opinion state. Affects ideology choice/switching, policy scoring, belief scoring, military strategy, unit/building production, and city specialization. |

| Function / enum / table | Decisions affected |
|---|---|
| `CvPolicyAI::WeighPolicy(...)`, `WeightPolicyAttributes(...)`, `WeighBranch(...)`, `AddFlavorWeights(...)`, `WeightPrereqs(...)`, `PropagateWeights(...)` (`CvPolicyAI.cpp`) | policy/tenet/branch scoring and prerequisite propagation. |
| `CvPlayerPolicies::DoPolicyAI()`, `DoChooseIdeology()`, `DoUnlockPolicyBranch(...)`, `SetPolicyBranchUnlocked(...)`, `DoSwitchIdeologies(...)` (`CvPolicyClasses.cpp`) | policy/ideology trigger and application routing. |
| `CvPlayer::doAdoptPolicy(...)`, `setHasPolicy(...)`, `processPolicies(...)` (`CvPlayer.cpp`) | policy/tenet application and broad side effects, including reformation belief trigger. |
| `CvGrandStrategyAI::DoTurn()`, `GetPersonalityAndGrandStrategy(...)`, `GetGrandStrategyPriority(...)` (`CvGrandStrategyAI.cpp`) | grand-strategy state and modified flavor inputs for policy, ideology, belief, specialization, production, and military helpers. |
| `CvDiplomacyAI::DoTurn(...)`, `IsGoingFor*Victory()`, `IsCloseTo*Victory()`, `GetVictoryDisputeLevel(...)`, `GetVictoryBlockLevel(...)`, `GetBoldness()`, `GetWonderCompetitiveness()` (`CvDiplomacyAI.cpp/.h`) | direct strategic/diplomatic inputs to ideology, policies, beliefs, grand strategy, military, and production scoring. |

- consumers: `CvPlayerPolicies::FlavorUpdate`, `CvWonderProductionAI::FlavorUpdate`, `CvCityStrategyAI::FlavorUpdate`, `CvReligionAI` direct personality-flavor reads, `CvPolicyAI` hardcoded flavor-name buckets.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` | policy weights, ideology scoring, city specialization, operation/army weights, military/economic helper thresholds, religion/production valuation. | RISK changes are global preference-scale changes, not isolated to grand strategy. |

- DESIGN_HYPOTHESIS future reusable appendices should split large estimators by boundary: `CvPolicyAI::WeightPolicyAttributes(...)`, `CvUnitProductionAI::CheckUnitBuildSanity(...)`, `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, and `CvReligionAI::ScoreBelief(...)`.

- Keep the selection/application split for beliefs and policies. `CvReligionAI::Choose*Belief(...)` selects `BeliefTypes`; `CvGameReligions::FoundPantheon(...)`, `FoundReligion(...)`, `EnhanceReligion(...)`, and `AddReformationBelief(...)` apply them. `CvPolicyAI::ChooseNextPolicy(...)` selects; `CvPlayer::doAdoptPolicy(...)`, `CvPlayerPolicies::DoUnlockPolicyBranch(...)`, and `SetPolicyBranchUnlocked(...)` apply.

- Reject/defer espionage/counter-intelligence behavior changes. The atlas only maps scattered consumers (`CvPolicyAI::WeighPolicy(...)`, `CvCityStrategyAI.cpp`, `CvEconomicAI.cpp`, object flavor tables); it does not yet map a complete upstream spy-pressure producer or runtime failure mode.

- RISK: The atlas cites many `Cv*Entry::CacheResults(...)` and SQL table inputs, but formula-level field coverage is incomplete for `CvBeliefEntry`, `CvPolicyEntry`, `CvBuildingEntry`, and `CvUnitEntry`. Do not treat the current atlas as a complete scoring formula appendix.

`CvPolicyAI` is declared in `CvGameCoreDLL_Expansion2/CvPolicyAI.h` and implemented in `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`. `CvPlayerPolicies` owns the turn trigger and application routing in `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`.

Inputs consumed: Candidate legality is delegated to `CvPlayerPolicies::CanAdoptPolicy`, `CanUnlockPolicyBranch`, `IsPolicyBranchUnlocked`, and `CvPolicyAI::IsBranchEffectiveInGame`. Scoring consumes `CvPolicyEntry`, `CvPolicyBranchEntry`, policy flavor rows from `Policy_Flavors`, prerequisites, player culture/free-policy/free-tenet state, era/tech state, city/building/unit/resource/specialist/religion state, corporations, traits, `CvGrandStrategyAI`, and `CvDiplomacyAI` victory focus.

Selection mechanism: `ChooseNextPolicy` stores branches as raw `PolicyBranchTypes` integers and policies/tenets as `PolicyTypes + GC.getNumPolicyBranchInfos()` in one `CvWeightedVector<int>`. It uses `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getPolicyChoiceCutoffThreshold(), CvSeeder::fromRaw(0x5fef0474).mix(player ID).mix(GetNumPoliciesOwned(false, false, true)))`. `DoChooseIdeology` does not use `CvWeightedVector`; after overrides and scoring, it chooses max priority with tie order Freedom, then Autocracy, then Order.

Policy selection is player-level in timing but manager-level in scoring. `CvPlayer::doTurnPostDiplomacy()` triggers `CvPlayerPolicies::DoChooseIdeology()` and `CvPlayerPolicies::DoPolicyAI()`; normal policy/branch/tenet candidate scoring is in `CvPolicyAI::ChooseNextPolicy(...)`, not in `CvPlayer`. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`; `CvPolicyClasses.cpp`, `CvPlayerPolicies::DoPolicyAI`; `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp`, `CvPolicyAI::ChooseNextPolicy`.

Policy final application is shared. `CvPlayer::doAdoptPolicy(...)` and `CvPlayer::setHasPolicy(...)` are state-changing application paths used after AI selection and by UI/network responses documented in `docs-ai/ai-policies.md`. `CvPlayer::processPolicies(...)` applies the actual `CvPolicyEntry` effects and can trigger reformation belief choice/application for AI players. Source: `CvPlayer.cpp`, `doAdoptPolicy`, `setHasPolicy`, `processPolicies`; `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp`, `ResponseAdoptPolicy`, `ResponseAdoptPolicyBranch`.

`CvPlayer::GetDiplomacyAI()` -> victory focus, war/approach/public-opinion inputs, diplomacy turn processing, policy and ideology scoring inputs.

`CvPlayer::GetPlayerPolicies()` -> policy/ideology selection state, policy modifier cache, branch/tenet application state.

- Attribute scoring: `CvPolicyAI::WeightPolicyAttributes(CvPlayer*, PolicyTypes)` is the large effect estimator for `CvPolicyEntry` fields loaded by `CvPolicyEntry::CacheResults(...)`. It returns `Firaxis::Array<int, NUM_YIELD_TYPES>` and evaluates current player, city, unit, building, resource, improvement, specialist, great work, corporation, religion, unhappiness, trade, and military/city-state state. INFERRED this function is the main formula appendix boundary; this decision record cites the function but does not enumerate every `CvPolicyEntry` field.

- Branch scoring: `CvPolicyAI::WeighBranch(CvPlayer*, PolicyBranchTypes)` sums `WeighPolicy(...)` for unowned policies in the branch, adds `CvPolicyBranchEntry::GetFreePolicy()` and `GetFreeFinishingPolicy()`, and adds branch-linked building value by checking `CvBuildingEntry::GetPolicyBranchType()` and delegating to the capital city's `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`.

- normal policies, normal branches, and tenets are packed into one `CvWeightedVector<int> m_AdoptablePolicies`: branch IDs are stored directly as `PolicyBranchTypes` integer values, while policies/tenets are stored as `PolicyTypes + GC.getNumPolicyBranchInfos()`. `CvPolicyAI::ChooseNextPolicy(...)` calls `m_AdoptablePolicies.StableSortItems()` and then `CvWeightedVector<int>::ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getPolicyChoiceCutoffThreshold(), seed)`. The seed is `CvSeeder::fromRaw(0x5fef0474).mix(pPlayer->GetID()).mix(pPlayer->GetPlayerPolicies()->GetNumPoliciesOwned(false, false, true))`.

- Selection semantics: this is threshold-random weighted selection, not deterministic top-score selection. The weighted-vector implementation is in `CvGameCoreDLLUtil/include/CvWeightedVector.h`; candidate order, candidate weights, `HandicapInfos.PolicyChoiceCutoffThreshold`, and the seed inputs all affect the chosen result.

### First Ideology Branch Choice

- Overrides before scoring: vassals are forced to the master's existing late-game policy tree by `CvPolicyAI::DoChooseIdeology(...)` calling `CvPlayerPolicies::SetPolicyBranchUnlocked(...)` and returning. Team-leader following can also bypass normal scoring if the leader has an ideology and a public-opinion unhappiness sanity check says following will not create severe unhappiness.

- Victory compatibility: when all three ideology multipliers remain nonzero, `CvPolicyAI::DoChooseIdeology(...)` can rule out the ideology least compatible with a clearly preferred victory path using `IDEOLOGY_PERCENT_CLEAR_VICTORY_PREF`. Conquest can zero Freedom, diplomacy can zero Order, and technology can zero Autocracy.

- first ideology branch selection is deterministic max-priority selection, not `CvWeightedVector` selection. `CvPolicyAI::DoChooseIdeology(...)` chooses Freedom if its priority is greater than or equal to both others, else Autocracy if its priority is greater than or equal to both others, else Order. Tie order is therefore Freedom, then Autocracy, then Order. Under `MOD_ISKA_HERITAGE`, if the player owns a religion present in most cities, `POLICY_BRANCH_HERITAGE` can replace the chosen branch unless Freedom, Order, or Autocracy has at least double the sum of the other two standard priorities.

### Ideology Switching

- Switching heuristics: `CvPolicyAI::DoConsiderIdeologySwitch(...)` is deterministic and conditional. It checks super-unhappy and very-unhappy state through `CvPlayer::IsEmpireSuperUnhappy()` / `IsEmpireVeryUnhappy()`, hypothetical public-opinion unhappiness through `CvPlayerCulture::ComputeHypotheticalPublicOpinionUnhappiness(...)`, team-leader pressure, city revolt counter, recent switch turn anti-flip-flop checks using `CvPlayerCulture::GetTurnIdeologySwitch()` and `GC.getGame().getGameTurn()`, and victory-path compatibility using `CvGrandStrategyAI::GetConquestPriority()`, `GetUnitedNationsPriority()`, `GetSpaceshipPriority()`, `GetCulturePriority()`, and `IDEOLOGY_PERCENT_CLEAR_VICTORY_PREF`.

- ideology switching does not score all ideology branches with `CvWeightedVector`. It either switches to the master's ideology for vassal forcing or to `CvPlayerCulture::GetPublicOpinionPreferredIdeology()` after the happiness, team-leader, revolt, anti-flip-flop, and victory-path checks pass.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
