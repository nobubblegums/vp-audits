# Optional event and mission dispatch

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

Where do optional hooks participate in authoritative actions?

Rebase/load, mission lifecycle, league proposal/voting and unit-created hooks have distinct timing, fallback and sentinel contracts. Optional switches and external handlers are part of applicability, not proof of runtime reachability.

## Reference synthesis

Optional event switches change dispatch contracts rather than merely adding passive observers. Rebase/load can use GameEvents or legacy raw Lua fallback names; friendly improvement airbases can return before the generic non-city hook. Custom-mission lifecycle hooks can affect built-in missions, whereas the unhandled-step path has a narrower scope. League probes may pass NO_PLAYER and AI hooks that report success may consume the whole proposal/vote action.

UnitCreated is reported inside initialization, before caller-side conversion/upgrade work copies final state. Handlers must not assume the final upgraded/captured unit state at that point. Military-log allocation/serialization and UI contracts have separate owners because later repair reports differ from the base event audit.

Canonical related owners: [Military event log and Lua contract](<military-log.md>), [Production queues and creation](<../mechanics/production/queue-and-creation.md>), [Religion founding and belief application](<../mechanics/religion/founding-and-beliefs.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Trigger and ownership

All mission queue execution through `CvUnitMission` can reach these hooks for unrecognized or custom mission cases.

Register handlers returning each supported custom action result and execute a multi-step custom mission.

Mission checks call `canRebaseAt` in `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:1119`, and mission start calls `rebase` in `CvUnitMission.cpp:1557`. Lua exposes `CanRebaseAt` and `rebase` in `CvGameCoreDLL_Expansion2/Lua/CvLuaUnit.cpp:1257`. UI rebase flow calls these methods in `(2) Vox Populi/Core Files/Overrides/WorldView.lua:838`.

Register all four handlers and test load/rebase into carriers, own cities, foreign cities, valid improvements, and empty plots.

Lua and UI proposal/vote operations route through league methods exposed by `CvLuaLeague`; AI sessions route through `CvLeague` turn/session processing.

Register handlers that veto proposals, consume AI proposals/votes, and observe proposal results for enact and repeal.

Normal unit creation through `CvPlayer::initUnit` and city creation paths eventually initializes `CvUnit` through this function.

Create units through city production, purchase, direct `initUnit`, event reward, capture/convert, and upgrade paths; compare which paths fire `UnitCreated`.

## Application and state

Listener return values can control mission possibility, activity type, mission timer, target plot, per-step action, and completion state.

The downstream effect is rebase eligibility and final unit plot/moves after `CvUnit::rebase`.

Hooks can veto proposal legality, mark AI proposals or votes as used, and observe proposal results.

Downstream consumers are Lua GameEvents listeners. The hook does not alter unit state unless listeners perform additional actions.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:893` with default `Value="0"`. Macro is `MOD_EVENTS_CUSTOM_MISSIONS` in `CvGameCoreDLL_Expansion2/CustomMods.h:784`; declaration is `CustomMods.h:1746`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:523`.

The option enables GameEvents callbacks for mission IDs not handled by built-in mission branches.

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1022` with default `Value="0"`. Macro is `MOD_EVENTS_REBASE` in `CvGameCoreDLL_Expansion2/CustomMods.h:908`; declaration is `CustomMods.h:1767`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:544`.

The option switches specific rebase/load extension points from raw Lua function calls to `GAMEEVENT_*` dispatch.

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1042` with default `Value="0"`. Macro is `MOD_EVENTS_RESOLUTIONS` in `CvGameCoreDLL_Expansion2/CustomMods.h:930`; declaration is `CustomMods.h:1775`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:552`.

The option enables GameEvents hooks around World Congress proposal, vote, and result paths.

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1087` with default `Value="0"`. Macro is `MOD_EVENTS_UNIT_CREATED` in `CvGameCoreDLL_Expansion2/CustomMods.h:974`; declaration is `CustomMods.h:1784`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:561`.

The option emits `GAMEEVENT_UnitCreated` from unit initialization.

| Configuration | Validity | Expected DLL behavior | Audit result |
| --- | --- | --- | --- |
| `EVENTS_REBASE=1` only | Supported | Rebase/load extensions use GameEvents instead of raw Lua fallback names. | Working with compatibility risk: improvement airbase path bypasses generic `CanRebaseTo` |
| `EVENTS_REBASE=0`, old raw Lua rebase handlers present | Supported legacy path | `CanLoadAt`, `CanRebaseInCity`, `CanRebaseTo`, and `RebaseTo` raw Lua functions can fire. | Working as intended |
| `EVENTS_REBASE=1`, only old raw Lua rebase handlers present | Unsupported migration mix | Old raw Lua handlers should not be expected to fire. | Compatibility risk |
| `EVENTS_CUSTOM_MISSIONS=1` only | Supported for external mission IDs | Unknown mission IDs can be authorized and processed by GameEvents. | Likely working for unknown IDs; compatibility risk for hooks firing on built-in mission lifecycle points |
| `EVENTS_RESOLUTIONS=1` only | Supported | Proposal legality, AI proposal/vote consumption, and result observer hooks fire. | Active and reachable; compatibility risk for `NO_PLAYER` proposal probes |
| `EVENTS_UNIT_CREATED=1` only | Supported | Unit creation emits `UnitCreated` after `CvUnit::initWithNameOffset` setup. | Active and reachable; timing is unsafe for final-state assumptions |
| All four event features enabled | Supported | Hooks are independent except where a custom mission is a rebase/ranged/nuke/built-in mission. | Compatibility risk: custom mission lifecycle hooks can observe or alter built-in mission processing |

## Observability and validation obligations

Log mission ID, all listener return values, timer increments, target plot, activity type, and completion decision.

Log target plot type, event dispatch path, listener return value, raw Lua fallback path, and final `setXY`.

Log league ID, player, resolution ID/type, proposal legality, listener result, and all-votes/all-proposals-used flags.

Log owner, unit ID, unit type, plot, creation source if available, and listener count/result.

1. `EVENTS_REBASE=1`: register all four rebase GameEvents. Test own city, foreign city accepted by event, carrier, friendly `AllowsRebaseTo` improvement, empty plot accepted by event, over-capacity city/improvement, and visible enemy on target.

2. `EVENTS_REBASE=0`: register old raw Lua `CanLoadAt`, `CanRebaseInCity`, `CanRebaseTo`, and `RebaseTo`; verify they fire only in disabled mode.

3. `EVENTS_CUSTOM_MISSIONS=1`: execute a custom mission ID with handlers returning `0`, `1`, `2`, and `3`; save mid-mission and reload; verify timer, target plot, completion, and mission deletion.

4. `EVENTS_CUSTOM_MISSIONS=1`: attach handlers that log mission IDs only, then run built-in move, rebase, nuke, ranged attack, build, and fortify missions to prove which custom hooks observe built-ins.

5. `EVENTS_RESOLUTIONS=1`: register handlers that veto enact/repeal, consume AI proposal, consume AI votes, and observe enact/repeal result; test human UI and AI World Congress sessions before and after save/load.

6. `EVENTS_RESOLUTIONS=1`: UI possible-proposal screen with a listener that asserts `iPlayer != -1`; verify current crash/error behavior or validate a guard patch.

7. `EVENTS_UNIT_CREATED=1`: create units by city production, purchase, direct Lua/player init, great person spawn, event reward, capture, gift, upgrade, trait conversion, and domain conversion; record whether listener sees final promotions, damage, moves, religion, origin city, and name.

Add diagnostic-only logging around each event dispatch with option state, owner/player, unit ID, mission ID, plot, listener return value, and whether built-in processing was skipped or continued.

Add a one-line warning when `CvLeague::CanProposeEnact` invokes or skips `PlayerCanPropose` with `NO_PLAYER`.

## Version differences, risks and unresolved questions

Local `git log -S'MOD_EVENTS_CUSTOM_MISSIONS'` finds production-source provenance at `54dee87` only.

INFERRED: Mission enum changes are savegame- and Lua-sensitive. This option does not itself add enum values, but custom mission IDs depend on stable assumptions outside this guarded block.

INFERRED: Incorrect listener return protocol can leave missions stuck, complete too early, or use invalid target plots.

Local `git log -S'MOD_EVENTS_REBASE'` finds production-source provenance at `54dee87` only.

No compile-time risk was found.

INFERRED: Mods listening to old raw Lua names may not fire when this option is enabled unless they migrate to GameEvents. Improvement rebase can bypass generic target events.

Local `git log -S'MOD_EVENTS_RESOLUTIONS'` finds production-source provenance at `54dee87` only.

INFERRED: Listener return values can suppress AI proposals/votes for a session if misused.

Local `git log -S'MOD_EVENTS_UNIT_CREATED'` finds production-source provenance at `54dee87` only.

INFERRED: Lua listeners may assume the unit is fully stable, but the hook fires during initialization immediately after core setup.

### Recovered finding: EI-1: `WH_MILITARY_LOG` does not cover nuclear combat

```text
CvUnitMission::ContinueMission(MISSION_NUKE)
  -> CvUnitCombat::AttackNuclear
  -> CvUnitCombat::GenerateNuclearCombatInfo
  -> CvUnitCombat::ResolveCombat
  -> CvUnitCombat::ResolveNuclearCombat
  -> CvUnitCombat::ApplyNuclearExplosionDamage
```

Possible regression risks: duplicate or overly verbose entries in high-unit-density late games; hidden-information leakage if logging ignores visibility/met-team checks.

Runtime validation needed: nuke city, nuke units outside a city, atomic bomb intercepted, missile partially intercepted, hidden/unrevealed target, human attacker, human defender, human observer.

### Recovered finding: EI-2: `CvLuaPlayer::lGetMilitaryLog` dereferences a null log

Intended behavior: a public Lua binding should be safe for unsupported players and disabled configurations.

Observed/static failure: calling the binding for an ineligible player or with the option disabled can dereference null.

Why exposed by current architecture: allocation/lifetime is option- and player-kind-dependent, but the Lua binding is always registered and does not mirror those guards.

Possible regression risks: external UI that used a crash as an implicit signal would now receive an empty table; that is acceptable.

Compatibility risk: `EVENTS_REBASE` has a source-verified bypass for friendly improvements with `AllowsRebaseTo`. `CvUnit::canRebaseAt` returns true for that DB-backed path at `CvGameCoreDLL_Expansion2/CvUnit.cpp:9921` through `CvUnit.cpp:9931`, before `GAMEEVENT_CanRebaseTo` at `CvUnit.cpp:9936`. This appears intentional VP architecture, but modmods expecting `CanRebaseTo` to observe every non-city/non-carrier target will miss airbase-style improvements.

Compatibility risk: `EVENTS_REBASE` changes old raw Lua fallback names into GameEvents only when enabled. Raw fallback calls remain in disabled branches at `CvGameCoreDLL_Expansion2/CvUnit.cpp:6486`, `CvUnit.cpp:9860`, `CvUnit.cpp:9941`, and `CvUnit.cpp:10042`. This is working as designed but is a migration hazard.

Compatibility risk: `EVENTS_CUSTOM_MISSIONS` is not limited to unknown mission IDs for every hook. `CustomMissionDoStep` is in the unhandled-mission `else if` at `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:687`, but `CustomMissionSetActivity`, `CustomMissionStart`, `CustomMissionTimerInc`, and `CustomMissionCompleted` can be called for built-in missions at `CvUnitMission.cpp:1451`, `CvUnitMission.cpp:1886`, `CvUnitMission.cpp:2035`, and `CvUnitMission.cpp:2327`. A listener that assumes only external mission IDs can accidentally alter built-in mission lifecycle.

Compatibility risk: `EVENTS_RESOLUTIONS` calls `GAMEEVENT_PlayerCanPropose` from `CvLeague::CanProposeEnact` regardless of whether `eProposer` is `NO_PLAYER` at `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:3138`. The repository UI uses `league:CanProposeEnactAnyChoice(resolutionType, kNoPlayer)` for possible-in-principle checks in `(2) Vox Populi/Core Files/Overrides/LeagueOverview.lua:299`. Lua listeners must handle `NO_PLAYER`, or the DLL should skip the hook for non-player probes.

Missing validation: `EVENTS_RESOLUTIONS` AI proposal/vote hooks consume the whole AI action if they return true at `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:9482` and `CvVotingClasses.cpp:9533`. Runtime tests must prove listeners actually spend all proposals/votes before returning true; otherwise the AI can skip a session.

Compatibility risk: `EVENTS_UNIT_CREATED` fires before caller-side post-init work. For upgrades, `CvUnit::DoUpgradeTo` creates the new unit at `CvGameCoreDLL_Expansion2/CvUnit.cpp:14179`, then calls `pNewUnit->convert(this, true)` at `CvUnit.cpp:14246`. The `UnitCreated` hook fires inside `initWithNameOffset` at `CvUnit.cpp:1253`, before `convert()` copies promotions, damage, moves, origin city, and other old-unit state. Capture and conversion paths using `initUnit` then `convert()` have the same timing shape.

Insufficient runtime evidence: No in-game runtime tests were executed in this environment.

Insufficient runtime evidence: No DLL build was run. This Linux workspace does not provide the Windows/MSVC Civ V DLL build environment referenced by `CvGameCoreDLL_Expansion2/VoxPopuli.vcxproj`.

2. Add bounded nuclear combat logging in `CvUnitCombat::ResolveNuclearCombat`, with visibility-safe entries and no per-tile unbounded spam.

3. Decide whether `EVENTS_REBASE` should document or expose the `AllowsRebaseTo` improvement path through `CanRebaseTo`; if changing behavior, call the event before the improvement return or add a dedicated improvement event.

4. Guard `GAMEEVENT_PlayerCanPropose` in `CvLeague::CanProposeEnact` for `eProposer == NO_PLAYER`, or document that listeners must accept `NO_PLAYER`.

5. For `EVENTS_UNIT_CREATED`, either document that the hook is pre-convert/pre-caller-finalization, or add a separate post-finalization hook for upgrade/capture/convert paths.

6. For `EVENTS_CUSTOM_MISSIONS`, document that lifecycle hooks can fire for built-in missions, or restrict start/activity/timer/completed hooks to mission IDs not handled by built-in branches.

`EVENTS_REBASE`: usable for controlled modmod testing; not fully ready for normal use unless the improvement-airbase bypass is documented or accepted.

`EVENTS_CUSTOM_MISSIONS`: usable for controlled external custom mission IDs; not ready for broad use without documenting or constraining built-in mission lifecycle hook interactions.

`EVENTS_RESOLUTIONS`: mostly ready, but `NO_PLAYER` proposal probes and AI consume semantics require validation.

`EVENTS_UNIT_CREATED`: active and reachable, but not ready for listeners that need final post-upgrade/capture/convert state unless documented as pre-finalization.

## Source and dependency lookup

Direct references are `CustomMods.h:784`, `CustomMods.h:1746`, `CustomMods.cpp:523`, `NewCustomModOptions.xml:893`, `CvUnitMission.cpp:687`, `CvUnitMission.cpp:810`, `CvUnitMission.cpp:918`, `CvUnitMission.cpp:1342`, `CvUnitMission.cpp:1451`, `CvUnitMission.cpp:1886`, `CvUnitMission.cpp:1958`, `CvUnitMission.cpp:2035`, and `CvUnitMission.cpp:2327`.

`CvUnitMission::ContinueMission` calls `GAMEEVENT_CustomMissionDoStep` at `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:687` and `GAMEEVENT_CustomMissionCompleted` at `CvUnitMission.cpp:810`.

`CvUnitMission::CanStartMission`/`CanDoMission` call `GAMEEVENT_CustomMissionPossible` at `CvUnitMission.cpp:1342`. `CvUnitMission::StartMission` calls `GAMEEVENT_CustomMissionSetActivity` at `CvUnitMission.cpp:1451` and `GAMEEVENT_CustomMissionStart` at `CvUnitMission.cpp:1886`.

Other hooks are `CustomMissionCameraTime` at `CvUnitMission.cpp:918`, `CustomMissionTargetPlot` at `CvUnitMission.cpp:1958`, `CustomMissionTimerInc` at `CvUnitMission.cpp:2035`, and `CustomMissionCompleted` at `CvUnitMission.cpp:2327`.

Event names are defined in `CustomMods.h:1239` through `CustomMods.h:1246`.

INFERRED: Custom mission IDs must not collide with built-in `MissionTypes` branches that are handled earlier.

INFERRED: Later hardcoded mission types added to `CvUnitMission` can supersede a previously custom mission ID by being handled before the custom-mission fallback.

Save during a custom mission with a nonzero timer, reload, and continue mission processing.

`COMPATIBILITY_RISK`

`CustomMissionDoStep` is only reached in the unhandled-mission branch of `CvUnitMission::ContinueMission` at `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:687`.

Other custom mission lifecycle hooks can fire for built-in mission IDs. `CustomMissionCompleted` is called after built-in completion checks at `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:810` and `CvUnitMission.cpp:2327`; `CustomMissionSetActivity` is called at `CvUnitMission.cpp:1451`; `CustomMissionStart` is called after built-in start handling at `CvUnitMission.cpp:1886`; `CustomMissionTimerInc` is called at `CvUnitMission.cpp:2035`.

Compatibility risk: the option name and comments imply custom mission support, but several lifecycle hooks are not restricted to custom or unknown mission IDs. A listener that returns action/done values for a built-in mission can alter built-in mission activity, deletion, timer, or completion behavior.

Missing validation: save/load of custom mission queues depends on stable mission integer IDs and mission data. This option does not add serialization fields, but external custom mission IDs remain savegame- and Lua-sensitive.

Smallest safe correction if built-in hook reach is unintended: gate lifecycle hooks behind an "unhandled mission" predicate or a documented custom mission ID range. Regression risk is that external mods may already use these hooks to observe built-in mission lifecycle.

Runtime scenario needed: register logging-only handlers and execute built-in move, rebase, nuke, ranged attack, build, fortify, and a true custom mission ID; then repeat with handlers returning each custom action code.

Direct references are `CustomMods.h:908`, `CustomMods.h:1767`, `CustomMods.cpp:544`, `NewCustomModOptions.xml:1022`, `CvUnit.cpp:6481`, `CvUnit.cpp:9858`, `CvUnit.cpp:9936`, and `CvUnit.cpp:10040`.

`CvUnit::canLoad(const CvPlot&)` calls `GAMEEVENT_CanLoadAt` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:6481` when enabled.

`CvUnit::canRebaseAt` calls `GAMEEVENT_CanRebaseInCity` for foreign-city cases at `CvUnit.cpp:9858` and `GAMEEVENT_CanRebaseTo` for empty/non-city/non-unit/non-improvement cases at `CvUnit.cpp:9936`.

`CvUnit::rebase` calls `GAMEEVENT_RebaseTo` at `CvUnit.cpp:10040`.

Event names are defined in `CustomMods.h`: `GAMEEVENT_CanLoadAt` at `CustomMods.h:1219`, `GAMEEVENT_CanRebaseInCity` at `CustomMods.h:1223`, `GAMEEVENT_CanRebaseTo` at `CustomMods.h:1224`, and `GAMEEVENT_RebaseTo` at `CustomMods.h:1317`.

INFERRED: Lua listeners must be registered against the GameEvents names when this option is enabled. Old raw Lua global functions remain relevant when it is disabled.

Raw Lua fallback functions are still called when the option is disabled at `CvUnit.cpp:6484`, `CvUnit.cpp:9865`, `CvUnit.cpp:9943`, and `CvUnit.cpp:10043`.

Audited source includes DB-backed improvement rebasing via `AllowsRebaseTo` in `(2) Vox Populi/Database Changes/Improvements/ImprovementTableChanges.sql:98` and `CvGameCoreDLL_Expansion2/CvUnit.cpp:9921`, before the generic event hook.

Save aircraft before rebase, reload, run the mission, and confirm event result and final unit plot.

The active mission path is `CvUnitMission::CanStartMission(MISSION_REBASE)` -> `CvUnit::canRebaseAt` at `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:1119`, and `CvUnitMission::StartMission(MISSION_REBASE)` -> `CvUnit::rebase` at `CvUnitMission.cpp:1557`.

`CvUnit::canLoad` dispatches `GAMEEVENT_CanLoadAt` when enabled at `CvGameCoreDLL_Expansion2/CvUnit.cpp:6481`; otherwise it calls the raw Lua fallback name `CanLoadAt` beginning at `CvUnit.cpp:6486`.

`CvUnit::canRebaseAt` dispatches `GAMEEVENT_CanRebaseInCity` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:9858` and `GAMEEVENT_CanRebaseTo` at `CvUnit.cpp:9936`; disabled behavior uses raw Lua fallback names at `CvUnit.cpp:9860` and `CvUnit.cpp:9941`.

`CvUnit::rebase` dispatches `GAMEEVENT_RebaseTo` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:10040`; disabled behavior uses raw Lua fallback name `RebaseTo` at `CvUnit.cpp:10042`.

Compatibility risk: friendly improvements with `AllowsRebaseTo` return true before `GAMEEVENT_CanRebaseTo`. The bypass is at `CvGameCoreDLL_Expansion2/CvUnit.cpp:9921` through `CvUnit.cpp:9931`; `GAMEEVENT_CanRebaseTo` is later at `CvUnit.cpp:9936`. This appears to be current VP source behavior, not a stale guide claim.

Smallest safe correction if this is unintended: move or duplicate a `CanRebaseTo` observation point before the `AllowsRebaseTo` return, while preserving the existing capacity/enemy checks. Regression risk is that old airbase mods may receive a new callback they did not expect.

Runtime scenario needed: aircraft rebasing to own city, foreign city accepted by event, carrier, `AllowsRebaseTo` improvement, empty plot accepted by event, and over-capacity target with `EVENTS_REBASE=0` and `EVENTS_REBASE=1`.

Direct references are `CustomMods.h:930`, `CustomMods.h:1775`, `CustomMods.cpp:552`, `NewCustomModOptions.xml:1042`, `CvVotingClasses.cpp:3138`, `CvVotingClasses.cpp:3186`, `CvVotingClasses.cpp:8664`, `CvVotingClasses.cpp:8702`, `CvVotingClasses.cpp:9482`, and `CvVotingClasses.cpp:9533`.

`CvLeague::CanProposeEnact` calls `GAMEEVENT_PlayerCanPropose` at `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:3138`; `CvLeague::CanProposeRepeal` calls it at `CvVotingClasses.cpp:3186`.

`CvLeague::NotifyProposalResult` calls `GAMEEVENT_ResolutionResult` for enact and repeal result paths at `CvVotingClasses.cpp:8664` and `CvVotingClasses.cpp:8702`.

League AI proposal and voting processing can be consumed by `GAMEEVENT_ResolutionProposing` and `GAMEEVENT_ResolutionVoting` at `CvVotingClasses.cpp:9482` and `CvVotingClasses.cpp:9533`.

Event names are defined in `CustomMods.h:1301`, `CustomMods.h:1322`, `CustomMods.h:1323`, and `CustomMods.h:1324`.

INFERRED: Lua listeners must understand the enact/repeal boolean argument and the resolution ID/type distinction in proposal checks.

INFERRED: Direct manipulation of league state outside `CvLeague` proposal/vote methods would bypass these hooks. No such authoritative replacement was identified in this pass.

Save during a league session before proposal/vote/result resolution, reload, and confirm callbacks and state changes remain consistent.

`ACTIVE_AND_REACHABLE_WITH_COMPATIBILITY_RISK`

`CvLeague::CanProposeEnact` invokes `GAMEEVENT_PlayerCanPropose` at `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:3138`, after built-in validity checks. The repository UI calls `league:CanProposeEnactAnyChoice(resolutionType, kNoPlayer)` for possible-in-principle checks in `(2) Vox Populi/Core Files/Overrides/LeagueOverview.lua:299`.

Compatibility risk: `PlayerCanPropose` listeners can receive `iPlayer == NO_PLAYER` from UI or other "possible" probes, because `CanProposeEnact` does not skip the hook for `eProposer == NO_PLAYER`. Listener code must handle that, or the DLL should guard this dispatch.

`CvGameLeagues::DoPlayerTurn` treats a true `ResolutionProposing` result as "all proposals used" at `CvGameCoreDLL_Expansion2/CvVotingClasses.cpp:9482`, and a true `ResolutionVoting` result as "all votes used" at `CvVotingClasses.cpp:9533`.

Missing validation: runtime tests must prove event listeners actually spend or intentionally suppress all AI proposals/votes before returning true; otherwise the AI can skip proposal or voting work for the session.

Smallest safe correction: skip `GAMEEVENT_PlayerCanPropose` when `eProposer == NO_PLAYER`, or document the current `NO_PLAYER` probe contract in the option comment. Regression risk is external listeners that intentionally use `NO_PLAYER` probes to hide proposals from the UI.

Runtime scenario needed: human UI proposal list, AI proposal phase, AI voting phase, enact result, repeal result, and a listener that logs or rejects `NO_PLAYER` proposal probes.

Direct references are `CustomMods.h:974`, `CustomMods.h:1784`, `CustomMods.cpp:561`, `NewCustomModOptions.xml:1087`, and `CvUnit.cpp:1253`.

`CvUnit::initWithNameOffset` invokes `GAMEEVENT_UnitCreated(getOwner(), GetID(), getUnitType(), getX(), getY())` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1253`, after core unit initialization, graphical setup, and water safety handling.

`GAMEEVENT_UnitCreated` is defined in `CvGameCoreDLL_Expansion2/CustomMods.h:1345`.

INFERRED: The feature assumes all desired creation mechanisms pass through `CvUnit::initWithNameOffset`.

Separate event-reward hooks named `GAMEEVENT_EventUnitCreated` exist in `CvGameCoreDLL_Expansion2/CvPlayer.cpp:8322`, `CvPlayer.cpp:8451`, `CvGameCoreDLL_Expansion2/CvCity.cpp:7068`, and `CvCity.cpp:7098`. These are semantically different from `MOD_EVENTS_UNIT_CREATED`.

INFERRED: Unit capture, conversion, upgrade, clone, or revive paths need runtime validation to prove they route through `initWithNameOffset`.

Save immediately after listener-side state is set by the event, reload, and verify external state persistence.

`ACTIVE_AND_REACHABLE_WITH_TIMING_RISK`

`UnitCreated` fires inside `CvUnit::initWithNameOffset` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1253`, after most unit initialization and optional graphical setup, but before caller-side post-init work in `CvPlayer::initUnit` returns at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:9120`.

Upgrade creation calls `CvPlayer::initUnit` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:14179`, then later calls `pNewUnit->convert(this, true)` at `CvUnit.cpp:14246`. `CvUnit::convert` copies promotions, damage, moves, origin city, and other old-unit state beginning at `CvUnit.cpp:1918`, after the `UnitCreated` hook has already fired.

Compatibility risk: listeners see a valid newly allocated unit, but not necessarily the final post-upgrade, post-capture, post-gift, or post-conversion state. This affects promotions, damage, moves, religion copied after capture, origin city, and other state transferred by `convert()` or caller-specific logic.

Smallest safe correction if final state is required: add a separate post-finalization event for upgrade/capture/convert paths, or move/create event dispatch after `convert()` in those paths while preserving a creation-source argument. Regression risk is changing event timing for existing listeners that rely on early setup.

Runtime scenario needed: unit creation by production, purchase, direct `initUnit`, great person spawn, event reward, capture, gift, upgrade, trait conversion, and domain conversion; listener should record state at callback and compare it to final state after the call returns.

Subsystem: Event infrastructure

All five options are runtime `CustomModOptions` switches defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml`, cached by `CustomMods::preloadCache` in `CvGameCoreDLL_Expansion2/CustomMods.cpp`, and wrapped by `MOD_*` macros in `CvGameCoreDLL_Expansion2/CustomMods.h`.

Activation rows and macros are:

| Feature | Row | Macro | Cache |
| --- | --- | --- | --- |
| `EVENTS_CUSTOM_MISSIONS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:893` | `CvGameCoreDLL_Expansion2/CustomMods.h:784` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:523` |
| `EVENTS_REBASE` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1022` | `CvGameCoreDLL_Expansion2/CustomMods.h:908` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:544` |
| `EVENTS_RESOLUTIONS` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1042` | `CvGameCoreDLL_Expansion2/CustomMods.h:930` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:552` |
| `EVENTS_UNIT_CREATED` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1087` | `CvGameCoreDLL_Expansion2/CustomMods.h:974` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:561` |

`EVENTS_REBASE` changes rebase/load extension dispatch in authoritative unit paths:

```text
CvUnitMission::CanStartMission(MISSION_REBASE)
  -> CvUnit::canRebaseAt
  -> GAMEEVENT_CanRebaseInCity / GAMEEVENT_CanRebaseTo

CvUnitMission::StartMission(MISSION_REBASE)
  -> CvUnit::rebase
  -> GAMEEVENT_RebaseTo

CvUnit::canLoad
  -> GAMEEVENT_CanLoadAt
```

`EVENTS_CUSTOM_MISSIONS` hooks the authoritative mission queue manager `CvUnitMission`, including `ContinueMission`, `CanStartMission`, `StartMission`, target plot, timer, and completion checks at `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:687`, `CvUnitMission.cpp:810`, `CvUnitMission.cpp:1342`, `CvUnitMission.cpp:1451`, `CvUnitMission.cpp:1886`, `CvUnitMission.cpp:1958`, `CvUnitMission.cpp:2035`, and `CvUnitMission.cpp:2327`.

`EVENTS_RESOLUTIONS` hooks authoritative World Congress proposal, result, and AI session paths in `CvLeague`: `CanProposeEnact`, `CanProposeRepeal`, both `NotifyProposalResult` overloads, and `CvGameLeagues::DoPlayerTurn`.

`EVENTS_UNIT_CREATED` fires from `CvUnit::initWithNameOffset` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:1253`. Normal unit creation through `CvPlayer::initUnit` and `CvPlayer::initUnitWithNameOffset` reaches this function at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:9120` and `CvPlayer.cpp:9144`.

Missing coverage includes nuclear combat (`CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:2238`, `CvUnitCombat.cpp:2880`, `CvUnitCombat.cpp:4118`) and likely lacks structured coverage for every affected unit/city in a nuclear blast.

INFERRED: Naval-to-land and land-to-naval attacks route through the same `CvUnitCombat::Attack`, `AttackRanged`, `AttackCity`, or `AttackAir` entry points depending on unit/city/air/ranged mode, so they are covered if the relevant resolution function logs the visible result. Runtime scenarios are still required for embarked targets, ships attacking cities, coastal city melee, carrier aircraft, submarines, and invisible attacker cases.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `EVENTS_CUSTOM_MISSIONS` | Register custom mission handlers returning each documented action code. | Save during multi-step custom mission; reload and continue mission. | `CvUnitMission::CanStartMission`, `StartMission`, `ContinueMission`, `IsMissionDone`. | `CvUnitMission.cpp:687`, `CvUnitMission.cpp:1451`, `CvUnitMission.cpp:2327` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `EVENTS_REBASE=1` GameEvents path | Register all four rebase GameEvents; test own city, foreign city accepted by event, carrier, empty plot accepted by event, and invalid target. | Save aircraft before each rebase; reload and execute mission. | GameEvents path should determine eligibility where no built-in city/carrier/improvement rule already decides. | Source path at `CvUnit.cpp:6481`, `CvUnit.cpp:9858`, `CvUnit.cpp:9936`, and `CvUnit.cpp:10040`. |
| `EVENTS_CUSTOM_MISSIONS=1` custom ID | Register custom mission handlers returning `0`, `1`, `2`, and `3`; execute a multi-step custom mission. | Save with queued custom mission and nonzero timer; reload and continue. | Custom mission should start, continue, set timer/target, complete, and delete according to listener protocol. | Main path at `CvUnitMission.cpp:687`, `CvUnitMission.cpp:1342`, `CvUnitMission.cpp:1886`, and `CvUnitMission.cpp:2327`. |
| `EVENTS_CUSTOM_MISSIONS=1` built-in mission observation | Register logging-only handlers and execute built-in move, rebase, nuke, ranged attack, build, and fortify missions. | Save before each built-in mission; reload and execute. | Document exactly which custom hooks fire for built-ins; no handler should accidentally alter built-in state. | Compatibility risk at `CvUnitMission.cpp:1451`, `CvUnitMission.cpp:1886`, `CvUnitMission.cpp:2035`, and `CvUnitMission.cpp:2327`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
