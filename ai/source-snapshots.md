# Source baselines, snapshots and incremental revalidation

Publication copy: machine-specific paths are omitted. Source versions, content hashes and verification limits are preserved. Recovered artifacts and DB snapshots are not included in this repository.

## Lightweight document provenance

A durable audit is a versioned technical reference describing the best established understanding of its subsystem. Each maintained technical owner records four fields near its title:

```text
Knowledge status: Maintained reference (or maintained design/reference)
Original audit baseline: named baseline/snapshot, version and identity limits
Last source verification: `audit-20261029-task1u-b` / `src-20261029-vp-548` for the added Task 1U-B sections and explicitly revalidated unit records; earlier recovered and c285a18 sections retain their stated historical scope.
Applicability / known version limits: VP 5.4.8 for the bounded Task 1U-B contracts; earlier sections retain original recovered/c285a18 scope unless explicitly revalidated. UPAI and effective data separately qualified. See [source identities](source-snapshots.md).
```

These fields apply to the document; claim-level evidence can be more specific. A design proposal or guide pattern must not inherit source verification simply because it shares a document with source findings. A later partial revalidation lists the revalidated sections/paths and retains older qualification for the remainder. Do not replace the whole-document baseline with a newer version when only one branch was checked.

Use existing maintained knowledge first. Older verification is a version qualification, not grounds to discard useful findings or repeat the entire investigation. The installed evidence labels remain scoped to the source/data/runtime against which they were established. Imported source-audit assertions are explicitly **reported recovered verification**, not newly reproduced source proof.

## Recovered baseline

Baseline ID: **`legacy-vp526`**. Kind: **RECOVERED_BASELINE**, not an immutable exact source snapshot.

| Field | Best recoverable identity / evidence |
|---|---|
| VP version | **VP 5.2.6 release family**; the documentary release anchor resolves to the exact `5.2.6 Release` commit |
| Repository | Readable identity source: `VP source checkout (local path omitted)`, remote `https://github.com/nobubblegums/Community-Patch-DLL.git`; the original audit process's absolute checkout path is not established |
| Release commit | **`54dee87c630406f9c36f25bbb03d7562351ede51`**, subject `5.2.6 Release`, commit timestamp `2026-05-14T16:05:59+02:00`. This identifies the release anchor, not every later audit's working tree |
| Branch | Recovered optional-feature reports name `audit/modmod-feature-health`; this is not proof that every AI report ran on that branch |
| AI report relationship | The optional-feature baseline says the existing AI reports/Atlas/investigations were already present as untracked material. This supports an approximate shared source era, not an exact per-document commit |
| Later revisions | Source capture commits for the recovery labels are resolved below. They contain different DLL tree identities. A report's presence in a capture does not prove each statement was verified against that capture; base observations and later repairs remain distinct |
| Data profile | VP+UPAI compiled-data observations were reported; only `database_tables.txt` survives from the named export directory. Referenced CSV row exports are unavailable in this artifact |
| Recovery labels | All nine directory labels resolve to source commits in the readable clone; documentation blobs were compared with recovered input. The merge combines variants, so no one capture is a universal verification baseline |
| Metadata investigation | Recovery `.git` is empty, but a metadata-only check of the readable VP clone resolves the release and recovery labels. Only commit metadata, tree IDs and recovered documentation blobs were examined; no current VP implementation archaeology or behavior revalidation was performed |

Evidence: `optional-feature baseline` (external recovery artifact; not distributed), `recovery merge report` (external recovery artifact; not distributed), and the recovery-capture identities recorded below. This establishes an exact release/capture lineage. It does not reconstruct the original audit worktree or prove modern applicability of individual mechanical claims.

Migration pass: `migration-20261029-canonical`, documentation consolidation only. Original input: `recovered corpus (not distributed)`, 255 files / 6,498,939 bytes. No current implementation target is selected. Task 0's `design-task0-20261029` remains a design-pass identity, not source verification. The pinned civ5-live VP 5.4.8 profile is not the universal target.

## Resolved recovery captures

These are **RECOVERY_CAPTURE** identities, not new source-verification passes. A matching Markdown blob proves document capture provenance; it does not prove the audit ran on that commit or with a clean tree. The preserved corpus is a merge of several captures.

| Recovery label | Full source commit | Subject | Matching recovered Markdown blobs |
|---|---|---|---:|
| `7ad1837` | `7ad1837297f28ea732c6eb4704c5af7478e8da08` | linked: fixed retreat bugs and visual glitches | 58 |
| `d60c83f` | `d60c83f21f8470cbca631b353ba5c26c849ab94d` | explored integrating wh military log to base | 59 |
| `d18eacd` | `d18eacd1115555840debfe1d9533e8c3564a31fe` | modmod: fixes to attrition and heavy charge | 64 |
| `ae9d8f3` | `ae9d8f3bfa23bc11b8f25548e822c5ecb053c784` | reverted popup | 60 |
| `298f536` | `298f536dd2fe0b0984b28440460209fa6a37b4b5` | fixed linked movement bugs, updated files, added better support for squads | 58 |
| `32b2ae4` | `32b2ae48111cf2f97c11535c34eee80037276eb0` | log for linked unit movement | 58 |
| `9066b70` | `9066b70b21914f1900f23f5dc4eb44af5be25297` | docs: restore Codex project files and modmod audit | 57 |
| `edff702` | `edff702c0a00d275183b0d5044c261ef11e4a5f8` | wh-military-log: fixes & added serialization | 58 |
| `f99a2cb` | `f99a2cbc063bc71e3b35c8d977c52d290cfb725f` | fix for local generals crash | 58 |

Metadata evidence: `git rev-parse --verify <label>^{commit}`, `git show -s`, source subtree object IDs, and byte comparisons of `codex/docs/` Markdown blobs with recovered documents. The DLL subtree hashes differ across captures; release identity alone therefore cannot certify the repaired optional-feature observations. No source file contents or current-source behavior were audited.

## Precise snapshot record

For a new current-source pass, select the actual repository and register a stable immutable ID such as `src-YYYYMMDD-target-01`. Never repoint it to changed content.

| Field | Required content |
|---|---|
| Snapshot ID / kind | Unique content identity; PRECISE_SNAPSHOT, separate from a recovered approximate baseline |
| Repository identity | Public repository identity and exact commit; omit machine-specific paths |
| Branch / HEAD | Actual branch or detached state and full commit hash |
| VP version/release | Identified version with evidence, or explicitly unknown |
| Working-tree identity | Clean/dirty; participating staged/unstaged changes, untracked inputs and submodules |
| Content preservation | Durable patches/content artifacts and hashes sufficient to recover relevant local changes; HEAD alone does not identify a dirty tree |
| Configuration | Compile flags, options, data packages/load order and examined enabled/disabled/unknown variants |
| Data/runtime correspondence | Identified DB/build/runtime evidence and relationship/mismatches, or not examined |
| Notes / scope limits | Excluded content and identity uncertainty |

Keep relevant identity artifacts with durable evidence, not `/tmp`. Preserve unrelated work; registration does not authorize cleanup or copying policy/whole repositories. At migration there were no precise current-source snapshots; the Task 1U records below are the first registered current-source identities.

## Audit-pass and claim evidence

A pass ID such as `audit-YYYYMMDD-task1u-01` records date, snapshot(s), auditor/session, decision/section scope, files/symbols/data dependencies, variants, exclusions, evidence added/rechecked and coverage changes. Each current-source claim/term resolves to exact repository-relative source and symbol/location. Data observations additionally identify effective snapshot, keys/columns, freshness/load-order limits; defaults alone do not prove effective values. Runtime evidence separately identifies the running build/content, scenario and recorded checks.

## Incremental revalidation

1. Consult the canonical owner and identify its last verification baseline and scope.
2. Identify supporting files, callers, helpers, state producers, Defines, options, loaders and SQL/XML/data dependencies for the relevant claims.
3. Where exact identities exist, inspect diff/history over that **dependency surface**, not only the headline `.cpp` file. Record the compared identities and evidence of material equivalence.
4. Reuse findings whose supporting implementation/configuration is materially unchanged and record the new revalidation; inspect changed or uncertain branches and their dependencies directly. Apparent stable naming is insufficient.
5. If the old exact commit/content is unknown, diff-based shortcutting cannot prove equivalence. The first modern pass re-establishes a precise baseline for the relevant subsystem/paths, using the existing reference as its investigation map and retained understanding.
6. Update the canonical owner, its provenance and registry applicability/coverage. Do not create a newer-version duplicate. For partial revalidation retain the older baseline for untouched sections.
7. Current task-relevant source wins in a conflict. Preserve resolved version differences and still-unresolved contradictions with their evidence; do not silently overwrite uncertainty or transfer old verification labels to the new target.

A precise identity enables more efficient later incremental passes. It does not itself prove any behavior, complete the source inventory or establish which build/content is running.

## Task 1U selected-source pass

Pass `audit-20261029-task1u-01`: 2026-10-29, source/data inspection, no runtime. **PARTIAL audit**, not a Task 1U completion certificate. This first pass selects the readable current repository because the requested recovered root has no DLL tree or usable Git metadata. UPAI remains a separate supplied artifact. Previous source-era observations remain qualified; the compared release anchor is a diff aid, not the unknown original audit worktree.

| Field | Base snapshot `src-20261029-vp-c285a18` |
|---|---|
| Kind / repository | PRECISE_SNAPSHOT; `VP source checkout (local path omitted)` |
| Remotes | origin `https://github.com/nobubblegums/Community-Patch-DLL.git`; upstream `https://github.com/LoneGazebo/Community-Patch-DLL/` |
| Branch / HEAD | `feature/ipc-overlay`; `c285a18f97d33438b23fb87d6a0a4eefc037c451` |
| Tree state | `git status --short` empty at inspection; no source edits by this pass |
| Release | `git describe --tags --always`: `Release-5.4.6-5-gc285a18`; do not label this 5.2.6 or the pinned runtime 5.4.8 |
| Configuration | `CustomMods.h/.cpp`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml`; repository defaults differ from effective DB and loaded-game state |
| Runtime / build | No build or running-content correspondence established; NOT_TESTED |
| Preservation | Clean tracked source recoverable from full commit; UPAI content has independent manifest below |

| Field | UPAI snapshot `src-20261029-upai-recovered32` |
|---|---|
| Path | `recovered artifact: modmod_resources/Unit Production AI (v 3.2)` |
| Identity | Directory label v3.2, manifest version 3, mod ID `b2eb6d1e-1fcb-4a0d-upai-51691aa312da`; recovered artifact, no independent repository/commit established |
| Target | Dependency bounds CP 101–999, VP 15–999; these do not identify an exact VP release. Intended VP revision UNKNOWN; old reported profile is approximately 5.2.6 family |
| Surfaces | SQL/XML data and triggers only in supplied package; no DLL replacement listed. Enables existing `AI_UNIT_PRODUCTION` option. DLL role/strategy support is upstream optional code |
| Load order | NewStrategies → NewUnitAITypes → NewLeaderFlavors → **NewTechFlavors** → NewUnitFlavors → Formations → Triggers, per manifest |
| Packaging mismatch | Manifest names `NewTechFlavors.sql`; artifact supplies `NewTechPolicyFlavors.sql`. Effective loading/failure behavior unverified. No repair performed |
| Compatibility | Fixed role/flavor/strategy IDs and unit/tech keys require profile validation. Presence of matching upstream consumers does not prove compatibility |

UPAI file SHA-256 identities (all regular supplied files):

```text
f4338aa80e9fb2e83cc598a8eb2e0439209d383b817e1d14405fbad3e5743747  Formations.xml
c888807822d61a22ea851a9fa3302bb2a0935574edabb3e91bb81935af12db6a  NewLeaderFlavors.sql
2d31b4204ac5a3fc41806d410e989b1056198490e8656b63dda49aafa461bc72  NewStrategies.sql
5afc65fa260b822b29236efcec505bb18b07ebd9deaa115947884fba8b27b54c  NewTechPolicyFlavors.sql
d4effa9f2f35db636a514f22b764fc5b24944a94bee975a5b49e824699265735  NewUnitAITypes.sql
4da76ec489e111d70ef5314660d66d9786842da29c14372eafbf49459ed3190b  NewUnitFlavors.sql
27a077100112bdb786153e6e324edf1e729b66a17cc71006d01abb8995cf8eaf  Triggers.sql
2564e68cffccfef5e1da8ecd5e6e8fadd9e133604cf05e18104a393c6efe55ec  Unit Production AI (v 3).modinfo
```

DB evidence `db-20261029-reference`: `Civ5DebugDatabase.db (reference snapshot; not distributed)`, 6,770,688 bytes, SHA-256 `429d65ededbdfac295a67b95a97c255e2d5a5c43177c1d476fad816aeaf8cca4`; opened SQLite `mode=ro`. No exact generating source/build identified. It is a separate effective snapshot, **not certified effective data for either selected source profile**. It contains 519 `Unit_Flavors` rows, `AI_UNIT_PRODUCTION=0`, no eight UPAI role flavors, `BALANCE_VP=1`, `BALANCE_CORE_JFD=0`, `BALANCE_UNIT_INVESTMENTS=0`, `CORE_PUPPET_BUILDING_LIMITATIONS=1`. Defines observed: operation base 5000, army base 750, operation flavor multiplier 250, duration base 0.15, duration per-turn 0.015. These are VERIFIED_IN_DB only for this hash. No hypothetical UPAI final database was constructed.

Incremental evidence: diff `54dee87c630406f9c36f25bbb03d7562351ede51..c285a18f97d33438b23fb87d6a0a4eefc037c451` inspected for unit sanity (changed own/their lands belief bonuses now summed and multiplied by 2/3), and change statistics for city selector/military manager. Current bodies independently inspected for AddFlavorWeights/GetWeight/RecommendUnit/CheckUnitBuildSanity, ChooseProduction/duration, flavor latest/GS consumers, recommended sizes/soft cap, operation peek/commit/uncommit/finish and city acquisition. Broader upstream operation creation predicates, all configuration dependencies and every shared producer are not yet revalidated. See current coverage in [scope](units/task-1u-scope.md#task-1u-current-pass-coverage).

## Task 1U-B authoritative VP 5.4.8 identity

Snapshot **src-20261029-vp-548**, pass **audit-20261029-task1u-b**, 2026-10-29. Repository `VP 5.4.8 source checkout (local path omitted)` is the sole upstream authority for this pass. `git rev-parse HEAD` directly returned `7307aefdf985a4f401a7996a896283557b896e2b`; porcelain status was empty. Detached HEAD; origin `https://github.com/LoneGazebo/Community-Patch-DLL.git`; subject `5.4.8 Release`. Pinned read-only 5.4.8 baseline, not a working implementation checkout. No change to this identity is inferred from any comparison repository's branch or HEAD. No runtime/build identity claimed.

Prior snapshot src-20261029-vp-c285a18 / audit-20261029-task1u-01 remains historical comparison only. Its commit content was read for targeted diffs; all new mechanical claims refer to actual files in the baseline above. UPAI remains **src-20261029-upai-recovered32**, with its independent eight file hashes registered above; its intended VP version is still UNRESOLVED_EXTERNAL, not reidentified as 5.4.8.

Revalidation: normalized-line comparisons of 32 participating .cpp/.h files against c285a18; 497 CP/VP SQL/XML/modinfo paths compared. UnitProductionAI, CityStrategyAI, CityAI, EconomicAI, PlayerAI, AIOperation, ArmyAI, FlavorManager, GrandStrategyAI, ProcessProductionAI and Infos bodies/headers unchanged. Changed surfaces: City/City.h return-by-reference optimizations; MilitaryAI IsPossibleMusterCity fixes ownership `==` rejection to `!=`; Player resource eligibility iterates cached positive requirement/total IDs; UnitClasses adds that cache, populated by Globals::GameDataPostCache; CustomMods removes IPC and changes minidump compile control. Unit formulas and relevant flavor/role/strategy/formation data unchanged. CP Define addition UNITCOMBAT_MOUNTED is a cached type key, not a direct unit-score coefficient. Other data deltas are package metadata, text/concepts/art, not unit decision formulas. Changes were followed in the 5.4.8 bodies, not treated as sufficient proof alone.

Evidence and scope: revalidated t001–t043; traced producer contracts, flavor state, operation creation/acquisition/recruitment/stage interfaces, all scorer/recommender callers, unit/process proxy, native legality, CvSeeder arithmetic and configuration. Current verification is section-bounded; recovered 5.2.6 narrative is not globally promoted. Build/runtime NOT_TESTED. Reference gameplay DB remains separately identified `db-20261029-reference` with unchanged SHA-256; no certified 5.4.8 or 5.4.8+UPAI effective DB found in references, baseline or recovered corpus. Local transformation test is described in role-strategies; it is not effective loaded data.

Direct source content hashes for the compared dependency files:

```text
b992b2d2006278d8c5179f7c2c51e986e5f148a38dbfce334d8f572c75849d06  CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp
261d3dd2bc69268f11a7206c5af9ca37b72687660effb17c2c0131285b180cf5  CvGameCoreDLL_Expansion2/CvUnitProductionAI.h
4b770c66de154ed8f1021f910270c6b6d6acd99f9c7cc698cfb5a132f403b26c  CvGameCoreDLL_Expansion2/CvCityStrategyAI.cpp
8f4f0eaf960540153066783f9a5e36e4222627b9556517fd199945be92eafcfa  CvGameCoreDLL_Expansion2/CvCityStrategyAI.h
512277a3c248d121365701cd300661bb737aed1031f4e18c4ddd12d222dba558  CvGameCoreDLL_Expansion2/CvCityAI.cpp
0be3c59ecdc61a7edc59874197b37f91aa8b98d3427b77625968d3d2a5e2678d  CvGameCoreDLL_Expansion2/CvCityAI.h
28f7c00314fb0d9e3317608052705706f411b582c2959d2093cae092d5306a24  CvGameCoreDLL_Expansion2/CvCity.cpp
e2c015cc0c545ac786c0af72c5ff6dfe2b3d8f0f6431e23a4bcf20d885bf9126  CvGameCoreDLL_Expansion2/CvCity.h
371b566037cb864644e23286b4bc3c8e17de3710db9087cb2744ecb905983043  CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp
94961a4df9e59bd3f6346f64cd50af1463fc579657c0bb39c08cf8a68d1570de  CvGameCoreDLL_Expansion2/CvMilitaryAI.h
d500c9dcd9bf1bf7a85230eeb07b9157fde1d4e29dd88278d8652581b3a64e6f  CvGameCoreDLL_Expansion2/CvEconomicAI.cpp
5c1c16864ee0feb1e6a3c51540845769e938c2a7ed86a3b459648d1a3da917e3  CvGameCoreDLL_Expansion2/CvEconomicAI.h
2635c84053dc212744e22e23a0774532923c258acb2426572a77adde5e4995e3  CvGameCoreDLL_Expansion2/CvPlayerAI.cpp
fdd17f809a93b8c1ee1595347dc11ac713939e47b4ae24162d69379fcb3f9216  CvGameCoreDLL_Expansion2/CvPlayerAI.h
53923343086605c6ae65732366c1e115b958a13f9762dcb4d23bede2db118879  CvGameCoreDLL_Expansion2/CvAIOperation.cpp
56c71f0caa194571a9323ac632707f9341924657c5d2cb3b07cbc375c8a42e53  CvGameCoreDLL_Expansion2/CvAIOperation.h
b174c41411afe56548fd3255d4bef55e5a96a1a4b7b055c255147b3ab83c8bf5  CvGameCoreDLL_Expansion2/CvArmyAI.cpp
a904eab0c22a8c1aff8bbe67bd624b4ef1a2812d4b07730b8e4bef1ac0a825f3  CvGameCoreDLL_Expansion2/CvArmyAI.h
833b2d398e8c479961c6b6c2e631f58a0ef12c2eb5f0853f7e258a457beff29d  CvGameCoreDLL_Expansion2/CvFlavorManager.cpp
6489df3060754f4e56f4c125ccd935d38d769dbb0f0e75f7380914fb59ff2a9e  CvGameCoreDLL_Expansion2/CvFlavorManager.h
59f435b8a45f62d75924a0f1ead4a8bd6956610b1bc81e2d83708f75430c4204  CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp
2f6645bf0050ac68b3e31d4ac5afbe005b31ae74f8a3b4640110a3aeb89ff301  CvGameCoreDLL_Expansion2/CvGrandStrategyAI.h
97e27d1dda97c85283fc3024a5cbc5f22ae9ae2d1d45bcb98c02554a683033b7  CvGameCoreDLL_Expansion2/CvProcessProductionAI.cpp
fc0096e2a2686e841d36ad9f3ce2048fc9ad876fd172980cc2bd59449751ed64  CvGameCoreDLL_Expansion2/CvProcessProductionAI.h
d70bf730136016e786d99ea2eb2c673f90892f4ffd0aadf25348052d8e38398e  CvGameCoreDLL_Expansion2/CustomMods.cpp
511e706f87a9bc860865e7ce1e6ccc39f2291fb777ecd861043a9c4d75e50880  CvGameCoreDLL_Expansion2/CustomMods.h
cdda69b4cfcac3a9efbbc82917a21764f7728fe00fdbce7acb1edfb0e365d818  CvGameCoreDLL_Expansion2/CvPlayer.cpp
64d426d7926089b9b915bafd6d338181e0f500c8c855ecf96d9629d2444a19f0  CvGameCoreDLL_Expansion2/CvPlayer.h
9df5d31857c1fedb19efa4563a75b3a336d9aceb651ee5fd77206d0b17373b86  CvGameCoreDLL_Expansion2/CvUnitClasses.cpp
b9fd9bf7f75a8b407781d3f2945e407150ac9c720a7993477194a6ba0dc69b0c  CvGameCoreDLL_Expansion2/CvUnitClasses.h
bf39a27b54adada2e1fb447ff036273f0655c00d4c9b6153b284d3256d39df95  CvGameCoreDLL_Expansion2/CvInfos.cpp
814f885bd16a8bcf3f3c4e38b0824fe4d2861147f86cc6c04a4198477c1f19f4  CvGameCoreDLL_Expansion2/CvInfos.h
```
