# AI trigger and turn lifecycle

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../source-snapshots.md#recovered-baseline>).

## Decision or contract question

When do decision owners run, and which same-turn state can they observe?

Player/city processing and the later movement slice are distinct. This document owns the cross-decision ordering reference and links to topic owners rather than reproducing a file-first Atlas.

Canonical related owners: [Strategic pressure and manager outputs](<../strategies/managers.md>), [City production selection](<../production/city-selection.md>), [Policies, branches, tenets and ideology](<../policies/selection.md>), [Belief selection](<../beliefs/selection.md>), [Operation creation and recruitment planning](<../operations/planning.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

for a major AI player, `CvPlayer::doTurn()` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp` calls `AI_doTurnPre()` near the start, then does broad player state updates, then for major civs calls `CvGrandStrategyAI::DoTurn()` and `CvDiplomacyAI::DoTurn(...)`. If diplomacy does not block, it continues into `CvPlayer::doTurnPostDiplomacy()`.

`CvPlayer::doTurnPostDiplomacy()` updates city threat, plots, area effects, danger plots, tactical map invalidation, military stats, current/future wars, then calls these AI managers before city turns:

- `CvEconomicAI::DoTurn()`

- `CvMilitaryAI::DoTurn()`

- for majors: `CvReligionAI::DoTurn()`, `CvTradeAI::DoTurn()`, `CvCitySpecializationAI::DoTurn()`, `CvLeagueAI::DoTurn()`

after those manager turns, `CvPlayer::doTurnPostDiplomacy()` calls `AI_doSpaceshipProduction()`, then loops cities and calls `CvCity::doTurn()`. City production selection therefore sees economic/military strategy state, operation creation, recommended counts, and city specialization changes from earlier in the same player turn.

`CvPlayerAI::AI_doTurnPost()` later handles AI launch checks, great people, espionage AI, and trade AI. `CvPlayer::doTurnPostDiplomacy()` then calls `doTurnUnits()`.

tactical and homeland unit AI run from `CvGame::updateMoves()` in `CvGameCoreDLL_Expansion2/CvGame.cpp`, which calls `CvPlayerAI::AI_unitUpdate(bool)`. For non-human AI units, `AI_unitUpdate(...)` calls `CvTacticalAI::Update()`, then `CvHomelandAI::Update(true)`, then `CvTacticalAI::CleanUp()` in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`.

`CvTacticalAI::Update()` calls `UpdateVisibility()`, `DropOldFocusAreas()`, `FindTacticalTargets()`, `RecruitUnits()`, and `ProcessDominanceZones()`. `ProcessDominanceZones()` calls `AssignGlobalHighPrioMoves()`, which calls `PlotOperationalArmyMoves()`. `PlotOperationalArmyMoves()` calls `CvAIOperation::DoTurn()` for each operation in `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`.

INFERRED: the exact interleaving between `CvPlayer::doTurnPostDiplomacy()` city production and later `CvGame::updateMoves()` tactical/homeland updates depends on the engine turn loop, but the source clearly shows player-level economic/military/city-specialization AI before city `doTurn()` and tactical/homeland unit movement through `AI_unitUpdate(...)`.

`CvGame::updateMoves` in `CvGameCoreDLL_Expansion2/CvGame.cpp` drives AI unit movement updates by calling `CvPlayerAI::AI_unitUpdate(bool)`. `CvPlayerAI::AI_unitUpdate` in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp` calls `CvTacticalAI::Update`, `CvHomelandAI::Update(true)`, and `CvTacticalAI::CleanUp` for non-human AI units.

`CvGame::doTurn()` in `CvGameCoreDLL_Expansion2/CvGame.cpp` is the game-turn rollover hub. It runs game/global systems, including `CvGameDeals::DoTurn()`, each alive `CvTeam::doTurn()`, `CvMap::doTurn()`, `CvGameReligions::DoTurn()`, `CvGameTrade::DoTurn()`, `CvGameLeagues::DoTurn()`, `CvGameCulture::DoTurn()`, `CvGameCorporations::DoTurn()`, and `CvGameContracts::DoTurn()`, then increments the game turn and activates players or teams with `CvPlayer::setTurnActive(true)` / `CvTeam::setTurnActive(true)`.

`CvPlayer::setTurnActive(bool bNewValue, bool bDoTurn)` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp` is the player activation gate. On turn start with `bDoTurn`, it calls `CvPlayer::SetAllUnitsUnprocessed()`, updates unit conditional promotions, calls `CvCityConnections::Update()` and `CvTreasury::DoUpdateCityConnectionGold()`, may invalidate tactical/homeland AI for human-controlled units or network multiplayer, starts diplomacy requests, and then calls `CvPlayer::doTurn()`.

`CvPlayer::doTurn()` in `CvPlayer.cpp` is the main player-turn entry. It begins with `CvPlayerAI::AI_doTurnPre()` in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp`, runs many player-state updates, then for alive major civs calls `CvGrandStrategyAI::DoTurn()` and `CvDiplomacyAI::DoTurn(...)`. If diplomacy is not blocking on human input, it immediately continues into `CvPlayer::doTurnPostDiplomacy()`.

RISK: The source comment in `CvPlayer::setTurnActive()` says "actual turn processing now happens in `CvGame::updateMoves()`", and `CvPlayer::doTurnPostDiplomacy()` later says "this isn't actually the end of the turn - `AI_unitUpdate` is called later." For AI decision docs, distinguish player/city-turn decisions from the later movement slice.

`CvPlayer::doTurn()` is the main player-turn entry after activation. It resets player supply caches (`m_iNumUnitsSuppliedCached`, `m_iNumUnitsSuppliedCachedWarWeariness`) and `m_bUnlockedGrowthAnywhereThisTurn`, then calls `CvPlayerAI::AI_doTurnPre()`. Source: `CvGameCoreDLL_Expansion2/CvPlayer.cpp`, `CvPlayer::doTurn`; `CvPlayerAI.cpp`, `CvPlayerAI::AI_doTurnPre`.

`CvPlayerAI::AI_doTurnPre()` does AI pre-turn preparation, not broad strategic scoring: it orders AI-controlled units by combat power for unit iteration when `!isHuman(ISHUMAN_AI_UNITS)`, and calls `AI_considerAnnex()` / `AI_considerRaze()` when city management is AI-controlled. Source: `CvPlayerAI.cpp`, `CvPlayerAI::AI_doTurnPre`.

`CvPlayer::doTurn()` performs broad player-state updates before `doTurnPostDiplomacy()`: team status, best military cities via `CvPlayer::UpdateBestMilitaryCities()`, happiness/revolt/cramped state, `CvPlayer::SetBestWonderCities()`, `CvGrandStrategyAI::DoTurn()`, and `CvDiplomacyAI::DoTurn(...)`. Active diplomacy can delay `CvPlayer::doTurnPostDiplomacy()` with `CvGame::SetWaitingForBlockingInput(m_eID)`. Source: `CvPlayer.cpp`, `CvPlayer::doTurn`.

`CvPlayer::doTurnPostDiplomacy()` is the central pre-city player AI hub. Before city turns it updates city threat criteria, plots, area-effect units/plots, danger plots, tactical-analysis invalidation, military stats, war-declaration cache, and current/future wars, then calls `CvEconomicAI::DoTurn()`, `CvMilitaryAI::DoTurn()`, and for major civs `CvReligionAI::DoTurn()`, `CvTradeAI::DoTurn()`, `CvCitySpecializationAI::DoTurn()`, and `CvLeagueAI::DoTurn()`. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`.

City production happens after those pre-city managers. `CvPlayer::doTurnPostDiplomacy()` calls `CvPlayerAI::AI_doSpaceshipProduction()` and then each city's `CvCity::doTurn()`. This ordering means normal city production can see same-turn economic/military strategy, operation demand, desired-wonder state, danger plots, and current/future war state. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`; production path continues through `CvCity::doTurn()` / `CvCity::doProduction()` in `CvGameCoreDLL_Expansion2/CvCity.cpp`.

Policy and ideology selection happen after city turns and culture accumulation in `CvPlayer::doTurnPostDiplomacy()`. Non-human policy-choice players call `CvPlayerPolicies::IsTimeToChooseIdeology()`, `CvPlayerPolicies::DoChooseIdeology()`, and `CvPlayerPolicies::DoPolicyAI()`. Human policy-choice players receive notifications/UI but still call `DoPolicyAI()` to force applicable ideology-update handling. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`; `CvGameCoreDLL_Expansion2/CvPolicyClasses.cpp`, `CvPlayerPolicies::DoPolicyAI`.

Religion has split timing. `CvReligionAI::DoTurn()` runs before city production inside `CvPlayer::doTurnPostDiplomacy()` for AI religion spending/purchases. Later, after policy/research/espionage, `CvGameReligions::DoPlayerTurn(CvPlayer&)` runs faith accumulation and pantheon/reformation-related processing. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`; `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, `CvGameReligions::DoPlayerTurn`.

`CvPlayer::doTurnPostDiplomacy()` explicitly notes that this is not the actual end of the turn because `AI_unitUpdate` is called later, then calls `CvPlayerAI::AI_doTurnPost()`, Lua hook `PlayerDoTurn`, military rating decay, achievements, and `CvPlayer::doTurnUnits()`. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`.

`CvPlayerAI::AI_doTurnPost()` handles post-player-turn AI actions for non-barbarian, non-minor players: AI launch checks via `AI_launch(...)`, great-person processing through `ProcessGreatPeople()`, espionage AI via `CvEspionageAI::DoTurn()`, and trade AI via `CvTradeAI::DoTurn()` depending on human-control masks. Source: `CvPlayerAI.cpp`, `CvPlayerAI::AI_doTurnPost`.

`CvPlayer::doTurnUnits()` wraps unit `doTurn()` processing with `CvPlayerAI::AI_doTurnUnitsPre()` and `CvPlayerAI::AI_doTurnUnitsPost()`. `AI_doTurnUnitsPre()` deletes delayed-death units, kills delayed-death armies, and kills operations whose `CvAIOperation::ShouldAbort()` is true. `AI_doTurnUnitsPost()` runs AI promotions through `CvUnit::AI_promote()` and checks garrison consistency. Source: `CvPlayer.cpp`, `CvPlayer::doTurnUnits`; `CvPlayerAI.cpp`, `AI_doTurnUnitsPre`, `AI_doTurnUnitsPost`.

Movement/tactical/homeland AI is reached later through `CvPlayerAI::AI_unitUpdate(bool bUpdateHomelandAI)`. It fires Lua hook `PlayerPreAIUnitUpdate`, returns if the player has a busy unit/city, then for human unit-AI control updates tactical visibility and homeland AI; for non-human unit AI it calls `CvTacticalAI::Update()`, `CvHomelandAI::Update(true)`, and `CvTacticalAI::CleanUp()`. Source: `CvPlayerAI.cpp`, `CvPlayerAI::AI_unitUpdate`; game loop entry noted in `CvGameCoreDLL_Expansion2/CvGame.cpp`, `CvGame::updateMoves`.

RISK: Moving decision code across the `CvPlayer::doTurnPostDiplomacy()` city-loop boundary changes which same-turn manager state city production sees. Moving movement/purchase behavior between `doTurnPostDiplomacy()` and `AI_unitUpdate()` changes whether normal queue production or tactical emergency purchases happen first.

`CvPlayer` allocates and owns the major AI/state manager pointers in its constructor/setup area, including `m_pPlayerPolicies`, `m_pEconomicAI`, `m_pMilitaryAI`, `m_pCitySpecializationAI`, `m_pWonderProductionAI`, `m_pGrandStrategyAI`, `m_pDiplomacyAI`, `m_pReligionAI`, `m_pTacticalAI`, and `m_pHomelandAI`. Source: `CvPlayer.cpp`, constructor allocation around `m_pPlayerPolicies = FNEW(...)`; member declarations in `CvPlayer.h`.

`CvPlayer::init` initializes these managers with their XML/database entry collections and player pointers: `CvEconomicAI::Init(GC.GetGameEconomicAIStrategies(), this)`, `CvMilitaryAI::Init(GC.GetGameMilitaryAIStrategies(), this, GetDiplomacyAI())`, `CvCitySpecializationAI::Init(GC.GetGameCitySpecializations(), this)`, `CvWonderProductionAI::Init(GC.GetGameBuildings(), this, false)`, `CvGrandStrategyAI::Init(GC.GetGameAIGrandStrategies(), this)`, `CvDiplomacyAI::Init(this)`, `CvReligionAI::Init(GC.GetGameBeliefs(), this)`, `CvPlayerPolicies::Init(GC.GetGamePolicies(), this, false)`, `CvTacticalAI::Init(this)`, and `CvHomelandAI::Init(this)`. Source: `CvPlayer.cpp`, `CvPlayer::init`.

Player-level flavor recipients are attached during init and load. `CvFlavorManager::AddFlavorRecipient` registers `m_pPlayerTechs`, `m_pPlayerPolicies`, and `m_pWonderProductionAI`; on load, `CvPlayer::Serialize` re-adds those recipients with `bPropogateFlavors=false`. Source: `CvPlayer.cpp`, `CvPlayer::init`; `CvPlayer.cpp`, `CvPlayer::Serialize`.

Accessors expose manager/state ownership to downstream AI code:

- `CvPlayer::GetEconomicAI()`

- `CvPlayer::GetMilitaryAI()`

- `CvPlayer::GetCitySpecializationAI()`

- `CvPlayer::GetWonderProductionAI()`

- `CvPlayer::GetGrandStrategyAI()`

- `CvPlayer::GetDiplomacyAI()`

- `CvPlayer::GetReligionAI()`

- `CvPlayer::GetPlayerPolicies()`

- `CvPlayer::GetPlayerTechs()`

- `CvPlayer::GetFlavorManager()`

- `CvPlayer::GetTacticalAI()`

- `CvPlayer::GetHomelandAI()`

Source: `CvPlayer.cpp`, accessor functions; declarations in `CvPlayer.h`.

RISK: `CvPlayer::Serialize` serializes manager objects directly (`visitor(*player.m_pPlayerPolicies)`, `m_pEconomicAI`, `m_pCitySpecializationAI`, `m_pWonderProductionAI`, `m_pMilitaryAI`, `m_pGrandStrategyAI`, `m_pDiplomacyAI`, `m_pReligionAI`, `m_pTacticalAI`, `m_pHomelandAI`) and serializes `m_armyAIs` plus `m_AIOperations`. Manager fields, operation type IDs, enum ordering, and serialized container shapes are savegame-sensitive. Source: `CvPlayer.cpp`, `CvPlayer::Serialize`.

Belief triggers:

- Pantheon-from-first-city trait path: `CvPlayer::foundCity(...)` checks `CvPlayerTraits::StartsWithPantheon`, then non-human religion-choice players call `CvReligionAI::ChoosePantheonBelief(GetID())` and `CvGameReligions::FoundPantheon(GetID(), eBelief)`. Source: `CvPlayer.cpp`, `CvPlayer::foundCity`; `CvReligionClasses.cpp`, `CvReligionAI::ChoosePantheonBelief`, `CvGameReligions::FoundPantheon`.

- Pantheon/reformation turn path: `CvPlayer::doTurnPostDiplomacy()` calls `CvGameReligions::DoPlayerTurn(*this)` after policy/research/espionage. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`; `CvReligionClasses.cpp`, `CvGameReligions::DoPlayerTurn`.

- Policy side-effect reformation path: `CvPlayer::processPolicies(...)` checks `CvPolicyEntry::IsAddReformationBelief()`, owned religion, and missing reformation belief; non-human religion-choice players call `CvReligionAI::ChooseReformationBelief(...)` and `CvGameReligions::AddReformationBelief(...)`. Source: `CvPlayer.cpp`, `CvPlayer::processPolicies`.

Policy/ideology triggers:

- Turn spending path: `CvPlayer::doTurnPostDiplomacy()` calls `CvPlayerPolicies::DoChooseIdeology()` and `CvPlayerPolicies::DoPolicyAI()` for non-human policy-choice players after city turns and culture accumulation. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`.

- Shared application path: `CvPlayer::doAdoptPolicy(...)` validates with `CvPlayer::canAdoptPolicy(...)`, spends free tenets/free policies/culture, calls `CvPlayer::setHasPolicy(...)`, updates next cost, unlocks branch with `CvPlayerPolicies::SetPolicyBranchUnlocked(...)`, and applies notifications/UI dirties and side effects. Source: `CvPlayer.cpp`, `CvPlayer::doAdoptPolicy`, `setHasPolicy`, `processPolicies`.

City production triggers:

- `CvPlayer::doTurnPostDiplomacy()` runs `CvEconomicAI::DoTurn()`, `CvMilitaryAI::DoTurn()`, `CvReligionAI::DoTurn()`, and `CvCitySpecializationAI::DoTurn()` before the city loop, then calls each `CvCity::doTurn()`. Source: `CvPlayer.cpp`, `CvPlayer::doTurnPostDiplomacy`.

- City-local final production selection is outside `CvPlayer`: `CvCity::doProduction(...)` -> `CvCityAI::AI_chooseProduction(...)` -> `CvCityStrategyAI::ChooseProduction(...)`. Source: `CvCity.cpp`, `CvCity::doProduction`; `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, `CvCityAI::AI_chooseProduction`; `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, `CvCityStrategyAI::ChooseProduction`.

Movement/emergency-purchase triggers:

- `CvPlayerAI::AI_unitUpdate(...)` runs tactical and homeland AI after player/city turn processing. `CvTacticalAI::Update()` can later reach tactical emergency purchase paths such as `CvTacticalAI::PlotEmergencyPurchases(...)` and `CvMilitaryAI::BuyEmergencyUnit(...)`. Source: `CvPlayerAI.cpp`, `CvPlayerAI::AI_unitUpdate`; `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, `CvTacticalAI::Update`; `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`, `CvMilitaryAI::BuyEmergencyUnit`.

INFERRED: `CvPlayer` is best documented as the timing and state visibility boundary for these decisions, while `CvPolicyAI`, `CvReligionAI`, `CvCityStrategyAI`, `CvUnitProductionAI`, and related managers are the scoring/selection authorities.

GUIDE_PATTERN: The guide states that `CvGame::doTurn()` runs after human players end their turn. It calls turn-processing functions for world-level systems, then iterates alive players in turn order and activates their turns through `CvPlayer::setTurnActive(true)`. `CvPlayer::doTurn()` handles some AI diplomacy requests; after those are processed, `CvPlayer::doTurnPostDiplomacy()` handles other turn-start work and triggers city turn processing through `CvCity::doTurn()`.

GUIDE_PATTERN: For each AI choice, document whether it occurs:

GUIDE_PATTERN: - During player turn start.

GUIDE_PATTERN: - During city turn processing.

GUIDE_PATTERN: - During production selection.

GUIDE_PATTERN: - During religion processing.

GUIDE_PATTERN: - During policy acquisition/application.

GUIDE_PATTERN: - During tactical/homeland/military/economic AI updates.

## Candidates and gates

- generic unit loop over `GC.GetGameUnits()->GetNumUnits()`, requiring `CvCity::canTrain`; operation/army candidates are separate catalog entries.

## Scoring and inputs

- hard skip reasons from `ProductionSkipReason`, supply caps, bankruptcy/maintenance, valid placement plots, underdeveloped/siege city restrictions, resource availability, role caps, underrepresented unit types, tiny army/navy pressure, city investments, and current-production inertia.

- indirect through `RecommendUnit` and unit/building cached weights; tactical role choice is direct.

- city threat/siege, purchase legality, requested `UnitAITypes`, operation remaining-unit count, treasury withdrawal rules, faith/gold purchase type, and `MOD_AI_UNIT_PRODUCTION` counter-unit semantics.

## Selection and control

After the pre-city managers, `CvPlayer::doTurnPostDiplomacy()` calls `CvPlayerAI::AI_doSpaceshipProduction()` and then loops all cities with `CvCity::doTurn()`.

Inside `CvCity::doTurn()` in `CvGameCoreDLL_Expansion2/CvCity.cpp`, city-local state updates and citizen checks happen before city strategy and city AI hooks. The relevant ordering is `CvCityStrategyAI::DoTurn()` in `CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp`, then `CvCityAI::AI_doTurn()` in `CvGameCoreDLL_Expansion2/CvCityAI.cpp`, then later `CvCity::doProduction(!doCheckProduction())`.

`CvCityStrategyAI::DoTurn()` updates city strategies before production selection. It precomputes yield stats, loops `AICityStrategyTypes`, tests strategy start/end conditions using `CvAICityStrategyEntry`, and strategy changes can affect city flavor state used by production AIs.

`CvCity::doProduction(bool bAllowNoProduction)` chooses production for AI-controlled or automated cities if the city has no production, is producing a process, or `CvCityAI::AI_isChooseProductionDirty()` is true. It calls `CvCityAI::AI_chooseProduction(false, false)`.

`CvCityAI::AI_chooseProduction(...)` handles a special desired-wonder case from `CvCitySpecializationAI::GetNextWonderDesired()` and Great Engineer availability, otherwise calls `CvCityStrategyAI::ChooseProduction(NO_BUILDING, NO_UNIT, bInterruptBuildings, bInterruptWonders)`.

`CvCityStrategyAI::ChooseProduction(...)` in `CvCityStrategyAI.cpp` is the final normal queue selector for units, buildings, projects, and processes. It injects operation/army unit candidates (`CITY_BUILDABLE_UNIT_FOR_OPERATION`, `CITY_BUILDABLE_UNIT_FOR_ARMY`), generic trainable units, constructible buildings, projects, and processes, runs type-specific sanity checks such as `CvUnitProductionAI::CheckUnitBuildSanity(...)` and `CvBuildingProductionAI::CheckBuildingBuildSanity(...)`, sorts `CvWeightedVector<CvCityBuildable>`, selects with `CvWeightedVector::ChooseAbovePercentThreshold(...)`, and applies the selected item through `CvCity::pushOrder(...)` such as `ORDER_TRAIN` or `ORDER_CONSTRUCT`.

## Application and state

- production queue changes, unit/building/project/process progress, unit creation through `CvCity::produce` and `CvCity::CreateUnit`, building creation through `CvCity::CreateBuilding`.

- `CvCity::pushOrder(ORDER_TRAIN, eUnitType, pkUnitInfo->GetDefaultUnitAIType(), ...)`, later `CvCity::produce` and `CvCity::CreateUnit`.

- selected operation unit is queued with `CvCity::pushOrder(ORDER_TRAIN, ...)`; operation queue commitment lifecycle is not fully traced.

## Data and configuration

- `CvUnitEntry`, `CvBuildingEntry`, purchase yield types, unit AI roles, city purchase rules, economic purchase types.

## Version differences, risks and unresolved questions

RISK: Changing candidate ordering, zero-weight behavior, cutoff thresholds, or seed inputs can alter deterministic AI outcomes and multiplayer synchronization.

RISK: Local-looking changes in these functions can affect multiple decisions and score estimators.

RISK: Flavor deltas rely on start/end symmetry and serialized recipient state. Broken symmetry can leave stale AI preferences.

`CvPlayer::doTurnPostDiplomacy` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp` is an ordering hub for player managers, city turns, policy AI, religion turn processing, research, espionage, events, and trade.

RISK: Moving decisions across this ordering boundary can change which same-turn state they observe.

RISK: Operation and army changes can affect both movement behavior and city production demand.

OPEN_QUESTION: The exact engine-level interleaving between `CvPlayer::setTurnActive(true)`, `CvPlayer::doTurn()`, `CvPlayer::doTurnPostDiplomacy()`, `CvPlayer::doTurnUnits()`, and repeated `CvGame::updateMoves()` slices is not fully documented here. The verified source facts are that city production occurs inside `CvPlayer::doTurnPostDiplomacy()` and tactical/homeland movement is called later from `CvGame::updateMoves()` through `CvPlayerAI::AI_unitUpdate(...)`.

OPEN_QUESTION: Active diplomacy can delay `doTurnPostDiplomacy()` through `CvGame::SetWaitingForBlockingInput(m_eID)`, and `CvGame::update()` / waiting diplomacy resume paths should be traced separately before making strict ordering claims for human-blocked diplomacy cases.

OPEN_QUESTION: `CvCitySpecializationAI::DoTurn()` can directly call `CvCity::AI_chooseProduction(...)` for dirty cities before the main city loop. A future production-specific page should trace how often this pre-city interruption changes the queue compared with the later `CvCity::doProduction(...)` call.

OPEN_QUESTION: `CvMilitaryAI::MakeEmergencyPurchases()` occurs before city production, while `CvTacticalAI::PlotEmergencyPurchases(...)` occurs after production during movement. A purchase-focused page should separate pre-production emergency purchases, tactical post-production purchases, economic hurry purchases, and faith purchases.

OPEN_QUESTION: `CvGameReligions::DoPlayerTurn(CvPlayer&)` happens after city production, but `CvReligionAI::DoTurn()` happens before city production. Future belief/religion docs should avoid treating "religion turn" as a single timing point.

RISK: Moving policy/religion/city-production decisions across the `CvPlayer::doTurnPostDiplomacy()` city-loop boundary would change same-turn state visibility. In audited source, policy adoption and `CvGameReligions::DoPlayerTurn(...)` happen after city production, while economic/military/religion faith spending/city specialization happen before it.

- Known improvement hooks: OPEN_QUESTION trace `CommitToBuildNextUnit`, `UncommitToBuildUnit`, and `FinishedBuildingUnit`; verify whether non-operation armies exist.

- Known improvement hooks: OPEN_QUESTION trace `CvCity::PurchaseUnit` side effects and all purchase logging hooks.

- Known improvement hooks: DESIGN_HYPOTHESIS create a flavor signal registry listing producer, consumer, scope, lifetime, scale, and log path for each important flavor.

- RISK `CvPlayer::doTurnPostDiplomacy()` and `CvGame::updateMoves()`: player/city turn decisions and tactical movement/purchase decisions run in different slices. Moving purchase or production logic between these phases changes whether normal queues or emergency purchases see state first.

- OPEN_QUESTION `CvCity::canTrain`, `CvCity::canConstruct`, `CvCity::canCreate`, `CvCity::canMaintain`, and `CvCity::IsCanPurchase` are not fully decomposed in this atlas page. They are legality gates with likely Lua/game-event/resource/schema side effects.

- OPEN_QUESTION purchase-specific logging is partially mapped; a future purchase page should verify every path where gold/faith purchase decisions log or silently apply.

- OPEN_QUESTION `CvHomelandAI` is mapped for movement/worker planning and prophet mission context only at a high level. No direct production requester role is verified in this page.

- OPEN_QUESTION tech selection (`CvPlayerTechs`, `CvTechAI`) uses the same flavor/weighted-vector infrastructure but is outside the requested decision set; avoid importing tech behavior into these AI production/policy/religion docs without a separate source pass.

## Source and dependency lookup

Scope: high-level source map for focused AI decision-making archaeology. Source search used `scripts-ai/find_ai_symbols.sh` and targeted `rg` searches before reading function ranges. `docs-ai/stale-vp-modding-guide-notes.md` was not read.

Labels:
- ``: confirmed in audited source code.
- `INFERRED`: plausible from nearby code, but not yet traced end-to-end.

GUIDE_PATTERN: The recurring shape from `docs-ai/vp-ai-quickref.md` is still useful: candidate generation -> non-negative scoring -> weighted or sorted selection -> application. Audited source must verify each instance because some decisions are deterministic, some are threshold-random, and some are split across player and city managers.

Player-level AI managers create strategic pressure and long-lived state. Important managers include:

Movement AI is reached from `CvGame::updateMoves` through `CvPlayerAI::AI_unitUpdate`. `CvTacticalAI::Update` in `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp` updates visibility, targets, recruited units, and dominance zones. `CvHomelandAI::Update` in `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp` recruits and assigns existing homeland units.

AI decisions are distributed by responsibility rather than by one master AI file:

- Turn orchestration lives in `CvGame`, `CvPlayer`, and `CvPlayerAI`.

INFERRED: A useful archaeology path is to start with a decision, then trace outward in this order: trigger, candidates, scoring, selection, application, logging, save/sync state, and data tables. Starting from a file often hides cross-system ownership.

`CvPlayer::doTurnPostDiplomacy()` is the main pre-city AI manager hub. Before city turns, it calls `UpdateCityThreatCriteria()`, `UpdatePlots()`, `UpdateAreaEffectUnits()`, `UpdateAreaEffectPlots()`, `UpdateDangerPlots()`, invalidates `CvTacticalAnalysisMap`, calls `UpdateMilitaryStats()`, clears war declaration cache, and calls `UpdateCurrentAndFutureWars()`.

For non-barbarian players, `CvPlayer::doTurnPostDiplomacy()` then calls these managers before city `CvCity::doTurn()`:

- `CvEconomicAI::DoTurn()` in `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp`.

- `CvMilitaryAI::DoTurn()` in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`.

- For major civs: `CvReligionAI::DoTurn()`, `CvTradeAI::DoTurn()`, `CvCitySpecializationAI::DoTurn()`, and `CvLeagueAI::DoTurn()`.

- For minor civs: `CvMinorCivAI::DoTurn()`.

`CvGrandStrategyAI::DoTurn()` happens earlier in `CvPlayer::doTurn()`, before `CvPlayer::doTurnPostDiplomacy()`. `CvEconomicAI::DoTurn()` loops economic strategies and can change active strategies and strategy-derived flavor state. `CvMilitaryAI::DoTurn()` calls `ScanForBarbarians()`, `UpdateBaseData()`, `UpdateDefenseState()`, `UpdateMilitaryStrategies()`, `UpdateWarType()`, `UpdateAttackTargets()`, and for AI-controlled units `UpdateOperations()`, `MakeEmergencyPurchases()`, and `DisbandObsoleteUnits()`.

`CvCitySpecializationAI::DoTurn()` runs before the normal city loop. It may choose a desired wonder through `CvWonderProductionAI::ChooseWonder(int& iWonderWeight)`, assign city specializations with `AssignSpecializations()`, and in some dirty/interrupt cases directly call `CvCity::AI_chooseProduction(bool bInterruptWonders, bool bInterruptBuildings)` for cities already producing buildings.

`CvPlayer::doTurnPostDiplomacy()` calls `CvPlayerAI::AI_doTurnPost()`, Lua hook `PlayerDoTurn`, `CvPlayer::doTurnUnits()`, and then returns to the engine/game loop. `CvPlayer::doTurnUnits()` first calls `CvPlayerAI::AI_doTurnUnitsPre()`, then calls `CvUnit::doTurn()` by domain pass, then calls `CvPlayerAI::AI_doTurnUnitsPost()`.

`CvPlayerAI::AI_doTurnUnitsPre()` deletes delayed-death units, kills delayed-death armies, and kills operations whose `CvAIOperation::ShouldAbort()` is true. `CvPlayerAI::AI_doTurnUnitsPost()` handles AI unit promotions through `CvUnit::AI_promote()` and garrison sanity checks.

Tactical/homeland movement AI is reached from `CvGame::updateMoves()` in `CvGame.cpp`, which calls `CvPlayerAI::AI_unitUpdate(bool bUpdateHomelandAI)` when a player is active or has automated units/homeland updates. For non-human AI units, `AI_unitUpdate(...)` calls `CvTacticalAI::Update()`, `CvHomelandAI::Update(true)`, and `CvTacticalAI::CleanUp()`.

`CvTacticalAI::Update()` calls `UpdateVisibility()`, `DropOldFocusAreas()`, `FindTacticalTargets()`, `RecruitUnits()`, and `ProcessDominanceZones()`. `CvTacticalAI::ProcessDominanceZones()` calls `AssignGlobalHighPrioMoves()`, then per-zone `PlotEmergencyPurchases(CvTacticalDominanceZone*)`, target extraction, and tactical move assignment. `CvTacticalAI::PlotOperationalArmyMoves()` calls `CvAIOperation::DoTurn()` for each operation.

`CvHomelandAI::Update(bool bUpdateImprovements)` recruits automated/homeland units, optionally calls `PlanImprovements()` and `PlanWorkerDistribution()`, then runs `FindHomelandTargets()` and `AssignHomelandMoves()` if units are available.

Diplomacy has an early player-turn pass. `CvPlayer::doTurn()` calls `CvGrandStrategyAI::DoTurn()` and then `CvDiplomacyAI::DoTurn(...)`; if an active diplomacy request or UI diplomacy block exists in single-player for AI diplomacy, `CvGame::SetWaitingForBlockingInput(m_eID)` delays `CvPlayer::doTurnPostDiplomacy()`.

Religion has two distinct timings. `CvReligionAI::DoTurn()` runs before city production inside `CvPlayer::doTurnPostDiplomacy()` and spends faith / handles religious purchases for AI faith spending. Later, after city turns, culture/policy processing, science, and espionage, `CvGameReligions::DoPlayerTurn(CvPlayer&)` adds faith per turn, can call `CvReligionAI::ChoosePantheonBelief(...)` and `CvGameReligions::FoundPantheon(...)`, can spawn great prophets, and can call `CvReligionAI::ChooseReformationBelief(...)` / `CvGameReligions::AddReformationBelief(...)`.

Policy and ideology choice happen after city production in `CvPlayer::doTurnPostDiplomacy()`. The ordering is city loop, treasury/gold, culture accumulation, `DoUpdateNextPolicyCost()`, then for AI policy choice `CvPlayerPolicies::IsTimeToChooseIdeology()`, `CvPlayerPolicies::DoChooseIdeology()`, and `CvPlayerPolicies::DoPolicyAI()`. `CvPlayerPolicies::DoPolicyAI()` calls `CvPolicyAI::DoConsiderIdeologySwitch(...)`, then loops `CvPolicyAI::ChooseNextPolicy(...)` and applies branches via `CvPlayerPolicies::DoUnlockPolicyBranch(...)` or policies via `CvPlayer::doAdoptPolicy(...)`.

Production selection is city-local but sees same-turn pre-city manager state. `CvEconomicAI::DoTurn()`, `CvMilitaryAI::DoTurn()`, `CvReligionAI::DoTurn()`, `CvCitySpecializationAI::DoTurn()`, danger-plot updates, and current/future war updates all occur before `CvCity::doTurn()` and `CvCityStrategyAI::ChooseProduction(...)`.

Tactical emergency purchases happen after normal city production, during `CvGame::updateMoves()` via `CvTacticalAI::PlotEmergencyPurchases(...)`, which can call `CvMilitaryAI::BuyEmergencyBuilding(...)` and `CvMilitaryAI::BuyEmergencyUnit(...)`. These bypass the normal city queue chosen by `CvCityStrategyAI::ChooseProduction(...)`.

The following state is produced or refreshed before normal city production:

- Threat, plot, danger, tactical-analysis invalidation, military stats, and current/future war state from `CvPlayer::doTurnPostDiplomacy()`.

- Active grand strategy and grand-strategy priorities from `CvGrandStrategyAI::DoTurn()` and accessors such as `CvGrandStrategyAI::GetPersonalityAndGrandStrategy(...)`.

- Economic strategy state from `CvEconomicAI::DoTurn()`, including recon/antiquity state and economic strategy start/end effects.

- Military strategy, recommended force context, operations, and emergency purchases from `CvMilitaryAI::DoTurn()`, including `UpdateBaseData()`, `UpdateDefenseState()`, `UpdateMilitaryStrategies()`, `UpdateOperations()`, and `MakeEmergencyPurchases()`.

- Desired wonder and city specialization state from `CvCitySpecializationAI::DoTurn()`, including `m_eNextWonderDesired`, `m_iNextWonderWeight`, specialization assignments, and production dirty/interrupt flags.

- City-local strategy state from `CvCityStrategyAI::DoTurn()` before `CvCity::doProduction(...)`.

- Best wonder city flags from `CvPlayer::SetBestWonderCities()` in `CvPlayer::doTurn()` before `CvCitySpecializationAI::DoTurn()` and city production.

INFERRED: Because `CvMilitaryAI::UpdateOperations()` runs before the city loop and `CvCityStrategyAI::ChooseProduction(...)` explicitly considers operation/army unit buildables, normal production can consume operation demand created earlier in the same player turn. The full operation commitment lifecycle is still documented as an open question in `docs-ai/ai-unit-production-upstream.md`.

After city production, `CvPlayer::doTurnPostDiplomacy()` consumes or updates state in these later phases:

- Treasury/gold through `CvTreasury::DoGold()` and vassal tax expense calculation.

- Culture accumulation, `CvPlayer::DoUpdateNextPolicyCost()`, ideology choice, and policy AI through `CvPlayerPolicies::DoChooseIdeology()` and `CvPlayerPolicies::DoPolicyAI()`.

- Research through `CvPlayer::doResearch()`.

- Espionage through `CvEspionage::DoTurn()` and later AI espionage through `CvPlayerAI::AI_doTurnPost()` / `CvEspionageAI::DoTurn()`.

- Religion player turn through `CvGameReligions::DoPlayerTurn(CvPlayer&)`, including pantheon/reformation decisions.

- League player turn, trade player turn, events, replay/yield history, instant-yield notifications, military rating decay, achievements, and unit `CvUnit::doTurn()` processing.

After the player/city turn, `CvGame::updateMoves()` consumes current unit/city/operation/tactical state for movement. `CvPlayerAI::AI_unitUpdate(...)` runs tactical and homeland updates; `CvTacticalAI::PlotEmergencyPurchases(...)` can consume threatened city state and treasury/economic purchase permissions to create immediate purchases after queue production has already been chosen.

Operation and army state is consumed both before and after production. Before production, city selection can read operation/army demand through `CvCity::GetUnitForOperation()`, `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, and `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)`. After production and during movement, `CvTacticalAI::PlotOperationalArmyMoves()` calls `CvAIOperation::DoTurn()`, and `CvPlayerAI::AI_doTurnUnitsPre()` can kill aborted operations.

| Class / manager | Decisions affected |
|---|---|
| `CvTacticalAI` (`CvGameCoreDLL_Expansion2/CvTacticalAI.cpp`, `.h`) | tactical update and `PlotEmergencyPurchases(...)`. Affects emergency purchases after normal city production. |

| Function / enum / table | Decisions affected |
|---|---|
| `CvCity::doTurn()`, `doProduction(...)`, `setProductionAutomated(...)` (`CvCity.cpp`) | city production timing and reselection triggers. |
| `CvPlayer::addAIOperation(...)` (`CvPlayer.cpp`) | operation creation and initial missing-unit cap. |
| `CvTacticalAI::PlotEmergencyPurchases(...)` (`CvTacticalAI.cpp`) | tactical post-production emergency purchase trigger. |

- application: `CvCity::pushOrder`, `popOrder`, `produce`, `CreateUnit`, `CreateBuilding`.

- selection/application: shared city production selector and `CvCity::pushOrder(ORDER_TRAIN, eUnitType, CvUnitEntry::GetDefaultUnitAIType(), ...)`; completion through `CvCity::produce(UnitTypes, UnitAITypes, bool)` and `CvCity::CreateUnit`.

- operation creation: `CvPlayer::addAIOperation`; concrete operation selection in `CreateAIOperation`.

- tactical post-production trigger: `CvTacticalAI::ProcessDominanceZones` -> `CvTacticalAI::PlotEmergencyPurchases`.

- data/enums: `PurchaseType`, `YieldTypes` (`YIELD_GOLD`, `YIELD_FAITH`), `UnitCreationReason` (`REASON_BUY`, `REASON_FAITH_BUY`), `UnitAITypes`, unit/building purchase cost and faith-purchase fields.

| Shared function / concept | Decisions affected | Risk |
|---|---|---|
| `CvCity::PurchaseUnit(...)` / `CvCity::PurchaseBuilding(...)` | emergency purchases, economic hurry, faith purchases, operation unit paths. | RISK immediate creation vs investment/cooldown/event side effects differ from normal queue application. |

`CvPlayer::UpdateDangerPlots()` -> city/unit danger inputs for military, tactical, homeland, and production sanity.

`CvPlayer::UpdateMilitaryStats()` -> player military aggregate state used by diplomacy/military/production logic.

`CvPlayer::UpdateCurrentAndFutureWars()` -> war-context state for military strategies, diplomacy, production and purchase pressure.

`CvPlayer::Serialize(...)` -> save/load behavior for player managers, flavor recipients, armies, operations, policy/religion/military cached state.

GUIDE_PATTERN: The guide uses pantheon choice as an example. If a player can create a pantheon, humans get a UI popup and Lua eventually calls the DLL application function. AI players call `kPlayer.GetReligionAI()->ChoosePantheonBelief(ePlayer)`, then `FoundPantheon(ePlayer, eBelief)`. The important rule is that decision-making differs, but the final effect processing should be shared.

GUIDE_PATTERN: For each documented system, identify:

GUIDE_PATTERN: - The human/UI route.

GUIDE_PATTERN: - The AI route.

GUIDE_PATTERN: - The shared final application function.

GUIDE_PATTERN: - Any places where AI bypasses UI but still uses the same effect code.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
