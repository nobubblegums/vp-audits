# Flagship grants and naval promotion interactions

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

When is the flagship promotion granted and how does movement consume it?

Automatic grant data and movement-time promotion effects have different reachability. The recovered audit did not find the required base promotion row; an external modmod can change that data boundary.

Canonical related owners: [Generals, admirals and optional local/era behavior](<commanders.md>), [Optional feature activation and dependency map](<../../architecture/optional-features.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

The automatic promotion grant path requires `pFromUnit`, which is provided by `GLOBAL_LOCAL_GENERALS` through `CvUnit::changeExperienceTimes100` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753` and `CvUnit.cpp:21767`.

Unsourced Great Admiral point paths call `changeNavalCombatExperienceTimes100` or `setNavalCombatExperienceTimes100` without `pFromUnit`, including `CvGameCoreDLL_Expansion2/CvPlayer.cpp:18946`, `CvPlayer.cpp:19526`, `CvPlayer.cpp:24302`, `CvPlayer.cpp:36852`, `CvPlayer.cpp:36858`, and Lua wrappers at `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:10423` and `CvLuaPlayer.cpp:10433`. These paths can create Admirals but cannot grant flagship.

Enable local generals and flagship, cross the naval Great Admiral threshold with a naval combat unit, and inspect the triggering unit. Repeat once with audited repository data and once with an external valid `PROMOTION_FLAGSHIP` row that sets `GreatGeneralReceivesMovement=1`.

## Application and state

The intended downstream state is the `PROMOTION_FLAGSHIP` promotion on the triggering naval unit. Movement restoration consumes movement-donor state through `CvUnit::GetGreatGeneralStackMovement` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:23985` through `CvUnit.cpp:24010`.

`CvUnit::GetGreatGeneralStackMovement` does not look specifically for `PROMOTION_FLAGSHIP`; it scans same-plot, same-domain units for `IsGreatGeneralReceivesMovement()` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:24005`.

The option adds no serialized field. It relies on ordinary unit promotion state and cached promotion-derived counters.

Unit promotion bitfields are serialized through `CvGameCoreDLL_Expansion2/CvUnit.cpp:28785` and `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:3690` through `CvPromotionClasses.cpp:3693`.

The movement-transfer counter `m_iGreatGeneralReceivesMovementCount` is serialized at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28656` and included in the sync archive in `CvGameCoreDLL_Expansion2/CvUnit.h:2987`.

When a promotion is granted or removed, `CvUnit::setPromotionActive` updates `m_iGreatGeneralReceivesMovementCount` from `CvPromotionEntry::IsGreatGeneralReceivesMovement()` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28037`, and `CvUnit::setHasPromotion` invalidates area-effect state through `GET_PLAYER(getOwner()).UpdateAreaEffectUnit(this)` at `CvUnit.cpp:27752`.

SAVEGAME_SENSITIVE: Adding, removing, or reordering `UnitPromotions` rows is savegame-sensitive because unit promotion bitfields are serialized by promotion index. A future bundled `PROMOTION_FLAGSHIP` row should be append-only and must preserve stable promotion ordering expectations.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:725` with default `Value="0"`. Macro is `MOD_PROMOTIONS_FLAGSHIP` in `CvGameCoreDLL_Expansion2/CustomMods.h:634`; declaration is `CustomMods.h:1712`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:489`.

The option gates two separable behaviors: promotion grant during sourced local Great Admiral generation, and Great Admiral movement restoration through the existing Great-General movement-transfer helper.

`PROMOTIONS_FLAGSHIP` is a runtime `CustomModOptions` switch defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:725`, cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:489`, and wrapped by `MOD_PROMOTIONS_FLAGSHIP` in `CvGameCoreDLL_Expansion2/CustomMods.h:634`.

The option has two active source effects:

- Great Admirals use Great-General stack movement transfer in `CvGameCoreDLL_Expansion2/CvUnit.cpp:21499`.

- A naval unit that triggered local Great Admiral creation is granted `PROMOTION_FLAGSHIP` in `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32342` or `CvPlayer.cpp:32361`.

The automatic grant path practically depends on `GLOBAL_LOCAL_GENERALS`, because `pFromUnit` is only passed from `CvUnit::changeExperienceTimes100` into naval combat XP counters when `MOD_GLOBAL_LOCAL_GENERALS` is true at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753` and `CvUnit.cpp:21767`.

`PROMOTION_FLAGSHIP` is declared as a global define with default `-1` in `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql:36`, mapped to `UnitPromotions` lookup at `CoreDefineChanges.sql:1512`, initialized in `CvGameCoreDLL_Expansion2/CvGlobals.cpp:2356`, and cached at `CvGlobals.cpp:7601`.

Audited repository search found no `UnitPromotions` row defining `PROMOTION_FLAGSHIP`; the only data/code references are the global define, option row, and two grant sites. Therefore the grant branch is inert in audited repository data unless an external modmod supplies that promotion row.

For a supplied `PROMOTION_FLAGSHIP` row to produce the documented movement-transfer behavior, it must set `GreatGeneralReceivesMovement = 1`, the promotion field cached in `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:637` and applied to units in `CvGameCoreDLL_Expansion2/CvUnit.cpp:28037`.

| `PROMOTIONS_FLAGSHIP` | `GLOBAL_LOCAL_GENERALS` | `LOCAL_GENERALS_NEAREST_CITY` | `PROMOTION_FLAGSHIP` row | Expected DLL behavior | Audit classification |
| --- | --- | --- | --- | --- | --- |
| 0 | any | any | absent or present | No flagship grant; Great Admirals use ordinary `maxMoves()` on restore. | working as intended |
| 1 | 0 | any | absent | No automatic grant because no `pFromUnit`; movement-transfer branch is enabled but only matters for other existing same-domain `GreatGeneralReceivesMovement` donors. | compatibility risk / dependency unsafe |
| 1 | 0 | any | valid | Same as above: valid row exists, but automatic grant still cannot fire without `pFromUnit`. Existing externally granted flagship promotions can donate movement. | compatibility risk |
| 1 | 1 | 0 | absent | Local Admiral spawn occurs on triggering naval unit, but flagship grant is inert because `GD_INT_GET(PROMOTION_FLAGSHIP) == -1`. | missing validation |
| 1 | 1 | 0 | valid with `GreatGeneralReceivesMovement=1` | Admiral spawns on the triggering naval unit; triggering unit receives flagship promotion; later same-plot Great Admirals restore to donor movement. | working as intended, runtime unvalidated |
| 1 | 1 | 1 | absent | Admiral spawns at closest friendly coastal city or source plot fallback; flagship grant is inert without data row. | missing validation |
| 1 | 1 | 1 | valid with `GreatGeneralReceivesMovement=1` | Admiral spawns near city when found, but the triggering naval unit at the combat plot receives flagship promotion; movement effect requires later stacking with a Great Admiral. | working as intended, runtime unvalidated |
| 1 | 1 | any | valid but lacks `GreatGeneralReceivesMovement=1` | Promotion can be granted and serialized, but Great Admirals will not receive extra movement from it. | likely defect in external data |
| toggled across save | any | any | present | Promotion state persists, but future Great Admiral movement restore follows the current option state. | savegame-sensitive compatibility risk |

No invalid DLL combination was found. The unsupported operational case is enabling `PROMOTIONS_FLAGSHIP` in the audited repository without also supplying the external `PROMOTION_FLAGSHIP` data expected by the define mapping.

## Observability and validation obligations

Log global define resolution, source unit, source unit delayed-death state, promotion grant result, generated admiral plot, donor movement, and `restoreFullMoves` movement calculation.

1. Audited repository data, `PROMOTIONS_FLAGSHIP=1`, `GLOBAL_LOCAL_GENERALS=1`, `LOCAL_GENERALS_NEAREST_CITY=0`: cross the naval Great Admiral threshold with a surviving naval combat unit. Expected current result: Admiral spawns at source plot, `PROMOTION_FLAGSHIP` is unresolved, and no promotion is granted.

2. External valid `PROMOTION_FLAGSHIP` row with `GreatGeneralReceivesMovement=1`, same options: cross the threshold. Expected: source naval unit has the promotion; save/reload preserves it and `m_iGreatGeneralReceivesMovementCount`.

3. Same as test 2 with `LOCAL_GENERALS_NEAREST_CITY=1`: expected Admiral spawns at closest friendly coastal city when available, while source naval unit still receives the promotion at the combat plot.

4. `PROMOTIONS_FLAGSHIP=0`, valid `PROMOTION_FLAGSHIP` row and promoted naval unit stacked with Great Admiral: call turn reset. Expected: Great Admiral restores to ordinary `maxMoves()`, not donor movement.

5. `PROMOTIONS_FLAGSHIP=1`, valid donor promotion, Great Admiral stacked with same-domain donor: call turn reset or another `restoreFullMoves` path. Expected: Great Admiral moves become `max(admiral maxMoves, donor maxMoves)`.

6. `PROMOTIONS_FLAGSHIP=1`, donor on adjacent plot but not stacked: expected no movement transfer, because `GetGreatGeneralStackMovement` scans only the supplied/current plot.

7. `PROMOTIONS_FLAGSHIP=1`, Great Admiral stacked with land donor or Great General stacked with sea donor: expected no cross-domain movement transfer.

8. Unsourced Great Admiral point threshold from city yields, historic-event yields, trade/minor yields, or Lua `ChangeNavalCombatExperience`: expected Admiral can spawn, but no flagship promotion grant occurs because `pFromUnit == NULL`.

9. Threshold-crossing combat where the source naval unit dies: verify whether `pFromUnit` is still valid long enough for the grant and whether delayed-death cleanup removes or preserves the promoted unit as expected.

Add diagnostic-only logging in `CvPlayer::setNavalCombatExperienceTimes100`, `CvUnit::changeExperienceTimes100`, and `CvUnit::restoreFullMoves`.

Log player, unit ID/type/domain, `MOD_PROMOTIONS_FLAGSHIP`, `MOD_GLOBAL_LOCAL_GENERALS`, `MOD_LOCAL_GENERALS_NEAREST_CITY`, threshold before/after, source unit pointer and delayed-death state, resolved `PROMOTION_FLAGSHIP` ID, whether `setHasPromotion` was attempted, final promotion state, Admiral unit type, spawn plot, donor plot, donor max moves, and restored Admiral moves.

Add a one-time warning or debug assertion when `MOD_PROMOTIONS_FLAGSHIP` is enabled and `GD_INT_GET(PROMOTION_FLAGSHIP) == NO_PROMOTION`.

## Version differences, risks and unresolved questions

Local `git log -S'MOD_PROMOTIONS_FLAGSHIP'` finds production-source provenance at `54dee87` only.

No compile-time risk was found because the promotion is resolved through runtime global defines.

MISSING_VALIDATION: Missing or invalid `PROMOTION_FLAGSHIP` data makes the grant path inert. `CvUnit::setHasPromotion` returns early for `NO_PROMOTION` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:27634`, so this is not a crash in the current grant path, but there is no diagnostic when the option is enabled and the promotion cannot be resolved.

COMPATIBILITY_RISK: The Great Admiral movement branch is broader than the automatic grant path. With `PROMOTIONS_FLAGSHIP=1`, any same-plot same-domain unit that already has a promotion with `GreatGeneralReceivesMovement=1` can donate movement to a Great Admiral, even if it is not `PROMOTION_FLAGSHIP`.

### Missing validation: current repository data leaves automatic flagship grant inert

Classification: missing validation / dependency unsafe.

Exact location: `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32342`, `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32361`, `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql:36`, and `CoreDefineChanges.sql:1512`.

```text
CvUnitCombat XP resolution
  -> CvUnit::changeExperienceTimes100(..., bUpdateGlobal=true)
  -> CvPlayer::changeNavalCombatExperienceTimes100(..., pFromUnit)
  -> CvPlayer::setNavalCombatExperienceTimes100
  -> MOD_PROMOTIONS_FLAGSHIP grant
  -> CvUnit::setHasPromotion((PromotionTypes)GD_INT_GET(PROMOTION_FLAGSHIP), true)
```

Intended behavior: A naval unit that generates a local Great Admiral should become a flagship movement donor so nearby or same-plot Great Admirals can receive its movement during restore.

Observed or statically inferred failure: Audited repository data maps `PROMOTION_FLAGSHIP` but does not define the promotion row. `GD_INT_GET(PROMOTION_FLAGSHIP)` therefore remains `-1`, and `CvUnit::setHasPromotion` returns at `CvGameCoreDLL_Expansion2/CvUnit.cpp:27634` without setting any promotion. The grant path is silent and has no runtime warning.

Why the surrounding rewrite caused or exposed it: INFERRED: Local history is grafted at `54dee87`, so this pass cannot identify the original upstream change. The current architecture exposes the dependency because the DLL-side feature is enabled by one option row while its actual promotion payload is external or absent from the repository.

Smallest safe correction: Add a valid-ID guard around the grant and log a one-time warning when `MOD_PROMOTIONS_FLAGSHIP` is enabled but `PROMOTION_FLAGSHIP` is unresolved. If the feature is intended to be self-contained in VP, add an append-only `UnitPromotions` row for `PROMOTION_FLAGSHIP` with `GreatGeneralReceivesMovement=1` and appropriate display metadata.

Possible regression risks: Adding a new promotion row is savegame- and ordering-sensitive. A pure guard/log patch changes no gameplay but can reveal misconfigured external modmods. Bundling the row may affect existing external mods that already define `PROMOTION_FLAGSHIP`.

Runtime scenario needed: Enable `PROMOTIONS_FLAGSHIP=1` and `GLOBAL_LOCAL_GENERALS=1`, cross the naval Great Admiral threshold with audited repository data, and verify no flagship promotion is present. Repeat with an external `PROMOTION_FLAGSHIP` row that sets `GreatGeneralReceivesMovement=1`; verify the promotion is granted and persists through save/load.

### Compatibility risk: movement branch is broader than the automatic grant

Classification: compatibility risk.

`CvUnit::restoreFullMoves` checks only `MOD_PROMOTIONS_FLAGSHIP && IsGreatAdmiral()` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21499`; `CvUnit::GetGreatGeneralStackMovement` then scans for any same-domain unit with `IsGreatGeneralReceivesMovement()` at `CvUnit.cpp:24005`, not specifically a unit with `PROMOTION_FLAGSHIP`.

Observed/static risk: Enabling `PROMOTIONS_FLAGSHIP` changes Great Admiral movement restoration for any naval unit that has any promotion with `GreatGeneralReceivesMovement=1`, including external or repurposed promotions. This can be desirable, but it means the movement effect is not limited to the automatic `PROMOTION_FLAGSHIP` grant path.

Smallest safe correction: No code correction is required if the generic movement-donor semantics are intended. Documentation and tests should explicitly cover externally granted movement-donor promotions.

Classification: insufficient runtime evidence.

The repository contains Visual Studio solution/project files, including `CvGameCoreDLL.vs2010.sln`, `VoxPopuli_vs2013.sln`, and `CvGameCoreDLL_Expansion2/*.vcxproj`.

No local `msbuild`, `xbuild`, `dotnet`, `cl`, or `devenv` executable was found on `PATH` in this Linux workspace, so no compile was attempted. A successful build would not substitute for runtime validation of promotion data, threshold crossing, save/load, or UI-visible movement.

### Working as intended: flagship movement state uses ordinary promotion serialization

Classification: working as intended.

A valid granted flagship promotion would be stored in ordinary unit promotion bitfields at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28785` and `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:3690` through `CvPromotionClasses.cpp:3693`. The movement-transfer cached counter is serialized at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28656`.

`PROMOTIONS_FLAGSHIP` does not share mutable state directly with `LINKED_MOVEMENT`, `BAL_ATTRITION`, or the Lua event options.

COMPATIBILITY_RISK: The automatic grant path shares practical dependencies with the Great General/Admiral cluster. `GLOBAL_LOCAL_GENERALS=0` or source-unit bypasses make the grant unreachable; `LOCAL_GENERALS_NEAREST_CITY=1` can spawn the Admiral away from the promoted combat unit, so the movement benefit is delayed until the Admiral and flagship stack later.

COMPATIBILITY_RISK: `BAL_ATTRITION` and normal combat damage can kill the triggering naval unit before any later movement-sharing validation, but the grant itself happens after `createGreatAdmiral` inside the XP threshold update. Runtime tests should include threshold-crossing combat where the source unit survives and where it dies.

1. Add a valid-promotion guard and diagnostic warning around both `PROMOTION_FLAGSHIP` grant sites in `CvPlayer::setNavalCombatExperienceTimes100`.

2. Decide whether `PROMOTION_FLAGSHIP` should remain an external modmod dependency or be bundled. If bundled, add an append-only `UnitPromotions` row with `GreatGeneralReceivesMovement=1`, text, icon/display metadata, and any intended eligibility rules.

3. Add diagnostic-only logging around naval XP threshold crossing, source unit, resolved promotion ID, grant result, Admiral spawn plot, and Great Admiral movement restoration.

4. If design wants only `PROMOTION_FLAGSHIP` donors to affect Great Admirals, narrow `GetGreatGeneralStackMovement` or add a naval-specific helper. If generic donor semantics are intended, document them and leave code unchanged.

`PROMOTIONS_FLAGSHIP`: not ready for normal self-contained use from the audited repository data. The C++ hooks are active and authoritative, movement-transfer state would serialize correctly, but the automatic grant is inert unless an external modmod supplies a valid `PROMOTION_FLAGSHIP` row. With that dependency supplied, the source audit shows the path is plausible but still needs runtime validation.

## Source and dependency lookup

Direct references are `CustomMods.h:634`, `CustomMods.h:1712`, `CustomMods.cpp:489`, `NewCustomModOptions.xml:725`, `CvUnit.cpp:21499`, `CvPlayer.cpp:32340`, and `CvPlayer.cpp:32359`.

`CvPlayer::setNavalCombatExperienceTimes100` attempts to grant `PROMOTION_FLAGSHIP` to `pFromUnit` when a local admiral is generated at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32340` through `CvPlayer.cpp:32342` and `CvPlayer.cpp:32359` through `CvPlayer.cpp:32361`.

`CvUnit::restoreFullMoves` treats `IsGreatAdmiral()` units as eligible for `CvUnit::GetGreatGeneralStackMovement` when the option is enabled at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21499`.

`PROMOTION_FLAGSHIP` is cached as a global define in `CvGameCoreDLL_Expansion2/CvGlobals.cpp:2356` and `CvGlobals.cpp:7601`; the define row exists in `(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql:36` and the lookup mapping exists at `CoreDefineChanges.sql:1512`.

Audited repository search found no `UnitPromotions` row defining `PROMOTION_FLAGSHIP`; the only data/code references are the global define, option row, and two grant sites. Therefore `GD_INT_GET(PROMOTION_FLAGSHIP)` remains `-1` unless an external modmod supplies the promotion row.

The field required for movement donation is `GreatGeneralReceivesMovement`, cached in `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:637` and applied to unit state in `CvGameCoreDLL_Expansion2/CvUnit.cpp:28037`.

INFERRED: The feature assumes the define resolves to a valid external or bundled `UnitPromotions` row at runtime, and that the row sets `GreatGeneralReceivesMovement=1`.

Great Admirals or naval units created outside sourced naval XP threshold crossing will not receive `PROMOTION_FLAGSHIP` from this guarded block because the other `changeNavalCombatExperienceTimes100` callers pass no source unit.

Save after promotion grant, reload, and confirm promotion and movement effect persist.

`DEPENDENCY_UNSAFE`

The active sourced grant path is:

```text
CvUnitCombat XP resolution
  -> CvUnit::changeExperienceTimes100(..., bUpdateGlobal=true)
  -> CvUnit.cpp:21753 or CvUnit.cpp:21767 passes this when MOD_GLOBAL_LOCAL_GENERALS
  -> CvPlayer::changeNavalCombatExperienceTimes100
  -> CvPlayer::setNavalCombatExperienceTimes100
  -> CvPlayer.cpp:32342 or CvPlayer.cpp:32361 attempts PROMOTION_FLAGSHIP grant
```

The active movement path is:

```text
CvPlayer::DoUnitReset
  -> CvUnit::restoreFullMoves
  -> MOD_PROMOTIONS_FLAGSHIP && IsGreatAdmiral()
  -> CvUnit::GetGreatGeneralStackMovement
  -> same-domain unit with IsGreatGeneralReceivesMovement()
```

MISSING_VALIDATION: In audited repository data, the automatic grant is silent no-op because `PROMOTION_FLAGSHIP` is mapped but no `UnitPromotions` row exists. The smallest safe code-side correction is a valid-promotion guard plus one-time diagnostic warning. If the feature should be self-contained, the data-side correction is an append-only `PROMOTION_FLAGSHIP` row with `GreatGeneralReceivesMovement=1`, text, icon/display metadata, and any intended eligibility constraints.

WORKING_AS_INTENDED: When a valid movement-donor promotion exists, promotion state and `m_iGreatGeneralReceivesMovementCount` are serialized through ordinary unit promotion state at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28785`, `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp:3690` through `CvPromotionClasses.cpp:3693`, and `CvGameCoreDLL_Expansion2/CvUnit.cpp:28656`.

SAVEGAME_SENSITIVE: Adding a bundled `PROMOTION_FLAGSHIP` row is promotion-index sensitive because unit promotion bitfields are serialized by index.

Runtime validation required: See `docs/modmod-audit/groups/naval-promotions.md` for the full configuration matrix and runtime tests.

Readiness: Not ready for normal self-contained use from audited repository data. With an external valid promotion row, the C++ path is statically plausible but runtime-unvalidated.

Subsystem: Naval promotions

Features audited in this pass: `PROMOTIONS_FLAGSHIP`

```text
Melee/ranged/air-sweep combat XP
  -> CvUnitCombat calls CvUnit::changeExperienceTimes100(..., bUpdateGlobal=true)
  -> CvUnit::changeExperienceTimes100
  -> domain sea branch
  -> CvPlayer::changeNavalCombatExperienceTimes100(..., pFromUnit)
  -> CvPlayer::setNavalCombatExperienceTimes100
  -> createGreatAdmiral
  -> optional PROMOTION_FLAGSHIP grant to pFromUnit
```

Representative combat XP callers are `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:345` through `CvUnitCombat.cpp:361` and `CvUnitCombat.cpp:1047` through `CvUnitCombat.cpp:1052`. The naval source-unit propagation is in `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753` and `CvUnit.cpp:21767`. The threshold and grant logic is in `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32269` through `CvPlayer.cpp:32368`.

```text
Player turn/unit reset or other restore point
  -> CvUnit::restoreFullMoves
  -> if Great General, or if PROMOTIONS_FLAGSHIP and Great Admiral
  -> CvUnit::GetGreatGeneralStackMovement
  -> same-plot same-domain unit with GreatGeneralReceivesMovement
  -> set admiral moves to max(current admiral moves, flagship unit max moves)
```

The regular turn reset calls `CvUnit::restoreFullMoves` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:10716`. `restoreFullMoves` applies the flagship Great Admiral branch at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21499` through `CvUnit.cpp:21501`. The movement donor scan is `CvUnit::GetGreatGeneralStackMovement` at `CvUnit.cpp:23985` through `CvUnit.cpp:24010`.

`CvPlayer::setNavalCombatExperienceTimes100` is still the authoritative threshold-crossing function for Great Admiral creation from naval XP counters. It handles both sourced (`pFromUnit`) and unsourced (`pBestCity`) Great Admiral creation paths in `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32269` through `CvPlayer.cpp:32368`.

Newer yield-based or script-based Great Admiral point paths call `changeNavalCombatExperienceTimes100` or `setNavalCombatExperienceTimes100` without `pFromUnit`, including `CvGameCoreDLL_Expansion2/CvPlayer.cpp:18946`, `CvPlayer.cpp:19526`, `CvPlayer.cpp:24302`, `CvPlayer.cpp:36852`, `CvPlayer.cpp:36858`, and Lua wrappers at `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:10423` and `CvLuaPlayer.cpp:10433`. These paths can create Great Admirals but cannot grant `PROMOTION_FLAGSHIP`.

`CvUnit::restoreFullMoves` remains an active movement reset authority. Besides player turn reset, it is used after capture, purchase/creation, conversion, worldbuilder load, and minor-civ reward paths, including `CvGameCoreDLL_Expansion2/CvUnit.cpp:3060`, `CvCity.cpp:29801`, `CvUnit.cpp:24339`, `CvUnit.cpp:24359`, `CvWorldBuilderMapLoader.cpp:740`, and `CvMinorCivAI.cpp:1063`.

`CvUnit::setHasPromotion` guards `NO_PROMOTION` and out-of-range promotion IDs before reading promotion data at `CvGameCoreDLL_Expansion2/CvUnit.cpp:27634`, so a missing `PROMOTION_FLAGSHIP` row does not crash at the grant site. It returns without mutating unit state.

With `PROMOTIONS_FLAGSHIP=0`, `CvUnit::restoreFullMoves` treats Great Admirals like ordinary non-General units for movement restoration and calls `maxMoves()` instead of `GetGreatGeneralStackMovement()` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:21499` through `CvUnit.cpp:21504`. The guarded promotion grant sites do not execute.

With `PROMOTIONS_FLAGSHIP=1` and no valid `PROMOTION_FLAGSHIP` data row, Great Admirals can still use movement transfer from any same-plot same-domain unit that already has a `GreatGeneralReceivesMovement` promotion, but the automatic grant sites do not add any promotion because `GD_INT_GET(PROMOTION_FLAGSHIP)` remains `-1` and `CvUnit::setHasPromotion` returns.

With `PROMOTIONS_FLAGSHIP=1`, `GLOBAL_LOCAL_GENERALS=1`, and a valid `PROMOTION_FLAGSHIP` row with `GreatGeneralReceivesMovement=1`, a naval combat unit that crosses the Great Admiral XP threshold grants itself movement-donor state for future Great Admiral `restoreFullMoves` calls.

`git log -- CvGameCoreDLL_Expansion2/CvUnit.cpp CvGameCoreDLL_Expansion2/CvPlayer.cpp CvGameCoreDLL_Expansion2/CustomMods.h CvGameCoreDLL_Expansion2/CustomMods.cpp "(1) Community Patch/Database Changes/NewCustomModOptions.xml" "(1) Community Patch/Database Changes/Defines/CoreDefineChanges.sql"` shows production-source provenance at `54dee87` (`5.2.6 Release`) only.

`git log -S'MOD_PROMOTIONS_FLAGSHIP'` and targeted `git log -G'PROMOTION_FLAGSHIP|PROMOTIONS_FLAGSHIP|setNavalCombatExperienceTimes100|GetGreatGeneralStackMovement|restoreFullMoves|GreatGeneralReceivesMovement'` over relevant source/data files also find `54dee87` only.

INFERRED: The local history is grafted/compressed, so this pass cannot identify the original upstream rewrite that introduced or exposed the missing promotion-data dependency.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `PROMOTIONS_FLAGSHIP` | With naval local admiral spawn, verify triggering naval unit receives flagship promotion and movement sharing works. | Save after promotion grant; reload and confirm promotion and movement effect remain. | `CvPlayer::setNavalCombatExperienceTimes100`, `CvUnit::restoreFullMoves`, `CvUnit::GetGreatGeneralStackMovement`. | `CvPlayer.cpp:32340`, `CvPlayer.cpp:32359`, `CvUnit.cpp:21499` |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
