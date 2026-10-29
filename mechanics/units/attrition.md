# Attrition and Heavy Charge displacement

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How do optional attrition damage and Heavy Charge retreat interact with movement and danger?

The later recovered implementation report uses BAL_ATTRITION, a shared projected-damage helper, configurable Heavy Charge controls and narrow city/fort/citadel protection. Earlier generic names and data-absence claims describe a different profile/revision.

## Revision reconciliation

The later report renames generic ATTRITION/MOD_ATTRITION to BAL_ATTRITION/MOD_BAL_ATTRITION and aligns the Dynamic Battle Lines activation row. It describes a side-effect-free projected-damage helper shared by actual damage, danger and path-end evaluation, plus configurable Heavy Charge damage/retreat controls. Those are reported recovered changes, not edits made during this migration.

Earlier base-data absence of an immunity promotion is reconciled with the supplied modmod defining and granting it. City/fort/citadel protection is intentionally narrow; unique replacement improvements and nonlethal AI displacement still require validation. Older line numbers/default names are revision-qualified below.

Canonical related owners: [Linked movement, quick groups and Squads](<movement-groups.md>), [Optional feature activation and dependency map](<../../architecture/optional-features.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Reported revision changes

### Recovered finding: Confirmed changes

`CvUnit::GetProjectedAttritionDamage(const CvPlot*, bool)` centralizes attrition eligibility and damage. It has no side effects and stores no per-unit state.

`CvUnit::DoAttrition` now uses the shared helper for actual turn-start damage while preserving the existing last-move-turn rule.

`CvDangerPlots::GetDanger` includes projected attrition for per-unit plot danger. `PathEndTurnCost` ensures path end-turn danger is at least projected attrition damage without adding a duplicate independent penalty.

Configurable defines are cached through the standard `CvGlobals` pattern:

- `BAL_ATTRITION_DAMAGE`, fallback/default `5`

- `BAL_HEAVY_CHARGE_NO_RETREAT_MODIFIER`, fallback/default `25`

- `BAL_HEAVY_CHARGE_RETREAT_MAX_HP`, fallback/default `100`

Negative attrition damage and no-retreat modifier values are clamped to zero at use sites. Retreat threshold is clamped to the defender's valid HP range.

Dynamic Battle Lines Heavy Charge forced retreat now requires the defender's post-combat remaining HP to be below `BAL_HEAVY_CHARGE_RETREAT_MAX_HP`. The default `100` preserves prior behavior for ordinary surviving damaged defenders; `50` enables the planned lighter variant; `0` disables forced retreat.

Cities, forts, and citadels are protected from Dynamic Battle Lines forced retreat and from the Dynamic Battle Lines no-retreat modifier. This protection is intentionally not generalized to every `NoFollowUp` improvement.

Tactical AI now treats projected successful nonlethal Heavy Charge retreat as displacement, with the attacker evaluated as advancing into the vacated defender plot. The existing flat modifier remains accounted for only through normal combat strength calculation.

STALE_OR_MISMATCH: The earlier report said `PROMOTION_ATTRITION_IMMUNITY` was absent. That remains true for base VP data alone, but the supplied Dynamic Battle Lines `Attrition.sql` defines the promotion and grants it to recon and submarine units.

STALE_OR_MISMATCH: The earlier report used old line references and old names `ATTRITION` / `MOD_ATTRITION`; audited source uses `BAL_ATTRITION` / `MOD_BAL_ATTRITION`.

CONFIRMED_AND_FIXED: AI danger/pathing did not model optional flat attrition. The shared projected-damage helper is now consumed by danger and path end-turn evaluation.

CONFIRMED_AND_FIXED: Dynamic Battle Lines referenced stale `CIVILIANS_RETREAT_WITH_MILITARY`. It now references `CORE_CIVILIANS_RETREAT_WITH_MILITARY`.

## Data and configuration

The optional Dynamic Battle Lines attrition switch was renamed from `ATTRITION` / `MOD_ATTRITION` to `BAL_ATTRITION` / `MOD_BAL_ATTRITION`.

The option row is `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1170` with default `Value="0"`. The macro is `CvGameCoreDLL_Expansion2/CustomMods.h:1046`; the declaration is `CustomMods.h:1819`; the cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:584`.

A audited source and supplied-resource search found no maintained consumer of the old generic `ATTRITION` option outside the renamed DLL branches and Dynamic Battle Lines activation SQL. Remaining `ATTRITION` strings in tactical posture enum/logging names are unrelated AI terminology, not the optional feature switch.

## Observability and validation obligations

Runtime tests still needed:

1. Dynamic Battle Lines activates `BAL_ATTRITION` and no old `MOD_ATTRITION` checks remain.

2. `BAL_ATTRITION_DAMAGE` values `5`, custom positive, `0`, and negative database values.

3. Immune and nonimmune units, civilians, barbarians, trade units, land units, and naval units.

4. Recently moved versus stationary hostile-territory units, including save/load before owner-turn activation.

5. AI plot choice where attrition is the only difference, wounded-unit avoidance, and no path penalty for every intermediate hostile plot.

6. Heavy Charge no-retreat modifier values `25`, custom positive, `0`, and protected city/fort/citadel cases.

7. Retreat thresholds `100`, `50`, `0`, exact-threshold equality, legal and illegal fallback plots, civilian followers, and save/load.

8. Tactical AI prediction matching actual nonlethal Heavy Charge displacement.

1. Feature rename: `BAL_ATTRITION` activates from Dynamic Battle Lines; `MOD_BAL_ATTRITION` is active; no maintained `MOD_ATTRITION` checks remain; disabled behavior remains standard VP behavior.

2. Attrition damage: defaults to `5`; honors custom positive, zero, and negative-clamped values; respects immunity and excluded unit classes; applies to land and naval nonimmune units in hostile territory; respects recent movement and save/load.

3. AI attrition: AI prefers otherwise equivalent safe plots, avoids lethal attrition when wounded, still enters hostile territory when tactically valuable, and does not count every intermediate hostile plot as separate attrition.

4. Heavy Charge modifier: defaults to `25`; honors custom positive and zero values; does not apply in city/fort/citadel protected cases; is not duplicated in AI simulation.

5. Retreat threshold and displacement: thresholds `100`, `50`, `0`; exact threshold equality does not retreat; legal and illegal fallback plots; civilian followers; attacker advance; defender remains when conditions fail; AI prediction matches actual combat.

## Version differences, risks and unresolved questions

RUNTIME_UNTESTED: No in-game DLL runtime scenario has been executed in this Linux workspace.

RUNTIME_UNTESTED: Tactical AI uses the existing `A_MELEEKILL` advancing assignment to model nonlethal Heavy Charge displacement. This is intentionally narrow, but runtime logs should verify that simulated displacement matches actual combat resolution under threshold, protected-plot, and fallback-failure cases.

RUNTIME_UNTESTED: If a modmod supplies unique fort/citadel replacement improvements that are not typed as `IMPROVEMENT_FORT` or `IMPROVEMENT_CITADEL`, they are not protected by the new narrow check unless separately added.

## Source and dependency lookup

`codex/modmod_resources/Dynamic Battle Lines (v 1)/Attrition.sql:1` now activates `BAL_ATTRITION`.

`Attrition.sql` defines the configurable values:

- `BAL_ATTRITION_DAMAGE = 5`

- `BAL_HEAVY_CHARGE_NO_RETREAT_MODIFIER = 25`

- `BAL_HEAVY_CHARGE_RETREAT_MAX_HP = 100`

`Attrition.sql` supplies `PROMOTION_ATTRITION_IMMUNITY` and grants it to recon and submarine combat classes. The earlier base-repo finding that the immunity promotion was absent is outdated for the supplied Dynamic Battle Lines package, but the DLL now still guards a missing promotion ID safely.

`codex/modmod_resources/Dynamic Battle Lines (v 1)/Main.sql:2` now enables `CORE_CIVILIANS_RETREAT_WITH_MILITARY` instead of the stale `CIVILIANS_RETREAT_WITH_MILITARY` name.

```text
CvPlayer::setTurnActive(true)
  -> CvPlayer::DoUnitAttrition
  -> CvUnit::DoAttrition
  -> CvUnit::GetProjectedAttritionDamage
  -> CvUnit::changeDamage / setDamage
```

`CvUnit::GetProjectedAttritionDamage` is side-effect-free and is shared by runtime attrition, danger evaluation, and path end-turn evaluation. It returns zero when `MOD_BAL_ATTRITION` is disabled, for excluded units, outside enemy territory, when the actual current unit did not move recently, when the immunity promotion is present, or when `BAL_ATTRITION_DAMAGE <= 0`.

Runtime attrition preserves the old `GC.getGame().getGameTurn() - getLastMoveTurn() < 2` semantics. AI hypothetical destinations pass `bAssumeMovedThisTurn = true`, so projected movement into hostile territory is assessed without reusing stale unit movement timestamps.

```text
CvUnit::GetDanger / CvPlayer::GetPlotDanger
  -> CvDangerPlots::GetDanger
  -> CvUnit::GetProjectedAttritionDamage

PathEndTurnCost
  -> CvUnit::GetDanger
  -> max(existing danger, projected attrition damage)
```

Pathing does not add a second independent attrition penalty; it uses the helper to ensure end-turn danger is at least the projected attrition damage.

Dynamic Battle Lines keeps the flat no-retreat attack-strength modifier. With `MOD_BAL_ATTRITION` enabled, `CvUnit::GetMaxAttackStrength` adds `max(0, BAL_HEAVY_CHARGE_NO_RETREAT_MODIFIER)` only when the defender has zero fallback plots and is not on a protected plot. With `MOD_BAL_ATTRITION` disabled, the existing VP `+50` behavior remains.

Protected plots for the Dynamic Battle Lines modifier and forced-retreat path are cities, `IMPROVEMENT_FORT`, and `IMPROVEMENT_CITADEL`. This intentionally avoids broadening the exemption to every `NoFollowUp` improvement.

`CvUnit::CanHeavyChargeForceRetreat` gates Dynamic Battle Lines forced retreat by:

- `MOD_BAL_ATTRITION`

- attacker has Heavy Charge

- defender is not delayed death

- defender is not on a protected plot

- `BAL_HEAVY_CHARGE_RETREAT_MAX_HP` clamped to `[0, defender max HP]`

- post-combat remaining HP is below the threshold

- at least one legal fallback plot exists

Default threshold `100` preserves retreat for ordinary surviving damaged defenders. Threshold `50` restricts retreat to defenders below 50 HP. Threshold `0` disables Dynamic Battle Lines forced retreat.

Civilian followers remain handled by `CvUnit::DoFallBack`; they move only when the military unit actually falls back.

Tactical AI now predicts successful nonlethal Heavy Charge displacement by using `CvUnit::CanHeavyChargeForceRetreat` after simulated combat damage. When the defender is projected to retreat, the assignment is treated like an advancing melee result so the attacker is evaluated on the defender's vacated plot.

The flat no-retreat modifier is still part of ordinary attack-strength calculation, so tactical simulated damage receives it through `GetMaxAttackStrength`; no separate AI damage bonus was added.

Subsystem: Dynamic Battle Lines attrition and Heavy Charge movement.

Feature audited and implemented in this pass: `BAL_ATTRITION`.

The old generic optional switch `ATTRITION` / `MOD_ATTRITION` was renamed to `BAL_ATTRITION` / `MOD_BAL_ATTRITION` across maintained DLL option rows, macros, cache declarations, C++ feature checks, and Dynamic Battle Lines SQL activation.

The active option path is:

```text
NewCustomModOptions.xml BAL_ATTRITION
  -> CustomMods::preloadCache MOD_OPT_CACHE(BAL_ATTRITION)
  -> gCustomMods.isBAL_ATTRITION()
  -> MOD_BAL_ATTRITION
```

Dynamic Battle Lines now explicitly enables the current VP civilian-retreat option `CORE_CIVILIANS_RETREAT_WITH_MILITARY`; the previous resource row used a stale option name.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `BAL_ATTRITION` | Move non-civilian non-barbarian unit into enemy territory without immunity; test Heavy Charge combat. | Save before turn attrition and before Heavy Charge combat; reload and compare damage/modifier/retreat threshold. | `CvPlayer::DoUnitAttrition`, `CvUnit::DoAttrition`, `CvUnit::GetProjectedAttritionDamage`, `CvUnitCombat` heavy-charge checks. | `CvPlayer.cpp:10788`, `CvUnit.cpp:8182`, `CvUnit.cpp:8303`, `CvUnitCombat.cpp:524` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `BAL_ATTRITION=0` flat damage control | Recently moved non-civilian non-barbarian combat unit starts owner turn in enemy territory with no other damage sources. | Save before player turn activation; reload and process turn. | No optional `5` damage should apply; always-active damage sources remain independently testable. | Disabled behavior parity for `CvUnit::DoAttrition`. |
| `BAL_ATTRITION=1`, no immunity row | Same unit starts owner turn in enemy territory in a database that does not define `PROMOTION_ATTRITION_IMMUNITY`. | Save before activation; reload and process turn in release and debug builds. | Release/debug should apply configured damage without calling `isHasPromotion(NO_PROMOTION)`. | Guarded helper at `CvUnit.cpp:8303`. |
| `BAL_ATTRITION=1`, immunity row present | External test data defines `PROMOTION_ATTRITION_IMMUNITY`; eligible unit with that promotion starts in enemy territory. | Save after promotion grant and before activation; reload and process turn. | Optional flat `5` damage should be skipped while other attrition-like damage sources still apply. | Regression check for immunity guard/caching patch. |
| `BAL_ATTRITION=1` eligibility controls | Test civilian, barbarian, trade/cargo, stationary combat unit, neutral territory, friendly territory, and enemy territory. | Save before activation for each control case. | Optional flat branch applies only to recently moved non-civilian non-barbarian combat units in enemy territory. | Current eligibility semantics in `CvUnit.cpp:8199`. |
| `BAL_ATTRITION=1` stacked damage | Unit starts on enemy terrain/feature damage and has promotion `EnemyDamage`/`NeutralDamage` effects. | Save before activation; reload and compare final damage/death. | Damage order should be optional flat attrition, promotion enemy/neutral damage, terrain/feature damage, then downstream death/notification handling. | Compatibility risk with always-active attrition-like systems. |
| `BAL_ATTRITION=1` Heavy Charge protected plots | Heavy Charge attacker strikes defender in city, fort, and citadel with fallback plots available. | Save before combat; reload and repeat. | Defender should not fallback and should not grant the Dynamic Battle Lines no-retreat modifier. | Helper branch in `CvUnit.cpp:25157` and combat branch in `CvUnitCombat.cpp:524`. |
| Heavy Charge no-fallback comparison | Heavy Charge attacker strikes defender with zero fallback plots with `BAL_ATTRITION=0` and `BAL_ATTRITION=1`. | Save before combat in each config; reload and repeat. | Attack modifier should be `+50` when disabled and configured `BAL_HEAVY_CHARGE_NO_RETREAT_MODIFIER` when enabled. | Guarded modifier in `CvUnit.cpp:16834`. |
| `BAL_ATTRITION=1` AI pathing/danger | AI units evaluate movement into enemy territory where the only extra risk is optional flat attrition. | Save before AI tactical turn; reload with and without projected-damage integration. | Projected risk should be visible to danger/pathing without changing damage authority or counting intermediate hostile plots repeatedly. | `CvDangerPlots.cpp:810`, `CvDangerPlots.cpp:924`, and `CvAStar.cpp:1292`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
