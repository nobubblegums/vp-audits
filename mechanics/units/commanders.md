# Generals, admirals and optional local/era behavior

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How are commanders created, scoped and applied under the optional features?

Local-city origin, nearest-city selection, era restriction and actual combat-bonus authority are different paths. External promotion/resource activation and missing-capital cases materially limit applicability.

Canonical related owners: [Optional feature activation and dependency map](<../../architecture/optional-features.md>), [Flagship grants and naval promotion interactions](<naval-promotions.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

`CvPlayerAI::AI_updateGreatPeople` calls `GetDirectiveGeneral` at `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1390` and `GetDirectiveAdmiral` at `CvPlayerAI.cpp:1396`, then writes directives through `SetGreatPeopleDirective` at `CvPlayerAI.cpp:1405`.

Create Great General and Great Admiral units with the negate promotions and force AI great-person directive update.

Combat XP gain calls `CvUnit::changeExperienceTimes100`, which updates player-level great-person progress.

Earn land and naval combat XP from a specific unit until the great-person threshold is crossed.

Requires `GLOBAL_LOCAL_GENERALS` to pass `pFromUnit` from `CvUnit::changeExperienceTimes100`.

Cross land and naval great-person XP thresholds near an owned city, far from an owned city, and with no reachable city.

## Application and state

When the obsolete promotion condition is met, both directive functions return `NO_GREAT_PEOPLE_DIRECTIVE_TYPE` instead of field-command or improvement directives. `CvHomelandAI` consumes great-person directives in great-person movement and mission paths, including `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp:4237`, `CvHomelandAI.cpp:4357`, `CvHomelandAI.cpp:4476`, `CvHomelandAI.cpp:4595`, and `CvHomelandAI.cpp:4839`.

Downstream effects are Great General or Great Admiral unit creation at a local plot or nearest city when the XP threshold is crossed.

Downstream state is the spawn plot of the generated Great General or Great Admiral.

`GLOBAL_LOCAL_GENERALS` and `LOCAL_GENERALS_NEAREST_CITY` add no serialized field. They change only which plot is passed to `createGreatGeneral` or `createGreatAdmiral` during threshold crossing.

Combat XP counters are serialized at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:43989` through `CvPlayer.cpp:43991` and sync-archived at `CvGameCoreDLL_Expansion2/CvPlayer.h:4123` through `CvPlayer.h:4125`.

Great General/Admiral creation increments created counters and threshold modifiers in `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44482` through `CvPlayer.cpp:44490` and `CvPlayer.cpp:44534` through `CvPlayer.cpp:44542`.

LIKELY_DEFECT: `CvPlayer::createGreatGeneral` and `CvPlayer::createGreatAdmiral` both dereference `pOriginCity` after an optional null check. `createGreatGeneral` calls `pOriginCity->addProductionExperience(...)` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44480`; `createGreatAdmiral` does the same at `CvPlayer.cpp:44532`.

Local non-city spawns do not set `m_iOriginCity` directly unless `plot()->getOwningCity()` exists and the plot owner matches the unit owner at `CvGameCoreDLL_Expansion2/CvUnit.cpp:597` through `CvUnit.cpp:602`. `CvUnit::getOriginCity()` falls back to the capital when `m_iOriginCity == -1` and `getCapitalCity() != NULL` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21117` through `CvUnit.cpp:21118`, then returns `NULL` if no origin and no capital exist at `CvUnit.cpp:21120` through `CvUnit.cpp:21121`.

INFERRED: This origin-city risk is exposed by `GLOBAL_LOCAL_GENERALS` when `LOCAL_GENERALS_NEAREST_CITY=0` and by naval nearest-city fallback when no suitable coastal city exists, but it likely requires a no-capital/no-origin edge case. It is less likely in city-spawn paths, where `initUnit` is called on an owned city plot.

`createGreatGeneral` has a downstream normalization step: if the new unit's plot is water or the unit is a combat unit, it calls `jumpToNearestValidPlot()` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44493` through `CvPlayer.cpp:44498`. `createGreatAdmiral` has no equivalent relocation after creation at `CvPlayer.cpp:44522` through `CvPlayer.cpp:44553`.

`BALANCE_ERA_RESTRICTED_GENERALS` relies on ordinary unit promotion serialization at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28785` and `CvPromotionClasses.cpp:3690` through `CvPromotionClasses.cpp:3693`. Missing negate promotion rows are runtime data dependencies, not serialized mod-option state.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:509` with default `Value="0"` and `DbUpdates="1"`. Macro is `MOD_BALANCE_ERA_RESTRICTED_GENERALS` in `CvGameCoreDLL_Expansion2/CustomMods.h:437`; declaration is `CvGameCoreDLL_Expansion2/CustomMods.h:1633`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:410`.

The XML option is cached by `CustomMods::preloadCache`; C++ guarded behavior checks `gCustomMods.isBALANCE_ERA_RESTRICTED_GENERALS()` through the macro.

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:269` with default `Value="0"`. Macro is `MOD_GLOBAL_LOCAL_GENERALS` in `CvGameCoreDLL_Expansion2/CustomMods.h:224`; declaration is `CustomMods.h:1562`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:339`.

The macro enables passing the combat XP source unit into player great-general and great-admiral XP counters.

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:270` with row default `Value="1"`. Macro is `MOD_LOCAL_GENERALS_NEAREST_CITY` in `CvGameCoreDLL_Expansion2/CustomMods.h:225`; declaration is `CustomMods.h:1563`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:340`.

The macro is `(MOD_GLOBAL_LOCAL_GENERALS && gCustomMods.isLOCAL_GENERALS_NEAREST_CITY())`, so the row default is ineffective unless `GLOBAL_LOCAL_GENERALS` is enabled.

`GLOBAL_LOCAL_GENERALS` is defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:269`, cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:339`, and exposed as `MOD_GLOBAL_LOCAL_GENERALS` in `CvGameCoreDLL_Expansion2/CustomMods.h:224`.

`LOCAL_GENERALS_NEAREST_CITY` is defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:270`, cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:340`, and exposed as `MOD_LOCAL_GENERALS_NEAREST_CITY` in `CvGameCoreDLL_Expansion2/CustomMods.h:225`. The macro is `(MOD_GLOBAL_LOCAL_GENERALS && gCustomMods.isLOCAL_GENERALS_NEAREST_CITY())`, so the row default `Value="1"` is inert unless `GLOBAL_LOCAL_GENERALS` is enabled.

`BALANCE_ERA_RESTRICTED_GENERALS` is defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:509` with `Value="0"` and `DbUpdates="1"`, cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:410`, and exposed as `MOD_BALANCE_ERA_RESTRICTED_GENERALS` in `CvGameCoreDLL_Expansion2/CustomMods.h:437`.

`DbUpdates="1"` is an activation dependency. `CustomMods::getOption` disables a true option value unless `CustomModDbUpdates` has `Name='<option>' AND Value=1` at `CvGameCoreDLL_Expansion2/CustomMods.cpp:230` through `CvGameCoreDLL_Expansion2/CustomMods.cpp:244`. `CommunityTables.xml:16` through `CommunityTables.xml:24` documents the same requirement.

MISSING_VALIDATION: Repository search found no `CustomModDbUpdates` row for `BALANCE_ERA_RESTRICTED_GENERALS`, and no repository data rows for the hardcoded negate promotions `PROMOTION_NEGATE_GENERAL`, `PROMOTION_NEGATE_GENERAL_S`, `PROMOTION_NEGATE_GENERAL_P`, `PROMOTION_NEGATE_GENERAL_SP`, `PROMOTION_NEGATE_ADMIRAL`, `PROMOTION_NEGATE_ADMIRAL_S`, `PROMOTION_NEGATE_ADMIRAL_P`, or `PROMOTION_NEGATE_ADMIRAL_SP`.

| `GLOBAL_LOCAL_GENERALS` | `LOCAL_GENERALS_NEAREST_CITY` | `BALANCE_ERA_RESTRICTED_GENERALS` | Required external data | Expected DLL behavior | Audit classification |
| --- | --- | --- | --- | --- | --- |
| 0 | 0 or 1 | 0 | none | Combat thresholds spawn Great Generals/Admirals from ranked city paths; nearest-city row is macro-inert. | working as intended |
| 1 | 0 | 0 | none | Combat thresholds spawn on the triggering unit plot; unsourced point paths still use city paths. | compatibility risk / likely defect in no-capital local-spawn edge cases |
| 1 | 1 | 0 | none | Land attempts nearest owned city and can null-deref; naval uses nearest suitable coastal city or source-plot fallback. | confirmed defect for land null path; compatibility risk for source-plot fallback origin-city handling |
| 0 | any | 1 | `CustomModDbUpdates` row plus negate promotions | Spawn behavior unchanged; AI directive suppression can apply to promoted Great Generals/Admirals. | compatibility risk / runtime unvalidated |
| 1 | 0 | 1 | same as above | Local spawning and AI directive suppression are independent; local spawn can create units whose later AI directive is suppressed if external promotions are present. | likely defect plus compatibility risk |
| 1 | 1 | 1 | same as above | Combines nearest-city placement risks with directive suppression. | confirmed defect plus dependency unsafe |
| any | any | 1 | missing `CustomModDbUpdates` row | Option remains disabled even if `CustomModOptions.Value` is set to 1. | missing validation |
| any | any | 1 | missing negate promotions | Debug precondition noise and no directive suppression; release likely inert. | missing validation / compatibility risk |

Unsupported operational combinations: enabling `BALANCE_ERA_RESTRICTED_GENERALS` without its data update marker and negate promotion rows is unsupported. Enabling `GLOBAL_LOCAL_GENERALS=1` with `LOCAL_GENERALS_NEAREST_CITY=1` is supported by macro intent, but the current land path is unsafe when the closest-city cache cannot return a city.

## Observability and validation obligations

Log option state, resolved promotion IDs, `isHasPromotion` result, and final directive in `CvPlayerAI::GetDirectiveGeneral` and `CvPlayerAI::GetDirectiveAdmiral`.

Log `CvUnit::changeExperienceTimes100`, player XP totals, `pFromUnit`, threshold crossing, and final spawn plot.

Log `pFromUnit`, source plot, closest city result, coastal city result, and final spawn plot.

1. `GLOBAL_LOCAL_GENERALS=0`, `LOCAL_GENERALS_NEAREST_CITY=1`: cross land and naval combat thresholds. Expected: row is macro-inert; both use ranked city/coastal city spawn paths.

2. `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=0`: cross a land threshold on a land combat plot and a naval threshold on a water combat plot. Expected current risk: local spawn path reaches `createGreatGeneral`/`createGreatAdmiral`; validate `getOriginCity()` null behavior.

3. `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1`: cross a land threshold near a normal owned city. Expected: Great General spawns at nearest city and does not crash.

4. Same as test 3, but make `GetClosestCityByPathLength` return null through a no-city/stale-cache/disconnected edge case. Expected current result: crash at `CvPlayer.cpp:32228`; patched result should fall back safely.

5. `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1`: cross a naval threshold near an owned coastal city connected to the source landmass/ocean. Expected: Admiral spawns at that coastal city.

6. Same as test 5 with no suitable coastal city. Expected current result: naval fallback spawns at source plot; validate origin-city null behavior.

7. Unsourced land and naval point thresholds from city/religion/policy/minor yields and Lua player XP setters. Expected: no local placement; ranked city/coastal-city path remains authoritative.

8. `BALANCE_ERA_RESTRICTED_GENERALS=1` with missing `CustomModDbUpdates`: verify `gCustomMods.isBALANCE_ERA_RESTRICTED_GENERALS()` remains false.

9. `BALANCE_ERA_RESTRICTED_GENERALS=1` with activation marker but missing negate promotions in debug build: verify invalid-promotion preconditions at `CvPromotionClasses.cpp:3728` and `CvPromotionClasses.cpp:3729`; release should be inert.

10. `BALANCE_ERA_RESTRICTED_GENERALS=1` with valid external negate promotions: give each promotion to AI Great General and Great Admiral units, save before `AI_updateGreatPeople`, reload, run the update, and verify directive remains `NO_GREAT_PEOPLE_DIRECTIVE_TYPE`.

11. With valid negate promotions, place obsolete Great General/Admiral near friendly combat units and resolve combat. Expected current-source uncertainty: directive suppression alone may not suppress combat aura; validate actual combat modifier behavior before claiming the two-era combat limit works.

Add diagnostic-only logging in `CvUnit::changeExperienceTimes100`, `CvPlayer::setCombatExperienceTimes100`, `CvPlayer::setNavalCombatExperienceTimes100`, `CvPlayer::createGreatGeneral`, `CvPlayer::createGreatAdmiral`, `CvPlayerAI::GetDirectiveGeneral`, and `CvPlayerAI::GetDirectiveAdmiral`.

Log option states, source unit owner/ID/type/domain/plot, source unit delayed-death/projected-death state, previous/new XP counters, threshold, selected great-person unit type, city lookup result, final spawn plot, `getOriginCity()` result, negate promotion names/IDs, directive before/after, and whether combat aura modifiers are still applied to nearby units.

Add debug assertions after nearest-city lookup only if a fallback exists in release code. Add one-time warnings for active `BALANCE_ERA_RESTRICTED_GENERALS` with missing negate promotion IDs or missing activation marker.

## Version differences, risks and unresolved questions

`git log -S'MOD_BALANCE_ERA_RESTRICTED_GENERALS' -- CvGameCoreDLL_Expansion2 '(1) Community Patch'` found current production-source provenance at commit `54dee87` only. `git blame` on the guarded lines also attributes them to `54dee87`.

No compile-time dependency on the promotion rows exists because promotion IDs are looked up by string at runtime.

DEPENDENCY_UNSAFE: Missing promotion rows may make the feature inert or unsafe depending on `NO_PROMOTION` handling in promotion storage. Runtime validation is required.

Local `git log -S'MOD_GLOBAL_LOCAL_GENERALS'` finds production-source provenance at `54dee87` only.

No compile-time risk was found for this option alone.

INFERRED: Incorrect or null `pFromUnit` propagation would silently fall back to city-rank spawn behavior in downstream code.

Local `git log -S'MOD_LOCAL_GENERALS_NEAREST_CITY'` finds production-source provenance at `54dee87` only.

No compile-time risk was found.

INFERRED: The land path has a possible null dereference if no closest city is found. The naval path is null-safe and falls back to `pFromUnit->plot()` at `CvPlayer.cpp:32355`.

### Recovered finding: Confirmed defect: land nearest-city spawn null-dereferences `pNearestCity`

Classification: confirmed defect.

Exact location: `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32226` through `CvPlayer.cpp:32229`.

```text
CvUnitCombat XP
  -> CvUnit::changeExperienceTimes100(..., bUpdateGlobal=true)
  -> CvPlayer::changeCombatExperienceTimes100(..., pFromUnit=this)
  -> CvPlayer::setCombatExperienceTimes100
  -> pFromUnit && MOD_LOCAL_GENERALS_NEAREST_CITY
  -> GetClosestCityByPathLength(pFromUnit->plot())
  -> pNearestCity->plot()
```

Intended behavior: With `LOCAL_GENERALS_NEAREST_CITY`, a combat-generated Great General should spawn at the nearest suitable owned city instead of on the combat tile.

Observed/static failure: `CvGame::GetClosestCityByPathLength` can return `NULL`, and the land branch dereferences it immediately in both `CUSTOMLOG` and `createGreatGeneral`. The naval branch already shows the expected null-safe pattern.

Why the surrounding rewrite caused or exposed it: INFERRED: Current architecture uses cached city-distance lookups that can represent no reachable/known city by returning `NO_PLAYER` and `NULL`. The local-generals rewrite assumed an owned city was always available from the source combat plot.

Smallest safe correction: Mirror the naval fallback: if `pNearestCity != NULL`, spawn at city; otherwise log and spawn at `pFromUnit->plot()` or fall back to a ranked `pBestCity` if one exists.

Possible regression risks: Source-plot fallback can expose the existing `createGreatGeneral` origin-city null risk. Ranked-city fallback preserves city-origin assumptions but may violate the feature's "nearest city" intent when the cache misses.

Runtime scenario needed: Enable `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1`; trigger a land Great General threshold on a plot where `GetClosestCityByPathLength` returns null, such as after razing/losing the only city, stale cache, unusual minor/cityless state, or a disconnected/path-cache edge case.

### Recovered finding: Likely defect: no-capital local Great General/Admiral creation can dereference null origin city

Classification: likely defect.

Exact location: `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44476` through `CvPlayer.cpp:44480` and `CvPlayer.cpp:44528` through `CvPlayer.cpp:44532`.

```text
CvPlayer::setCombatExperienceTimes100 or setNavalCombatExperienceTimes100
  -> createGreatGeneral/createGreatAdmiral at source plot or naval fallback plot
  -> initUnit
  -> pGreatPeopleUnit->getOriginCity()
  -> optional DoGreatPersonSpawnBonus
  -> unconditional pOriginCity->addProductionExperience(...)
```

Intended behavior: A Great General or Admiral generated from combat should be created and have normal spawn bonuses/threshold increments applied.

Observed/static failure: The function guards `DoGreatPersonSpawnBonus` with `if (pOriginCity)` but then unconditionally dereferences `pOriginCity`. `CvUnit::getOriginCity()` falls back to the player's capital when no origin city is stored, but can return null if the player has no capital. In that no-capital/no-origin edge case, local creation can crash.

Why the surrounding rewrite caused or exposed it: INFERRED: The older city-spawn path likely made a non-null origin city common. `GLOBAL_LOCAL_GENERALS` and naval fallback introduce normal threshold paths that call these helpers with arbitrary combat plots.

Smallest safe correction: Guard `addProductionExperience` with the same `if (pOriginCity)` used for `DoGreatPersonSpawnBonus`, or derive a fallback origin city before applying production experience.

Possible regression risks: Skipping `addProductionExperience` for no-origin local spawns may change starting XP for locally spawned Great Generals/Admirals if some building effects are expected. Choosing a fallback city changes which city's production experience is applied.

Runtime scenario needed: Enable `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=0`; trigger land and naval thresholds on non-city plots with a normal capital and in a no-capital edge case, then inspect whether the spawned great person's `getOriginCity()` is null before `addProductionExperience`.

### Missing validation: era-restricted generals option cannot self-activate from current repository data

Classification: missing validation.

Exact location: `(1) Community Patch/Database Changes/NewCustomModOptions.xml:509`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:230` through `CustomMods.cpp:244`, and `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1715` through `CvPlayerAI.cpp:1728` / `CvPlayerAI.cpp:1864` through `CvPlayerAI.cpp:1877`.

```text
CustomModOptions row with DbUpdates=1
  -> CustomMods::getOption requires CustomModDbUpdates row
  -> MOD_BALANCE_ERA_RESTRICTED_GENERALS
  -> CvPlayerAI::GetDirectiveGeneral/GetDirectiveAdmiral
  -> hardcoded negate-promotion lookup
```

Intended behavior: Great Generals and Great Admirals beyond their allowed era window should stop receiving AI commander directives.

Observed/static failure: The repository provides neither the activation marker nor the negate-promotion rows. If an external config sets only `CustomModOptions.Value=1`, the DLL still disables the option. If the option is forced active without negate promotions, the lookup returns `NO_PROMOTION` and the check is release-inert while debug builds hit invalid-index preconditions.

Why the surrounding rewrite caused or exposed it: INFERRED: The current VP repository no longer carries the database payload that this optional DLL code expects, while the C++ guard remains.

Smallest safe correction: Add valid-promotion guards before `isHasPromotion`, add a one-time diagnostic for missing negate rows when the option is active, and document or restore the required `CustomModDbUpdates`/promotion data package.

Possible regression risks: Guard/log-only changes do not alter release gameplay except diagnostics. Restoring promotion rows is savegame-sensitive because promotion bitfields are serialized by promotion index.

Runtime scenario needed: Load with a valid external data package; verify `MOD_BALANCE_ERA_RESTRICTED_GENERALS` resolves true, each negate promotion ID resolves non-negative, and promoted Great Generals/Admirals receive `NO_GREAT_PEOPLE_DIRECTIVE_TYPE`.

### Compatibility risk: era restriction is not a combat-bonus authority

Classification: compatibility risk.

The only guarded C++ references to `MOD_BALANCE_ERA_RESTRICTED_GENERALS` are in `CvPlayerAI::GetDirectiveGeneral` and `CvPlayerAI::GetDirectiveAdmiral`. Combat-bonus paths continue to inspect nearby/stacked Great General and Great Admiral state independently.

Observed/static risk: The feature name/comment says combat bonuses are limited to two eras, but the current guarded source only suppresses AI directives for units carrying external negate promotions. Existing nearby units may still receive combat bonuses unless external data/promotion mechanics remove, negate, or hide the great-person aura.

Smallest safe correction: Before changing code, validate the intended external data model. If the feature is meant to enforce combat bonuses in DLL, the combat modifier functions need explicit era-window checks or validated negate-promotion handling at modifier sites, not only directive selection.

BASELINE_DEFECT_REPORT: `GLOBAL_LOCAL_GENERALS=1` plus `LOCAL_GENERALS_NEAREST_CITY=1` exposes the land nearest-city null dereference.

LIKELY_DEFECT: `GLOBAL_LOCAL_GENERALS=1` plus `LOCAL_GENERALS_NEAREST_CITY=0` exposes no-capital/non-city origin assumptions in `createGreatGeneral` and `createGreatAdmiral`.

COMPATIBILITY_RISK: `LOCAL_GENERALS_NEAREST_CITY=1` naval fallback can still spawn on the source plot when no suitable coastal city exists, so it can expose the same origin-city risk as local spawning even though the nearest-city lookup itself is null-safe.

COMPATIBILITY_RISK: `BALANCE_ERA_RESTRICTED_GENERALS` is mechanically independent from local spawn placement. It can suppress AI use of a locally generated Great General/Admiral only after external promotions are applied and `AI_updateGreatPeople` runs; it does not prevent spawn, threshold changes, notifications, or combat aura state by itself.

1. Add null handling in the land `LOCAL_GENERALS_NEAREST_CITY` branch before using `pNearestCity`.

2. Guard `pOriginCity->addProductionExperience(...)` in `CvPlayer::createGreatGeneral` and `CvPlayer::createGreatAdmiral`, or choose an explicit fallback city before applying production XP.

3. Add valid-promotion checks and diagnostics for `BALANCE_ERA_RESTRICTED_GENERALS` negate promotion lookups.

4. Decide whether `BALANCE_ERA_RESTRICTED_GENERALS` remains an external-data compatibility hook or should be made self-contained. If self-contained, add append-only promotion/data rows and `CustomModDbUpdates` marker with savegame/order risk called out.

5. If design requires true two-era combat-bonus enforcement in DLL, add validated era-window checks at combat-aura/modifier authority points after runtime evidence confirms the intended behavior.

`GLOBAL_LOCAL_GENERALS`: conditionally usable only after runtime validation of local non-city spawns. It still has a likely no-capital origin-city edge case in `createGreatGeneral` and `createGreatAdmiral`.

`LOCAL_GENERALS_NEAREST_CITY`: not ready for normal use. The land nearest-city path has a confirmed null-dereference defect; the naval path is null-safe for city lookup but can still fall back into source-plot origin-city risk.

`BALANCE_ERA_RESTRICTED_GENERALS`: not ready for normal self-contained use from audited repository data. The C++ directive hook is active when the option is truly enabled, but the required activation marker and negate promotion rows are absent from repository data, and the source does not prove combat-bonus enforcement.

## Source and dependency lookup

Direct references are `CustomMods.h:437`, `CustomMods.h:1633`, `CustomMods.cpp:410`, `NewCustomModOptions.xml:509`, `CvPlayerAI.cpp:1715`, and `CvPlayerAI.cpp:1864`.

`CvPlayerAI::GetDirectiveGeneral(CvUnit*)` in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1712` checks hardcoded promotion names `PROMOTION_NEGATE_GENERAL`, `PROMOTION_NEGATE_GENERAL_S`, `PROMOTION_NEGATE_GENERAL_P`, and `PROMOTION_NEGATE_GENERAL_SP` at `CvPlayerAI.cpp:1718`.

`CvPlayerAI::GetDirectiveAdmiral(CvUnit*)` in `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1861` checks hardcoded promotion names `PROMOTION_NEGATE_ADMIRAL`, `PROMOTION_NEGATE_ADMIRAL_S`, `PROMOTION_NEGATE_ADMIRAL_P`, and `PROMOTION_NEGATE_ADMIRAL_SP` at `CvPlayerAI.cpp:1867`.

The guarded code depends on promotion IDs resolved by `GC.getInfoTypeForString(...)`. Repository search found the C++ string literals but did not find current XML/SQL definitions for the `PROMOTION_NEGATE_GENERAL*` or `PROMOTION_NEGATE_ADMIRAL*` rows.

INFERRED: The feature assumes missing promotion names do not produce unsafe `NO_PROMOTION` behavior when passed into `CvUnit::isHasPromotion`.

INFERRED: The feature assumes current great-person AI directive calculation remains the authoritative path before Homeland AI consumes directives.

INFERRED: No replacement directive path was found for great general/admiral directive selection. However, manual unit orders or Lua/unit effects that bypass `CvPlayerAI::AI_updateGreatPeople` would not be affected.

Save before `CvPlayerAI::AI_updateGreatPeople`, reload, and confirm the same promoted great person still receives no directive.

`DEPENDENCY_UNSAFE`

This option has an extra activation dependency because its `CustomModOptions` row has `DbUpdates="1"` at `(1) Community Patch/Database Changes/NewCustomModOptions.xml:509`. `CustomMods::getOption` requires a matching `CustomModDbUpdates` row when an option has `DbUpdates=1` at `CvGameCoreDLL_Expansion2/CustomMods.cpp:230` through `CustomMods.cpp:244`; `CommunityTables.xml:16` through `CommunityTables.xml:24` documents that requirement.

MISSING_VALIDATION: Repository search found no `CustomModDbUpdates` row for `BALANCE_ERA_RESTRICTED_GENERALS`. Therefore setting only `CustomModOptions.Value=1` is not sufficient to activate the feature in the current architecture.

MISSING_VALIDATION: Repository search found no current XML/SQL definitions for the hardcoded negate promotions checked in `CvPlayerAI::GetDirectiveGeneral` and `CvPlayerAI::GetDirectiveAdmiral`: `PROMOTION_NEGATE_GENERAL`, `PROMOTION_NEGATE_GENERAL_S`, `PROMOTION_NEGATE_GENERAL_P`, `PROMOTION_NEGATE_GENERAL_SP`, `PROMOTION_NEGATE_ADMIRAL`, `PROMOTION_NEGATE_ADMIRAL_S`, `PROMOTION_NEGATE_ADMIRAL_P`, and `PROMOTION_NEGATE_ADMIRAL_SP`.

COMPATIBILITY_RISK: Missing negate promotions are not fully silent in debug builds. The directive code casts `GC.getInfoTypeForString(..., true)` to `PromotionTypes` and passes it to `isHasPromotion` at `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1726` through `CvPlayerAI.cpp:1728` and `CvPlayerAI.cpp:1875` through `CvPlayerAI.cpp:1877`; `CvUnitPromotions::HasPromotion` has invalid-index preconditions at `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:3728` and `CvPromotionClasses.cpp:3729`.

The guarded source does not directly enforce combat-bonus limits. The only `MOD_BALANCE_ERA_RESTRICTED_GENERALS` checks are in AI directive functions, while combat bonus state is consumed by normal Great General/Admiral helpers such as `CvUnit::IsNearGreatGeneral`, `CvUnit::IsStackedGreatGeneral`, `CvUnit::GetReverseGreatGeneralModifier`, and `CvUnit::GetGreatGeneralCombatModifier`.

INFERRED: The current C++ hook can make AI stop using obsolete promoted Great Generals/Admirals as field commanders, but the advertised two-era combat-bonus limit requires external data/promotion behavior or additional DLL enforcement not verified in audited source.

Smallest safe correction: Add valid-promotion guards and diagnostics in the directive functions, then decide whether the required activation marker and negate promotion rows remain external dependencies or should be restored/bundled.

Runtime validation required: Load a configuration with `CustomModDbUpdates('BALANCE_ERA_RESTRICTED_GENERALS', 1)` and valid negate promotions, assign each promotion to AI Great General/Admiral units, run `AI_updateGreatPeople`, and separately verify whether nearby combat units still receive Great General/Admiral combat modifiers.

Readiness: Not ready for normal self-contained use from audited repository data.

Direct references are `CustomMods.h:224`, `CustomMods.h:1562`, `CustomMods.cpp:339`, `NewCustomModOptions.xml:269`, `CvUnit.cpp:21753`, `CvUnit.cpp:21757`, `CvUnit.cpp:21767`, and `CvUnit.cpp:21771`.

`CvUnit::changeExperienceTimes100` passes `this` instead of `NULL` into `CvPlayer::changeCombatExperienceTimes100` and `CvPlayer::changeNavalCombatExperienceTimes100` when enabled at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753` and `CvUnit.cpp:21767`.

The resulting authoritative spawn paths are `CvPlayer::setCombatExperienceTimes100` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32157` and `CvPlayer::setNavalCombatExperienceTimes100` in `CvPlayer.cpp:32269`.

`LOCAL_GENERALS_NEAREST_CITY` depends on this option in `CustomMods.h:225`. `PROMOTIONS_FLAGSHIP` practically depends on this feature for its guarded promotion grant path because the grant is under `pFromUnit` admiral spawn handling in `CvPlayer.cpp:32340` and `CvPlayer.cpp:32359`.

INFERRED: The triggering unit pointer must remain valid through the XP update and spawn logic.

INFERRED: Any XP path that updates player combat experience without going through `CvUnit::changeExperienceTimes100` would bypass the local unit pointer.

Save immediately before a threshold-crossing combat XP event, reload, and confirm local spawn behavior.

`ACTIVE_AND_REACHABLE`

`MOD_GLOBAL_LOCAL_GENERALS` changes only source-unit propagation. In `CvUnit::changeExperienceTimes100`, combat XP with `bUpdateGlobal=true` passes `this` to `CvPlayer::changeNavalCombatExperienceTimes100` or `CvPlayer::changeCombatExperienceTimes100` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753`, `CvUnit.cpp:21757`, `CvUnit.cpp:21767`, and `CvUnit.cpp:21771`; disabled behavior passes `NULL`.

The authoritative downstream functions are still `CvPlayer::setCombatExperienceTimes100` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32157` and `CvPlayer::setNavalCombatExperienceTimes100` at `CvPlayer.cpp:32269`.

Unsourced point paths still call the same player XP counters without `pFromUnit`, including `CvGameCoreDLL_Expansion2/CvPlayer.cpp:18922`, `CvPlayer.cpp:18946`, `CvPlayer.cpp:19502`, `CvPlayer.cpp:19526`, `CvPlayer.cpp:24216`, `CvPlayer.cpp:24302`, `CvPlayer.cpp:26691`, `CvPlayer.cpp:26700`, `CvPlayer.cpp:36852`, `CvPlayer.cpp:36858`, and Lua wrappers at `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:10375`, `CvLuaPlayer.cpp:10385`, `CvLuaPlayer.cpp:10423`, and `CvLuaPlayer.cpp:10433`. These bypass local placement by design.

LIKELY_DEFECT: Local source-plot spawning can expose a no-capital/null-origin-city crash in `CvPlayer::createGreatGeneral` and `CvPlayer::createGreatAdmiral`. Both functions guard `DoGreatPersonSpawnBonus` with `if (pOriginCity)` but then unconditionally call `pOriginCity->addProductionExperience(...)` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44480` and `CvPlayer.cpp:44532`.

A local non-city spawn sets no explicit origin city unless `plot()->getOwningCity()` exists and the plot owner matches the unit owner at `CvGameCoreDLL_Expansion2/CvUnit.cpp:597` through `CvUnit.cpp:602`. `CvUnit::getOriginCity()` normally falls back to the capital at `CvUnit.cpp:21117` through `CvUnit.cpp:21118`, but returns null if there is no stored origin and no capital at `CvUnit.cpp:21120` through `CvUnit.cpp:21121`.

INFERRED: The rewrite from city-ranked spawn to source-plot spawn exposed this assumption because a Great General or Admiral initialized on a combat plot may rely on capital fallback instead of an actual origin city.

Smallest safe correction: Guard `pOriginCity->addProductionExperience(...)` or select an explicit fallback city before applying production experience.

Runtime validation required: Enable `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=0`, cross land and naval thresholds on non-city plots in normal and no-capital edge cases, and inspect `getOriginCity()` before production experience is applied.

Readiness: Conditionally usable only after runtime validation of local non-city spawns; still carries a likely no-capital origin-city edge case.

Direct references are `CustomMods.h:225`, `CustomMods.h:1563`, `CustomMods.cpp:340`, `NewCustomModOptions.xml:270`, `CvPlayer.cpp:32219`, `CvPlayer.cpp:32224`, `CvPlayer.cpp:32335`, and `CvPlayer.cpp:32345`.

`CvPlayer::setCombatExperienceTimes100` creates a Great General at the triggering unit plot when disabled and at the closest city path result when enabled at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32219` and `CvPlayer.cpp:32224`.

`CvPlayer::setNavalCombatExperienceTimes100` creates a Great Admiral at the triggering unit plot when disabled and at `OperationalAIHelpers::GetClosestFriendlyCoastalCity(...)` when enabled at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32335` and `CvPlayer.cpp:32345`.

Explicit macro dependency on `GLOBAL_LOCAL_GENERALS`.

INFERRED: The land general path assumes `GetClosestCityByPathLength(pFromUnit->plot())` returns a non-null city before dereferencing `pNearestCity->plot()` at `CvPlayer.cpp:32228`.

INFERRED: City-rank great-person spawn remains a fallback only when `pFromUnit` is null; with local generals enabled, this option controls location selection for threshold crossings sourced from units.

Save just before threshold crossing and after spawn; reload and verify spawn plot consistency.

The land branch in `CvPlayer::setCombatExperienceTimes100` calls `GetClosestCityByPathLength(pFromUnit->plot())` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32226` and then dereferences `pNearestCity->plot()` at `CvPlayer.cpp:32228` and `CvPlayer.cpp:32229` without a null check.

The helper contract does not make that dereference safe. `CvPlayer::GetClosestCityByPathLength` delegates to `GC.getGame().GetClosestCityByPathLength(pPlot, GetID())` for majors at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:40744` through `CvPlayer.cpp:40748`; `CvGame::GetClosestCityByPathLength(const CvPlot*, PlayerTypes)` explicitly returns `NULL` for null input or no cached owner at `CvGameCoreDLL_Expansion2/CvGame.cpp:13475` through `CvGame.cpp:13485`.

BASELINE_DEFECT_REPORT: With `GLOBAL_LOCAL_GENERALS=1` and `LOCAL_GENERALS_NEAREST_CITY=1`, a land Great General threshold crossing can crash if the closest-city lookup returns null.

The naval branch already contains the safer pattern. It calls `OperationalAIHelpers::GetClosestFriendlyCoastalCity(...)` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32347`, checks `pNearestCity != NULL` at `CvPlayer.cpp:32349`, and falls back to the source plot at `CvPlayer.cpp:32354` through `CvPlayer.cpp:32357`.

LIKELY_DEFECT: The naval source-plot fallback can still expose the same no-capital `createGreatAdmiral` origin-city risk documented for `GLOBAL_LOCAL_GENERALS`, because `CvPlayer::createGreatAdmiral` unconditionally calls `pOriginCity->addProductionExperience(...)` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44532`.

Smallest safe correction: For land, mirror the naval null fallback before logging or spawning. Separately guard origin-city production experience in both Great General and Great Admiral creation helpers.

Runtime validation required: Test land threshold crossing with a forced null closest-city result, normal land nearest-city spawn, naval nearest-coastal-city spawn, and naval no-coastal-city fallback.

Readiness: Not ready for normal use. The land nearest-city branch has a confirmed null dereference, and the naval fallback remains tied to the no-capital source-plot origin-city risk.

Subsystem: Generals and Admirals

Features audited in this pass: `GLOBAL_LOCAL_GENERALS`, `LOCAL_GENERALS_NEAREST_CITY`, `BALANCE_ERA_RESTRICTED_GENERALS`

```text
combat resolution
  -> CvUnitCombat calls CvUnit::changeExperienceTimes100(..., bFromCombat=true, bUpdateGlobal=true)
  -> CvUnit::changeExperienceTimes100
  -> domain split
  -> CvPlayer::changeCombatExperienceTimes100(..., pFromUnit) or changeNavalCombatExperienceTimes100(..., pFromUnit)
  -> CvPlayer::setCombatExperienceTimes100 or setNavalCombatExperienceTimes100
  -> threshold-crossing Great General or Great Admiral creation
```

The `GLOBAL_LOCAL_GENERALS` branch passes `this` as `pFromUnit` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753`, `CvUnit.cpp:21757`, `CvUnit.cpp:21767`, and `CvUnit.cpp:21771`. Disabled behavior passes `NULL`, preserving city-ranked spawn behavior.

Land threshold authority is `CvPlayer::setCombatExperienceTimes100` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32157`. When `pFromUnit && !MOD_LOCAL_GENERALS_NEAREST_CITY`, it spawns at `pFromUnit->plot()` at `CvPlayer.cpp:32219` through `CvPlayer.cpp:32222`. When `pFromUnit && MOD_LOCAL_GENERALS_NEAREST_CITY`, it calls `GetClosestCityByPathLength(pFromUnit->plot())` and then dereferences `pNearestCity->plot()` at `CvPlayer.cpp:32224` through `CvPlayer.cpp:32229`.

Naval threshold authority is `CvPlayer::setNavalCombatExperienceTimes100` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32269`. When `pFromUnit && !MOD_LOCAL_GENERALS_NEAREST_CITY`, it spawns at `pFromUnit->plot()` at `CvPlayer.cpp:32335` through `CvPlayer.cpp:32338`. When `pFromUnit && MOD_LOCAL_GENERALS_NEAREST_CITY`, it calls `OperationalAIHelpers::GetClosestFriendlyCoastalCity(...)` at `CvPlayer.cpp:32347`, spawns at the city if non-null at `CvPlayer.cpp:32349` through `CvPlayer.cpp:32352`, and falls back to the source plot at `CvPlayer.cpp:32354` through `CvPlayer.cpp:32357`.

```text
AI_updateGreatPeople
  -> UNITAI_GENERAL: CvPlayerAI::GetDirectiveGeneral
  -> UNITAI_ADMIRAL: CvPlayerAI::GetDirectiveAdmiral
  -> CvUnit::SetGreatPeopleDirective
  -> CvHomelandAI directive consumers
```

`CvPlayerAI::AI_updateGreatPeople` calls `GetDirectiveGeneral` at `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1390` and `GetDirectiveAdmiral` at `CvPlayerAI.cpp:1396`, then stores the result with `SetGreatPeopleDirective` at `CvPlayerAI.cpp:1405`. The directive field is serialized at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28705` and sync-archived at `CvGameCoreDLL_Expansion2/CvUnit.h:3036`.

`CvPlayer::setCombatExperienceTimes100` and `CvPlayer::setNavalCombatExperienceTimes100` remain the authoritative threshold-crossing functions for combat XP meters. Unsourced point paths call the same setters without `pFromUnit`, including city/religion/policy/minor-yield paths at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:18922`, `CvPlayer.cpp:18946`, `CvPlayer.cpp:19502`, `CvPlayer.cpp:19526`, `CvPlayer.cpp:24216`, `CvPlayer.cpp:24302`, `CvPlayer.cpp:26691`, `CvPlayer.cpp:26700`, `CvPlayer.cpp:36852`, and `CvPlayer.cpp:36858`, plus Lua wrappers at `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:10375`, `CvLuaPlayer.cpp:10385`, `CvLuaPlayer.cpp:10423`, and `CvLuaPlayer.cpp:10433`.

These unsourced paths bypass local placement by design: with `pFromUnit == NULL`, land picks a ranked city at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32172` through `CvPlayer.cpp:32192`, and naval picks a ranked coastal city at `CvPlayer.cpp:32284` through `CvPlayer.cpp:32309`.

`CvPlayer::GetClosestCityByPathLength` delegates major civs to `GC.getGame().GetClosestCityByPathLength(pPlot, GetID())` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:40744` through `CvPlayer.cpp:40748`; minors return `getCapitalCity()` at `CvPlayer.cpp:40750` through `CvPlayer.cpp:40751`.

`CvGame::GetClosestCityByPathLength(const CvPlot*, PlayerTypes)` explicitly returns `NULL` when the input plot is null or the cached closest-city owner is `NO_PLAYER` at `CvGameCoreDLL_Expansion2/CvGame.cpp:13475` through `CvGame.cpp:13485`. Therefore the land nearest-city branch is not protected by the helper contract.

`OperationalAIHelpers::GetClosestFriendlyCoastalCity` explicitly returns `NULL` for invalid player/input and when no owned coastal city satisfies `isCoastal(iMinWaterSize)` and `HasAccessToLandmassOrOcean(...)` at `CvGameCoreDLL_Expansion2/CvAIOperation.cpp:3367` through `CvAIOperation.cpp:3391`; the naval branch handles this.

`CvPlayerAI::GetDirectiveGeneral` and `CvPlayerAI::GetDirectiveAdmiral` remain active directive authorities for AI-owned Great Generals and Great Admirals. No newer replacement directive manager was found for these unit AI types.

`BALANCE_ERA_RESTRICTED_GENERALS` does not modify combat-bonus calculation directly. Combat modifiers still use Great General/Admiral state in `CvUnit::GetReverseGreatGeneralModifier`, `CvUnit::IsNearGreatGeneral`, `CvUnit::IsStackedGreatGeneral`, and `CvUnit::GetGreatGeneralCombatModifier` paths around `CvGameCoreDLL_Expansion2/CvUnit.cpp:16156`, `CvUnit.cpp:16204`, `CvUnit.cpp:16924`, `CvUnit.cpp:16990`, and `CvUnit.cpp:23929` through `CvUnit.cpp:24021`.

INFERRED: The era-restriction feature is implemented by making AI stop assigning field-command or improvement directives to units that already have external negate promotions. It does not enforce the two-era combat-bonus limit for human units, existing deployed units, tactical positioning, or raw combat modifier calculation unless external data/promotions also remove or suppress the combat effects.

With `GLOBAL_LOCAL_GENERALS=0`, combat-generated land and naval great people use city-ranked spawn paths because `CvUnit::changeExperienceTimes100` passes `NULL` to player XP counters.

With `GLOBAL_LOCAL_GENERALS=1` and `LOCAL_GENERALS_NEAREST_CITY=0`, land Great Generals and naval Great Admirals spawn at the triggering unit's plot. Land Great Generals may be relocated afterward by `jumpToNearestValidPlot()` if the plot is water or invalid for the created unit.

With both local options enabled, land Great Generals are intended to spawn at the closest owned city by path length, while naval Great Admirals are intended to spawn at the closest owned suitable coastal city. Current land code lacks the null fallback present in the naval branch.

With `BALANCE_ERA_RESTRICTED_GENERALS=0`, AI Great General/Admiral directives ignore the hardcoded negate-promotion list and proceed through army assignment, war commander needs, improvement checks, and default field-command behavior.

With `BALANCE_ERA_RESTRICTED_GENERALS=1` and valid negate promotions on a Great General or Admiral, `GetDirectiveGeneral` and `GetDirectiveAdmiral` suppress field-command assignment and return `NO_GREAT_PEOPLE_DIRECTIVE_TYPE` when no later improvement/use-power branch applies. With missing negate promotions, `GC.getInfoTypeForString(..., true)` resolves to `NO_PROMOTION`, and `CvUnitPromotions::HasPromotion` trips debug preconditions before returning false at `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:3726` through `CvPromotionClasses.cpp:3735`.

`git log -- CvGameCoreDLL_Expansion2/CvPlayer.cpp CvGameCoreDLL_Expansion2/CvUnit.cpp CvGameCoreDLL_Expansion2/CvPlayerAI.cpp CvGameCoreDLL_Expansion2/CvHomelandAI.cpp CvGameCoreDLL_Expansion2/CustomMods.h CvGameCoreDLL_Expansion2/CustomMods.cpp "(1) Community Patch/Database Changes/NewCustomModOptions.xml"` shows production-source provenance at `54dee87` (`5.2.6 Release`) only.

Targeted `rg` and `git log -S/-G` probes for `MOD_GLOBAL_LOCAL_GENERALS`, `MOD_LOCAL_GENERALS_NEAREST_CITY`, `MOD_BALANCE_ERA_RESTRICTED_GENERALS`, `setCombatExperienceTimes100`, `setNavalCombatExperienceTimes100`, `GetDirectiveGeneral`, `GetDirectiveAdmiral`, and negate promotion names found no later local production-source rewrite after `54dee87`.

INFERRED: The local history is grafted/compressed, so this pass cannot identify the original upstream rewrite that introduced the null assumptions or removed/omitted the era-restriction data package.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `BALANCE_ERA_RESTRICTED_GENERALS` | Great General and Great Admiral each with matching `PROMOTION_NEGATE_*` promotion are evaluated for directives. | Save before `CvPlayerAI::AI_updateGreatPeople`, reload, confirm directive remains suppressed. | `CvPlayerAI::GetDirectiveGeneral`, `CvPlayerAI::GetDirectiveAdmiral`, promotion lookup result. | `CvPlayerAI.cpp:1712`, `CvPlayerAI.cpp:1861` |
| `GLOBAL_LOCAL_GENERALS` | Earn land and naval combat XP from a specific unit until great person threshold is crossed. | Save just before threshold, reload, confirm spawn location and XP counter behavior. | `CvUnit::changeExperienceTimes100`, `CvPlayer::setCombatExperienceTimes100`, `CvPlayer::setNavalCombatExperienceTimes100`. | `CvUnit.cpp:21753`, `CvPlayer.cpp:32157`, `CvPlayer.cpp:32269` |
| `LOCAL_GENERALS_NEAREST_CITY` | With local generals enabled, cross thresholds near owned city, no reachable city, and coastal city cases. | Save before threshold and after spawn; reload to check spawned unit location. | Land nearest-city lookup, naval nearest-coastal-city lookup, null handling. | `CvPlayer.cpp:32224`, `CvPlayer.cpp:32345` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `PROMOTIONS_FLAGSHIP=1`, `GLOBAL_LOCAL_GENERALS=1`, audited repository data | Cross the naval Great Admiral threshold with a surviving naval combat unit. | Save immediately before threshold-crossing combat; reload and repeat. | Admiral should spawn through the local path, but current data leaves `PROMOTION_FLAGSHIP` unresolved and no promotion is granted. | Missing validation/inert grant at `CvPlayer.cpp:32342`, `CvPlayer.cpp:32361`, `CoreDefineChanges.sql:36`, and `CoreDefineChanges.sql:1512`. |
| `PROMOTIONS_FLAGSHIP=1`, `GLOBAL_LOCAL_GENERALS=1`, external valid `PROMOTION_FLAGSHIP` row | Same threshold crossing with `PROMOTION_FLAGSHIP` defined and `GreatGeneralReceivesMovement=1`. | Save after promotion grant; reload and inspect unit promotion and movement donor counter. | Triggering naval unit should have `PROMOTION_FLAGSHIP`; `m_iGreatGeneralReceivesMovementCount` should persist. | Dependency validation for external/bundled promotion data. |
| `PROMOTIONS_FLAGSHIP=1`, `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1`, external valid row | Cross threshold far from a friendly coastal city and near one. | Save before threshold and after spawn; reload and inspect Admiral and source unit. | Admiral should spawn at closest friendly coastal city when found or source plot fallback; source unit should still receive flagship. | Interaction with nearest-city admiral spawn in `CvPlayer.cpp:32347`. |
| `PROMOTIONS_FLAGSHIP=0`, external valid flagship donor | Stack a Great Admiral with a naval unit carrying a movement-donor promotion. | Save before turn reset; reload and process reset. | Great Admiral should restore only to ordinary `maxMoves()`. | Disabled behavior parity for `CvUnit::restoreFullMoves`. |
| `PROMOTIONS_FLAGSHIP=1`, same-plot donor | Stack a Great Admiral with a same-domain unit carrying `GreatGeneralReceivesMovement=1`. | Save before turn reset; reload and process reset. | Great Admiral movement should become `max(admiral maxMoves, donor maxMoves)`. | Movement authority in `CvUnit.cpp:21499` and `CvUnit.cpp:23985`. |
| `PROMOTIONS_FLAGSHIP=1`, adjacent or cross-domain donor | Put a donor on an adjacent plot, then stack a Great Admiral with a land donor or Great General with sea donor. | Save before reset; reload and process reset. | No movement transfer should occur because the scan is same-plot and same-domain. | Boundary behavior in `CvUnit::GetGreatGeneralStackMovement`. |
| Unsourced Great Admiral points | Cross threshold via city/yield paths or Lua player XP setters. | Save before threshold; reload and process the yield/Lua trigger. | Great Admiral can spawn, but no flagship grant should occur because `pFromUnit == NULL`. | Bypass paths at `CvPlayer.cpp:24302` and `CvLuaPlayer.cpp:10423`. |
| `GLOBAL_LOCAL_GENERALS=0`, `LOCAL_GENERALS_NEAREST_CITY=1` | Cross land and naval combat XP thresholds with a specific source unit. | Save before threshold crossing; reload and repeat. | `LOCAL_GENERALS_NEAREST_CITY` should be macro-inert; spawn should use ranked city/coastal-city paths because `pFromUnit == NULL`. | Disabled behavior parity at `CvUnit.cpp:21753`, `CvUnit.cpp:21757`, `CvPlayer.cpp:32172`, and `CvPlayer.cpp:32284`. |
| `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=0` land local spawn | Cross a land Great General threshold on a non-city land plot, with normal capital and no-capital edge cases. | Save before threshold crossing and after spawn. | Great General should spawn locally or relocate if invalid; audited source must validate whether `getOriginCity()` can be null before `pOriginCity->addProductionExperience`. | Likely no-capital origin-city null defect at `CvPlayer.cpp:44476` through `CvPlayer.cpp:44480`. |
| `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=0` naval local spawn | Cross a naval Great Admiral threshold on a water combat plot, with normal capital and no-capital edge cases. | Save before threshold crossing and after spawn. | Great Admiral should spawn at source plot; audited source must validate whether `getOriginCity()` can be null before `pOriginCity->addProductionExperience`. | Likely no-capital origin-city null defect at `CvPlayer.cpp:44528` through `CvPlayer.cpp:44532`. |
| `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1` land normal nearest city | Cross a land Great General threshold near an owned city with valid city-distance cache. | Save before threshold crossing and after spawn. | Great General should spawn at nearest owned city and not crash. | Intended nearest-city behavior at `CvPlayer.cpp:32224`. |
| `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1` land null closest city | Force or reproduce a source plot where `GetClosestCityByPathLength(pFromUnit->plot())` returns null, then cross a land Great General threshold. | Save before threshold crossing; reload to reproduce crash or patched fallback. | Audited source can crash at `pNearestCity->plot()`; patched source should fall back safely. | Confirmed null dereference at `CvPlayer.cpp:32226` through `CvPlayer.cpp:32229`. |
| `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1` naval coastal city | Cross a naval Great Admiral threshold near an owned coastal city connected to the source water/landmass. | Save before threshold crossing and after spawn. | Great Admiral should spawn at closest suitable coastal city. | Naval nearest-city authority at `CvPlayer.cpp:32347` through `CvPlayer.cpp:32352`. |
| `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=1` naval no coastal city | Cross a naval Great Admiral threshold when no owned coastal city satisfies `GetClosestFriendlyCoastalCity`. | Save before threshold crossing and after spawn. | Audited source falls back to source plot without city lookup crash; origin-city null behavior still needs validation. | Null-safe naval fallback at `CvPlayer.cpp:32354` through `CvPlayer.cpp:32357`, plus origin-city risk at `CvPlayer.cpp:44532`. |
| `BALANCE_ERA_RESTRICTED_GENERALS=1` without `CustomModDbUpdates` | Set only `CustomModOptions.Value=1` for the option, then inspect cached option state. | Save after database load; reload and inspect option state if possible. | DLL should keep the option disabled because `DbUpdates=1` lacks a marker. | Activation dependency at `CustomMods.cpp:230` through `CustomMods.cpp:244`. |
| `BALANCE_ERA_RESTRICTED_GENERALS=1` with marker but missing negate promotions | Force the option active without defining `PROMOTION_NEGATE_*` rows; run AI great-person directive update in debug and release. | Save before `CvPlayerAI::AI_updateGreatPeople`; reload and run update. | Debug build may hit invalid promotion preconditions; release should be inert and not suppress directives. | Missing promotion validation at `CvPlayerAI.cpp:1726`, `CvPlayerAI.cpp:1875`, and `CvPromotionClasses.cpp:3728`. |
| `BALANCE_ERA_RESTRICTED_GENERALS=1` with valid negate promotions | Give AI Great General and Great Admiral units each valid negate promotion and run directive update. | Save before update; reload and verify directive state after update. | Promoted obsolete units should receive `NO_GREAT_PEOPLE_DIRECTIVE_TYPE` unless a later explicitly allowed branch applies. | Directive suppression at `CvPlayerAI.cpp:1731` through `CvPlayerAI.cpp:1798` and `CvPlayerAI.cpp:1880` through `CvPlayerAI.cpp:1956`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
