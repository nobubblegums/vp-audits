# Belief selection

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../source-snapshots.md#recovered-baseline>).

## Decision or contract question

Which pantheon, founder, follower, enhancer, bonus or reformation belief should be selected?

The six belief categories share scoring and weighted-selection infrastructure but have distinct triggers, category legality and seed contexts. Direct personality reads differ from propagated latest flavor state. Shared religion application is documented in mechanics.

## Reference synthesis

The category choosers use availability lists, shared `ScoreBelief` helpers and thresholded weighted selection, but their triggers differ. Pantheons can be chosen during player religion processing or trait-start founding; founder/follower/bonus and enhancement choices enter through prophet missions; reformation can enter through turn or policy application. Human choices use UI/network validation before the shared mechanics path.

Scoring combines plot, owned-city, potential-city and player components, with special trait-start scoring and civilization/bonus adjustments. Personality reads bypass propagated latest state. The reports include yield-to-flavor scale factors and category-specific seeds below; these are older-baseline findings, not current runtime values. Full belief-field and secondary-table enumeration remains unfinished.

Canonical related owners: [Religion founding and belief application](<../../mechanics/religion/founding-and-beliefs.md>), [Flavor production, propagation and consumption](<../infrastructure/flavors.md>), [Shared weighted selection and seeded draws](<../infrastructure/weighted-selection.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

Pantheon turn entry is `CvGameReligions::DoPlayerTurn` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`. It checks faith, major-civ status, era cutoff, `CvGameReligions::CanCreatePantheon`, and human/AI split; AI calls `CvReligionAI::ChoosePantheonBelief` and applies with `CvGameReligions::FoundPantheon`.

Trait-start pantheon entry is `CvPlayer::foundCity` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp`. For a major civ's first city with `CvPlayerTraits::StartsWithPantheon`, it grants enough faith, calls `CanCreatePantheon`, then either notifies a human or calls `CvReligionAI::ChoosePantheonBelief` and `CvGameReligions::FoundPantheon`.

AI founding entry is `CvHomelandAI::ExecuteProphetMoves` in `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp`, which calls `CvUnit::CanFoundReligion` and `CvUnit::DoFoundReligion`. The actual AI belief choices are in `CvUnit::DoFoundReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp`.

AI enhancement entry is the same great prophet homeland path in `CvHomelandAI::ExecuteProphetMoves` in `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp`, calling `CvUnit::CanEnhanceReligion` and `CvUnit::DoEnhanceReligion`. `CvUnit::DoEnhanceReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp` chooses one follower belief with `CvReligionAI::ChooseFollowerBelief` and one enhancer with `CvReligionAI::ChooseEnhancerBelief`.

Reformation entry appears in two places: `CvGameReligions::DoPlayerTurn` checks owned religion plus `CvPlayerPolicies::HasPolicyGrantingReformationBelief` or `CvPlayer::IsReformation`, and `CvPlayer::doAdoptPolicy` checks `CvPolicyEntry::IsAddReformationBelief`. Non-human players call `CvReligionAI::ChooseReformationBelief` and apply with `CvGameReligions::AddReformationBelief`.

Human/UI/network paths share final application but not AI selection. `CvDllNetMessageHandler::ResponseFoundReligion` and `CvDllNetMessageHandler::ResponseEnhanceReligion` in `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp` validate with `CvGameReligions::CanFoundReligion` or `CanEnhanceReligion`, then call `FoundReligion` or `EnhanceReligion`.

- tactical and military purchase paths use `CvTacticalAI`, `CvMilitaryAI`, `CvAIOperation`, `CvEconomicAI`, and `CvReligionAI`.

- `CvTacticalAI::PlotEmergencyPurchases` during tactical dominance-zone processing; `CvMilitaryAI::MakeEmergencyPurchases`; `CvAIOperation::BuyFinalUnit`; economic hurry/purchase logic; faith purchase logic in `CvReligionAI`.

Decision role: `CvReligionAI` is the scorer/selector for AI belief IDs. It does not apply the selected belief. Application is in `CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, and `AddReformationBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`; prophet entry points are `CvUnit::DoFoundReligion` and `CvUnit::DoEnhanceReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp`.

- AI choice of pantheon, founder, follower, enhancer, bonus, and reformation beliefs. All six category choosers are in `CvReligionAI` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` and share the same high-level shape: get available beliefs from `CvGameReligions`, score each `CvBeliefEntry` with `CvReligionAI::ScoreBelief(...)`, sort a `CvWeightedVector<BeliefTypes>`, then choose with `CvWeightedVector<BeliefTypes>::ChooseAbovePercentThreshold(...)`.

- selection is owned by `CvReligionAI` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`: `ChoosePantheonBelief(PlayerTypes)`, `ChooseFounderBelief(PlayerTypes, ReligionTypes)`, `ChooseFollowerBelief(PlayerTypes, ReligionTypes)`, `ChooseEnhancerBelief(PlayerTypes, ReligionTypes)`, `ChooseBonusBelief(PlayerTypes, ReligionTypes, int, int, int)`, and `ChooseReformationBelief(PlayerTypes, ReligionTypes)`. religion state and availability/application are owned by `CvGameReligions`, `CvPlayerReligions`, `CvReligion`, and `CvReligionBeliefs` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` and `CvGameCoreDLL_Expansion2/CvReligionClasses.h`.

- pantheon choice is reached from `CvGameReligions::DoPlayerTurn(CvPlayer&)` after faith gain when `CanCreatePantheon(...) == FOUNDING_OK`; non-human players call `CvReligionAI::ChoosePantheonBelief(...)` and then `CvGameReligions::FoundPantheon(...)`. trait-start pantheon choice is also reached from `CvPlayer::foundCity(...)` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp` for major civs with `CvPlayerTraits::StartsWithPantheon()`. founder and first follower choices are reached from the great prophet mission path `CvUnit::DoFoundReligion()` in `CvGameCoreDLL_Expansion2/CvUnit.cpp`; if the AI lacks a pantheon, that function also chooses a pantheon first. enhancement choices are reached from `CvUnit::DoEnhanceReligion()` in `CvUnit.cpp`, which chooses one follower and one enhancer belief. reformation choice is reached from `CvGameReligions::DoPlayerTurn(...)` when the player owns a religion, has not reformed, and `CvPlayerPolicies::HasPolicyGrantingReformationBelief()` or `CvPlayer::IsReformation()` is true; it is also reached immediately during `CvPlayer::doAdoptPolicy(PolicyTypes)` when the adopted `CvPolicyEntry::IsAddReformationBelief()` policy grants reformation.

## Candidates and gates

`CvGameReligions::GetAvailablePantheonBeliefs` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` loops `CvBeliefXMLEntries` from `GC.GetGameBeliefs()`, requires `CvBeliefEntry::IsPantheonBelief`, filters already-used beliefs unless `MOD_BALANCE_ANY_PANTHEON` is enabled, applies `GAMEEVENT_PlayerCanHaveBelief` when `MOD_EVENTS_ACQUIRE_BELIEFS` is enabled, and enforces `CvBeliefEntry::GetRequiredCivilization` plus `CvPlayerTraits::IsUniqueBeliefsOnly`.

`CvGameReligions::GetAvailableFounderBeliefs`, `GetAvailableFollowerBeliefs`, `GetAvailableEnhancerBeliefs`, and `GetAvailableReformationBeliefs` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` have the same broad shape: loop all belief entries, require the relevant `CvBeliefEntry::Is*Belief` flag, reject beliefs already in some religion with `CvGameReligions::IsInSomeReligion`, apply `GAMEEVENT_ReligionCanHaveBelief` under `MOD_EVENTS_ACQUIRE_BELIEFS`, and enforce unique-civilization requirements.

`CvGameReligions::GetAvailableBonusBeliefs` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` is broader than the normal category choosers. It accepts enhancer, follower, founder, or pantheon beliefs, then `CvReligionAI::ChooseBonusBelief` in the same file excludes up to three already selected belief IDs before scoring. This is used by `CvUnit::DoFoundReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp` for `CvPlayerTraits::IsBonusReligiousBelief`.

`CvUnit::CanFoundReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp` additionally requires a religion-founding unit, unused prophet spread charges, owned-team city plot, non-minor team, no delayed death, religions still available unless `CvPlayerTraits::IsAlwaysReligion`, no existing owned religion, Lua/game-event permission, and at least one available founder and follower belief.

`CvUnit::CanEnhanceReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp` requires a religion-founding unit at the owned religion's holy-city coordinates, no prior enhancement, and at least one available enhancer and follower belief.

STALE_OR_MISMATCH: Any guide implication that all belief categories are chosen during `CvGameReligions::DoPlayerTurn` is stale. Audited source chooses founder/follower/enhancer beliefs from the prophet mission path in `CvUnit::DoFoundReligion` and `CvUnit::DoEnhanceReligion`.

- candidate lists come from `CvGameReligions::GetAvailablePantheonBeliefs`, `GetAvailableFounderBeliefs`, `GetAvailableFollowerBeliefs`, `GetAvailableEnhancerBeliefs`, `GetAvailableBonusBeliefs`, and `GetAvailableReformationBeliefs` in `CvReligionClasses.cpp`. These functions loop `CvBeliefXMLEntries` / `GC.GetGameBeliefs()`, null-check `CvBeliefEntry`, require category flags from `CvBeliefEntry::IsPantheonBelief`, `IsFounderBelief`, `IsFollowerBelief`, `IsEnhancerBelief`, or `IsReformationBelief`, and reject already-used beliefs with `CvGameReligions::IsInSomeReligion(...)` except pantheons under `MOD_BALANCE_ANY_PANTHEON`. unique-civ filtering uses `CvBeliefEntry::GetRequiredCivilization()` and `CvPlayerTraits::IsUniqueBeliefsOnly()`. event vetoes use `GAMEEVENT_PlayerCanHaveBelief` for pantheons and `GAMEEVENT_ReligionCanHaveBelief` for founder/follower/enhancer/bonus/reformation when `MOD_EVENTS_ACQUIRE_BELIEFS` is enabled. `GetAvailableBonusBeliefs(...)` accepts enhancer, follower, founder, or pantheon beliefs; `CvReligionAI::ChooseBonusBelief(...)` then excludes up to three belief IDs already selected in the same founding sequence.

## Scoring and inputs

All chooser functions call `CvReligionAI::ScoreBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, passing plot weights from `CvReligionAI::CalculatePlotWeightsForBeliefSelection`. Pantheons pass `bConsiderExpansion=true`; founder, follower, enhancer, bonus, and reformation pass `false`.

`CvReligionAI::ScoreBelief` immediately special-cases `CvPlayerTraits::StartsWithPantheon`: beliefs flagged by `CvBeliefEntry::IsAIGoodStartingPantheon` score `1000`, all others score `1`. Because selection is still thresholded, this creates a heavily weighted starting-pantheon pool rather than a hard single best pick.

Plot scoring is `CvReligionAI::ScoreBeliefAtPlot`, which calls `CvReligionAI::GetValidPlotYieldTimes100` for yields up to `YIELD_FAITH`, then multiplies by `CvReligionAI::ScoreYieldForReligionTimes100`. `GetValidPlotYieldTimes100` evaluates terrain, plot type, features, natural wonders, lakes, resources, current/future improvements, required resource/improvement/no-feature gates, tech availability, unique improvements, and likely feature removal.

Pantheon city scoring uses `CvReligionAI::ScorePantheonBeliefAtCity`. It evaluates river/city/building happiness, population and follower yields, yield per birth, WLTKD yields, great-person points, city/capital/coastal/trade-route/specialist/luxury/building/great-work/construction yields, and minimum population/follower penalties.

Non-pantheon city scoring uses `CvReligionAI::ScoreBeliefAtCity`, multiplied in `ScoreBelief` by `RELIGION_BELIEF_SCORE_CITY_MULTIPLIER`. It evaluates growth, peace requirements, border growth, plot culture cost, city range strike, healing, wonder/unit production, city/building happiness, expansion-related yields, holy-city/trade/luxury/border-growth yields, great-person points, WLTKD synergies, era-adjusted city yield changes, building-class yields, world-wonder yields, follower-based yield modifiers, and construction yields.

Player-level non-pantheon scoring is `CvReligionAI::ScoreBeliefForPlayer`. It builds categories for war, happiness, culture, science, gold, spread, great people, buildings, diplomacy, policy gain, and golden ages, then weights parts of them by grand-strategy priorities from `CvGrandStrategyAI::GetGrandStrategyPriority`.

Player-level pantheon scoring is `CvReligionAI::ScorePantheonBeliefForPlayer`. It classifies the player as tall, warmonger, or expansion from unlocked early policy branches (`POLICY_BRANCH_TRADITION`, `POLICY_BRANCH_HONOR`, `POLICY_BRANCH_LIBERTY`) or player traits, then scores war, anti-warmonger defense, wonder production, world-wonder yields, growth, border growth, plot-cost reductions, trade-route yields, great-work yields, and luxury yields.

Yield valuation is centralized in `CvReligionAI::ScoreYieldForReligionTimes100` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`: `YIELD_FOOD` uses `FLAVOR_GROWTH * 50`, production uses `FLAVOR_PRODUCTION * 50`, gold uses `FLAVOR_GOLD * 50`, science uses `FLAVOR_SCIENCE * 80`, culture uses `FLAVOR_CULTURE * 100`, faith uses `FLAVOR_RELIGION * 110`, and other yields default to `500`.

Flavor inputs are read through `CvFlavorManager::GetPersonalityIndividualFlavor` in `CvReligionAI::ScoreYieldForReligionTimes100`, `CvReligionAI::ScorePantheonBeliefAtCity`, `CvReligionAI::ScoreBeliefAtCity`, and `CvReligionAI::ScorePantheonBeliefForPlayer` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`. Explicit flavors include growth, production, gold, science, culture, religion, great people, offense, defense, city defense, happiness, diplomacy, and wonder.

Civilization and leader-trait inputs are read by `CvReligionAI::ScoreBelief`, `CvReligionAI::GetValidPlotYieldTimes100`, `CvReligionAI::ScorePantheonBeliefAtCity`, `CvReligionAI::ScoreBeliefAtCity`, `CvReligionAI::ScoreBeliefForPlayer`, and `CvReligionAI::ScorePantheonBeliefForPlayer` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`. Inputs include `CvPlayerTraits::StartsWithPantheon`, `IsAIGoodStartingPantheon` on belief entries, `IsUniqueBeliefsOnly`, `IsBonusReligiousBelief`, `IsAlwaysReligion`, `IsSmaller`, `IsExpansionist`, `IsWarmonger`, `IsNerd`, `IsDiplomat`, `IsTourism`, `IsReligious`, `IsPopulationBoostReligion`, `IsNoNaturalReligionSpread`, `IsForeignReligionSpreadImmune`, `NoTrain`, `IsWoodlandMovementBonus`, unique luxury/import/export traits, WLTKD traits, wonder modifiers, and unique improvements/buildings through civilization mappings.

Terrain, city, and map inputs are used by `CvReligionAI::CalculatePlotWeightsForBeliefSelection`, `CvReligionAI::GetValidPlotYieldTimes100`, `CvReligionAI::ScorePantheonBeliefAtCity`, and `CvReligionAI::ScoreBeliefAtCity` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`. Examples include revealed map bounds, owned/worked plots, unowned expansion plots, mountains in city range, nearby known enemy cities, resources, features, improvements, city population, capital/holy city/coastal/river status, specialists, trade routes, great works, buildings, local luxuries, world target city count, estimated number of cities, and current era.

Diplomacy and military state enter through `CvReligionAI::ScoreBeliefForPlayer`, `CvReligionAI::ScorePantheonBeliefForPlayer`, and `CvReligionAI::GetNumCitiesWithReligionCalculator` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`: proximity, known territory, conversion theft targets, civ approach, current wars, meanness, wonder competitiveness, city-state friends/allies, active league votes/host, spies, foreign religions in owned cities, and military strategy `MILITARYAISTRATEGY_ERADICATE_BARBARIANS`.

Chosen victory or grand-strategy inputs are read in `CvReligionAI::ScoreBeliefForPlayer` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` by looping all AI grand strategies and matching type strings `AIGRANDSTRATEGY_CONQUEST`, `AIGRANDSTRATEGY_CULTURE`, `AIGRANDSTRATEGY_UNITED_NATIONS`, and `AIGRANDSTRATEGY_SPACESHIP`. Their priorities become conquest, culture, diplomacy, and science interest modifiers.

Existing religion state modifies scoring. `CvReligionAI::ScoreBeliefForPlayer` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` reads the player's state religion, current religion beliefs, holy city, enhanced religion count, nearby cities with/without the religion through `CvReligionAI::GetNumCitiesWithReligionCalculator`, existing faith building availability through `CvReligionAI::FaithBuildingAvailable`, and recursively samples existing beliefs' category scores to bias future belief synergy.

Scoring shape: Every chooser calls `CvReligionAI::ScoreBelief`. `ScoreBelief` combines `CalculatePlotWeightsForBeliefSelection`, `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity` or `ScoreBeliefAtCity`, `ScorePantheonBeliefForPlayer` or `ScoreBeliefForPlayer`, and yield valuation from `ScoreYieldForReligionTimes100`. The final score is clamped non-negative with `max(0, iRtnValue)`.

- every chooser calls `CvReligionAI::ScoreBelief(CvBeliefEntry*, CvWeightedVector<int>, bool bForBonus, bool bConsiderFutureTech)`. `ChoosePantheonBelief(...)` passes plot weights from `CalculatePlotWeightsForBeliefSelection(true)`; founder, follower, enhancer, bonus, and reformation pass `CalculatePlotWeightsForBeliefSelection(false)`. `ScoreBelief(...)` sums plot value, owned-city value, potential-city value, and player-level value, then applies a unique-civilization multiplier and bonus-belief modifiers before returning `max(0, iRtnValue)`. a `StartsWithPantheon()` player bypasses normal scoring: `CvBeliefEntry::IsAIGoodStartingPantheon()` returns `1000`, all others return `1`.

- Scoring components: plot scoring uses `CvReligionAI::ScoreBeliefAtPlot(...)`, `GetValidPlotYieldTimes100(...)`, and `ScoreYieldForReligionTimes100(...)`. Plot weights from `CalculatePlotWeightsForBeliefSelection(...)` cover revealed owned/worked plots, workable owned plots, mountains in city range, and, for pantheons only, unowned revealed expansion plots based on capital distance, nearby cities, early policy branches `POLICY_BRANCH_TRADITION`, `POLICY_BRANCH_HONOR`, `POLICY_BRANCH_LIBERTY`, and traits such as `IsExpansionist()` / `IsSmaller()`. city scoring uses `ScorePantheonBeliefAtCity(...)` for pantheon beliefs and `RELIGION_BELIEF_SCORE_CITY_MULTIPLIER * ScoreBeliefAtCity(...)` for non-pantheons. player-level scoring uses `ScorePantheonBeliefForPlayer(...)` for pantheons and `ScoreBeliefForPlayer(...)` for other categories. INFERRED these helpers are the practical score-component boundary for future formula tables; each helper internally evaluates many belief fields rather than a single generic flavor row.

- belief choice does not use a `Belief_Flavors` object-flavor table. It reads personality flavors through `CvFlavorManager::GetPersonalityIndividualFlavor(...)` in `CvReligionAI` helpers. `ScoreYieldForReligionTimes100(YieldTypes)` maps `YIELD_FOOD` to `FLAVOR_GROWTH * 50`, `YIELD_PRODUCTION` to `FLAVOR_PRODUCTION * 50`, `YIELD_GOLD` to `FLAVOR_GOLD * 50`, `YIELD_SCIENCE` to `FLAVOR_SCIENCE * 80`, `YIELD_CULTURE` to `FLAVOR_CULTURE * 100`, `YIELD_FAITH` to `FLAVOR_RELIGION * 110`, and other yields to `500`. other scoring helpers read direct flavors such as `FLAVOR_GREAT_PEOPLE`, `FLAVOR_OFFENSE`, `FLAVOR_DEFENSE`, `FLAVOR_CITY_DEFENSE`, `FLAVOR_HAPPINESS`, `FLAVOR_DIPLOMACY`, and `FLAVOR_WONDER`. `CvReligionAI::ScoreBeliefForPlayer(...)` also reads grand-strategy priorities from `CvGrandStrategyAI::GetGrandStrategyPriority(...)` for strings including `AIGRANDSTRATEGY_CONQUEST`, `AIGRANDSTRATEGY_CULTURE`, `AIGRANDSTRATEGY_UNITED_NATIONS`, and `AIGRANDSTRATEGY_SPACESHIP`.

## Selection and control

Each chooser in `CvReligionAI::ChoosePantheonBelief`, `ChooseFounderBelief`, `ChooseFollowerBelief`, `ChooseEnhancerBelief`, `ChooseBonusBelief`, and `ChooseReformationBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` sorts `CvWeightedVector<BeliefTypes>` with `StableSortItems()` and chooses with `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getBeliefChoiceCutoffThreshold(), seed)`. This means selection is thresholded weighted choice among sufficiently high-scoring beliefs, not always top-score deterministic.

Seeds are category-specific and mix player ID plus the available-belief count:

- Pantheon: `CvReligionAI::ChoosePantheonBelief`, seed `0x0af7fe29`.

- Founder: `CvReligionAI::ChooseFounderBelief`, seed `0x9db23f3c`.

- Follower: `CvReligionAI::ChooseFollowerBelief`, seed `0x93c8983d`.

- Enhancer: `CvReligionAI::ChooseEnhancerBelief`, seed `0x3a862bb8`.

- Bonus: `CvReligionAI::ChooseBonusBelief`, seed `0xc0809801`.

- Reformation: `CvReligionAI::ChooseReformationBelief`, seed `0x894eaafe`.

`CvReligionAI::ScoreBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` clamps the final score to `max(0, iRtnValue)`, so negative internal terms can reduce a belief to zero but not below zero.

`CvReligionAI::ChooseBonusBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` uses the full available bonus-belief count in its seed even after excluding already chosen beliefs from scoring. This is deterministic for a given available list and exclusions, but edits to availability or belief ordering can change outcomes.

- candidate generation: `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvGameReligions::GetAvailablePantheonBeliefs`, `GetAvailableFounderBeliefs`, `GetAvailableFollowerBeliefs`, `GetAvailableEnhancerBeliefs`, `GetAvailableBonusBeliefs`, `GetAvailableReformationBeliefs`.

- scoring: `CvReligionAI::ScoreBelief`, `CalculatePlotWeightsForBeliefSelection`, `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, `ScorePantheonBeliefForPlayer`, `ScoreBeliefForPlayer`, `ScoreYieldForReligionTimes100`.

- application: `CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, `AddReformationBelief`; prophet mission triggers in `CvGameCoreDLL_Expansion2/CvUnit.cpp`, `CvUnit::DoFoundReligion` and `CvUnit::DoEnhanceReligion`; first-city/policy triggers in `CvPlayer::foundCity` and `CvPlayer::processPolicies`.

- data: `CvBeliefEntry`, `CvBeliefEntry::CacheResults(...)` in `CvGameCoreDLL_Expansion2/CvBeliefClasses.cpp`; VP belief data under `(2) Vox Populi/Database Changes/Beliefs/`.

- Seeded randomness: category seeds in `CvReligionClasses.cpp` are `CvSeeder::fromRaw(...).mix(GET_PLAYER(ePlayer).GetID()).mix(availableBeliefs.size())`: pantheon `0x0af7fe29`, founder `0x9db23f3c`, follower `0x93c8983d`, enhancer `0x3a862bb8`, bonus `0xc0809801`, and reformation `0x894eaafe`. RISK selection is sensitive to availability filters, database belief ordering/counts, score magnitudes, `HandicapInfos.BeliefChoiceCutoffThreshold`, and seed inputs.

## Data and configuration

- `CvUnitEntry`, `Unit_Flavors`, `UnitAITypes` in `CvGameCoreDLLUtil/include/CvEnums.h`, unit classes, domains, resources, promotions, beliefs, policies, strategies, and custom-mod option `AI_UNIT_PRODUCTION`.

| Folder / data area | Relevant tables and rows | Current-source consumer |
|---|---|---|
| `(2) Vox Populi/Database Changes/Beliefs/` | modifies `Beliefs` and many `Belief_*` tables, including category flags (`Pantheon`, `Founder`, `Follower`, `Enhancer`, `Reformation`), yield tables such as `Belief_ImprovementYieldChanges`, `Belief_BuildingClassYieldChanges`, `Belief_YieldPerXFollowers`, faith-purchase tables, civilization requirements, and AI flags such as `AIGoodStartingPantheon`. | `CvBeliefEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvBeliefClasses.cpp`); belief candidate and scoring consumers in `CvGameReligions::GetAvailable*Beliefs` and `CvReligionAI::ScoreBelief` (`CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`). |

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvBeliefEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvBeliefClasses.cpp`) | `Beliefs` and `Belief_*` tables | Belief availability and `CvReligionAI::ScoreBelief`; no verified `Belief_Flavors` object-flavor path in current belief selection. |
| `CvTechEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvTechClasses.cpp`) | `Technologies`, `Technology_Flavors`, prereq/grid/cost/era fields | Tech AI flavor cache and tech unlock paths that indirectly affect units/buildings/policies/beliefs. |

Belief path is `Beliefs` and `Belief_*` tables -> `CvBeliefEntry::CacheResults` -> candidate generation in `CvGameReligions::GetAvailablePantheonBeliefs`, `GetAvailableFounderBeliefs`, `GetAvailableFollowerBeliefs`, `GetAvailableEnhancerBeliefs`, `GetAvailableBonusBeliefs`, and `GetAvailableReformationBeliefs` -> scoring in `CvReligionAI::ScoreBelief`, `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, `ScoreBeliefForPlayer`, and `ScorePantheonBeliefForPlayer` -> application in `CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, and `AddReformationBelief`.

Belief selection does not use object `*_Flavors` rows for `CvBeliefEntry`; it reads database belief effect fields plus personality flavors and grand-strategy priorities inside `CvReligionAI` (`CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`).

- source classes include `CvReligionAI`, `CvGameReligions`, `CvPlayerReligions`, `CvReligion`, `CvReligionBeliefs`, `CvBeliefEntry`, `CvBeliefXMLEntries`, `CvPlayerTraits`, `CvPlayerPolicies`, `CvFlavorManager`, `CvGrandStrategyAI`, `CvUnit`, and `CvCityReligions`. enum/order-sensitive types include database-backed `BeliefTypes`, `ReligionTypes`, `YieldTypes`, `PolicyBranchTypes`, `CivilizationTypes`, `BuildingClassTypes`, `ImprovementTypes`, `ResourceTypes`, `TerrainTypes`, `FeatureTypes`, and `FlavorTypes`. `CvBeliefEntry::CacheResults(...)` in `CvGameCoreDLL_Expansion2/CvBeliefClasses.cpp` reads category columns `Beliefs.Pantheon`, `Founder`, `Follower`, `Enhancer`, `Reformation`, gating columns such as `RequiresImprovement`, `RequiresResource`, `RequiresNoImprovement`, `RequiresNoImprovementFeature`, `RequiresOwnTerritory`, special AI column `AI_GoodStartingPantheon`, and `CivilizationType`. VP/CP database files include `(1) Community Patch/Database Changes/Religions/BeliefTableChanges.sql` and `(1) Community Patch/Database Changes/Religions/NewBeliefTables.xml`, plus VP belief updates under `(2) Vox Populi/Database Changes/Beliefs/`.

## Observability and validation obligations

`CvReligionAI::LogBeliefChoices` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` logs every scored candidate and the chosen belief to `GC.getGame().GetGameReligions()->GetLogFileName()` when both `GC.getLogging()` and `GC.getAILogging()` are enabled.

`CvReligionAI::ScoreBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` logs the total plot, owned-city, potential-city, and player components to `TotalBeliefScoringReligionLog.csv` under the same logging gates.

`CvReligionAI::ScoreBeliefForPlayer` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` logs war, happiness, culture, science, gold, spread, great people, diplomacy, building, policy, golden-age, total, and grand-strategy interest values to `PlayerBeliefReligionLog.csv` under the same logging gates.

Final application functions also log high-level religion events through `CvGameReligions::LogReligionMessage` in `CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, and `AddReformationBelief` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`.

Lua/game-event hooks can veto or observe belief paths in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` and `CvGameCoreDLL_Expansion2/CvUnit.cpp`: `CvGameReligions::CanCreatePantheon` uses `PlayerCanFoundPantheon`; availability functions use `PlayerCanHaveBelief` or `ReligionCanHaveBelief`; `CvUnit::CanFoundReligion` uses `PlayerCanFoundReligion`; application functions fire `PantheonFounded`, `ReligionFounded`, `ReligionEnhanced`, and `ReligionReformed`; AI founding can receive an extra belief through `GAMEEVENT_GetBeliefToFound` in `CvUnit::DoFoundReligion`.

6. Runtime logging adequacy test: for one fixed save, correlate `CityStrategyAIProductionLog...csv`, `CityStrategyAIHurryLog...csv`, `PolicyAILog...csv`, `PolicyAIYieldAttributesLog...csv`, `ReligionAILog...csv`, `TotalBeliefScoringReligionLog.csv`, operation logs, and tactical logs before any scoring patch.

Logging: `CvReligionAI::LogBeliefChoices` logs scored candidates and the selected belief. `CvReligionAI::ScoreBelief` logs component totals to `TotalBeliefScoringReligionLog.csv`; `ScoreBeliefForPlayer` logs player-level components to `PlayerBeliefReligionLog.csv`; final religion events log through `CvGameReligions::LogReligionMessage`.

- `CvReligionAI::LogBeliefChoices(...)` logs every candidate weight and the chosen belief to `GC.getGame().GetGameReligions()->GetLogFileName()` when `GC.getLogging()` and `GC.getAILogging()` are enabled. `CvReligionAI::ScoreBelief(...)` logs plot, owned-city, potential-city, and player components to `TotalBeliefScoringReligionLog.csv`. `CvReligionAI::ScoreBeliefForPlayer(...)` logs player-level subcomponents to `PlayerBeliefReligionLog.csv`. final application functions call `CvGameReligions::LogReligionMessage(...)` for high-level religion events.

## Version differences, risks and unresolved questions

INFERRED: The exact behavior of `CvWeightedVector<BeliefTypes>::ChooseAbovePercentThreshold` should be documented from `CvGameCoreDLLUtil/include/CvWeightedVector.h` in a shared scoring-infrastructure doc. This file confirms chooser usage but this pass did not restate that template's internals.

INFERRED: `CvBeliefEntry` database caching and all belief getter storage fields should be mapped separately when documenting XML/schema assumptions. This pass cites the `CvBeliefEntry` getters consumed by scoring, but does not enumerate every belief table column or `CvBeliefEntry::CacheResults` field.

INFERRED: Some scoring formulas contain likely copy/paste or legacy quirks, such as `ScoreBeliefForPlayer` checking `GetYieldFromConquest(iI) > 0` before adding `GetYieldFromRemoveHeresy(...)`. This is source behavior, but intent is not verified.

Belief selection is randomness-sensitive. Changing available-list ordering, filtering, score magnitudes, `BeliefChoiceCutoffThreshold`, or seed inputs in `CvReligionAI::Choose*Belief` can change deterministic AI outcomes and multiplayer synchronization.

Belief and religion state is savegame-sensitive. `CvReligion`, `CvReligionInCity`, and `CvGameReligions` serialize religion fields through serialization visitors and `CvGameReligions::Serialize` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`; changes to stored belief counts, flags such as `m_bEnhanced`/`m_bReformed`, or enum ordering need save compatibility review.

Open enum and database assumptions are high risk. Candidate loops cast integer indices to `BeliefTypes`, `YieldTypes`, `BuildingClassTypes`, `ImprovementTypes`, `PolicyBranchTypes`, `AIGrandStrategyTypes`, and other enums, and scoring relies on XML-backed `CvBeliefEntry` getters.

Lua and game-event hooks are behavioral extension points. Edits to `GetAvailable*Beliefs`, `CanCreatePantheon`, `CanFoundReligion`, `FoundPantheon`, `FoundReligion`, `EnhanceReligion`, or `AddReformationBelief` can affect modmod vetoes, extra beliefs, notifications, and script-visible event order.

Do not treat the guide as proof for future patches. The audited source shows founder/follower/enhancer AI belief choice in `CvUnit::DoFoundReligion` and `CvUnit::DoEnhanceReligion`, not in a single religion-turn chooser.

- RISK religion application functions (`CvGameReligions::FoundPantheon`, `FoundReligion`, `EnhanceReligion`, `AddReformationBelief`): side effects include religion state, city religion updates, traits, faith thresholds/costs, prophets, notifications, replay/logs, Lua/game events, and UI dirties.

- RISK direct purchase paths (`CvMilitaryAI::BuyEmergencyUnit`, `BuyEmergencyBuilding`, `CvEconomicAI::DoHurry`, `CvReligionAI::DoFaithPurchases*`, `CvCity::PurchaseUnit`, `PurchaseBuilding`): purchases can bypass normal queue scoring, consume treasury/faith, satisfy operation demand, or use investment paths that insert urgent queue items.

- RISK open/database-backed IDs: `FlavorTypes`, `UnitTypes`, `BuildingTypes`, `PolicyTypes`, `PolicyBranchTypes`, `BeliefTypes`, `ProjectTypes`, `ProcessTypes`, and formation/unit role data are array-indexed and serialized in multiple managers. Enum/database ordering changes can affect saves, scoring caches, and seeded selection order.

- Classification: UNDERUSED_SIGNAL

- Affected decision: Belief selection.

- Source functions: `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvReligionAI::ScoreYieldForReligionTimes100(...)` reads `CvFlavorManager::GetPersonalityIndividualFlavor(...)` for growth, production, gold, science, culture, and religion yield valuation. Other religion scoring helpers read personality flavors such as `FLAVOR_OFFENSE`, `FLAVOR_DEFENSE`, `FLAVOR_CITY_DEFENSE`, `FLAVOR_HAPPINESS`, `FLAVOR_DIPLOMACY`, and `FLAVOR_WONDER`. Flavor propagation docs verify active/city recipient latest values are maintained by `CvFlavorManager` and `CvFlavorRecipient`, but belief scoring does not use a `CvFlavorRecipient` or `Belief_Flavors`.

- Why current behavior may be suboptimal: INFERRED active economic/military pressures and city strategy deltas can affect production and some policy/wonder caches, but they do not naturally affect belief valuation. An AI under current war, happiness, or economic pressure may still value yields primarily by static personality in belief selection.

- Lowest-risk investigation: Documentation/logging only: augment belief score logs conceptually by comparing personality flavor value, active personality value, and relevant grand-strategy priority for the same player at belief choice time.

- Smallest possible patch point: Do not patch `ScoreYieldForReligionTimes100(...)` first. A lower-risk experiment would add diagnostic logging near `CvReligionAI::ScoreBelief(...)`; if behavior change is later desired, introduce a helper that explicitly chooses personality versus active/grand-strategy flavor per belief category.

- Runtime/logging test: Enable `TotalBeliefScoringReligionLog.csv` and `PlayerBeliefReligionLog.csv`; compare chosen beliefs for an AI before/after activation of major military/economic strategies. Confirm whether only static personality flavor explains yield multipliers.

- Risk of double-counting or unintended downstream effects: RISK belief choices are infrequent and high impact. Adding active strategy flavor can double-count direct belief heuristics that already read neighbor/warmonger/religion/city state, and can push religion choices toward short-term crises rather than durable strategy.

- Source functions: purchase paths in `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, `CvTacticalAI::PlotEmergencyPurchases(...)`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::BuyEmergencyUnit(...)` / `BuyEmergencyBuilding(...)`; `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `CvEconomicAI::DoHurry(...)`; `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvReligionAI::DoFaithPurchasesInCities(...)`; and `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseHurry(...)`. Shared helpers include `CvUnitProductionAI::CheckUnitBuildSanity(...)` and `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`; application paths include `CvCity::pushOrder(...)`, `CvCity::PurchaseUnit(...)`, `CvCity::PurchaseBuilding(...)`, and direct `CvCity::CreateBuilding(...)`.

- Smallest possible patch point: Add runtime assertions/log-only tags to `CvCityStrategyAI::ChooseHurry(...)`, `CvMilitaryAI::BuyEmergencyUnit(...)`, and `CvReligionAI::DoFaithPurchasesInCities(...)` showing which application subtype occurred: immediate creation, investment, urgent queue insertion, or direct create.

- OPEN_QUESTION: Purchase logging remains source-thin. Known paths include `CvCityStrategyAI::ChooseHurry(...)`, `CvEconomicAI::DoHurry(...)`, `CvMilitaryAI::BuyEmergencyUnit(...)`, `CvMilitaryAI::BuyEmergencyBuilding(...)`, `CvReligionAI::DoFaithPurchasesInCities(...)`, `CvAIOperation::BuyFinalUnit()`, `CvCity::PurchaseUnit(...)`, and `CvCity::PurchaseBuilding(...)`.

Patch-risk notes: RISK: Belief availability and ordering, `BeliefTypes` database-backed IDs, `CvBeliefEntry` schema/cache fields, seeded randomness, Lua/game-event vetoes such as `PlayerCanHaveBelief` / `ReligionCanHaveBelief`, and `CvGameReligions` serialized state are sensitive. STALE_OR_MISMATCH: Any doc or guide implication that founder/follower/enhancer choices are all made in `CvGameReligions::DoPlayerTurn` is stale; audited source chooses those from prophet mission paths.

- Risk notes: RISK belief selection is multiplayer/replay sensitive because seeded random selection depends on candidate vectors, scores, and database ordering. RISK belief/religion save compatibility is sensitive because `CvReligion` serializes `m_Beliefs`, `m_bPantheon`, `m_bEnhanced`, and `m_bReformed` through serialization in `CvReligionClasses.cpp`; changes to enum order, stored flags, or belief-count semantics need save review. RISK Lua/game-event hooks can veto or add beliefs, so source-only candidate assumptions may differ in a modded runtime. RISK changing `CvBeliefEntry::CacheResults(...)` fields or belief database schema affects both scoring and final application effects.

- Known improvement hooks: DESIGN_HYPOTHESIS split a future formula appendix by helper boundary: `CalculatePlotWeightsForBeliefSelection`, `GetValidPlotYieldTimes100`, `ScoreBeliefAtPlot`, `ScorePantheonBeliefAtCity`, `ScoreBeliefAtCity`, `ScorePantheonBeliefForPlayer`, and `ScoreBeliefForPlayer`. OPEN_QUESTION fully enumerate every `CvBeliefEntry::CacheResults(...)` field and secondary `Belief_*` table consumed by these scoring helpers. OPEN_QUESTION document `CvGameReligions::CanFoundReligion(...)` and `CanEnhanceReligion(...)` final validation paths separately from the AI prophet mission gates. STALE_OR_MISMATCH any guide pattern implying all belief categories are chosen during `CvGameReligions::DoPlayerTurn(...)` is stale; audited source chooses founder/follower/enhancer/bonus beliefs from `CvUnit::DoFoundReligion()` and `CvUnit::DoEnhanceReligion()`.

## Source and dependency lookup

Vox Populi belief selection follows the general guide pattern: each chooser in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` class `CvReligionAI` gets an available-belief list from `CvGameReligions`, scores each `CvBeliefEntry` with `CvReligionAI::ScoreBelief`, stores scores in `CvWeightedVector<BeliefTypes>`, calls `StableSortItems()`, then uses `ChooseAbovePercentThreshold(...)` with a seeded `CvSeeder` and the handicap belief-choice cutoff.

GUIDE_PATTERN: Section 3.6.4's candidate generation -> scoring -> weighted/thresholded selection -> application model is a useful description of the current chooser shape, but the audited source is more detailed and is the authority.

Unresolved questions for focused pass:
- Trace founder/follower/enhancer final AI call chain from prophet/unit/UI responses through `CanFoundReligion(...)`, `CanEnhanceReligion(...)`, and final application.
- Split `ScoreBelief(...)` into plot, city, player, flavor, yield, trait, and diplomacy inputs with exact constants.
- Identify all Lua/game-event vetoes in available-belief generation.
- Flag serialization or sync-sensitive religion state touched by final application functions.

This atlas treats the audited repository source code as the authority. Existing docs such as `docs-ai/ai-source-map.md`, `docs-ai/ai-beliefs.md`, `docs-ai/ai-policies.md`, `docs-ai/ai-wonders.md`, `docs-ai/ai-unit-production.md`, `docs-ai/ai-unit-production-upstream.md`, `docs-ai/ai-flavors.md`, and `docs-ai/ai-flavor-evaluation.md` are maps into the source, not proof by themselves.

- `CvReligionAI` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`: belief scoring and religious purchase logic.

| Class / manager | Decisions affected |
|---|---|
| `CvReligionAI` (`CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvReligionClasses.h`) | belief scoring/selection through `Choose*Belief(...)` and `ScoreBelief(...)`; AI faith purchases through `DoTurn()`, `DoFaithPurchases()`, and `DoFaithPurchasesInCities(...)`. Affects belief selection and emergency/direct purchases. |
| `CvGameReligions`, `CvPlayerReligions`, `CvReligion`, `CvReligionBeliefs` (`CvReligionClasses.cpp/.h`) | belief candidate availability and final religion application: `FoundPantheon`, `FoundReligion`, `EnhanceReligion`, `AddReformationBelief`. Affects belief selection application and religion side effects. |
| `CvEconomicAI` (`CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`, `.h`) | economic strategy deltas, direct economic strategy checks, hurry/purchase selection, operation requests for civilian missions. Affects city production, unit production, policy scoring, belief scoring, operation demand, emergency purchases, and flavor propagation. |

| Function / enum / table | Decisions affected |
|---|---|
| `CvReligionAI::ScoreBelief(...)`, `ScoreBeliefAtPlot(...)`, `ScorePantheonBeliefAtCity(...)`, `ScoreBeliefAtCity(...)`, `ScorePantheonBeliefForPlayer(...)`, `ScoreBeliefForPlayer(...)` (`CvReligionClasses.cpp`) | belief scoring components. |
| `CvGameReligions::GetAvailable*Beliefs(...)` (`CvReligionClasses.cpp`) | belief candidate generation and Lua/game-event vetoes. |

- production/purchase consumers: `CvCityStrategyAI::ChooseProduction`, `CvCityStrategyAI::ChooseHurry`, `CvCity::CheckForOperationUnits`, `CvAIOperation::BuyFinalUnit`, `CvMilitaryAI::BuyEmergencyUnit`, faith-purchase logic in `CvReligionAI`.

- pre-city purchase triggers: `CvMilitaryAI::MakeEmergencyPurchases`, `CvAIOperation::BuyFinalUnit`, `CvEconomicAI::DoHurry`, `CvReligionAI::DoFaithPurchases` / `DoFaithPurchasesInCities`.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvPlayer::processPolicies(...)` | broad policy side effects and reformation belief trigger. | RISK policy application can invoke religion selection/application and many player/city/unit modifiers. |

- Keep the flavor-layer split. `CvFlavorManager::GetPersonalityIndividualFlavor(...)`, `CvFlavorRecipient::GetLatestFlavorValue(...)`, and `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)` are different signals. The atlas correctly flags this as high risk in production, policy, wonder, and belief records.

- INFERRED: Revise broad "improvement hook" wording that implies behavior should change where the current evidence only proves traceability gaps. Production component logs, policy flavor logs, belief active-flavor diagnostics, and wonder race-pressure logs are documentation/debugging improvements first; they are not behavior-change justifications.

- Reject/defer changing belief scoring to use active/latest flavors. `CvReligionAI::ScoreYieldForReligionTimes100(...)` and other religion helpers use `CvFlavorManager::GetPersonalityIndividualFlavor(...)`; INFERRED this may underuse current strategic pressure, but belief decisions are infrequent, high-impact, and already include direct game-state heuristics in `CvReligionAI::ScoreBeliefForPlayer(...)`.

4. Source-pass purchase application variants: direct create (`CvMilitaryAI::BuyEmergencyBuilding(...)`), normal purchase (`CvCity::PurchaseBuilding(...)` / `PurchaseUnit(...)`), investment, front-of-queue operation build, and faith purchase in `CvReligionAI`.

`CvReligionAI` is declared in `CvGameCoreDLL_Expansion2/CvReligionClasses.h` and implemented in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`. It chooses `BeliefTypes` for pantheon, founder, follower, enhancer, bonus, and reformation slots through `ChoosePantheonBelief`, `ChooseFounderBelief`, `ChooseFollowerBelief`, `ChooseEnhancerBelief`, `ChooseBonusBelief`, and `ChooseReformationBelief`.

Inputs consumed: Candidate lists come from `CvGameReligions::GetAvailablePantheonBeliefs`, `GetAvailableFounderBeliefs`, `GetAvailableFollowerBeliefs`, `GetAvailableEnhancerBeliefs`, `GetAvailableBonusBeliefs`, and `GetAvailableReformationBeliefs`. Scoring consumes `CvBeliefEntry`, plot/city/player state, religion state, `CvPlayerTraits`, diplomacy and military state, `CvGrandStrategyAI::GetGrandStrategyPriority`, and personality flavors through `CvFlavorManager::GetPersonalityIndividualFlavor`.

Selection mechanism: Choosers push scored candidates into `CvWeightedVector<BeliefTypes>`, call `StableSortItems`, then call `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getBeliefChoiceCutoffThreshold(), seed)`. Seeds are per slot: pantheon `0x0af7fe29`, founder `0x9db23f3c`, follower `0x93c8983d`, enhancer `0x3a862bb8`, bonus `0xc0809801`, and reformation `0x894eaafe`, each mixed with player ID and available-belief count.

Religion selection is similarly split. `CvPlayer::foundCity(...)` and `CvPlayer::processPolicies(...)` can trigger AI belief choices, but final religion application is in `CvGameReligions::FoundPantheon(...)` and `CvGameReligions::AddReformationBelief(...)`; founder/follower/enhancer choices use prophet mission paths outside this class pair. Source: `CvPlayer.cpp`, `foundCity`, `processPolicies`; `CvReligionClasses.cpp`, `CvReligionAI::ChoosePantheonBelief`, `ChooseReformationBelief`, `CvGameReligions::FoundPantheon`, `AddReformationBelief`; `CvGameCoreDLL_Expansion2/CvUnit.cpp`, `CvUnit::DoFoundReligion`, `CvUnit::DoEnhanceReligion`.

Direct purchase paths can bypass normal city queue selection but still use player managers. Examples include `CvMilitaryAI::BuyEmergencyUnit(...)` using `CvEconomicAI::CanWithdrawMoneyForPurchase(...)` and `CvCity::PurchaseUnit(...)`, and `CvReligionAI` faith-purchase code calling `CvCity::PurchaseUnit(...)` and checking `CvCity::GetUnitForOperation()` / `CvMilitaryAI::GetUnitTypeForArmy(...)`. Source: `CvMilitaryAI.cpp`, `CvMilitaryAI::BuyEmergencyUnit`; `CvReligionClasses.cpp`, `CvReligionAI::DoTurn` purchase logic; `CvCity.cpp`, `CvCity::PurchaseUnit`.

STALE_OR_MISMATCH: Any guide-level claim that one central player AI method directly selects beliefs, policies, wonders, and units is stale for audited source. Audited source routes timing through `CvPlayer` / `CvPlayerAI`, but scoring and candidate selection live in specialized managers and city classes.

`CvPlayer::GetReligionAI()` -> belief choice, AI faith spending, religious unit/building purchase decisions.

- all six chooser functions call `StableSortItems()` and then `ChooseAbovePercentThreshold(GC.getGame().getHandicapInfo().getBeliefChoiceCutoffThreshold(), seed)`. `CvWeightedVector<T>::ChooseAbovePercentThreshold(...)` in `CvGameCoreDLLUtil/include/CvWeightedVector.h` computes the highest weight, randomly chooses among all entries if the highest weight is zero, otherwise filters to entries whose weight is at least `highest * percent / 100`, then calls `ChooseByWeight(...)`; `ChooseByWeight(...)` uses `GC.getGame().randRangeExclusive(...)`. This is thresholded weighted random selection, not deterministic top-score selection.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
