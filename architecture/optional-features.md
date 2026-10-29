# Optional feature activation and dependency map

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How are optional feature switches activated and what contracts do they depend on?

Rows in CustomModOptions, cached switches, macros, dependent flags and external resources must be distinguished. This is the activation/dependency index; detailed mechanics have topic owners.

Canonical related owners: [Optional event and mission dispatch](<event-dispatch.md>), [Generals, admirals and optional local/era behavior](<../mechanics/units/commanders.md>), [Attrition and Heavy Charge displacement](<../mechanics/units/attrition.md>), [Linked movement, quick groups and Squads](<../mechanics/units/movement-groups.md>), [Role strategies and the VP+UPAI profile](<../ai/units/role-strategies.md>), [Military event log and Lua contract](<military-log.md>), [Settler-carried religion](<../mechanics/religion/settler-religion.md>), [Puppet faith purchasing](<../mechanics/production/puppet-faith-purchasing.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Data and configuration

Requested options are defined as database rows in `(1) Community Patch/Database Changes/NewCustomModOptions.xml`, declared and wrapped as `MOD_*` macros in `CvGameCoreDLL_Expansion2/CustomMods.h`, and cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp::CustomMods::preloadCache`.

Each requested top-level feature row defaults to `Value="0"` in `(1) Community Patch/Database Changes/NewCustomModOptions.xml`, except `LOCAL_GENERALS_NEAREST_CITY`, which has row default `Value="1"` but is macro-gated by `MOD_GLOBAL_LOCAL_GENERALS`.

`MOD_LOCAL_GENERALS_NEAREST_CITY` is defined as `(MOD_GLOBAL_LOCAL_GENERALS && gCustomMods.isLOCAL_GENERALS_NEAREST_CITY())` in `CvGameCoreDLL_Expansion2/CustomMods.h:225`.

`MOD_RELIGIOUS_SETTLERS_WEIGHTED` is defined as `(MOD_GLOBAL_RELIGIOUS_SETTLERS && gCustomMods.isRELIGIOUS_SETTLERS_WEIGHTED())` in `CvGameCoreDLL_Expansion2/CustomMods.h:242`. It is not one of the requested feature reports but is an explicit dependency of `GLOBAL_RELIGIOUS_SETTLERS`.

| Feature | Option row | Macro | Cache | Default | Preliminary status |
| --- | --- | --- | --- | --- | --- |
| `BALANCE_ERA_RESTRICTED_GENERALS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:509` | `CvGameCoreDLL_Expansion2/CustomMods.h:437` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:410` | `0` | `DEPENDENCY_UNSAFE / MISSING_VALIDATION` |
| `GLOBAL_RELIGIOUS_SETTLERS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:286` | `CvGameCoreDLL_Expansion2/CustomMods.h:241` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:345` | `0` | `ACTIVE_AND_REACHABLE` |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:282` | `CvGameCoreDLL_Expansion2/CustomMods.h:237` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:344` | `0` | `ACTIVE_AND_REACHABLE` |
| `GLOBAL_LOCAL_GENERALS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:269` | `CvGameCoreDLL_Expansion2/CustomMods.h:224` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:339` | `0` | `LIKELY_DEFECT` |
| `LOCAL_GENERALS_NEAREST_CITY` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:270` | `CvGameCoreDLL_Expansion2/CustomMods.h:225` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:340` | row `1`, effective off unless `GLOBAL_LOCAL_GENERALS` | `BASELINE_DEFECT_REPORT` |
| `BAL_ATTRITION` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1170` | `CvGameCoreDLL_Expansion2/CustomMods.h:1046` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:584` | `0` | `ACTIVE_WITH_AI_INTEGRATION` |
| `LINKED_MOVEMENT` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1140` | `CvGameCoreDLL_Expansion2/CustomMods.h:1022` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:573` | `0` | `POSSIBLY_SUPERSEDED` |
| `AI_UNIT_PRODUCTION` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1131` | `CvGameCoreDLL_Expansion2/CustomMods.h:1019` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:570` | `0` | `DEPENDENCY_UNSAFE` |
| `WH_MILITARY_LOG` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:768` | `CvGameCoreDLL_Expansion2/CustomMods.h:673` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:501` | `0` | `PARTIALLY_REACHABLE` |
| `PROMOTIONS_FLAGSHIP` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:725` | `CvGameCoreDLL_Expansion2/CustomMods.h:634` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:489` | `0` | `DEPENDENCY_UNSAFE` |
| `EVENTS_REBASE` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1022` | `CvGameCoreDLL_Expansion2/CustomMods.h:908` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:544` | `0` | `POSSIBLY_SUPERSEDED` |
| `EVENTS_CUSTOM_MISSIONS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:893` | `CvGameCoreDLL_Expansion2/CustomMods.h:784` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:523` | `0` | `STATICALLY_PLAUSIBLE_RUNTIME_UNTESTED` |
| `EVENTS_RESOLUTIONS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1042` | `CvGameCoreDLL_Expansion2/CustomMods.h:930` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:552` | `0` | `ACTIVE_AND_REACHABLE` |
| `EVENTS_UNIT_CREATED` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1087` | `CvGameCoreDLL_Expansion2/CustomMods.h:974` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:561` | `0` | `ACTIVE_AND_REACHABLE` |

```text
NewCustomModOptions.xml row
  -> CustomMods::preloadCache MOD_OPT_CACHE(...)
  -> gCustomMods.isOPTION()
  -> CustomMods.h MOD_* macro
  -> guarded DLL code path
```

## Source and dependency lookup

Local production-source history is effectively compressed at `54dee87` (`5.2.6 Release`). `git log -S'MOD_*'` and targeted `git log -G` probes for the audited symbols and surrounding functions found the requested guarded code in `54dee87` only.

Current branch commits after `54dee87` are documentation/scripts/snapshot work, including `03aa0ca`, `abaa66b`, `a72791b`, `91d3bab`, `8f949d1`, `27c5a8d`, `c64eaad`, `422d55c`, `c36d694`, `632b7a9`, `165d1ed`, `6d57804`, and `2d8f854`.

INFERRED: Because the local history is grafted/compressed, this pass cannot prove when a modmod feature was introduced or whether upstream rewrites happened before `54dee87`. It can only state that the audited source snapshot at `54dee87` contains the cited behavior and that no later local production-source rewrite is visible.

`BALANCE_ERA_RESTRICTED_GENERALS` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:437`, `CvGameCoreDLL_Expansion2/CustomMods.h:1633`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:410`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:509`, `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1715`, and `CvGameCoreDLL_Expansion2/CvPlayerAI.cpp:1864`.

`GLOBAL_RELIGIOUS_SETTLERS` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:241`, `CvGameCoreDLL_Expansion2/CustomMods.h:1568`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:345`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:286`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:1031`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:10622`, and `CvGameCoreDLL_Expansion2/CvPlayer.cpp:36779`.

`GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:237`, `CvGameCoreDLL_Expansion2/CustomMods.h:1567`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:344`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:282`, and `CvGameCoreDLL_Expansion2/CvCity.cpp:30256`.

`GLOBAL_LOCAL_GENERALS` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:224`, `CvGameCoreDLL_Expansion2/CustomMods.h:1562`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:339`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:269`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:21753`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:21757`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:21767`, and `CvGameCoreDLL_Expansion2/CvUnit.cpp:21771`.

`LOCAL_GENERALS_NEAREST_CITY` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:225`, `CvGameCoreDLL_Expansion2/CustomMods.h:1563`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:340`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:270`, `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32219`, `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32224`, `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32335`, and `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32345`.

`BAL_ATTRITION` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:1046`, `CvGameCoreDLL_Expansion2/CustomMods.h:1819`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:584`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1170`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:8305`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:16837`, and `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:524`.

`LINKED_MOVEMENT` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:1022`, `CvGameCoreDLL_Expansion2/CustomMods.h:1796`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:573`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1140`, `CvGameCoreDLL_Expansion2/CvUnit.h:844`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:145`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:1295`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:2578`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:3212`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:5487`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:14607`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:29800`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:30631`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:117`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:550`, and `CvGameCoreDLL_Expansion2/CvPlayer.cpp:10968`.

`AI_UNIT_PRODUCTION` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:1019`, `CvGameCoreDLL_Expansion2/CustomMods.h:1793`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:570`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1131`, `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp:1780`, `CvGameCoreDLL_Expansion2/CvTacticalAI.cpp:10826`, `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp:2061`, `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp:2063`, `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp:2085`, `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp:3925`, `CvGameCoreDLL_Expansion2/CvMilitaryAI.cpp:3941`, `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp:696`, `CvGameCoreDLL_Expansion2/CvUnitProductionAI.cpp:1580`, `CvGameCoreDLL_Expansion2/CvEconomicAI.cpp:1053`, `CvGameCoreDLL_Expansion2/CvAdvisorRecommender.cpp:394`, `CvGameCoreDLL_Expansion2/CvGrandStrategyAI.cpp:648`, and `CvGameCoreDLL_Expansion2/CvHomelandAI.cpp:6338`.

`WH_MILITARY_LOG` is directly referenced in its macro/cache/option files, the `MILITARYLOG` macro at `CvGameCoreDLL_Expansion2/CustomMods.h:1089`, player log lifecycle and Lua exposure, and many `CvUnit.cpp`/`CvUnitCombat.cpp` combat log sites. See `features/wh-military-log.md` for the full line list.

`PROMOTIONS_FLAGSHIP` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:634`, `CvGameCoreDLL_Expansion2/CustomMods.h:1712`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:489`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:725`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:21499`, `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32340`, and `CvGameCoreDLL_Expansion2/CvPlayer.cpp:32359`. MISSING_VALIDATION: audited repository data maps but does not define `PROMOTION_FLAGSHIP`, so the automatic grant is inert unless an external modmod supplies the promotion row.

`EVENTS_REBASE` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:908`, `CvGameCoreDLL_Expansion2/CustomMods.h:1767`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:544`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1022`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:6481`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:9858`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:9936`, and `CvGameCoreDLL_Expansion2/CvUnit.cpp:10040`.

`EVENTS_CUSTOM_MISSIONS` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:784`, `CvGameCoreDLL_Expansion2/CustomMods.h:1746`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:523`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:893`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:687`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:810`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:918`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:1342`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:1451`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:1886`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:1958`, `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:2035`, and `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:2327`.

`EVENTS_RESOLUTIONS` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:930`, `CvGameCoreDLL_Expansion2/CustomMods.h:1775`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:552`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1042`, `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:3138`, `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:3186`, `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:8664`, `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:8702`, `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:9482`, and `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:9533`.

`EVENTS_UNIT_CREATED` is directly referenced in `CvGameCoreDLL_Expansion2/CustomMods.h:974`, `CvGameCoreDLL_Expansion2/CustomMods.h:1784`, `CvGameCoreDLL_Expansion2/CustomMods.cpp:561`, `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1087`, and `CvGameCoreDLL_Expansion2/CvUnit.cpp:1253`.

```text
GLOBAL_RELIGIOUS_SETTLERS
  -> RELIGIOUS_SETTLERS_WEIGHTED
  -> CvUnit::initWithNameOffset carried settler religion
  -> CvUnit::foundCity
  -> CvPlayer::foundCity / CvCity::initCity
```

```text
GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS
  -> CvCity::IsCanPurchase puppet exception
  -> Lua CvLuaCity::lIsCanPurchase
  -> UI production purchase checks
  -> AI religion faith-spending checks
```

```text
GLOBAL_LOCAL_GENERALS
  -> CvUnit::changeExperienceTimes100 passes pFromUnit
  -> CvPlayer::setCombatExperienceTimes100
  -> CvPlayer::setNavalCombatExperienceTimes100
```

```text
LOCAL_GENERALS_NEAREST_CITY
  -> depends on GLOBAL_LOCAL_GENERALS in macro
  -> land general nearest-city spawn
  -> naval admiral nearest-coastal-city spawn
```

```text
PROMOTIONS_FLAGSHIP
  -> practical grant path depends on naval pFromUnit admiral spawn
  -> therefore practically depends on GLOBAL_LOCAL_GENERALS
  -> automatic grant also depends on external or bundled PROMOTION_FLAGSHIP UnitPromotions row
  -> CvUnit::restoreFullMoves
  -> CvUnit::GetGreatGeneralStackMovement
```

```text
BALANCE_ERA_RESTRICTED_GENERALS
  -> CvPlayerAI::GetDirectiveGeneral / GetDirectiveAdmiral
  -> CvPlayerAI::AI_updateGreatPeople
  -> CvHomelandAI great-person handling
```

```text
BAL_ATTRITION
  -> CvPlayer::DoUnitAttrition
  -> CvUnit::DoAttrition
  -> enemy-territory flat damage
  -> CvUnitCombat heavy-charge fallback suppression
  -> CvUnit combat modifier when no fallback exists
```

```text
LINKED_MOVEMENT
  -> CvLuaUnit linked/grouped methods
  -> CvUnit::LinkUnits / MoveLinkedLeader / DoGroupMovement
  -> CvUnitMission mission queue clearing
  -> CvUnit::ReadyToMove / maxMoves / canHandleAction
```

```text
MOD_SQUADS
  -> reuses linked movement state and execution helpers
  -> overlaps LINKED_MOVEMENT in CvUnit::move and turn processing
```

```text
AI_UNIT_PRODUCTION
  -> CvMilitaryAI role need/enough strategies
  -> CvUnitProductionAI sanity/balance scoring
  -> CvTacticalAI emergency purchase and formation placement
  -> CvEconomicAI / CvHomelandAI / advisor flavor classification
```

```text
WH_MILITARY_LOG
  -> CvPlayer military log allocation
  -> MILITARYLOG macro
  -> CvUnit and CvUnitCombat logging sites
  -> CvLuaPlayer::lGetMilitaryLog
```

```text
EVENTS_REBASE
  -> CvUnit::canLoad / canRebaseAt / rebase
  -> GameEvents CanLoadAt / CanRebaseInCity / CanRebaseTo / RebaseTo
  -> CvUnitMission MISSION_REBASE
  -> Lua/UI rebase calls
```

```text
EVENTS_CUSTOM_MISSIONS
  -> CvUnitMission custom mission dispatch
  -> GameEvents CustomMission* hooks
```

```text
EVENTS_RESOLUTIONS
  -> CvLeague proposal/vote/result paths
  -> GameEvents PlayerCanPropose / ResolutionProposing / ResolutionVoting / ResolutionResult
```

```text
EVENTS_UNIT_CREATED
  -> CvUnit::initWithNameOffset
  -> GameEvents UnitCreated
```

`EVENTS_REBASE` coexists with raw Lua fallback names (`CanLoadAt`, `CanRebaseInCity`, `CanRebaseTo`, `RebaseTo`) in `CvGameCoreDLL_Expansion2/CvUnit.cpp:6484`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:9865`, `CvGameCoreDLL_Expansion2/CvUnit.cpp:9943`, and `CvGameCoreDLL_Expansion2/CvUnit.cpp:10043`. The option changes event-dispatch infrastructure, not the whole semantic extension surface.

`EVENTS_REBASE` also coexists with current database-backed improvement rebasing through `ImprovementTableChanges.sql:98` and `CvGameCoreDLL_Expansion2/CvUnit.cpp:9921`.

`LINKED_MOVEMENT` coexists with `MOD_SQUADS`, which uses linked/grouped state and movement helpers in `CvGameCoreDLL_Expansion2/CvUnit.cpp`.

`BAL_ATTRITION` coexists with always-active promotion/improvement damage fields in `CvGameCoreDLL_Expansion2/CvPromotionClasses.cpp`, `CvGameCoreDLL_Expansion2/CvPlot.cpp`, and promotion SQL under `(2) Vox Populi/Database Changes/UnitPromotions/PromotionChanges.sql`.

`AI_UNIT_PRODUCTION` coexists with always-compiled role-flavor references in `CvGameCoreDLL_Expansion2/CvTechClasses.cpp:1258`, `CvGameCoreDLL_Expansion2/CvPolicyAI.cpp:4883`, and role unit AI enum entries in `CvGameCoreDLL_Expansion2/CvEnums.h:1873`.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `EVENTS_REBASE` | Register Lua handlers for all four rebase events, test carrier/city/improvement/empty target cases. | Save aircraft before rebase; reload and verify event result and final plot. | `CvUnit::canLoad`, `CvUnit::canRebaseAt`, `CvUnit::rebase`. | `CvUnit.cpp:6481`, `CvUnit.cpp:9858`, `CvUnit.cpp:9936`, `CvUnit.cpp:10040` |
| `EVENTS_RESOLUTIONS` | Register handlers that veto proposals, consume AI proposals/votes, and alter result handling. | Save during league session before proposals/votes/results; reload and compare callbacks. | `CvLeague::CanProposeEnact`, `CanProposeRepeal`, proposal/vote/result processing. | `CvVotingClasses.cpp:3138`, `CvVotingClasses.cpp:9482`, `CvVotingClasses.cpp:9533` |
| `EVENTS_UNIT_CREATED` | Create units through city production, purchase, initUnit, event reward, capture/convert, and upgrade paths. | Save immediately after unit creation; reload and verify callback state persisted externally if required. | `CvUnit::initWithNameOffset`, alternative event-created hooks. | `CvUnit.cpp:1253` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| Threshold combat where source naval unit dies | Cross threshold in combat where the source unit is killed or delayed-death. | Save before combat; reload and repeat. | Validate whether grant happens before cleanup and whether any promoted unit survives; document intended behavior before patching. | Source-unit lifetime risk around `CvPlayer.cpp:32342` and `CvPlayer.cpp:32361`. |
| Unsourced Great General/Admiral point thresholds | Cross thresholds via city/religion/policy/minor yields and Lua player XP setters. | Save before threshold crossing; reload and repeat for each source. | No local placement should occur because these paths pass no `pFromUnit`; ranked city/coastal-city behavior remains authoritative. | Bypass paths at `CvPlayer.cpp:24216`, `CvPlayer.cpp:24302`, `CvLuaPlayer.cpp:10375`, and `CvLuaPlayer.cpp:10423`. |
| Era-restricted combat bonus validation | With valid negate promotions, place obsolete Great General/Admiral near same-domain combat units and resolve combat. | Save before combat; reload and compare with option on/off. | Audited source does not prove combat aura suppression; runtime must confirm whether external promotions or code paths actually limit combat bonuses. | Compatibility risk: option hooks directive functions, not `CvUnit` combat modifier authority. |
| `EVENTS_REBASE=1` improvement airbase path | Rebase to a friendly unpillaged `AllowsRebaseTo` improvement with capacity and no visible enemy. | Save before rebase and reload before executing. | Audited source returns true before `CanRebaseTo`; decide whether this bypass is intended or should become observable. | Compatibility risk at `CvUnit.cpp:9921` through `CvUnit.cpp:9936`. |
| `EVENTS_REBASE=0` raw Lua fallback | Register old raw Lua `CanLoadAt`, `CanRebaseInCity`, `CanRebaseTo`, and `RebaseTo` handlers. | Save before rebase and reload before executing. | Raw Lua fallback names should fire only with `EVENTS_REBASE=0`. | Disabled behavior parity at `CvUnit.cpp:6486`, `CvUnit.cpp:9860`, `CvUnit.cpp:9941`, and `CvUnit.cpp:10042`. |
| `EVENTS_RESOLUTIONS=1` proposal veto | Register `PlayerCanPropose` handler that vetoes one enact and one repeal proposal. | Save before opening proposal UI and before AI proposal turn; reload and repeat. | Human UI and AI proposal code should respect veto without corrupting session state. | Proposal checks at `CvVotingClasses.cpp:3138` and `CvVotingClasses.cpp:3186`. |
| `EVENTS_RESOLUTIONS=1` `NO_PLAYER` UI probe | Register a handler that logs or rejects `iPlayer == -1`; open League Overview proposal list. | Save before opening UI; reload and repeat. | Audited source can pass `NO_PLAYER`; patched behavior should either skip the hook or explicitly document it. | Compatibility risk from `LeagueOverview.lua:299` into `CvVotingClasses.cpp:3138`. |
| `EVENTS_RESOLUTIONS=1` AI consume hooks | Register `ResolutionProposing` and `ResolutionVoting` handlers that return true only after spending all proposals/votes. | Save before AI proposal phase and AI voting phase; reload and process. | Returning true should suppress default AI only when listener performed equivalent work. | Missing validation at `CvVotingClasses.cpp:9482` and `CvVotingClasses.cpp:9533`. |
| `EVENTS_UNIT_CREATED=1` normal creation | Create units by city production, purchase, direct player/Lua init, great person spawn, and event reward. | Save immediately after callback-side state is set; reload and verify external state. | Listener should receive owner, unit ID, unit type, and plot for each `initWithNameOffset` creation. | Main hook at `CvUnit.cpp:1253`. |
| `EVENTS_UNIT_CREATED=1` final-state timing | Create units by upgrade, capture, gift, domain conversion, barbarian conversion, and religious-unit conversion. | Save before conversion/upgrade/capture, reload, execute, and compare callback state with final state. | Current hook fires before `convert()` finalizes many transferred fields; tests should document or guard this timing. | Timing risk at `CvUnit.cpp:1253`, `CvUnit.cpp:14179`, `CvUnit.cpp:14246`, and `CvUnit.cpp:1918`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
