# Database loading, options and effective-data provenance

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How do data rows become cached entries and runtime configuration?

Repository SQL/XML establishes provenance, not necessarily the final loaded value. This reference owns generic loading/cache/option contracts and snapshot qualification; system-specific score interpretations remain with consumers.

## Reference synthesis and data limits

Generic DB-to-entry loading and runtime option caching are shared architecture. Exact table/column keys, defaults, loaders, cached getters and override order belong to the provenance chain. A repository grep cannot establish the final effective database or loaded mod options.

The recovered VP+UPAI notes report compiled table/row observations; only their table-name inventory survives in this artifact. The missing CSVs and original source identity limit independent reproduction. No database was opened, copied or transformed by this migration. Future effective-data verification follows the installed DB skill and task-selected source/profile.

Canonical related owners: [Optional feature activation and dependency map](<optional-features.md>), [Flavor production, propagation and consumption](<../ai/infrastructure/flavors.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

Unresolved questions for focused pass:
- Identify all flavor recipients beyond the five target systems and which are savegame-serialized.
- Separate base leader/minor flavor, randomized flavor, active flavor deltas, city flavor deltas, and grand-strategy-adjusted flavor reads.
- Trace database tables and cache paths for `FlavorTypes` and entry-level flavor values.
- Confirm how flavor updates are triggered after strategy changes, policy changes, city founding, leader change, and load.

## Scoring and inputs

Flavor identity is database-driven at runtime. `FlavorTypes` is an open enum in `CvGameCoreDLLUtil/include/CvEnums.h`, and `CvDllDatabaseUtility::PrefetchGlobalDefines` in `CvGameCoreDLL_Expansion2/CvDllDatabaseUtility.cpp` loads flavor names from the `Flavors` table.

Runtime flavor access is split:

RISK: The personality/latest/grand-strategy flavor split is a high-risk documentation and patch area because different decisions consume different flavor layers.

GUIDE_PATTERN: The guide says scoring functions can use information available to a human player, such as:

GUIDE_PATTERN: - selected policy trees

GUIDE_PATTERN: - owned tiles and improvements

GUIDE_PATTERN: - neighboring civilizations

GUIDE_PATTERN: - traits and unique buildings

GUIDE_PATTERN: - AI-internal calculations

GUIDE_PATTERN: - victory-condition intentions

GUIDE_PATTERN: - flavors

GUIDE_PATTERN: Victory intention examples from diplomacy AI:

GUIDE_PATTERN: - `CvDiplomacyAI::IsGoingForWorldConquest() const`

GUIDE_PATTERN: - `CvDiplomacyAI::IsGoingForDiploVictory() const`

GUIDE_PATTERN: - `CvDiplomacyAI::IsGoingForCultureVictory() const`

GUIDE_PATTERN: - `CvDiplomacyAI::IsGoingForSpaceshipVictory() const`

GUIDE_PATTERN: The guide describes AI decision-making as looping over available options, assigning each a non-negative integer score, then choosing the highest scoring option or sometimes randomly choosing among high-scoring options. `CvWeightedVector<T>` is a common data structure for storing candidate-score pairs and then sorting or selecting from them.

GUIDE_PATTERN: The pantheon example in `CvReligionAI::ChoosePantheonBelief(PlayerTypes ePlayer)` is the canonical pattern to search for in other systems:

GUIDE_PATTERN: - Get available options, such as `GetAvailablePantheonBeliefs(ePlayer)`.

GUIDE_PATTERN: - Create `CvWeightedVector<BeliefTypes>` or similar.

GUIDE_PATTERN: - Optionally precompute/calculate helper values, such as plot weights.

GUIDE_PATTERN: - Loop candidates.

GUIDE_PATTERN: - Read the relevant database entry, for example `CvBeliefEntry*`.

GUIDE_PATTERN: - Score the entry, for example `ScoreBelief(...)`.

GUIDE_PATTERN: - Push candidate and score into the weighted vector.

GUIDE_PATTERN: - Sort candidates with `StableSortItems()` or equivalent.

GUIDE_PATTERN: - Select with a direct best choice, threshold choice, or seeded random choice.

GUIDE_PATTERN: When documenting beliefs, policies, wonders, and unit production, search for the same concepts even if function names differ: available list, score/evaluate function, weighted vector, sort, threshold, random seed, log function, final application.

## Data and configuration

1. Which exact Civ5 cache DB should be treated as the AI data truth for VP and VP+UPAI?

2. Which exact baseline VP compiled DB snapshot should be exported for comparison against VP+UPAI?

3. Which UPAI rows exist only in external modmod DB/XML and are not present in the current repo source/provenance files?

4. How should DB snapshots be versioned without committing large binaries?

### Which AI data truth exists for VP+UPAI?

REPORTED_DB_SNAPSHOT: The current VP+UPAI exported data truth is `docs-ai/db-snapshots/vp-upai/`. It contains per-table CSV snapshots plus `manifest.csv` and `database_tables.txt`.

REPORTED_DB_SNAPSHOT: `manifest.csv` records exported table names, row counts, output files, and notes. The binary Civ5 cache database path/version metadata is not preserved in this repo artifact, so this is closed for the exported VP+UPAI snapshot but not for reproducing the original local cache DB path.

### Which tables should be exported for formation/UPAI analysis?

REPORTED_DB_SNAPSHOT: The useful exported tables for the completed formation/UPAI pass are:

- `CustomModOptions.csv`

- `Flavors.csv`

- `AIMilitaryStrategies.csv`

- `AIMilitaryStrategy_Player_Flavors.csv`

- `AIMilitaryStrategy_City_Flavors.csv`

- `UnitAIInfos.csv`

- `Unit_AITypes.csv`

- `Units.csv`

- `Unit_Flavors.csv`

- `MultiUnitFormations.csv`

- `MultiUnitFormation_SlotEntries.csv`

- `MultiUnitPositions.csv`

- `TacticalMoves.csv`

- `Defines_AI_relevant.csv`

REPORTED_DB_SNAPSHOT: `manifest.csv` also includes broader AI-relevant exports for flavor context: leader, building, policy, technology, project, process, resource, economic strategy, city strategy, city specialization, grand strategy, and minor-civ flavor tables.

REPORTED_DB_SNAPSHOT: `CustomModOptions.csv` has columns `Name,Value,Class,DbUpdates`. Enabled state should be verified from the final compiled snapshot row, not repository XML alone.

REPORTED_DB_SNAPSHOT: In VP+UPAI, `CustomModOptions.csv` contains `AI_UNIT_PRODUCTION,1,6,0`, so `MOD_AI_UNIT_PRODUCTION` is enabled for this compiled data set.

### Which UPAI rows exist in the final VP+UPAI DB?

REPORTED_DB_SNAPSHOT: UPAI role flavors exist in `Flavors.csv`: `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, and `FLAVOR_FIGHTER`.

REPORTED_DB_SNAPSHOT: UPAI need/enough military strategies exist in `AIMilitaryStrategies.csv` for archer, siege, skirmisher, naval melee, naval ranged, submarine, fighter, and bomber. Their player and city flavor deltas exist in `AIMilitaryStrategy_Player_Flavors.csv` and `AIMilitaryStrategy_City_Flavors.csv`.

REPORTED_DB_SNAPSHOT: UPAI UnitAI roles exist in `UnitAIInfos.csv`: `UNITAI_SKIRMISHER` and `UNITAI_SUBMARINE`. Matching unit-role rows exist in `Unit_AITypes.csv`.

The VP modding guide identifies the cache DBs as useful for viewing final game data, especially `Civ5DebugDatabase.db` and `Localization-Merged.db`. These files are updated when Civ 5 starts and when mods are loaded/unloaded, so copy them to a stable project folder after generating each configuration.

Recommended local layout:

```text
local-db/
  baseline-vp/
    Civ5DebugDatabase.db
    Localization-Merged.db
    metadata.md
  vp-upai/
    Civ5DebugDatabase.db
    Localization-Merged.db
    metadata.md
```

For each DB snapshot, record:

- VP version;

- branch/commit if known;

- mod list;

- load order;

- UPAI enabled?

- relevant custom mod options;

- game options;

- date generated;

- cache-clearing status;

- save/scenario if applicable.

| Folder / data area | Relevant tables and rows | Current-source consumer |
|---|---|---|
| `(2) Vox Populi/Database Changes/Tech/` | modifies `Technologies`, eras, tech costs/tree data, and adds `TECH_CORPORATIONS`. AI flavor rows are in `(2) Vox Populi/Database Changes/AI/TechFlavorSweeps.sql` via `Technology_Flavors`. | `CvTechEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvTechClasses.cpp`), `CvPlayerTechs::FlavorUpdate`, `CvTechAI::AddFlavorWeights`, and `CvTechAI::ChooseNextTech` (`CvGameCoreDLL_Expansion2/CvTechAI.cpp`). |
| `(1) Community Patch/Database Changes/NewCustomModOptions.xml` | defines `CustomModOptions`, including AI-affecting toggles such as `AI_UNIT_PRODUCTION`, `AI_CONTROL_*`, `DIPLOAI_*`, `EVENTS_AI_OVERRIDE_TECH`, and `HUMAN_USES_AI_MECHANICS`. | `CustomMods.cpp::preloadCache` / `MOD_OPT_CACHE(...)`, `CustomMods.h` macros such as `MOD_AI_UNIT_PRODUCTION` and `MOD_EVENTS_AI_OVERRIDE_TECH`; consumers are spread across AI source. |

Generic flavor loading is centralized in `CvDatabaseUtility::SetFlavors` (`CvGameCoreDLL_Expansion2/CvDatabaseUtility.cpp`). It calls `PopulateArrayByValue(..., "Flavors", szTableName, "FlavorType", szFilterColumn, szFilterValue, "Flavor", ...)`, so object flavor arrays are indexed by runtime `Flavors.ID`.

Runtime flavor names are loaded before object caches by `CvDllDatabaseUtility::CacheGameDatabaseData` / `PrefetchGlobalDefines` (`CvGameCoreDLL_Expansion2/CvDllDatabaseUtility.cpp`), which uses `kUtility.MaxRows("Flavors")`, calls `GC.setNumFlavorTypes(iNumFlavors)`, and stores `Flavors.Type` strings in `GC.getFlavorTypes()`.

| Entry / info class | Tables loaded | Main AI decisions fed |
|---|---|---|
| `CvUnitEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvUnitClasses.cpp`) | `Units`, `Unit_Flavors`, role/default AI fields, prereq/resource/promotion/unit-class data | Generic unit production, operation/army role recommendation, purchases, unit-role counting. |
| `CvProjectEntry::CacheResults` (`CvGameCoreDLL_Expansion2/CvProjectClasses.cpp`) | `Projects`, `Project_Flavors` | City project production scoring. |
| `CvProcessInfo` / process production cache (`CvGameCoreDLL_Expansion2/CvProcessProductionAI.cpp`) | `Processes`, `Process_Flavors` | City process production scoring. |

Tech path is `Technologies` / `Technology_Flavors` / prereq and unlock fields -> `CvTechEntry::CacheResults` -> `CvPlayerTechs::FlavorUpdate` -> `CvTechAI::AddFlavorWeights` -> `CvTechAI::ChooseNextTech` -> research application in player tech paths. Tech AI is not one of the primary atlas decisions here, but tech data gates unit/building/policy/belief availability.

RISK: Tech changes can be data-only yet affect AI production/policy/religion indirectly by changing unlock timing, prerequisites, obsolescence, and available candidate pools.

Custom mod options are database-backed in `CustomModOptions` and cached by `CustomMods.cpp::preloadCache` using `MOD_OPT_CACHE(...)`. Macros in `CvGameCoreDLL_Expansion2/CustomMods.h` expose cached options to AI code.

| Option / macro | Default in repo | Verified AI relevance |
|---|---:|---|
| `AI_CONTROL_CITY_PRODUCTION`, `AI_CONTROL_CITY_MANAGEMENT`, `AI_CONTROL_POLICY_CHOICE`, `AI_CONTROL_RELIGION_CHOICE`, `AI_CONTROL_TECH_CHOICE`, `AI_CONTROL_UNITS`, `AI_CONTROL_DIPLOMACY`, `AI_CONTROL_ECONOMY`, etc. | `0` | INFERRED / RISK: option names and macros in `CustomMods.h` show human-to-AI-control toggles; each target path must be verified before documenting exact behavior. |
| `DIPLOAI_NO_FLAVOR_RANDOMIZATION` / `MOD_DIPLOAI_NO_FLAVOR_RANDOMIZATION` | `0` | affects diplomacy/personality randomness context through `CustomMods.h`; exact flavor-manager branch should be cited from the consumer when documenting behavior. |
| `EVENTS_AI_OVERRIDE_TECH` / `MOD_EVENTS_AI_OVERRIDE_TECH` | `0` | macro exists in `CustomMods.h`; relevant to tech-choice override paths, but exact current hook behavior needs a focused tech pass. |
| `HUMAN_USES_AI_MECHANICS`, `HUMAN_USES_AI_HANDICAP` | `0` | RISK: can blur human-vs-AI splits in final application or special treatment. Cite consuming source before applying to a decision doc. |

STALE_OR_MISMATCH: `AI_UNIT_PRODUCTION` is marked "SLATED FOR DELETION" in `NewCustomModOptions.xml` and is disabled by default, but C++ references remain active behind `MOD_AI_UNIT_PRODUCTION`. Treat it as opt-in/appendix behavior unless the merged database enables it.

STALE_OR_MISMATCH: Current C++ references role flavor names such as `FLAVOR_ARCHER`, `FLAVOR_SIEGE`, `FLAVOR_SKIRMISHER`, `FLAVOR_NAVAL_MELEE`, `FLAVOR_NAVAL_RANGED`, `FLAVOR_SUBMARINE`, `FLAVOR_BOMBER`, and `FLAVOR_FIGHTER` in `MOD_AI_UNIT_PRODUCTION`-adjacent code, but previous targeted repository searches did not find matching VP XML/SQL flavor definitions. If external data supplies them, update this map from the merged database.

GUIDE_PATTERN: The guide explains that game components such as buildings, units, policies, beliefs, promotions, technologies, and traits are represented in the database. Each main table generally has one row per object. Additional tables represent conditional or repeated effects, especially yield effects.

GUIDE_PATTERN: For many database tables, the DLL has open enums such as `BuildingTypes`, where only `NO_BUILDING = -1` is hard-coded and real values correspond to database row IDs. Use `GC.getNum*Infos()` to loop and `GC.get*Info(...)` to retrieve the `Cv*Entry` object. Null-checks are important because modmods can delete rows and leave gaps.

GUIDE_PATTERN: For each database column used by code, the corresponding `Cv*Entry` class normally has a member variable and getter. Values are loaded in `Cv*Entry::CacheResults(Database::Results& kResults, CvDatabaseUtility& kUtility)`. Simple columns use `GetInt`, `GetBool`, or `GetText`; references to other database rows are converted with `GC.getInfoTypeForString(...)`. Yield arrays are often loaded with helper functions such as `SetYields(...)`.

GUIDE_PATTERN: For current AI documentation, this means:

GUIDE_PATTERN: - Belief AI likely reads `CvBeliefEntry` values.

GUIDE_PATTERN: - Policy AI likely reads policy entries and policy branch/ideology/tenet data.

GUIDE_PATTERN: - Wonder choice likely reads `CvBuildingEntry`, building classes, prerequisites, and world/national wonder flags.

GUIDE_PATTERN: - Unit production likely reads `CvUnitEntry`, unit combat/domain/AI role data, prerequisites, and strategy state.

GUIDE_PATTERN: - Flavor documentation must connect database `Leader_Flavors` and flavor info tables to runtime accessors and scoring functions.

GUIDE_PATTERN: The guide distinguishes database entry data from processed/cached player, city, team, or unit state. For example, building effects are stored in `CvBuildingEntry`, then processed into `CvCity` or `CvPlayer` variables when the building is built or removed. The same broad pattern exists for policies with `CvPlayer::processPolicies(...)`, techs with `CvTeam::processTech(...)`, promotions with `CvUnit::setHasPromotion(...)`, and religions through `CvReligionBeliefs` and `CvCity::UpdateReligion(...)`.

GUIDE_PATTERN: For AI documentation, check whether scoring reads raw entry data, processed player/city values, or both.

GUIDE_PATTERN: The guide emphasizes caching because repeated loops over database objects or map/city state can be expensive. Cached values must be updated whenever relevant underlying state changes. Some caches are refreshed periodically as a safety measure.

GUIDE_PATTERN: For documentation, note if an AI scoring function:

GUIDE_PATTERN: - precomputes helper values before candidate loops

GUIDE_PATTERN: - uses cached player/city/unit state

GUIDE_PATTERN: - depends on values updated elsewhere

GUIDE_PATTERN: - has potential stale-cache risks

GUIDE_PATTERN: Main game objects such as buildings, policies, beliefs, units, promotions, techs, and traits have database tables. In the DLL, many are represented by open enums and `Cv*Entry` classes. Use `GC.getNum*Infos()` to loop through possible entries and `GC.get*Info(...)` or equivalent to get entry objects.

GUIDE_PATTERN: Important null-check rule: database-backed entry pointers can be null, especially with modmods or deleted database rows. Existing code should normally check pointers before use. When documenting AI candidate loops, record whether null entries are skipped.

GUIDE_PATTERN: For AI documentation, pay special attention to:

GUIDE_PATTERN: - `CvBeliefEntry`

GUIDE_PATTERN: - `CvPolicyEntry` or policy-related entries

GUIDE_PATTERN: - `CvBuildingEntry` and world wonder flags/classes

GUIDE_PATTERN: - `CvUnitEntry`, combat/domain/unit AI/default AI entries

GUIDE_PATTERN: - `FlavorTypes`, `YieldTypes`, `PolicyTypes`, `BeliefTypes`, `BuildingTypes`, `UnitTypes`

GUIDE_PATTERN: Caching is pervasive because looping over many objects repeatedly is expensive. Before assuming a scoring function computes directly from raw game state, check whether it reads cached player/city/unit values, precomputed vectors, or helper structs. For AI documentation, note which values are cached and where they are refreshed if the code makes that visible.

## Observability and validation obligations

GUIDE_PATTERN: The guide shows AI logging guarded by `GC.getLogging()` and `GC.getAILogging()`, then log messages written through subsystem-specific log functions such as `LogHomelandMessage(...)`. For each AI decision, search for log functions near scoring and selection. Good docs should identify how to enable and interpret logs if the code supports it.

GUIDE_PATTERN: - Log the scores and selected candidate if logging exists.

## Version differences, risks and unresolved questions

INFERRED:
- Any later behavioral edits in these systems should treat weighted vector serialization, enum ordering, seeded random inputs, and XML/database entry flavor schemas as high-risk compatibility points.

- Known improvement hooks: OPEN_QUESTION trace full `CvCity::canTrain` and `CvCity::PurchaseUnit` constraint trees for database, Lua/event, resource, and synchronization side effects.

OPEN_QUESTION: The complete base-game schema and initial `Flavors` rows are not fully listed in the VP patch folders. Current DLL behavior proves the runtime table model, but exact merged row IDs require a built/merged database inventory.

- RISK `FlavorTypes` is database-driven but array-indexed. `CvDllDatabaseUtility::PrefetchGlobalDefines`, `CvDatabaseUtility::SetFlavors`, `CvEnumMap<FlavorTypes, int>`, object `m_piFlavorValue` arrays, and hardcoded `GC.getInfoTypeForString("FLAVOR_*")` calls all depend on stable merged `Flavors` rows.

- RISK `CvFlavorManager::Serialize` stores `m_piPersonalityFlavor` and `m_piActiveFlavor`; recipient latest flavor vectors are serialized in `CvPlayerTechs::Serialize`, `CvPlayerPolicies::Serialize`, and `CvCityStrategyAI::Serialize`. Changing flavor counts/order or cache semantics can affect save compatibility.

- RISK production AI weights are serialized: `CvUnitProductionAI::Serialize` stores `m_UnitAIWeights`, `CvBuildingProductionAI::Serialize` stores `m_BuildingAIWeights`, `CvWonderProductionAI::Serialize` stores `m_WonderAIWeights`, and project/process caches have similar serialization. Cache invalidation and flavor update timing matter.

- RISK strategy deltas require start/end symmetry. `CvEconomicAI::UpdateEconomicStrategies` and `CvMilitaryAI::UpdateMilitaryStrategies` apply positive and negative deltas through `CvFlavorManager`; `CvCityStrategyAI` applies local city strategy/specialization deltas directly.

- RISK enum/database row ordering is selection-sensitive. `CvWeightedVector<T>::ChooseAbovePercentThreshold` (`CvGameCoreDLLUtil/include/CvWeightedVector.h`) is used by belief choice, policy choice, desired-wonder choice, city production, and hurry selection. Candidate order, zero-weight behavior, and seeded threshold selection can change deterministic outcomes.

- RISK policy branch/policy packing is schema-sensitive. `CvPolicyAI::ChooseNextPolicy` stores branch IDs directly and policy IDs offset by `GC.getNumPolicyBranchInfos()`. Branch count/order changes can break assumptions if not kept coherent with `CvPlayerPolicies`.

- RISK operation/army production demand is serialized. `CvAIOperation::Serialize` stores `m_viListOfUnitsWeStillNeedToBuild` and `m_viListOfUnitsCitiesHaveCommittedToBuild`; formation slot IDs and `UnitAITypes` changes affect production and movement readiness.

- RISK best-wonder-city and desired-wonder state are cached/serialized across player/city managers (`CvPlayer::SetBestWonderCities`, `CvCity::SetBestForWonder`, `CvWonderProductionAI`, `CvCitySpecializationAI`). Data-only wonder prerequisite or class changes can alter both candidate generation and final scoring.

- RISK Lua/game-event vetoes can make database-visible candidates unavailable at runtime. Policy, branch, ideology, belief, unit, building, and tech docs should keep legality hooks separate from scoring.

- OPEN_QUESTION exact merged schema constraints for some VP-added `Belief_*` and building/policy tables should be generated from the built database before claiming full modmod compatibility.

- existing object flavor rows can be tuned data-only when the target flavor and consumer already exist: `Unit_Flavors`, `Building_Flavors`, `Policy_Flavors`, `Technology_Flavors`, `Project_Flavors`, and `Process_Flavors`. Expected effect is cache-weight changes, not guaranteed final choices, because sanity/legality layers still dominate many decisions.

- leader personality changes can be data-only through `Leader_Flavors` / `LeaderPersonalitySweeps.sql`, with the caveat that `CvFlavorManager::RandomizeWeights`, map adjustments, and grand strategies can alter runtime values.

- strategy flavor deltas can be adjusted data-only through `AIEconomicStrategy_*_Flavors`, `AIMilitaryStrategy_*_Flavors`, `AICityStrategy_Flavors`, and `CitySpecialization_Flavors` when existing strategies already start/end in the desired conditions.

- policy, branch, belief, unit, building, and tech effect rows can be data-only when existing `Cv*Entry::CacheResults` fields and existing scoring code already value those fields. Examples: changing `Policy_Flavors`, policy prerequisites, `Beliefs` category/effect rows, unit default AI/prereq data, building prerequisites, and tech flavor rows.

- INFERRED flavor tuning is safest when it changes relative preference among already-legal candidates and does not depend on new local facts.

- adding a new flavor name is only partly data-driven. Generic object-weight consumers will multiply it if rows exist, but semantic consumers that compare literal names (`CvPolicyAI::WeighPolicy`, `CvTechEntry::GetGrandStrategyPriority`, `CvMilitaryAI` role helpers, `CvReligionAI` yield/flavor mappings) need source changes to interpret it.

- new candidate-generation rules require DLL changes unless an existing database field or Lua/event hook already controls the candidate. Examples: new unit role demand shape, new wonder race logic, new belief availability category, or new policy branch unlock rule.

- new local/tactical scoring inputs require DLL work. Current local facts live in functions such as `CvUnitProductionAI::CheckUnitBuildSanity`, `CvBuildingProductionAI::CheckBuildingBuildSanity`, `CvReligionAI::ScoreBelief*`, `CvPolicyAI::WeightPolicyAttributes`, `CvCityStrategyAI::ChooseProduction`, and `CvTacticalAI::PlotEmergencyPurchases`.

- changing selection mechanics, cutoff behavior, or randomness requires DLL work and is sync-sensitive because `CvWeightedVector<T>::ChooseAbovePercentThreshold` and seeded `CvSeeder` use are shared.

- changing save-visible cache/state layout requires DLL and save-compatibility planning: `CvFlavorManager`, `CvFlavorRecipient` subclasses, production AI weights, `CvPlayerPolicies`, `CvAIOperation`, `CvArmyAI`, and city/wonder state serialize relevant fields.

- DESIGN_HYPOTHESIS robust dynamic flavor additions should start with a "flavor signal registry" before behavior patches: producer, consumer, scope, lifetime, scale, logging, and double-counting risks for each signal.

OPEN_QUESTION: A complete data-only opportunity list should be regenerated from the merged database plus `Cv*Entry::CacheResults` field coverage. This page maps the main AI-relevant paths, not every VP schema extension.

- Why current behavior may be suboptimal: STALE_OR_MISMATCH the source contains a conceptually richer role-composition system, but the repository default disables it and the flavor names are not present in the searched VP/CP database files. If enabled externally without data, `GC.getInfoTypeForString("FLAVOR_*")` role lookups may not provide meaningful strategy thresholds.

- Lowest-risk investigation: Build a merged runtime database inventory of `Flavors`, `Unit_Flavors`, `Policy_Flavors`, `Technology_Flavors`, and military strategy rows to confirm whether external/base data supplies the role flavors.

4. OPEN_QUESTION: Build a merged database flavor inventory before reviving or documenting `MOD_AI_UNIT_PRODUCTION` as an active feature.

GUIDE_PATTERN: If a future edit adds a class member, the member normally needs to be included in the relevant `Serialize` function, otherwise it will not be saved. Changing serialization changes savegame compatibility. Some temporary cached values are intentionally not serialized.

GUIDE_PATTERN: For multiplayer synchronization, classes such as `CvCity`, `CvPlayer`, `CvPlot`, and `CvUnit` have sync archive sections. Adding member variables to synced classes may require adding them to `SYNC_ARCHIVE_*`. AI randomness and decision paths should remain deterministic across clients.

GUIDE_PATTERN: For documentation-only tasks, do not change code. For future patch tasks, flag risks when touching:

GUIDE_PATTERN: - `Serialize` functions, because they affect save compatibility.

GUIDE_PATTERN: - `SYNC_ARCHIVE_*` sections, because they affect multiplayer synchronization.

GUIDE_PATTERN: - Open enum ordering and hard-coded enum assumptions.

GUIDE_PATTERN: - XML/database schema, table columns, or `Cv*Entry::CacheResults`.

GUIDE_PATTERN: - Seeded randomness and multiplayer determinism.

GUIDE_PATTERN: - Cached values without clear invalidation/update paths.

GUIDE_PATTERN: - C++ syntax newer than C++03.

## Source and dependency lookup

VP and Civ V database state is layered. Repository XML/SQL alone may not represent the final runtime database because base game, expansions, VP components, modmods, SQL updates, deletes, replacements, load order, and custom options all matter.

For UPAI specifically, external modmod DB/XML entries may provide rows that are intentionally absent from the VP repo.

Export relevant tables to text/CSV for Codex and diffs:

- `Flavors`

- `Leader_Flavors`

- `Unit_Flavors`

- `Building_Flavors`

- `Policy_Flavors`

- `Technology_Flavors`

- `Project_Flavors`

- `Process_Flavors`

- economic strategy tables and flavor tables

- military strategy tables and flavor tables

- city strategy tables and flavor tables

- city specialization tables and flavor tables

- unit AI mapping tables

- formation tables and formation slot tables

- operation-related tables, if present

- `CustomModOptions`

- relevant defines

Codex should use exported snapshots for repeated documentation and direct SQLite queries for ad hoc inspection.

Do not let Codex infer final DB state from repository XML/SQL when compiled DB is available.

- Data interpretation lives in `Cv*Entry::CacheResults`, XML/SQL tables, global defines, enum-backed IDs, and object flavor arrays.

- data: `CvUnitEntry`, `Unit_Flavors`, `UnitAITypes`, `UnitTypes`, `UnitClassTypes`, `DomainTypes`, unit data under `(2) Vox Populi/Database Changes/Units/`.

- flavor IDs and object rows: `FlavorTypes` in `CvGameCoreDLLUtil/include/CvEnums.h`; runtime flavor names from database table `Flavors` via `CvDllDatabaseUtility::PrefetchGlobalDefines`; generic object flavor loader `CvDatabaseUtility::SetFlavors`.

- STALE_OR_MISMATCH: Revise any compressed statement that says `MOD_AI_UNIT_PRODUCTION` is "not data-backed" as if repository SQL is the runtime database. Source search verifies C++ references and `(1) Community Patch/Database Changes/NewCustomModOptions.xml` default `AI_UNIT_PRODUCTION = 0`, but the final runtime DB can include base game or external modmod rows. The safer claim is "not found in searched VP/CP database changes; runtime DB inventory required."

GUIDE_PATTERN: Source: `VP_Modding_Guide.pdf`, axatin, version 1.0, 2024-11-17. This file is intentionally secondary. Read it only when the quickref is not enough or when a task touches DLL/database mechanics.

GUIDE_PATTERN: The source code for the DLL is in `CvGameCoreDLL_Expansion2` and other folders starting with `Cv`. The repository also contains mod component folders such as `(1) Community Patch`, `(2) Vox Populi`, EUI compatibility folders, and UI folders. For this documentation project, focus on DLL source and database files only when they feed AI decisions.

GUIDE_PATTERN: The DLL source is C++. The guide says VP uses old C++03, so do not introduce C++11 or newer features in future patches.

GUIDE_PATTERN: Classes generally start with `Cv` and are usually split into `.h` and `.cpp`. Header files contain class attributes and function declarations; `.cpp` files contain function bodies. Interface classes named `CvDll*` are mostly for code we do not have access to. `CvLua*` files expose C++ functions to Lua and are relevant when tracing how human/UI actions call final DLL application functions.

GUIDE_PATTERN: The core guide statement: AI decision-making loops through available options, assigns each option a non-negative integer score, then selects the highest score or sometimes randomly selects one of the high-scoring options. `CvWeightedVector<T>` is commonly used for candidate-score pairs.

GUIDE_PATTERN: Canonical pantheon pattern in `CvReligionAI::ChoosePantheonBelief(PlayerTypes ePlayer)`:

GUIDE_PATTERN: 1. Get game religion manager with `GC.getGame().GetGameReligions()`.

GUIDE_PATTERN: 2. Create `CvWeightedVector<BeliefTypes> beliefChoices`.

GUIDE_PATTERN: 3. Get `availableBeliefs` from `GetAvailablePantheonBeliefs(ePlayer)`.

GUIDE_PATTERN: 4. Calculate helper plot weights with `CalculatePlotWeightsForBeliefSelection(...)`.

GUIDE_PATTERN: 5. Loop available beliefs.

GUIDE_PATTERN: 6. Get `CvBeliefEntry*` from `m_pBeliefs->GetEntry(eBelief)`.

GUIDE_PATTERN: 7. If entry exists, score it with `ScoreBelief(...)`.

GUIDE_PATTERN: 8. Add `(eBelief, iScore)` with `beliefChoices.push_back(...)`.

GUIDE_PATTERN: 9. Sort with `StableSortItems()`.

GUIDE_PATTERN: 10. Select with `ChooseAbovePercentThreshold(...)`, using handicap threshold and seeded randomness.

GUIDE_PATTERN: 11. Log with `LogBeliefChoices(...)`.

GUIDE_PATTERN: 12. Return selected belief.

GUIDE_PATTERN: `ScoreBelief(...)` should evaluate usefulness from belief properties. A score of 0 means not useful, higher values mean more useful. The guide explicitly says a scoring function should have code for every member variable of `CvBeliefEntry`, which is a useful standard when checking coverage.

GUIDE_PATTERN: This quickref is derived from the VP Modding Guide, which is useful but may be stale. It should guide source exploration, not replace it.

GUIDE_PATTERN: The recurring AI pattern from the guide is:
candidate generation -> non-negative integer scoring -> weighted/sorted selection -> seeded randomness in some cases -> final application path.

GUIDE_PATTERN: Before documenting any specific AI behavior, verify the actual current implementation in source.

GUIDE_PATTERN: Source: `VP_Modding_Guide.pdf`, especially sections 3.2, 3.4, 3.5, and 3.6.4. This file is a short routing aid. The source code remains authoritative.

GUIDE_PATTERN: Many turn processing paths split on `isHuman()`. For example, pantheon selection routes humans to UI selection, while AI calls `GetReligionAI()->ChoosePantheonBelief(...)`. The final application function should usually be shared between human and AI paths. For beliefs, the guide notes that `FoundPantheon(...)` processes the effects for both human and AI choices.

GUIDE_PATTERN: For each investigated decision, identify:

GUIDE_PATTERN: - Human path.

GUIDE_PATTERN: - AI path.

GUIDE_PATTERN: - Shared final application function.

GUIDE_PATTERN: - Whether Lua/UI enters the same final DLL function.

GUIDE_PATTERN: This helps avoid documenting UI code as AI code.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
