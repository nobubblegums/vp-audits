# Role strategies and the VP+UPAI profile

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.

Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](../source-snapshots.md).

## Decision or contract question

How do role-specific strategy pressure, formation roles and unit sanity interact?

This is the specialized VP+UPAI and optional AI_UNIT_PRODUCTION reference. Reported compiled-DB rows describe that profile; they are not upstream defaults. Several referenced CSV exports are absent from the recovered artifact.

## Profile reconciliation

Base repository reports describe AI_UNIT_PRODUCTION defaulting to zero and incomplete role-flavor rows in the searched data. The VP+UPAI investigation reports an effective profile with the option enabled and additional strategy/flavor/formation rows. Those claims concern different content profiles; repository absence cannot establish runtime inactivity.

Only `database_tables.txt` survives from the referenced compiled-DB export directory. Preserve reported row findings and profile limits, but do not claim the missing CSVs were independently checked during this migration. Required unfilled slots create demand; most described added skirmisher/submarine slots are optional. Primary/secondary recommendations are fallback-only. Whether need/enough pressure conflicts with operation roles remains a behavior/logging question.

Canonical related owners: [Unit production and recommendation](<production.md>), [Operation and army demand reaching unit production](<operation-demand.md>), [Operation and army state contracts](<../../architecture/operations-armies.md>), [Flavor production, propagation and consumption](<../infrastructure/flavors.md>), [Database loading, options and effective-data provenance](<../../architecture/database-loading.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

AI turn processing invokes military strategy updates and production AI scoring through normal player AI, city production, and tactical AI update loops.

Enable the option with complete role flavor/strategy data, run AI military strategy update, then evaluate unit production choices across land, naval, air, and submarine roles.

## Candidates and gates

Tactical AI consumes `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` in some places. `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp` includes them in a visibility/reveal candidate switch and treats them with ranged units in movement strategy selection. `UNITAI_SKIRMISHER` also affects fast goody-target scoring through `CvUnitEntry::GetUnitAIType(UNITAI_SKIRMISHER)`.

Tactical execution is not a full mirror of formation slots. Movement strategy also uses unit range, domain, movement, ranged-attack properties, and `MOD_AI_UNIT_PRODUCTION && canIntercept()`.

Homeland AI does not directly consume `UNITAI_SKIRMISHER` or `UNITAI_SUBMARINE` in the inspected role distinctions. It does use `MOD_AI_UNIT_PRODUCTION` to exclude `UNITAI_COUNTER` from sentry suitability.

`CvAIOperationMilitary::Init(...)`, `CvAIOperationCityAttackCombined::Init(...)`, `CvAIOperationNavalSuperiority::Init(...)`, `CvAIOperationNukeAttack::Init(...)`, and the civilian operation init paths call `AddArmy(OperationalAIHelpers::GetArmyFormationForOpType(m_eType))` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp`.

`CvAIOperation::AddArmy(...)` creates a `CvArmyAI`, stores its ID in `m_viArmyIDs`, sets operation ID/state/type, and calls `CvArmyAI::SetFormation(...)`. `CvArmyAI::SetFormation(MultiunitFormationTypes)` in `CvGameCoreDLL_Expansion2/CvArmyAI.cpp` releases current units, stores the formation enum, then creates one `CvArmyFormationSlot(-1, required)` per cached formation slot row.

`CvAIOperation::SetUpArmy(...)` sets target/muster plots and initial state. If `pArmyAI->GetOpenSlots(true).empty()`, it starts moving; otherwise it enters `AI_OPERATION_STATE_RECRUITING_UNITS`. It then calls `GrabUnitsFromTheReserves(...)`.

`CvAIOperation::GrabUnitsFromTheReserves(...)` builds reserve choices from existing units, sorts them, clears `m_viListOfUnitsWeStillNeedToBuild`, then iterates all free army slots. `FindBestFitReserveUnit(...)` may fill a slot. If it cannot and `pArmy->GetSlotInfo(iI).m_requiredSlot` is true, it pushes `OperationSlot(m_iID, pArmy->GetID(), iI)` into `m_viListOfUnitsWeStillNeedToBuild`.

optional slots do not enter `m_viListOfUnitsWeStillNeedToBuild` from `GrabUnitsFromTheReserves(...)`. `OperationalAIHelpers::IsUnitSuitableForRecruitment(...)` can match a reserve unit by current `AI_getUnitAIType()`, by slot primary/secondary UnitAI, or by `CvUnitEntry::GetUnitAIType(...)` multi-role eligibility.

RISK: UPAI optional slots can improve use of existing skirmisher/submarine reserves, but they usually do not force queued production because optional unfilled slots are not pushed into `m_viListOfUnitsWeStillNeedToBuild`.

RISK: `MUFORMATION_FAST_PILLAGERS` has two required slots with primary `UNITAI_FAST_ATTACK` and secondary `UNITAI_SKIRMISHER`. The production path tries primary first and secondary only if primary returns `NO_UNIT`; it does not compare a strong skirmisher against a trainable but weak fast-attack candidate.

## Application and state

Downstream effects include active `MILITARYAISTRATEGY_NEED_*` / `ENOUGH_*` strategies, production scoring results from `CvUnitProductionAI`, emergency purchase choice, and role-flavor aggregation for advisors and grand strategy.

## Data and configuration

`CvDllDatabaseUtility::PrefetchGlobalDefines()` loads `MultiUnitFormations` with `PrefetchCollection(GC.getMultiUnitFormationInfo(), "MultiUnitFormations")` in `CvGameCoreDLL_Expansion2/CvDllDatabaseUtility.cpp`. `CvDllDatabaseUtility::ValidatePrefetchProcess()` checks `getNumMultiUnitFormationInfos`.

`CvMultiUnitFormationInfo::CacheResults(Database::Results&, CvDatabaseUtility&)` in `CvGameCoreDLL_Expansion2/CvInfos.cpp` reads the parent formation row, stores `Name` in `m_strFormationName`, then selects `MultiUnitFormation_SlotEntries` by `MultiUnitFormationType = GetType()`. Each slot row is cached as `CvFormationSlotEntry` fields declared in `CvGameCoreDLL_Expansion2/CvInfos.h`: `m_requiredSlot`, `m_ePositionType`, `m_primaryUnitType`, and `m_secondaryUnitType`.

slot type strings are resolved with `GC.getInfoTypeForString(..., true)` into open enums: `MultiunitPositionTypes` and `UnitAITypes`. `CvMultiUnitFormationInfo::getNumFormationSlotEntriesRequired()` counts rows where `m_requiredSlot` is true.

RISK: `MultiunitFormationTypes` and `UnitAITypes` are open enum/data mappings. `CvGameCoreDLLUtil/include/CvEnums.h` lists `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` as "New UNITAI for MOD_AI_UNIT_PRODUCTION", and lists the hardcoded `MUFORMATION_*` enum order. Reordering or deleting rows/enums is savegame/sync-sensitive.

### UPAI Strategy And Flavor Data

REPORTED_DB_SNAPSHOT: `CustomModOptions.csv` in the VP+UPAI snapshot has `AI_UNIT_PRODUCTION,1,6,0`, so `MOD_AI_UNIT_PRODUCTION`-gated source paths should be treated as active for this compiled data set.

REPORTED_DB_SNAPSHOT: `Flavors.csv` defines the UPAI role flavors consumed by the source: `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, and `FLAVOR_FIGHTER`.

REPORTED_DB_SNAPSHOT: `AIMilitaryStrategies.csv` defines need/enough strategies for the same role split: archer, siege, skirmisher, naval melee, naval ranged, submarine, fighter, and bomber. `AIMilitaryStrategy_Player_Flavors.csv` assigns +/-15 player flavor deltas, and `AIMilitaryStrategy_City_Flavors.csv` assigns +/-150 city flavor deltas for those role flavors.

`CvMilitaryAI::UpdateMilitaryStrategies()` in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp` evaluates UPAI role-specific strategy tests only when `MOD_AI_UNIT_PRODUCTION` is enabled. `CvUnitProductionAI::CheckUnitBuildSanity(...)` in `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp` then adds a role-strategy bonus or returns `SR_BALANCE` for enough-role strategies for default UnitAI roles including `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE`.

INFERRED: For VP+UPAI, operation formation slots and generic city-production role strategies are both active sources of skirmisher/submarine demand. Formation slots mainly affect explicit army composition; role strategies affect generic unit scoring and sanity filtering.

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1131` with default `Value="0"` under the XML section labeled "SLATED FOR DELETION". Macro is `MOD_AI_UNIT_PRODUCTION` in `CvGameCoreDLL_Expansion2/CustomMods.h:1019`; declaration is `CustomMods.h:1793`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:570`.

The option gates military role strategy activation, unit production sanity/scoring, tactical emergency purchases, tactical placement, economic explorer classification, advisor flavor classification, grand-strategy flavor classification, and homeland sentry suitability.

## Observability and validation obligations

DESIGN_HYPOTHESIS: To test the claims above, logging should expose the full formation-to-production chain, not only the final city build.

Recommended fields:

| Location | Needed fields |
|---|---|
| `CvAIOperation::AddArmy(...)` / `CvArmyAI::SetFormation(...)` | player, operation ID/type, army ID, formation enum/name, total/required/optional slot counts |
| `CvAIOperation::GrabUnitsFromTheReserves(...)` | operation ID/type, army ID, slot index, required flag, position, primary UnitAI, secondary UnitAI, chosen reserve unit ID/type/default UnitAI/current UnitAI, no-fill reason |
| `m_viListOfUnitsWeStillNeedToBuild` updates | operation ID/type, army ID, slot index, required flag, primary/secondary UnitAI, queue size before/after |
| `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)` | city, operation ID/type, muster city/plot, bSameCity, access result, selected slot, skipped operation reason |
| `CvCity::GetUnitForOperation()` | city, operation ID/type, formation, slot index, primary/secondary UnitAI, primary returned unit, secondary returned unit, final unit |
| `CvMilitaryAI::GetUnitTypeForArmy(...)` | city, army ID, operation ID if present, open required slots, least-open-slots choice, primary/secondary candidates |
| `CvUnitProductionAI::RecommendUnit(...)` | requested UnitAI, candidate unit, default UnitAI, all matching UnitAI flags relevant to the slot, canTrain result, strategic-resource filter result, raw flavor weight, fallback flavor sum, turns, final weighted score |
| `CvCityStrategyAI::ChooseProduction(...)` | operation/army buildable type, unit, source operation/army/slot, precheck weight, sanity result, final selected buildable, skipped-operation counter |
| Tactical/homeland assignment | unit ID/type/default/current UnitAI, originating operation/army/slot if available, tactical move type, movement strategy, final plot/role |

RISK: Existing operation and tactical logs record some army/unit IDs but not enough slot metadata. Without primary/secondary UnitAI, required flag, selected formation, and primary-vs-secondary candidate results, it will be hard to distinguish "UPAI composition improved" from "existing broad role happened to fill the slot."

Log all role flavor IDs, strategy IDs, active need/enough strategies, `CheckUnitBuildSanity` role decisions, and final production score.

## Version differences, risks and unresolved questions

DESIGN_HYPOTHESIS: Improved army composition. VP+UPAI formations add explicit skirmisher and submarine slots, and `CvUnitProductionAI::RecommendUnit(...)` can request those UnitAI roles directly. This should make reserve recruitment and special operation/army production less dependent on broad `UNITAI_RANGED`, `UNITAI_FAST_ATTACK`, or `UNITAI_RESERVE_SEA` buckets.

DESIGN_HYPOTHESIS: Limited operation-production impact for optional UPAI slots. Most skirmisher/submarine formation rows are optional, and `GrabUnitsFromTheReserves(...)` only pushes unfilled required slots into `m_viListOfUnitsWeStillNeedToBuild`. These roles can improve recruited reserves and completed army shape, but usually do not create mandatory production demand.

RISK: Harder-to-fill required slots exist where UPAI roles are secondary rather than primary. `MUFORMATION_FAST_PILLAGERS` requires two `UNITAI_FAST_ATTACK/UNITAI_SKIRMISHER` slots. If trainable fast-attack units are poor fits or unavailable, fallback only occurs when no primary candidate is trainable; it is not a scored primary-vs-secondary competition.

RISK: Stalled or aborted operations are most likely from required slots whose primary and secondary roles have no trainable/recruitable candidate and from operations created with low `iMaxMissingUnits`. `CvPlayer::addAIOperation(...)` aborts if `GetNumUnitsNeededToBeBuilt() > iMaxMissingUnits`. For ongoing recruitment, required missing slots remain in `m_viListOfUnitsWeStillNeedToBuild`.

RISK: Poor secondary fallback behavior. Both `CvCity::GetUnitForOperation()` and `CvMilitaryAI::GetUnitTypeForArmy(...)` try the primary role first and never inspect secondary if any primary candidate is returned. This can suppress UPAI secondary roles even when they would produce a better or faster unit.

RISK: Formation recruitment and tactical execution can mismatch. Formation slots ask for UnitAI roles, but tactical movement later often places units by range/domain/movement and only partially by UnitAI. Skirmisher slots likely line up with second-line/mobile behavior; submarine slots are treated as ranged naval in movement strategy, but broader naval operation behavior may not preserve a special submarine mission identity.

REPORTED_DB_SNAPSHOT: The VP+UPAI snapshot has `AI_UNIT_PRODUCTION = 1`, so gated source behavior for role-specific production strategies, submarine naval exploration, counter-unit sentry changes, and AA backline tactical handling is expected to be active for this data set.

Local `git log -S'MOD_AI_UNIT_PRODUCTION'` finds production-source provenance at `54dee87` only. Later local commits include documentation/snapshot work such as `8f949d1`, but no production rewrite is visible.

Unit AI enum ordering and savegame compatibility are sensitive because role enum entries exist in `CvEnums.h`. The option itself does not change compile-time enum layout.

DEPENDENCY_UNSAFE: Missing database rows for hardcoded flavors or strategies may make guarded logic inert, skew scores, or activate no strategies.

## Source and dependency lookup

### Operations/formations/UPAI

1. How often do UPAI-related required slots actually stall or abort operations in controlled runs?

2. How often does primary-first UnitAI fallback choose a worse primary candidate while suppressing a better secondary candidate?

3. In real tactical execution, do skirmisher/submarine formation slots produce different battlefield behavior or do range/domain heuristics erase most of the distinction?

4. How often do operation slot demand and active need/enough strategies conflict in production scoring?

`OperationalAIHelpers::GetArmyFormationForOpType(AIOperationTypes)` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp` is the authoritative operation-to-formation map.

Direct mappings:

| Operation | Formation |
|---|---|
| `AI_OPERATION_FOUND_CITY` | `MUFORMATION_SETTLER_ESCORT` |
| `AI_OPERATION_MUSICIAN_CONCERT_TOUR` | `MUFORMATION_CONCERT_TOUR` |
| `AI_OPERATION_DIPLOMAT_DELEGATION` | `MUFORMATION_DIPLOMAT_ESCORT` |
| `AI_OPERATION_MERCHANT_DELEGATION` | `MUFORMATION_MERCHANT_ESCORT` |
| `AI_OPERATION_PILLAGE_ENEMY` | `MUFORMATION_FAST_PILLAGERS` |
| `AI_OPERATION_NUKE_ATTACK` | `MUFORMATION_NUKE_ATTACK` |
| `AI_OPERATION_CARRIER_GROUP` | `MUFORMATION_CARRIER_ESCORT` |
| `AI_OPERATION_NAVAL_SUPERIORITY` | `MUFORMATION_NAVAL_SQUADRON` |
| `AI_OPERATION_RAPID_RESPONSE` | `MUFORMATION_RAPID_RESPONSE_FORCE` |
| `AI_OPERATION_CITY_DEFENSE` | `MUFORMATION_CLOSE_CITY_DEFENSE` |

Era-dependent mappings use helpers in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`:

- `AI_OPERATION_CITY_ATTACK_LAND`: Ancient `MUFORMATION_EARLY_RUSH`; Classical/Medieval `MUFORMATION_SMALL_CITY_ATTACK_FORCE`; Renaissance/Industrial `MUFORMATION_BASIC_CITY_ATTACK_FORCE`; Modern/Atomic/Information `MUFORMATION_BIGGER_CITY_ATTACK_FORCE`.

- `AI_OPERATION_CITY_ATTACK_COMBINED`: through Medieval `MUFORMATION_NAVAL_INVASION`; after Medieval `MUFORMATION_LATE_GAME_INVASION_FORCE`.

- `AI_OPERATION_CITY_ATTACK_NAVAL`: through Medieval `MUFORMATION_PURE_NAVAL_CITY_ATTACK`; after Medieval `MUFORMATION_LATE_GAME_PURE_NAVAL_FORCE`.

### Which formation slots use UPAI-added `UnitAITypes`?

REPORTED_DB_SNAPSHOT: `UNITAI_SKIRMISHER` appears in formation slots for `MUFORMATION_EARLY_RUSH`, `MUFORMATION_SMALL_CITY_ATTACK_FORCE`, `MUFORMATION_BASIC_CITY_ATTACK_FORCE`, `MUFORMATION_BIGGER_CITY_ATTACK_FORCE`, `MUFORMATION_CITY_STATE_ATTACK_FORCE`, `MUFORMATION_FAST_PILLAGERS`, `MUFORMATION_ANTI_BARBARIAN_TEAM`, `MUFORMATION_NAVAL_INVASION`, `MUFORMATION_CITY_STATE_INVASION`, `MUFORMATION_RAPID_RESPONSE_FORCE`, and `MUFORMATION_LATE_GAME_INVASION_FORCE`.

REPORTED_DB_SNAPSHOT: `UNITAI_SUBMARINE` appears in formation slots for `MUFORMATION_NAVAL_SQUADRON`, `MUFORMATION_NAVAL_INVASION`, `MUFORMATION_NAVAL_BOMBARDMENT`, `MUFORMATION_LATE_GAME_INVASION_FORCE`, and `MUFORMATION_LATE_GAME_PURE_NAVAL_FORCE`.

REPORTED_DB_SNAPSHOT: For operation-relevant formations, most skirmisher/submarine slots are optional. The notable required UPAI-related case is `MUFORMATION_FAST_PILLAGERS`, which has two required `UNITAI_FAST_ATTACK / UNITAI_SKIRMISHER` slots where skirmisher is secondary.

### Are UPAI formation slots too specific, causing operation stalls?

Unfilled required slots become build requests; unfilled optional slots do not. `CvAIOperation::GrabUnitsFromTheReserves(...)` pushes only unfilled required slots into `m_viListOfUnitsWeStillNeedToBuild`.

REPORTED_DB_SNAPSHOT: Most UPAI-added skirmisher/submarine formation slots in operation-relevant formations are optional, so they should not usually force operation production or stalls by themselves.

RISK: Required slots can still stall or abort operations when no reserve or trainable unit can satisfy the slot and `iMaxMissingUnits` is exceeded in `CvPlayer::addAIOperation(...)`. `MUFORMATION_FAST_PILLAGERS` is the clearest UPAI-adjacent risk because its required slots request primary `UNITAI_FAST_ATTACK` and secondary `UNITAI_SKIRMISHER`.

INFERRED: The source/data answer reduces the stall concern for most UPAI slots, but actual stall frequency remains a behavior-log question.

Secondary roles are fallback-only. `CvCity::GetUnitForOperation()` and `CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` call `CvUnitProductionAI::RecommendUnit(primary, true)` first and only try `secondary` if primary returns `NO_UNIT`.

RISK: This is not a scored primary-vs-secondary comparison. A weak, slow, or tactically awkward primary candidate can block a stronger secondary candidate as long as the primary role has any trainable recommendation.

Operation demand is slot-specific. Required unfilled formation slots enter `m_viListOfUnitsWeStillNeedToBuild`, reach `CvCity::GetUnitForOperation()`, and request primary/secondary UnitAI roles through `CvUnitProductionAI::RecommendUnit(...)`.

Generic role pressure is strategy-specific. With `MOD_AI_UNIT_PRODUCTION` enabled, `CvMilitaryAI::UpdateMilitaryStrategies()` evaluates role need/enough strategies, and `CvUnitProductionAI::CheckUnitBuildSanity(...)` adds bonuses or returns `SR_BALANCE` by default UnitAI role.

REPORTED_DB_SNAPSHOT: In VP+UPAI both mechanisms are active because `AI_UNIT_PRODUCTION = 1` and UPAI role flavors/strategies exist in the compiled snapshot.

INFERRED: The two mechanisms reinforce each other when an operation slot and active need strategy point at the same role. They can conflict when a formation slot asks for a role while generic enough-role strategy suppresses that unit's default UnitAI, or when primary formation fallback blocks a better secondary role.

`OperationalAIHelpers::GetArmyFormationForOpType(AIOperationTypes eType)` in `CvGameCoreDLL_Expansion2/CvAIOperation.cpp` maps operation types to formations. Most mappings are direct, but city-attack operations call era helpers in `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp`.

| Operation | Selected formation |
|---|---|
| `AI_OPERATION_FOUND_CITY` | `MUFORMATION_SETTLER_ESCORT` |
| `AI_OPERATION_MUSICIAN_CONCERT_TOUR` | `MUFORMATION_CONCERT_TOUR` |
| `AI_OPERATION_DIPLOMAT_DELEGATION` | `MUFORMATION_DIPLOMAT_ESCORT` |
| `AI_OPERATION_MERCHANT_DELEGATION` | `MUFORMATION_MERCHANT_ESCORT` |
| `AI_OPERATION_PILLAGE_ENEMY` | `MUFORMATION_FAST_PILLAGERS` |
| `AI_OPERATION_NUKE_ATTACK` | `MUFORMATION_NUKE_ATTACK` |
| `AI_OPERATION_CARRIER_GROUP` | `MUFORMATION_CARRIER_ESCORT` |
| `AI_OPERATION_CITY_ATTACK_LAND` | `MilitaryAIHelpers::GetCurrentBestFormationTypeForLandAttack()` |
| `AI_OPERATION_CITY_ATTACK_COMBINED` | `MilitaryAIHelpers::GetCurrentBestFormationTypeForCombinedAttack()` |
| `AI_OPERATION_CITY_ATTACK_NAVAL` | `MilitaryAIHelpers::GetCurrentBestFormationTypeForPureNavalAttack()` |
| `AI_OPERATION_NAVAL_SUPERIORITY` | `MUFORMATION_NAVAL_SQUADRON` |
| `AI_OPERATION_RAPID_RESPONSE` | `MUFORMATION_RAPID_RESPONSE_FORCE` |
| `AI_OPERATION_CITY_DEFENSE` | `MUFORMATION_CLOSE_CITY_DEFENSE` |

`GetCurrentBestFormationTypeForLandAttack()` returns `MUFORMATION_EARLY_RUSH` in Ancient, `MUFORMATION_SMALL_CITY_ATTACK_FORCE` in Classical/Medieval, `MUFORMATION_BASIC_CITY_ATTACK_FORCE` in Renaissance/Industrial, and `MUFORMATION_BIGGER_CITY_ATTACK_FORCE` in Modern/Atomic/Information. `GetCurrentBestFormationTypeForCombinedAttack()` returns `MUFORMATION_NAVAL_INVASION` through Medieval and `MUFORMATION_LATE_GAME_INVASION_FORCE` after Medieval. `GetCurrentBestFormationTypeForPureNavalAttack()` returns `MUFORMATION_PURE_NAVAL_CITY_ATTACK` through Medieval and `MUFORMATION_LATE_GAME_PURE_NAVAL_FORCE` after Medieval.

### Formation Slots In VP+UPAI

REPORTED_DB_SNAPSHOT: The relevant formation slot rows in `MultiUnitFormation_SlotEntries.csv` are:

| Formation | Slots | Required | Optional | Slot composition |
|---|---:|---:|---:|---|
| `MUFORMATION_SETTLER_ESCORT` | 2 | 2 | 0 | R x1 `UNITAI_SETTLE/UNITAI_SETTLE`; R x1 `UNITAI_DEFENSE/UNITAI_ATTACK` |
| `MUFORMATION_CONCERT_TOUR` | 2 | 1 | 1 | R x1 `UNITAI_MUSICIAN/UNITAI_MUSICIAN`; O x1 `UNITAI_DEFENSE/UNITAI_COUNTER` |
| `MUFORMATION_DIPLOMAT_ESCORT` | 2 | 1 | 1 | R x1 `UNITAI_DIPLOMAT/UNITAI_DIPLOMAT`; O x1 `UNITAI_DEFENSE/UNITAI_COUNTER` |
| `MUFORMATION_MERCHANT_ESCORT` | 2 | 1 | 1 | R x1 `UNITAI_MERCHANT/UNITAI_MERCHANT`; O x1 `UNITAI_DEFENSE/UNITAI_COUNTER` |
| `MUFORMATION_FAST_PILLAGERS` | 3 | 2 | 1 | R x2 `UNITAI_FAST_ATTACK/UNITAI_SKIRMISHER`; O x1 `UNITAI_PARADROP/UNITAI_EXPLORE` |
| `MUFORMATION_NUKE_ATTACK` | 1 | 1 | 0 | R x1 `UNITAI_ICBM/UNITAI_ICBM` |
| `MUFORMATION_CARRIER_ESCORT` | 7 | 4 | 3 | R x1 `UNITAI_CARRIER_SEA/UNITAI_CARRIER_SEA`; R x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; R x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ADMIRAL/UNITAI_ADMIRAL` |
| `MUFORMATION_EARLY_RUSH` | 10 | 4 | 6 | R x1 `UNITAI_ATTACK/UNITAI_DEFENSE`; R x1 `UNITAI_DEFENSE/UNITAI_ATTACK`; R x2 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; O x1 `UNITAI_FAST_ATTACK/UNITAI_ATTACK`; O x2 `UNITAI_RANGED/UNITAI_CITY_BOMBARD`; O x1 `UNITAI_SKIRMISHER/UNITAI_SKIRMISHER`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL`; O x1 `UNITAI_CITY_SPECIAL/UNITAI_CITY_SPECIAL` |
| `MUFORMATION_SMALL_CITY_ATTACK_FORCE` | 10 | 4 | 6 | R x2 `UNITAI_ATTACK/UNITAI_DEFENSE`; R x1 `UNITAI_CITY_BOMBARD/UNITAI_CITY_BOMBARD`; R x1 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; O x1 `UNITAI_ATTACK/UNITAI_DEFENSE`; O x1 `UNITAI_ATTACK/UNITAI_FAST_ATTACK`; O x1 `UNITAI_FAST_ATTACK/UNITAI_ATTACK`; O x1 `UNITAI_SKIRMISHER/UNITAI_RANGED`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL`; O x1 `UNITAI_CITY_SPECIAL/UNITAI_CITY_SPECIAL` |
| `MUFORMATION_BASIC_CITY_ATTACK_FORCE` | 13 | 5 | 8 | R x3 `UNITAI_ATTACK/UNITAI_DEFENSE`; R x2 `UNITAI_CITY_BOMBARD/UNITAI_CITY_BOMBARD`; O x1 `UNITAI_ATTACK/UNITAI_FAST_ATTACK`; O x1 `UNITAI_FAST_ATTACK/UNITAI_ATTACK`; O x1 `UNITAI_RANGED/UNITAI_CITY_BOMBARD`; O x2 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; O x1 `UNITAI_SKIRMISHER/UNITAI_RANGED`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL`; O x1 `UNITAI_CITY_SPECIAL/UNITAI_CITY_SPECIAL` |
| `MUFORMATION_BIGGER_CITY_ATTACK_FORCE` | 15 | 7 | 8 | R x4 `UNITAI_ATTACK/UNITAI_DEFENSE`; R x2 `UNITAI_CITY_BOMBARD/UNITAI_CITY_BOMBARD`; R x1 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; O x1 `UNITAI_ATTACK/UNITAI_FAST_ATTACK`; O x1 `UNITAI_FAST_ATTACK/UNITAI_ATTACK`; O x1 `UNITAI_RANGED/UNITAI_CITY_BOMBARD`; O x1 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; O x1 `UNITAI_COUNTER/UNITAI_COUNTER`; O x1 `UNITAI_SKIRMISHER/UNITAI_RANGED`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL`; O x1 `UNITAI_CITY_SPECIAL/UNITAI_CITY_SPECIAL` |
| `MUFORMATION_NAVAL_INVASION` | 17 | 9 | 8 | R x2 `UNITAI_ATTACK/UNITAI_DEFENSE`; R x1 `UNITAI_ATTACK/UNITAI_FAST_ATTACK`; R x2 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; R x1 `UNITAI_ATTACK_SEA/UNITAI_ASSAULT_SEA`; R x1 `UNITAI_ATTACK_SEA/UNITAI_ATTACK_SEA`; R x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_CARRIER_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_SUBMARINE/UNITAI_RESERVE_SEA`; O x1 `UNITAI_FAST_ATTACK/UNITAI_ATTACK`; O x1 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; O x1 `UNITAI_SKIRMISHER/UNITAI_RANGED`; O x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ADMIRAL/UNITAI_ADMIRAL`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL` |
| `MUFORMATION_LATE_GAME_INVASION_FORCE` | 19 | 9 | 10 | R x1 `UNITAI_DEFENSE/UNITAI_ATTACK`; R x1 `UNITAI_ATTACK/UNITAI_DEFENSE`; R x1 `UNITAI_ATTACK/UNITAI_FAST_ATTACK`; R x1 `UNITAI_CITY_BOMBARD/UNITAI_CITY_BOMBARD`; R x1 `UNITAI_CITY_BOMBARD/UNITAI_RANGED`; R x2 `UNITAI_ATTACK_SEA/UNITAI_ASSAULT_SEA`; R x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_SUBMARINE/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ADMIRAL/UNITAI_ADMIRAL`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL`; O x1 `UNITAI_FAST_ATTACK/UNITAI_ATTACK`; O x1 `UNITAI_COUNTER/UNITAI_COUNTER`; O x1 `UNITAI_SKIRMISHER/UNITAI_RANGED`; O x1 `UNITAI_ATTACK_SEA/UNITAI_ASSAULT_SEA`; O x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA` |
| `MUFORMATION_PURE_NAVAL_CITY_ATTACK` | 8 | 4 | 4 | R x2 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; R x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_CARRIER_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ADMIRAL/UNITAI_ADMIRAL` |
| `MUFORMATION_LATE_GAME_PURE_NAVAL_FORCE` | 12 | 5 | 7 | R x3 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; R x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x3 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; O x2 `UNITAI_SUBMARINE/UNITAI_SUBMARINE`; O x1 `UNITAI_ADMIRAL/UNITAI_ADMIRAL` |
| `MUFORMATION_NAVAL_SQUADRON` | 7 | 3 | 4 | R x1 `UNITAI_ATTACK_SEA/UNITAI_RESERVE_SEA`; R x1 `UNITAI_ATTACK_SEA/UNITAI_ASSAULT_SEA`; R x1 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x2 `UNITAI_ASSAULT_SEA/UNITAI_RESERVE_SEA`; O x1 `UNITAI_SUBMARINE/UNITAI_RESERVE_SEA`; O x1 `UNITAI_ADMIRAL/UNITAI_ADMIRAL` |
| `MUFORMATION_RAPID_RESPONSE_FORCE` | 4 | 2 | 2 | R x2 `UNITAI_DEFENSE/UNITAI_ATTACK`; O x1 `UNITAI_FAST_ATTACK/UNITAI_SKIRMISHER`; O x1 `UNITAI_SKIRMISHER/UNITAI_FAST_ATTACK` |
| `MUFORMATION_CLOSE_CITY_DEFENSE` | 8 | 3 | 5 | R x2 `UNITAI_DEFENSE/UNITAI_ATTACK`; R x1 `UNITAI_RANGED/UNITAI_DEFENSE`; O x1 `UNITAI_COUNTER/UNITAI_COUNTER`; O x1 `UNITAI_ATTACK/UNITAI_DEFENSE`; O x2 `UNITAI_RANGED/UNITAI_DEFENSE`; O x1 `UNITAI_GENERAL/UNITAI_GENERAL` |

REPORTED_DB_SNAPSHOT: UPAI-added UnitAI slot use is mostly optional. `UNITAI_SKIRMISHER` appears in optional slots for the land city-attack formations, mixed naval invasions, rapid response, and in required secondary slots for `MUFORMATION_FAST_PILLAGERS`. `UNITAI_SUBMARINE` appears only in optional slots in the relevant operation formations: `MUFORMATION_NAVAL_SQUADRON`, `MUFORMATION_NAVAL_INVASION`, `MUFORMATION_LATE_GAME_INVASION_FORCE`, and `MUFORMATION_LATE_GAME_PURE_NAVAL_FORCE`.

REPORTED_DB_SNAPSHOT: `UnitAIInfos.csv` contains `UNITAI_SKIRMISHER` at ID 44 and `UNITAI_SUBMARINE` at ID 45. `Unit_AITypes.csv` maps `UNITAI_SUBMARINE` to `UNIT_SUBMARINE`, `UNIT_NUCLEAR_SUBMARINE`, and `UNIT_ATTACK_SUBMARINE`. It maps `UNITAI_SKIRMISHER` to skirmisher-line and related mobile ranged units including `UNIT_CHARIOT_ARCHER`, `UNIT_HEAVY_SKIRMISHER`, `UNIT_CUIRASSIER`, `UNIT_CAVALRY`, `UNIT_HELICOPTER_GUNSHIP`, `UNIT_ANTI_TANK_GUN`, and multiple unique replacements.

`CvAIOperation::PeekAtNextUnitToBuild()` returns the front of `m_viListOfUnitsWeStillNeedToBuild`. `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(CvCity*, bool&)` loops player operations, requires a muster plot, checks city/muster access and naval access, peeks the operation slot, verifies the army and free slot, and returns the first required slot found via `OperationalAIHelpers::IsSlotRequired(...)`.

`CvCity::GetUnitForOperation()` calls `CvPlayerAI::PeekAtNextUnitToBuildForOperationSlot(...)`, requires `bSameCity`, verifies landmass/ocean access, reads `CvArmyAI::GetSlotInfo(slotId)`, then calls `CvUnitProductionAI::RecommendUnit(primary, true)`. It calls `RecommendUnit(secondary, true)` only if the primary returns `NO_UNIT`.

`CvMilitaryAI::GetUnitTypeForArmy(CvCity*)` scans all `CvArmyAI` objects, gets required open slots with `CvArmyAI::GetOpenSlots(true)`, reads `CvArmyAI::GetSlotInfo(...)`, and tries primary then secondary through `CvUnitProductionAI::RecommendUnit(...)`. It returns a trainable unit from the army with the fewest required open slots.

`CvUnitProductionAI::RecommendUnit(UnitAITypes, bool)` filters units by `CvUnitEntry::GetUnitAIType(eUnitAIType)`, optional strategic-resource rejection, and `CvCity::canTrain(...)`. It weights candidates by cached unit flavor weight, falls back to the sum of unit flavor rows if the cached weight is zero, reweights by turns left, stable-sorts, logs, and returns the top unit.

`CvCityStrategyAI::ChooseProduction(...)` injects `CITY_BUILDABLE_UNIT_FOR_OPERATION` before generic units if `CvCity::GetUnitForOperation()` returns a unit. It injects `CITY_BUILDABLE_UNIT_FOR_ARMY` if `CvMilitaryAI::GetUnitTypeForArmy(...)` returns a unit. Both later pass through `CvUnitProductionAI::CheckUnitBuildSanity(...)`.

In the normal production sanity pass, `CITY_BUILDABLE_UNIT_FOR_OPERATION` and `CITY_BUILDABLE_UNIT_FOR_ARMY` share a `PeekAtNextUnitToBuildForOperationSlot(...)` and same-muster requirement before `CheckUnitBuildSanity(..., bForOperation=true, ...)`. This means the army-fill candidate is still constrained by the current operation slot check in normal queued production.

Final selection in `CvCityStrategyAI::ChooseProduction(...)` competes against other buildables. If a unit wins, `CvCity::pushOrder(ORDER_TRAIN, eUnitType, pkUnitInfo->GetDefaultUnitAIType(), ...)` queues the unit. `CvUnitEntry::GetDefaultUnitAIType()` comes from the `Units.DefaultUnitAI` column loaded in `CvUnitEntry::CacheResults(...)`; multi-role eligibility comes from `Unit_AITypes` loaded into `m_pbUnitAIType`.

### Tactical And Homeland Consumption Of UPAI Roles

tactical AI does consume `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` after recruitment/production, but not everywhere formation recruitment distinguishes them.

`CvTacticalAI.cpp` includes `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` in a visibility/reveal candidate switch, includes `UNITAI_SKIRMISHER` in fast goody-target scoring via `GetUnitAIType(UNITAI_SKIRMISHER)`, and treats `UNITAI_SKIRMISHER`/`UNITAI_SUBMARINE` with ranged units in movement strategy selection. In that movement strategy code, range greater than 2 or `MOD_AI_UNIT_PRODUCTION && canIntercept()` becomes third line, range 2 becomes second line, and otherwise ranged/mobile attack properties decide first/second/third line.

`CvEconomicAI.cpp` treats `UNITAI_SUBMARINE` as a potential naval explorer only when `MOD_AI_UNIT_PRODUCTION` is enabled.

`CvHomelandAI.cpp` does not consume `UNITAI_SKIRMISHER` or `UNITAI_SUBMARINE` directly in the inspected role distinctions. It changes sentry suitability by excluding `UNITAI_COUNTER` when `MOD_AI_UNIT_PRODUCTION` is enabled.

INFERRED: Formation recruitment uses slot primary/secondary roles as composition requests, while tactical execution often falls back to unit range, domain, movement, intercept capability, and broad combat behavior. The role distinction survives into some tactical choices, but execution is not a pure mirror of formation slot roles.

Direct references are `CustomMods.h:1019`, `CustomMods.h:1793`, `CustomMods.cpp:570`, `NewCustomModOptions.xml:1131`, `CvTacticalAI.cpp:1780`, `CvTacticalAI.cpp:10826`, `CvMilitaryAI.cpp:2061`, `CvMilitaryAI.cpp:2063`, `CvMilitaryAI.cpp:2085`, `CvMilitaryAI.cpp:3925`, `CvMilitaryAI.cpp:3941`, `CvUnitProductionAI.cpp:696`, `CvUnitProductionAI.cpp:1580`, `CvEconomicAI.cpp:1053`, `CvAdvisorRecommender.cpp:394`, `CvGrandStrategyAI.cpp:648`, and `CvHomelandAI.cpp:6338`.

`CvMilitaryAI::UpdateMilitaryStrategies` suppresses default ranged need/enough strategy handling when enabled at `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp:2061` and activates role-specific need/enough checks at `CvMilitaryAI.cpp:2085`.

`CvUnitProductionAI::CheckUnitBuildSanity` changes escort scoring at `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp:696` and applies role-based scoring and balance rejection at `CvUnitProductionAI.cpp:1580`.

`CvTacticalAI::PlotEmergencyPurchases` changes emergency defense purchase role at `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp:1780`, and tactical formation handling treats interceptors differently at `CvTacticalAI.cpp:10826`.

Role unit AI enum entries `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE` exist in `CvGameCoreDLL_Expansion2/CvEnums.h:1873`. Role flavor names are referenced in C++ helper paths such as `CvGameCoreDLL_Expansion2/CvTechClasses.cpp:1258`.

DEPENDENCY_UNSAFE: Existing docs-ai investigation and current search did not confirm complete repository XML/SQL data rows for all role flavors and military strategies required by the guarded code.

INFERRED: The feature assumes the database has matching flavor and military strategy rows for hardcoded names such as `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `MILITARYAISTRATEGY_NEED_ARCHER`, and related enough strategies.

Role-flavor references also exist in always-compiled paths, including `CvGameCoreDLL_Expansion2/CvTechClasses.cpp:1258` and `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp:4883`. These are not proof that `MOD_AI_UNIT_PRODUCTION` behavior is active, and they may indicate partial migration of role concepts outside the option.

Save before an AI city chooses production and before strategy update; reload and confirm the same role strategies and selected unit scores.

`DEPENDENCY_UNSAFE`

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.

## Task 1U feature and profile matrix

Source identity `src-20261029-vp-c285a18`; UPAI `src-20261029-upai-recovered32`; pass `audit-20261029-task1u-01`. **Matrix is partial**: every gate listed here has a disposition, but transitive canTrain/purchase/strategy/operation and compile-option discovery is not complete. DB column below means identified reference snapshot only, not loaded target.

| Gate / definition and configuration | Repository default / reference DB | OFF → ON and stage | UPAI relation / logging |
|---|---|---|---|
| MOD_AI_UNIT_PRODUCTION; CustomMods.h1048/1836, .cpp580, NewCustomModOptions.xml1164 | 0 / 0 | Legacy NEED/ENOUGH_RANGED dispatch → split role strategies; sanity adds100/rejects ENOUGH; COUNTER removed from escort-pressure classification; affects tactical purchase/advisor/GS/explorer consumers | UPSTREAM_GATE_ENABLED by NewStrategies.sql; data contract needed. No per-term trace |
| MOD_BALANCE_VP; option XML29, macro CustomMods.h | CP option default0; VP package overrides must be traced / DB1 | Automated/puppet worker availability, messenger/paper sanity, supply ceiling, top defense process override and spaceship investment | Shared upstream mechanism; UPAI depends on VP manifest; profile not set by UPAI itself |
| MOD_BALANCE_CORE_JFD; option XML1212 | 0 / 0 | Restrict artifact-yield iteration to CULTURE_LOCAL → all yields; soft-cap contract maintenance absent→included | UPSTREAM_UNCHANGED; extra yield/cost inputs unlogged |
| MOD_BALANCE_UNIT_INVESTMENTS; XML587 | 0 / 0 | Immediate gold unit purchase → investment/possible queue in CheckForOperationUnits and PurchaseUnit | UPSTREAM_UNCHANGED; changes acquisition/fulfillment; purchase message alone does not prove created unit |
| MOD_CORE_PUPPET_BUILDING_LIMITATIONS; XML72 | 1 / 1 | Removes puppet-specific competitor filter → restricts eras/maintenance competitors | UPSTREAM_UNCHANGED; indirectly changes unit competition |
| MOD_SQLITE_LOGGING; XML54 | 0 / not queried | Optional military logging branch at CvMilitaryAI.cpp3043 | Observability variant, source branch located but content not fully audited |
| Lua CityCanTrain / PlayerCanTrain listeners | Runtime registration UNKNOWN | No registered veto → listener rejection can remove unit | Dynamic extension LEGALITY, not an AI flavor; exact enabled listeners unknown |
| GC logging && AI logging; split AI logs | Runtime settings UNKNOWN | No recommendation/production logs → aggregate candidates/selection and skip messages | Shared observability; no detailed term logging in sanity |
| Human AI-control modes / isHumanAutomated / puppet / free / purchase / forOperation | Runtime state/parameters, not global defaults | Change entry, candidate restrictions and sanity gates as documented in production terms | Shared, retain these in profile/record rather than pretending one default control mode |

No compile guard occurs in the entire current CvUnitProductionAI.cpp body. This bounded absence does **not** establish no relevant compile guards in callers, native legality, CustomMods definitions or queue application.

### UPAI delta classification

| Supplied surface | Classification / consequence |
|---|---|
| NewStrategies.sql updates AI_UNIT_PRODUCTION=1 | UPSTREAM_GATE_ENABLED; enables maintained optional upstream code, not new DLL implementation |
| Eight flavor IDs38–45 and role strategy rows; removes old need/enough ranged rows/deltas | UPAI_NEW_ROLE/ENUM/DATA_CONTRACT + UPAI_DATA_ONLY; city ±150/player ±15 are source values, not verified effective profile |
| NewUnitAITypes.sql inserts IDs44 SKIRMISHER,45 SUBMARINE; updates default/membership roles; removes non-AA COUNTER membership | UPAI_NEW_ROLE/ENUM/DATA_CONTRACT using existing upstream enum consumers; compatibility with selected source IDs unproven |
| NewUnitFlavors.sql deletes mobile coefficient for skirmisher defaults; adds role coefficients (e.g. archer ARCHER4) | Shared cache mechanism with different data; UPAI_DATA_ONLY, no new scorer |
| NewLeaderFlavors.sql and leader-insert trigger | UPAI_DATA_ONLY personality role preferences; newly inserted leaders receive eight coefficients5 |
| Formations.xml | UPAI_DATA_ONLY formation definitions; full effective slot comparison still unresolved |
| Triggers.sql | UPAI_DATA_ONLY future inserted units/roles/leaders; ordering matters. Unit_AITypes trigger deletes COUNTER unless unit default COUNTER, differing from initial AirInterceptRange filter |
| NewTechPolicyFlavors.sql | File exists; manifest requests NewTechFlavors.sql instead. UNKNOWN effective load, not assumed active extension |
| Optional DLL code | No DLL source/binary listed in supplied UPAI manifest. Matching paths exist upstream, so UPSTREAM_UNCHANGED/UPSTREAM_GATE_ENABLED applies; no evidence of UPAI_NEW_CODE_PATH or UPSTREAM_CODE_EXTENDED in supplied package |

Current role-ratio evidence (CvMilitaryAI.cpp4118–4250): ratio=countRole*10/max(1,denominator), integer. Archer denominator melee: NEED<=GS(ARCHER)/2, ENOUGH>=GS(ARCHER). Siege/skirmisher denominator melee: NEED<=GS(role)/4, ENOUGH>=GS(role). Naval melee/ranged and bomber/fighter denominator total domain: NEED<=3*GS(role)/5, ENOUGH>=7*GS(role)/5. Submarine denominator naval: NEED<=GS(SUBMARINE)/5, ENOUGH>=3*GS(SUBMARINE)/5. GS is personality+grand-strategy layer, not active/latest. Full dispatch timing/tech/prereq/min-duration and MOBILE legacy helper formulas remain unresolved. These are upstream optional predicates with UPAI data supplying keys.

No effective UPAI SQLite snapshot identified. The recovered exported directory only supplies table names; neither historical CSV reports nor SQL intentions establish final values or successful load on 5.4.6. No compatibility claim is made.

### Additional bounded gate dispositions

Newly traced source gates (same snapshot/pass): CP XML is initialization default; VP `(2) Vox Populi/Database Changes/CustomModOptionChanges.sql` explicitly sets BALANCE_VP=1 and enables CULTURE_VICTORY_CHANGES, PURCHASED_UNIT_DAMAGE and SETTLERS_CONSUME_POPULATION. These source overrides do not prove a selected-source effective DB load.

| Gate | Default / OFF → ON / source stage / UPAI relationship |
|---|---|
| MOD_BALANCE_CULTURE_VICTORY_CHANGES | CP0, VP source1; CvCityAI.cpp85 omits→preserves current Citizen Earth Protocol project before shared selector; UPSTREAM_UNCHANGED |
| MOD_AI_CONTROL_CITY_PRODUCTION | CP0; CvCity.cpp1115 changes initial turn-zero production condition; broader isHuman mode routing unresolved; UPSTREAM_UNCHANGED |
| MOD_BALANCE_PURCHASED_UNIT_DAMAGE | CP0, VP source1; CreateUnit30138 purchased combat inherits city damage percent min80 with survival clamp instead of no damage; fulfillment state, not score; UPSTREAM_UNCHANGED |
| MOD_BALANCE_SETTLERS_CONSUME_POPULATION | CP0, VP source1; CreateUnit30160 eligible food-produced founder reduces city population1 when >1; completion feedback, not selection term; UPSTREAM_UNCHANGED |
| MOD_EVENTS_CITY | CP0; produce29922 uses native GAMEEVENT CityTrained versus Lua hook; listener behavior/effective setting unknown; UPSTREAM_UNCHANGED |
| GAMEOPTION_ONE_CITY_CHALLENGE | Runtime unknown; player canTrainUnit blocks human-mechanics founders when ON; no extra AI score; shared legality |
| GAMEOPTION_NO_RELIGION | Runtime unknown; blocks religion founder/spreader/heresy candidates; shared legality |
| No-nukes/nukes-valid/victory-valid and league nuclear prohibition | Runtime/league state unknown; player legality removes nuclear/spaceship candidates; no flavor path |
| Trait training/tech bypass/class mapping, purchaseOnly, control modes | DB/runtime predicates; meaningful native city/player gates enumerated in production owner; not CustomMod defaults |

No MOD/compile guards found in CvAIOperation.cpp search. Neighboring military terrain/resource gates (RESOURCE_MONOPOLIES, ALTERNATE_IROQUOIS_TRAIT) were located outside the bounded direct force/sanity formulas; their transitive tactical/resource contribution has not been resolved and remains UNKNOWN for Task 1U, rather than labeled immaterial. Matrix remains open for transitive closure.

## Task 1U-B final feature and UPAI contract matrix

Authority src-20261029-vp-548 / audit-20261029-task1u-b; UPAI remains src-20261029-upai-recovered32. Earlier partial gate tables are retained as history; this matrix supplements/corrects their open dispositions. Gate macros live in CustomMods.h, cached declarations/loads in CustomMods.h/.cpp; DB defaults in CP Database Changes/NewCustomModOptions.xml; VP source overrides in `(2) Vox Populi/Database Changes/CustomModOptionChanges.sql`. Unless stated otherwise UPAI does not set these gates. Effective reference values below belong only to the identified reference DB, not an asserted loaded 5.4.8 profile. All relevant extension boundaries must be recorded in Task 2U context.

| Gate family | Default/configuration / OFF→ON / stage / UPAI and logging disposition |
|---|---|
| AI_UNIT_PRODUCTION | CP0, reference0, UPAI NewStrategies sets1. Replaces legacy ranged dispatch with split role strategies; changes mobile ratio denominator, COUNTER escort pressure, sanity NEED100/ENOUGH rejection; tactical emergency role draw→DEFENSE, naval explorer classification adds SUBMARINE; advisor/GS role flavor handling. UPSTREAM_GATE_ENABLED + data-required contract; source exists even OFF, no default activation inferred |
| BALANCE_VP | CP0→VP source1, reference1. Worker puppet/automation allowance, supply cap/VP canal legality, messenger/paper bonus, defense-process override, production/purchase/maintenance/trait variants. Shared engine with profile data; UPAI manifest requires VP. Conditional globals are not unit flavor deltas |
| BALANCE_CORE_JFD | CP/reference0. Extends yield iteration beyond CULTURE_LOCAL; contract maintenance included in soft cap; contract unit system can change counts/resources. Direct score/producer branch known; detailed contract acquisition is independent existing-state interface |
| UNITS_RESOURCE_QUANTITY_TOTALS | CP/reference0. HasResourceForNewUnit ignores→tests unit resource TOTAL and nonnegative availability; distinct from per-unit requirement. 5.4.8 positive-resource cache preserves selected resource ordering. No UPAI override |
| BALANCE_PURCHASE_COST_ADJUSTMENTS | CP/reference0. Gold unit price adds half tech-progress percent, then ×8/10; changes t019 surplus and purchase eligibility. No UPAI override |
| BALANCE_MINOR_UNIT_SUPPLY_HANDICAP | CP/reference0. Switches minor city/population/tech-reduction accounting in Player17103–17253; changes force/legality inputs, not flavor. No UPAI override |
| BALANCE_UNIT_INVESTMENTS | CP/reference0. Immediate gold acquisition→investment/no immediate created unit, queue may be added by city root. Spaceship VP investment exception independent. UPAI unchanged; outcome must distinguish investment from successful purchase |
| BALANCE_BUILDING_INVESTMENTS | CP0, VP source/reference1. Emergency building purchases suppressed/converted and nonunit purchase competition changes; no unit sanity coefficient. Relevant acquisition interface/competitor condition, full building economics deferred |
| CORE_PUPPET_BUILDING_LIMITATIONS | CP/reference1. Enables puppet competitor era/maintenance filter; changes final unit competition; no UPAI override |
| BALANCE_CULTURE_VICTORY_CHANGES | CP0, VP source1. Current Citizen Earth Protocol preserved before selector; independent player victory production controller; no UPAI override |
| BALANCE_PURCHASED_UNIT_DAMAGE | CP0, VP source1. Tactical emergency bWantUnits suppressed if city>half damaged; CreateUnit inherited purchase damage capped80percent/survival. Changes acquisition candidate request/fulfillment state; no UPAI override |
| BALANCE_SETTLERS_CONSUME_POPULATION | CP0, VP source1. Food-produced founder creation consumes1 population when>1; completion feedback, not direct score multiplier |
| EVENTS_CITY | CP0. Native CityTrained hook replaces Lua hook path and purchase hooks; listener behavior runtime external, must correlate post-hook unit. No default listener assumed |
| AI_CONTROL_CITY_PRODUCTION / UNITS / ECONOMY / DIPLOMACY / FAITH_SPENDING | CP0. Player::isHuman9776 returns false for corresponding human reason when optionON; reroutes selector/operation/acquisition/personality/financial paths. Other AI_CONTROL_* options affect independent strategic inputs, not direct selector. HUMAN_USES_AI_HANDICAP/MECHANICS (CP/reference0) similarly changes difficulty/OCC semantics; preserve reason flags in logs |
| BALANCE_RECON_ONLY_ANCIENT_RUINS | CP0, VP/reference1. Explorer candidate ability/goody participation, modifies exploration-state interface; no direct unit-cache coefficient; scope includes explorer demand interface, not goody missions |
| BALANCE_RESOURCE_MONOPOLIES / ALTERNATE_IROQUOIS_TRAIT / PERMANENT_PANTHEONS | CP0, first two VP source1; pantheons reference0. Resource/terrain/religion producer variants; enter via resource ledger, tactical/path eligibility and effective religion/trait outputs. These are classified **producer configuration**, not newly invented direct sanity modifiers. Record effective state at interface; independent monopoly/religion/terrain systems not audited end-to-end |
| CARGO_SHIPS compile macro | CustomMods.h defines false; `#if defined(MOD_CARGO_SHIPS)` entry loader still compiles optional field loading. Cargo-special field not consumed by direct Task1U scorer; runtime cargo actions deferred. This is not proof featureON merely from defined guard |
| SQLITE_LOGGING | CP/reference0. MilitaryAI3043 mirrors MilitarySummary into stats.db with counters/recommended counts; EconomicAI has stats emitters. No ordered unit-score trace; improves producer observability only |
| Logging && AI logging / split | Runtime external. Enables aggregate recommendations/PRE/POST/choice/skip/flavor messages, routing only; independent of SQLITE_LOGGING. CustomMods diagnostic logging switches are not score behavior gates |
| MilitaryStrategyCanActivate / EconomicStrategyCanActivate / CityStrategyCanActivate | Test-all registered listeners may veto candidate strategy activation after native trigger, changing direct strategy state and flavor deltas. No listener→no extra veto; actual loaded scripts UNRESOLVED_EXTERNAL |
| PlayerCanTrain / CityCanTrain | Native-then-Lua test-all veto; affects pre-enumeration/application eligibility with caller args. Actual listener state UNRESOLVED_EXTERNAL |
| Runtime rules | OCC/human mechanics, NO_RELIGION, no-nukes/valid-nukes, victory validity, league nuclear prohibition, human automation, puppet, occupation, siege, purchase/free/operation parameters. All are gates/state, not flavors or build-default constants; structured legality owner enumerates their effects |
| Game speed / handicap / DB Defines | Train/Hurry percent, CityProductionChoiceCutoffThreshold, duration/wrapper Defines, supply/maintenance/settler/resource data. DB-controlled numeric/predicate parameters, not compileON flags; actual values must be profile-resolved |

Direct UnitProductionAI, FlavorManager, ArmyAI, AIOperation bodies have no feature compile guards; CustomMods macros and entry loaders do. Debug minidump/removed IPC deltas have no identified Task1U score/candidate semantics. Unit execution-only AI_UNIT_PRODUCTION branches in tactical placement and homeland sentry suitability are ADJACENT_DEFERRED after recording their role/state feedback boundary. This closes their relevance disposition without auditing movement.

### UPAI file-by-file against 5.4.8

| Artifact | Classification / 5.4.8 contract / evidence limit |
|---|---|
| modinfo | CONFIRMED_PACKAGE_MISMATCH **in recovered package**: both Files and OnModActivated request NewTechFlavors.sql, not present; actual NewTechPolicyFlavors.sql unlisted. No alternate matching reference/file. Missing-file handling/transaction continuation is executable-owned UNRESOLVED_EXTERNAL; no successful package load claimed. Whether recovery itself caused the mismatch is UNKNOWN; cannot assert original distributed package broken |
| NewStrategies.sql | UPSTREAM_GATE_ENABLED, UPAI_NEW_ROLE/ENUM/DATA_CONTRACT, UPAI_DATA_ONLY. AI_UNIT_PRODUCTION1; inserts8 flavors IDs38–45; removes legacy ranged strategy/deltas; adds16 split need/enough strategies IDs26–41 with timing/prereq rows; city±150/player±15. All corresponding optional 5.4.8 predicates exist |
| NewUnitAITypes.sql | Adds DB roles44/45 corresponding exactly to compiled UnitAITypes sequence SKIRMISHER44/SUBMARINE45, NUM_UNITAI_TYPES46; uses upstream identifiers absent from base reference DB but present in enum. Changes mounted archer defaults/membership; submarine defaults/membership; changes non-AA COUNTER to DEFENSE and deletes non-AA COUNTER memberships. Source-name match plus numeric bounds verified; other mods can still collide |
| NewUnitFlavors.sql | UPAI_DATA_ONLY shared coefficient mechanism. Deletes skirmisher mobile coefficient and adds role coefficients. Three flavor targets UNIT_VP_SLINGER, UNIT_MOUNTED_BOWMAN, UNIT_EARLY_DESTROYER absent from reference DB and searched target SQL/XML; those rows do not attach to entries in test. Current renamed unit families can therefore miss intended role coefficient, despite role reassignment working. No automatic replacement assumed |
| NewLeaderFlavors.sql | UPAI_DATA_ONLY; explicit role leader rows, tested44 leaders×8 flavors; coefficients mostly5, some7. Personalities later randomized/clamped as normal. IDs/names are symbolic joins, not evidence of loaded profile |
| NewTechPolicyFlavors.sql | SOURCE_ONLY, inert under supplied manifest action list. Inserts19 technology role flavor rows for archer/siege/skirmisher, no active policy INSERT; mobile/duplicate flavor blocks commented. This affects independent tech valuation if explicitly loaded, not city latest broadcast. No file rename or loading workaround applied |
| Formations.xml | UPAI_DATA_ONLY. Replaces slots for20 formations (20 Deletes,178 Rows); source operation/formation enum keys exist. Primary/secondary fallback and slot RequiredSlot semantics unchanged. Most explicit new skirmisher/submarine slots omit required flag; two FAST_PILLAGERS required FAST_ATTACK with secondary SKIRMISHER; ANTI_BARBARIAN_TEAM required primary SKIRMISHER also exists, correcting an overly broad 'all added skirmisher slots optional' reading. No new formation enum/dynamic operation code |
| Triggers.sql | UPAI_DATA_ONLY future insert compatibility: unit-default/role remapping, future leader default8×5, fallback rows for unlisted existing leaders. Initial COUNTER filter uses AirInterceptRange; trigger role filter uses defaultUnitAI, a meaningful difference. Only INSERT triggers, no universal later UPDATE repair; no automatic role flavors for every future inserted unit |

No C++/DLL/Lua/binary in the eight-file artifact or manifest. Thus no evidence for UPSTREAM_CODE_EXTENDED, UPSTREAM_GATE_MODIFIED or UPAI_NEW_CODE_PATH; code is upstream optional, data package enables/supplies it. UPAI intended VP release remains UNRESOLVED_EXTERNAL. Fixed IDs are compatible with observed base key ranges (flavors0–37, military strategies0–25 before deletion, roles0–43), and roles match compiled5.4.8 numeric values; this does not establish arbitrary mod-stack compatibility.

### Data validation and profile limits

No existing certified5.4.8+UPAI snapshot found in reference/baseline/recovered artifact locations. Original DBs opened mode=ro and never changed. **TRANSFORMATION_TEST**, not effective VP5.4.8: SQLite backup of gameplay reference hash429d65... to a temporary `upai-transform.db` copy; apply NewStrategies→NewUnitAITypes→NewLeaderFlavors→NewUnitFlavors→Formations→Triggers, deliberately exclude missing tech action. XML translator implemented only actual Row/Delete operations, true/false→1/0, retained row order and omitted fields as NULL; it is not CivV loader emulation. No claim executable continues after missing action. All6 supplied actions succeeded in isolated test. A savepoint inserted synthetic mounted archer plus COUNTER membership; both remapped SKIRMISHER, then rolled back.

Test results: gate1; roles44/45;8 new flavors;16 new strategies;20 skirmisher defaults/3 submarine defaults; no duplicate unit-role rows, missing role/flavor/formation-parent/prereq-tech references in tested joins;3 missing unit-flavor targets above. Leader additions44 rows per role flavor (min5/max7, bomber/fighter/submarine max5). Foreign-key absence checks were explicit joins, not a claim CivV validation ran. Source-only tech file was separately inspected, not silently applied. Unit/tech/formation IDs and all table claims are scoped to source or test, never a fabricated effective profile.

Transformed total/explicit required slot counts (all formations, including untouched civilian entries):

| Formation | Slots / explicit required |
|---|---|
| MUFORMATION_ANTI_BARBARIAN_TEAM | 3 / 2 |
| MUFORMATION_BASIC_CITY_ATTACK_FORCE | 13 / 5 |
| MUFORMATION_BIGGER_CITY_ATTACK_FORCE | 15 / 7 |
| MUFORMATION_CARRIER_ESCORT | 7 / 4 |
| MUFORMATION_CITY_STATE_ATTACK_FORCE | 11 / 4 |
| MUFORMATION_CITY_STATE_INVASION | 16 / 7 |
| MUFORMATION_CLOSE_CITY_DEFENSE | 8 / 3 |
| MUFORMATION_COLONIZATION_PARTY | 4 / 2 |
| MUFORMATION_CONCERT_TOUR | 2 / 1 |
| MUFORMATION_DIPLOMAT_ESCORT | 2 / 1 |
| MUFORMATION_EARLY_RUSH | 10 / 4 |
| MUFORMATION_FAST_PILLAGERS | 3 / 2 |
| MUFORMATION_LATE_GAME_INVASION_FORCE | 19 / 9 |
| MUFORMATION_LATE_GAME_PURE_NAVAL_FORCE | 12 / 5 |
| MUFORMATION_MERCHANT_ESCORT | 2 / 1 |
| MUFORMATION_NAVAL_BOMBARDMENT | 7 / 2 |
| MUFORMATION_NAVAL_INVASION | 17 / 9 |
| MUFORMATION_NAVAL_SQUADRON | 7 / 3 |
| MUFORMATION_NUKE_ATTACK | 1 / 1 |
| MUFORMATION_PURE_NAVAL_CITY_ATTACK | 8 / 4 |
| MUFORMATION_QUICK_COLONY_SETTLER | 2 / 1 |
| MUFORMATION_RAPID_RESPONSE_FORCE | 4 / 2 |
| MUFORMATION_SETTLER_ESCORT | 2 / 2 |
| MUFORMATION_SMALL_CITY_ATTACK_FORCE | 10 / 4 |

RequiredSlot omitted→NULL in this schema; CvInfos::CacheResults1578 uses database GetBool. Explicit true requirements are therefore reported separately from test omission rather than pretending a hidden XML schema default was validated. Effective engine boolean/load behavior and final mod stack remain UNRESOLVED_EXTERNAL; log cached slot flags actually consumed.

Role-state scheduling refinement: need/enough archer/siege/skirmisher/naval/bomber/fighter keys are selected by runtime strategy name only when AI_UNIT_PRODUCTION; MOBILE keys are tested earlier for both profiles with gate-dependent formula. Military minor eligibility/technology/firstTurn/adopted-turn interval/minimum-duration checks precede dispatch; veto after positive trigger; exact additive city/player flavor reversal follows. Need/Enough can overlap at low/zero thresholds; sanity tests NEED first. Naval-melee ENOUGH rejection remains commented, although its strategy can still produce negative flavor deltas. Enum/name compatibility does not mean all strategies are simultaneously active.

Additional source debug-gate disposition: EconomicAI3484 g_bNoNewExplorers defaultsfalse; ON skips DoReconState before reset and can retain cached explorer demand, OFF computes normally. TacticalAnalysisMap19 g_bLogZones defaultsfalse; ON emits refreshed zone analysis after calculation (logging only), OFF no zone dump. Neither is set by supplied UPAI or a DB CustomMod option; runtime debug alteration not observed. These globals belong in effective debugging context if altered.
