# Settler-carried religion

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How is a settler religion selected, saved, transferred and applied at founding?

The base audit found a zero-total-weight religious-selection defect. A recovered 2026-06-29 repair reports requiring positive total follower weight before selection; that report, not an assumed current tree, is the basis for the repaired reference.

## Revision reconciliation

The base audit reports that a nonempty religion vector with zero total follower weight can use the weighted-vector random fallback and assign a religion with no followers. The **2026-06-29 repair report** adds `vReligions.GetTotalWeight() > 0` before selection. Its claim preserves the positive-weight seed path and majority-only/disabled modes; it is not a current-source verification performed here.

The recovered resource enables GLOBAL_RELIGIOUS_SETTLERS while leaving dependent weighted-mode defaults in force. Assignment during unit initialization, serialized carried religion, conversion/capture/gift and city founding application remain separate stages. The recorded save/load and mixed-follower scenarios were still required.

Canonical related owners: [Religion founding and belief application](<founding-and-beliefs.md>), [Shared weighted selection and seeded draws](<../../ai/infrastructure/weighted-selection.md>), [Optional feature activation and dependency map](<../../architecture/optional-features.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Reported revision changes

Revalidated against audited source and the supplied modmod resources. `codex/modmod_resources/Merged Gameplay Mods/ReligiousSettlers/XML/GlobalReligiousSettlers.xml` enables `GLOBAL_RELIGIOUS_SETTLERS` and does not override `RELIGIOUS_SETTLERS_WEIGHTED`, so the current database default leaves weighted mode active when this resource is loaded.

The audited source still uses the same authoritative path: `CvPlayer::initUnit` -> `CvUnit::init` -> `CvUnit::initWithNameOffset` for assignment, and `CvUnit::foundCity` -> `CvPlayer::foundCity` -> `CvCity::init` for application. The audit finding survived revalidation: before this pass, `CvGameCoreDLL_Expansion2/CvUnit.cpp:1047` checked only that the weighted vector was non-empty, while `CvGameCoreDLLUtil/include/CvWeightedVector.h:217` through `CvWeightedVector.h:220` randomly selected an entry when total weight was zero.

REPORTED_REPAIR: `CvGameCoreDLL_Expansion2/CvUnit.cpp:1047` now requires `vReligions.GetTotalWeight() > 0` before sorting and calling `ChooseByWeight`. This preserves standard VP behavior while `GLOBAL_RELIGIOUS_SETTLERS` is disabled, preserves majority-only behavior while `RELIGIOUS_SETTLERS_WEIGHTED` is disabled, and preserves existing weighted seeded outcomes whenever at least one candidate has positive follower weight.

RECOVERED_CONCLUSION: `GLOBAL_RELIGIOUS_SETTLERS` already had a complete activation resource and an authoritative DLL/unit-founding path, but the older audit identified a real surviving weighted-mode defect. Current VP did not supersede this implementation. Resource-side integration was not the missing piece for the defect.

REMAINING_RUNTIME_TESTS: Enable `GLOBAL_RELIGIOUS_SETTLERS=1` with default `RELIGIOUS_SETTLERS_WEIGHTED=1`, create a settler in a city with zero pantheon/religion followers, save/reload before founding, and confirm the settler carries no random real religion. Also test a mixed-follower city to confirm only positive-weight religions can be selected, then save/reload and found to confirm `CvUnitReligion` serialization and `CvCity::init` application.

## Trigger and ownership

Normal unit creation through `CvPlayer::initUnit` reaches `CvUnit::initWithNameOffset`. City creation uses `CvUnit::foundCity`, which calls `CvPlayer::foundCity`.

Create settlers in cities with majority and mixed follower religions, then found cities and inspect founded city religion.

## Application and state

The downstream state is `m_Religion` on the settler unit and the religion parameter passed to `CvPlayer::foundCity`, then into city initialization.

```text
GLOBAL_RELIGIOUS_SETTLERS
CvPlayer::initUnit
  -> CvUnit::init
  -> CvUnit::initWithNameOffset
  -> settler religion assignment
  -> CvUnit::foundCity
  -> CvPlayer::foundCity
  -> CvPlayer::initCity
  -> CvCity::init
  -> CvCityReligions::AdoptReligionFully
```

`CvPlayer::initUnit` calls `CvUnit::init` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:9132`; `CvUnit::init` forwards to `CvUnit::initWithNameOffset` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:549`. The guarded settler branch is in `CvUnit.cpp:1031` through `CvUnit.cpp:1058`.

`CvUnit::foundCity` passes the carried religion at `CvGameCoreDLL_Expansion2/CvUnit.cpp:10622` into `CvPlayer::foundCity`, which calls `CvPlayer::initCity` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:13733`. `CvCity::init` applies `eInitialReligion` through `CvCityReligions::AdoptReligionFully` at `CvGameCoreDLL_Expansion2/CvCity.cpp:927` through `CvCity.cpp:930`.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:286` with default `Value="0"`. Macro is `MOD_GLOBAL_RELIGIOUS_SETTLERS` in `CvGameCoreDLL_Expansion2/CustomMods.h:241`; declaration is `CustomMods.h:1568`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:345`.

`CustomMods::preloadCache` caches the XML row. C++ checks `MOD_GLOBAL_RELIGIOUS_SETTLERS`.

Both feature rows are disabled by default in `(1) Community Patch/Database Changes/NewCustomModOptions.xml`: `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` at line 282 and `GLOBAL_RELIGIOUS_SETTLERS` at line 286. `RELIGIOUS_SETTLERS_WEIGHTED` is enabled by default at line 287 but is only effective through `MOD_RELIGIOUS_SETTLERS_WEIGHTED`.

`MOD_GLOBAL_RELIGIOUS_SETTLERS` is declared as `gCustomMods.isGLOBAL_RELIGIOUS_SETTLERS()` in `CvGameCoreDLL_Expansion2/CustomMods.h:241`, declared in `CustomMods.h:1568`, and cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:345`.

`MOD_RELIGIOUS_SETTLERS_WEIGHTED` is `(MOD_GLOBAL_RELIGIOUS_SETTLERS && gCustomMods.isRELIGIOUS_SETTLERS_WEIGHTED())` in `CvGameCoreDLL_Expansion2/CustomMods.h:242`, declared in `CustomMods.h:1569`, and cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:346`.

| `GLOBAL_RELIGIOUS_SETTLERS` | `RELIGIOUS_SETTLERS_WEIGHTED` | `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` | Validity | Expected DLL behavior | Audit classification |
| --- | --- | --- | --- | --- | --- |
| 0 | 0 or 1 | 0 | Supported default | Settlers do not carry religion; non-Venice puppets cannot faith-purchase ordinary faith buildings. | working as intended |
| 1 | 0 | 0 | Supported | Settlers take creation city majority religion if above pantheon; founded city adopts it. | working as intended, runtime unvalidated |
| 1 | 1 | 0 | Supported but unsafe edge case | Settlers choose among local religion followers by weight; all-zero religion follower cities can produce random real religion. | confirmed defect |
| 0 | 0 or 1 | 1 | Supported | Puppet faith-building DLL eligibility is broadened; UI exposure depends on Venice, `AllowsPuppetPurchase`, `PuppetPurchaseOverride`, and intended resources. | missing validation / compatibility risk |
| 1 | 0 | 1 | Supported | Both features operate on separate paths: settler religion and puppet faith-building purchase. | compatible, runtime unvalidated |
| 1 | 1 | 1 | Supported but unsafe edge case | Both features operate, with weighted-settler all-zero defect and resource-dependent puppet UI behavior. | confirmed defect plus missing validation |
| 0 | 1 | any | Supported no-op for weighted setting | Weighted option is macro-gated by `GLOBAL_RELIGIOUS_SETTLERS`, so it should have no effect. | working as intended |

## Observability and validation obligations

Log city religion composition, weighted roll result, unit religion, and `CvPlayer::foundCity` religion parameter.

1. `GLOBAL_RELIGIOUS_SETTLERS=1`, `RELIGIOUS_SETTLERS_WEIGHTED=1`: city with no pantheon/religion followers creates a settler. Expected after patch: settler religion remains `NO_RELIGION`; founded city follows normal no-initial-religion path.

2. `GLOBAL_RELIGIOUS_SETTLERS=1`, `RELIGIOUS_SETTLERS_WEIGHTED=1`: city with 3 followers of religion A and 1 of religion B creates multiple settlers from deterministic seeds. Expected: only A or B can be selected; no zero-follower religion can be selected.

3. `GLOBAL_RELIGIOUS_SETTLERS=1`, `RELIGIOUS_SETTLERS_WEIGHTED=0`: city with majority religion A creates settler, save/reload before founding, found city. Expected: `CvUnitReligion` persists and `CvCity::init` applies A through `AdoptReligionFully`.

- Add an assertion or guard that weighted settler selection does not choose a real religion when total real/pantheon follower weight is zero.

- In `CvUnit::foundCity`, log unit religion and `CvPlayer::foundCity` `eReligion`.

## Version differences, risks and unresolved questions

Local `git log -S'MOD_GLOBAL_RELIGIOUS_SETTLERS'` finds production-source provenance at `54dee87` only. No later local production rewrite was visible.

No enum or schema compile risk was found for the guarded blocks.

INFERRED: Units created outside normal `CvUnit::initWithNameOffset` or cities founded without `CvUnit::foundCity` would bypass the feature.

### Recovered finding: Confirmed Defect: Weighted settlers can receive a random religion from a city with zero religion followers

Classification: confirmed defect.

Exact location: `CvGameCoreDLL_Expansion2/CvUnit.cpp:1041` through `CvUnit.cpp:1058`; `CvGameCoreDLLUtil/include/CvWeightedVector.h:217` through `CvWeightedVector.h:220`.

```text
CvPlayer::initUnit
  -> CvUnit::initWithNameOffset
  -> MOD_GLOBAL_RELIGIOUS_SETTLERS branch
  -> MOD_RELIGIOUS_SETTLERS_WEIGHTED branch
  -> CvWeightedVector<ReligionTypes>::ChooseByWeight
```

Intended behavior: The XML comment at `(1) Community Patch/Database Changes/NewCustomModOptions.xml:284` says city-founding units take the city's majority religion, and line 285 says weighted mode chooses a religion with each religion weighted by local followers.

Observed or statically inferred failure: The weighted branch pushes every religion from `RELIGION_PANTHEON` through all real religions even when follower count is zero. `CvWeightedVector::ChooseByWeight` explicitly falls back to `ChooseAtRandom` when total weight is zero. In a city with no pantheon or religion followers, the settler can randomly receive any real religion; `CvUnit.cpp:1057` then accepts any result greater than `RELIGION_PANTHEON`.

Why a rewrite caused or exposed it: INFERRED: Local history is grafted at `54dee87`, so no later local rewrite can be identified. The issue is exposed by the current weighted-vector behavior in the present architecture.

Smallest safe correction: Before calling `ChooseByWeight`, require `vReligions.GetTotalWeight() > 0`, or only push candidate religions with `iNumFollowers > 0`. Preserve pantheon exclusion by keeping the existing `eReligion > RELIGION_PANTHEON` final gate.

Current revalidation update 2026-06-29: The supplied `codex/modmod_resources/Merged Gameplay Mods/ReligiousSettlers/XML/GlobalReligiousSettlers.xml` enables `GLOBAL_RELIGIOUS_SETTLERS` and leaves `RELIGIOUS_SETTLERS_WEIGHTED` at its current default. The defect survived in audited source and was fixed by adding the `vReligions.GetTotalWeight() > 0` guard in `CvGameCoreDLL_Expansion2/CvUnit.cpp`.

Possible regression risks: Filtering zero-weight candidates changes deterministic seeded outcomes for edge cases where the city has at least one follower and many zero-weight religions. Requiring total weight only avoids that broader change.

Runtime scenario needed: Enable `GLOBAL_RELIGIOUS_SETTLERS=1` and `RELIGIOUS_SETTLERS_WEIGHTED=1`, create a settler in a city with population but no pantheon/religion followers, and verify the unit religion stays `NO_RELIGION` or `RELIGION_PANTHEON` rather than a random founded religion.

Classification: working as intended.

Unit religion is serialized as `unit.m_Religion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp:28805`, and `CvUnitReligion::Serialize` saves religion, strength, max strength, and spreads used at `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp:5788` through `CvReligionClasses.cpp:5795`.

Capture copies unit religion into `CvUnitCaptureDefinition` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:2766` through `CvUnit.cpp:2770` and restores it to the captured unit at `CvUnit.cpp:2843` through `CvUnit.cpp:2845`. Gift conversion copies religion at `CvGameCoreDLL_Expansion2/CvUnit.cpp:6245` through `CvUnit.cpp:6247`.

The two requested features do not share mutable state directly. `GLOBAL_RELIGIOUS_SETTLERS` modifies unit/city religion state; `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` modifies purchase eligibility.

INFERRED: The practical interaction is indirect: a settler-founded city that adopts a carried religion can later become a city whose majority religion unlocks belief buildings; if that city is later puppeted, the puppet purchase feature can allow those buildings to be faith-purchased. No immediate source-level overwrite or cache collision was found.

`git log -S'MOD_GLOBAL_RELIGIOUS_SETTLERS'` and `git log -S'MOD_GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS'` also find `54dee87` only.

1. Fixed `GLOBAL_RELIGIOUS_SETTLERS` weighted all-zero behavior in `CvUnit::initWithNameOffset` by requiring total positive religion follower weight before `ChooseByWeight`.

3. Optional diagnostics around settler religion choice and puppet faith-building purchase eligibility were not added; no runtime logging change was needed for the concrete fix.

| Feature | Ready for normal use? | Reason |
| --- | --- | --- |
| `GLOBAL_RELIGIOUS_SETTLERS` | Runtime validation still required | Core path is authoritative, save/load looks sound, and the confirmed all-zero weighted-selection defect has been patched. Majority-only mode remains statically plausible. |

## Source and dependency lookup

Direct references are `CustomMods.h:241`, `CustomMods.h:1568`, `CustomMods.cpp:345`, `NewCustomModOptions.xml:286`, `CvUnit.cpp:1031`, `CvUnit.cpp:10622`, and `CvPlayer.cpp:36779`.

`CvUnit::initWithNameOffset` sets settler religion when the unit has `UNITAI_SETTLE` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1031`. It either uses weighted local city followers when `MOD_RELIGIOUS_SETTLERS_WEIGHTED` is true at `CvUnit.cpp:1035`, or the city majority religion at `CvUnit.cpp:1050`.

`CvUnit::foundCity` passes the carried religion into `CvPlayer::foundCity` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:10622` when the option is enabled and unit religion is above `RELIGION_PANTHEON`.

`CvPlayer.cpp:36779` uses captured settler religion in grateful settler logic.

`MOD_RELIGIOUS_SETTLERS_WEIGHTED` is an explicit dependent option defined in `CustomMods.h:242`, cached in `CustomMods.cpp:346`, and configured in `NewCustomModOptions.xml:287` with default `Value="1"`.

INFERRED: The feature assumes the settler creation plot has a city (`pPlotCity`) when religion should be assigned. It also assumes carried unit religion is serialized by the existing unit religion state.

Faith/free religious unit creation has separate religion assignment paths in `CvGameCoreDLL_Expansion2/CvCity.cpp:30690` and `CvGameCoreDLL_Expansion2/CvCity.cpp:35090`. These do not replace the settler path but show unit religion assignment is not centralized.

Save after settler creation and before founding. Reload and confirm the settler retains religion and applies it on founding.

`ACTIVE_AND_REACHABLE`

In the audited source, the authoritative creation path is `CvPlayer::initUnit` -> `CvUnit::init` -> `CvUnit::initWithNameOffset`. `CvPlayer::initUnit` calls `pUnit->init(...)` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:9132`, and `CvUnit::init` calls `initWithNameOffset` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:549`.

The guarded branch at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1031` is still authoritative for normal settler unit creation. The carried religion is serialized as `unit.m_Religion` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28805` through `CvUnitReligion::Serialize` at `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp:5788`.

The city-founding path is still authoritative: `CvUnit::foundCity` passes the carried religion at `CvGameCoreDLL_Expansion2/CvUnit.cpp:10622`, `CvPlayer::foundCity` passes it to `initCity` at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:13733`, and `CvCity::init` applies it with `CvCityReligions::AdoptReligionFully` at `CvGameCoreDLL_Expansion2/CvCity.cpp:927`.

Capture and gift paths copy unit religion after creating the new unit: capture stores religion in `CvUnitCaptureDefinition` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:2766` and restores it at `CvUnit.cpp:2843`; gift copies it at `CvGameCoreDLL_Expansion2/CvUnit.cpp:6245`.

BASELINE_DEFECT_REPORT: The default dependent mode `RELIGIOUS_SETTLERS_WEIGHTED=1` can assign a random real religion when the creation city has zero pantheon/religion followers. The weighted branch pushes every religion with its follower count at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1041` through `CvUnit.cpp:1045`, checks only that the vector is non-empty at `CvUnit.cpp:1047`, and calls `ChooseByWeight` at `CvUnit.cpp:1050`. `CvWeightedVector::ChooseByWeight` falls back to `ChooseAtRandom` when total weight is zero at `CvGameCoreDLLUtil/include/CvWeightedVector.h:217` through `CvWeightedVector.h:220`. `CvUnit.cpp:1057` then accepts any randomly chosen religion above `RELIGION_PANTHEON`.

Smallest safe correction: Require `vReligions.GetTotalWeight() > 0` before `ChooseByWeight`, or push only religions with positive follower counts. The lower-risk patch is the total-weight guard because it preserves seeded outcomes whenever at least one candidate has positive weight.

Runtime validation required: Enable `GLOBAL_RELIGIOUS_SETTLERS=1` and `RELIGIOUS_SETTLERS_WEIGHTED=1`, create a settler in a city with no pantheon/religion followers, save/reload before founding, and confirm no random real religion is carried or applied.

Readiness: Not ready for normal use with the default weighted sub-option. Majority-only mode (`RELIGIOUS_SETTLERS_WEIGHTED=0`) is statically plausible but still needs runtime validation.

With `GLOBAL_RELIGIOUS_SETTLERS` disabled, `CvUnit::foundCity` always passes `NO_RELIGION` for settler-founded cities at `CvGameCoreDLL_Expansion2/CvUnit.cpp:10622`. The city can still receive capital-religion, pantheon, or local-religion initialization from `CvCity::init` at `CvGameCoreDLL_Expansion2/CvCity.cpp:913` through `CvCity.cpp:938`.

With `GLOBAL_RELIGIOUS_SETTLERS` enabled and `RELIGIOUS_SETTLERS_WEIGHTED` disabled, settler religion is the creation city majority religion from `CvGameCoreDLL_Expansion2/CvUnit.cpp:1054`.

With both `GLOBAL_RELIGIOUS_SETTLERS` and `RELIGIOUS_SETTLERS_WEIGHTED` enabled, settler religion is chosen by `CvWeightedVector<ReligionTypes>::ChooseByWeight` from follower counts for `RELIGION_PANTHEON` through `GC.getNumReligionInfos() - 1` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1041` through `CvUnit.cpp:1050`.

- Should weighted religious settlers include `NO_RELIGION` as a weighted outcome for no-religion followers, or should no-religion simply mean no carried religion?

1. Confirmed findings: `GLOBAL_RELIGIOUS_SETTLERS` with default weighted mode had a confirmed all-zero follower defect; `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` has authoritative DLL/direct purchase paths and resource-dependent human UI behavior, not a confirmed source defect.

2. Changes made: guarded weighted settler selection against zero total weight in `CvUnit::initWithNameOffset`; no production source change remains for puppet production popup reachability.

3. Resource conclusion: supplied resources activate both options and add useful unit-side `PuppetPurchaseOverride` rows; audited source fixes were required only for weighted settlers.

4. Exact runtime tests: run the seven scenarios listed in the "Exact Runtime Tests" section, with emphasis on all-zero weighted settlers and non-Venice puppet faith-building UI access.

5. Recommended logging or assertions: log settler religion candidate weights and selected religion if runtime tests fail; log puppet faith-building purchase eligibility and UI-open decisions if resource/UI validation fails.

6. Ready for normal use: both features still need runtime validation in-game after the settler fix; no further source change is justified from static audit alone.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `GLOBAL_RELIGIOUS_SETTLERS` | Create settlers in cities with majority and mixed follower religions, then found cities. | Save after settler creation and before founding; reload and confirm unit religion is preserved and applied to new city. | `CvUnit::initWithNameOffset`, `CvUnit::foundCity`, `CvPlayer::foundCity`. | `CvUnit.cpp:1031`, `CvUnit.cpp:10622`, `CvPlayer.cpp:13721` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `GLOBAL_RELIGIOUS_SETTLERS=1`, `RELIGIOUS_SETTLERS_WEIGHTED=1` | Create a settler in a city with population but zero pantheon/religion followers. | Save after settler creation and before founding; reload and found. | Settler should not receive a random real religion; founded city should follow normal no-initial-religion path. | Confirmed all-zero weighted-selection defect in `CvUnit.cpp:1041` and `CvWeightedVector.h:217`. |
| `GLOBAL_RELIGIOUS_SETTLERS=1`, `RELIGIOUS_SETTLERS_WEIGHTED=1` | Create settlers in a city with positive followers for two religions and zero followers for others. | Save/reload before founding for each selected religion. | Only religions with positive follower weights should be selected; zero-follower religions should never be selected. | Regression check for any weighted-selection patch. |
| `GLOBAL_RELIGIOUS_SETTLERS=1`, `RELIGIOUS_SETTLERS_WEIGHTED=0` | Create a settler in a city with a clear majority religion. | Save after creation; reload and found city. | Unit religion persists through `CvUnitReligion` serialization and `CvCity::init` applies it with `AdoptReligionFully`. | Majority-mode authority and serialization validation. |
| Captured/gifted religious settler | Capture or gift a settler carrying religion, then found a city. | Save after ownership transfer; reload and found city. | Religion copied in capture/gift path should remain on the new unit and apply on founding. | Copy paths at `CvUnit.cpp:2766`, `CvUnit.cpp:2843`, and `CvUnit.cpp:6245`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
