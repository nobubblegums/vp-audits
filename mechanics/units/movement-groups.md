# Linked movement, quick groups and Squads

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How do linked units and grouped movement validate and execute a step?

The base audit, integration proposal and later implementation notes describe distinct revisions. The latter reports synchronized link commands, atomic follower validation, stale-ID handling and a separate QUICK_GROUP_MOVEMENT option; it does not establish present-source or runtime correctness.

## Revision reconciliation

The early feature/group audits report failed-move state mutation, follower missions queued on a stale iteration unit, null ideal plots and unsafe linked IDs. The integration report proposes alternative relationships with Squads. The later implementation note reports synchronized Link/Unlink commands, atomic step validation, stale-ID cleanup, a rigid all-or-nothing quick-group path and separate QUICK_GROUP_MOVEMENT gating. Keep proposals, pre-repair defects and reported implementation changes separate.

The latest recovered notes still require runtime ownership-transfer/capture/gift/conversion, embarkation, blocked-member, Squads coexistence and multiplayer checks. They report no DLL build or Lua bytecode check. Earlier readiness classifications cannot prove that the later implementation was compiled or tested.

Canonical related owners: [Optional event and mission dispatch](<../../architecture/event-dispatch.md>), [Attrition and Heavy Charge displacement](<attrition.md>), [Optional feature activation and dependency map](<../../architecture/optional-features.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Reported revision changes

- Standalone Link/Unlink is restored in EUI and non-EUI UnitPanel resources.

- UnitPanel Lua is based on current VP Lua, with only the Link/Unlink button logic carried forward.

- Linked followers can route movement through their linked leader from WorldView when `LINKED_MOVEMENT` is enabled.

- Linked leader movement validates all followers before embark/disembark, `setXY`, or movement-cost mutation. Blocked valid followers cancel the step without unlinking; invalid linked IDs clear link state instead of being dereferenced.

- Worker-first linked leader semantics are preserved for wake/order prompting, but mixed civilian/military linked stacks move non-civilian followers before a civilian leader so escorts enter the target plot first.

- Retreat/knockback fallback now keeps valid linked stacks on the same fallback plot, cancelling the fallback if a linked member cannot legally use the chosen retreat plot.

- Forced fallback movement now fires embark/disembark callbacks before `setXY` when the destination crosses an embarkation boundary, avoiding stale embarked visuals after retreat relocations.

- `CvUnit::LinkUnits` now queues fortify/sleep on the current follower unit, not the stale plot-iteration unit.

- Quick Group Movement is separated from `LINKED_MOVEMENT` behind `QUICK_GROUP_MOVEMENT`.

- Quick Group Movement is rigid and all-or-nothing: it records each member's offset from the selected unit and cancels without mutation if any ideal destination is invalid or blocked.

- Added `CvUnit::GetLinkedLeaderUnit()`.

- Added `CvUnit::TryCollectLinkedFollowers(...)`.

- Reused these helpers in kill cleanup, `SetIsLinkedLeader(false)`, `UnlinkUnits`, `MoveLinkedLeader`, linked leader movement, and Squads end cleanup.

- No save layout change.

- Stale linked IDs are repaired at use sites covered in this pass: kill cleanup, unlink, linked follower leader movement, linked leader movement, and Squads end cleanup.

- Ownership transfer, capture, gifting, conversion, teleport, airlift, and rebase still need runtime validation. Current policy is to clear invalid links when encountered rather than preserving unsafe references.

## Trigger and ownership

Linking:

```text
UnitPanel Link button
  -> UnitPanel.lua OnLinkUnitsClicked
  -> CvLuaUnit::lLinkUnits
  -> CvUnit::LinkUnits
  -> same-plot eligible units collected
  -> leader selected (worker first, else ordering unit)
  -> member flags, leader ID, follower IDs, linked max moves stored
  -> follower sleep/fortify missions attempted
```

Linked movement:

```text
Leader MISSION_MOVE_TO or follower MoveLinkedLeader
  -> CvUnitMission::ContinueMission
  -> CvUnit::UnitPathTo / UnitMove
  -> CvUnit::move
  -> if linked leader:
       validate each follower canMoveInto(next target)
       leader setXY(next target)
       follower PushMission(MISSION_MOVE_TO, same target, KEEP_LINK)
     else:
       ordinary setXY
```

Squad assignment:

```text
Squads panel / hotkey management mode
  -> WorldView interface-mode LuaEvents
  -> Squads.lua HandleLeftClickUp / DoBoxSelect / unit-list removal
  -> CvLuaUnit::lAssignToSquad or lRemoveFromSquad
  -> CvUnit::AssignToSquad / RemoveFromSquad
  -> per-unit m_iSquadNumber serialized
```

Squad movement:

```text
Squads movement mode RCLICK
  -> Squads.lua HandleMoveSquad
  -> first unit in player:UnitsInSquad
  -> CvLuaUnit::lDoSquadMovement(destination, escort option)
  -> CvUnit::DoSquadPlotAssignments
       partition stacking/land/sea members
       optionally LinkUnits for escorts
       assign unique destination plots near target and center of mass
  -> CvUnit::DoSquadMovement
       SetSquadDestination for squad
       sendPushMission for each unlinked unit or linked leader
  -> CvUnitMission/CvUnit movement, reroute, TryEndSquadMovement
```

## Application and state

Linked state is stored on `CvUnit` as `m_bIsLinked`, `m_bIsLinkedLeader`, `m_bIsGrouped`, `m_iLinkedMaxMoves`, `m_LinkedUnitIDs`, and `m_iLinkedLeaderID` in `CvGameCoreDLL_Expansion2/CvUnit.h:2320`.

State is initialized in the constructor/reset path at `CvGameCoreDLL_Expansion2/CvUnit.cpp:145` and `CvUnit.cpp:1295`.

State is serialized in `CvUnit::Serialize` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28429` through `CvUnit.cpp:28434` and in the sync archive at `CvGameCoreDLL_Expansion2/CvUnit.h:2766` through `CvUnit.h:2771`.

MISSING_VALIDATION: `CvUnit::read` calls `reset()` then `Serialize(...)` and reattaches map layer at `CvGameCoreDLL_Expansion2/CvUnit.cpp:28845`; no source path validates linked IDs after load or after option changes. Consumers dereference saved IDs without null checks in `CvUnit.cpp:2587`, `CvUnit.cpp:5494`, `CvUnit.cpp:14690`, `CvUnit.cpp:14900`, `CvUnit.cpp:14911`, and `CvUnit.cpp:15481`.

SAVEGAME_SENSITIVE: Linked state is serialized unconditionally, not only when `LINKED_MOVEMENT` or `SQUADS` is enabled. Toggling `LINKED_MOVEMENT` or `SQUADS` across saves can therefore leave dormant state that becomes active later.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1140` with default `Value="0"`. Macro is `MOD_LINKED_MOVEMENT` in `CvGameCoreDLL_Expansion2/CustomMods.h:1022`; declaration is `CustomMods.h:1796`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:573`.

`MOD_SQUADS` is a separate option defined as `gCustomMods.isSQUADS()` in `CvGameCoreDLL_Expansion2/CustomMods.h:58`, cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:289`, and defaulted off in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:51`. It reuses linked-movement state and helpers.

`LINKED_MOVEMENT=1` activates guarded branches in `CvUnit::doTurn`, `CvUnit::move`, `CvUnit::maxMoves`, `CvUnit::ReadyToMove`, `CvUnit::CanDoInterfaceMode`, `CvUnitMission::PushMission`, `CvUnitMission::ContinueMission`, and `CvPlayer::EndTurnsForReadyUnits`.

Linked methods are compiled and Lua-exposed regardless of runtime option: `CvGameCoreDLL_Expansion2/CvUnit.h:844`, `CvGameCoreDLL_Expansion2/Lua/CvLuaUnit.cpp:195`, and wrappers at `CvLuaUnit.cpp:2554` through `CvLuaUnit.cpp:2626`. The helper methods themselves do not check `MOD_LINKED_MOVEMENT` before changing linked state.

COMPATIBILITY_RISK: Because Lua can call `CanLinkUnits`, `LinkUnits`, `UnlinkUnits`, `MoveLinkedLeader`, and `DoGroupMovement` while `LINKED_MOVEMENT=0`, the option does not fully own its behavior surface. In the audited tree this is mostly dormant because repository Lua search found no linked-movement UI that calls those helpers.

| `LINKED_MOVEMENT` | `SQUADS` | Expected DLL behavior | Audit classification |
| --- | --- | --- | --- |
| 0 | 0 | No guarded linked/grouped behavior should run. Lua helpers are still exposed and can mutate state if external Lua calls them. | Compatibility risk |
| 1 | 0 | Linked state branches are active. Normal repository UI reachability is unproven; helper defects affect direct Lua/modmod use. | Likely defect / insufficient runtime evidence |
| 0 | 1 | Squads use linked state and helpers; `LINKED_MOVEMENT` end-turn and ready-state branches remain off. | Compatibility risk for shared helper defects |
| 1 | 1 | Both features share linked state. Followers can be both linked-movement and squad-managed; shared failure modes apply. | Compatibility risk / missing validation |
| toggled across save | any | Serialized linked state can become dormant or active after option change without validation. | Missing validation |

| Configuration | Validity | Expected DLL behavior | Audit result |
| --- | --- | --- | --- |
| `LINKED_MOVEMENT=0`, `SQUADS=0` | Supported default | No guarded linked movement branches execute. Lua helper methods remain callable but no repository UI calls them. | Working as intended for default; compatibility risk for external Lua |
| `LINKED_MOVEMENT=1`, `SQUADS=0` | Intended optional feature | Linked followers should suppress orders, share max movement, and follow linked leader movement. | Not ready: confirmed defects and missing UI/runtime validation |
| `LINKED_MOVEMENT=0`, `SQUADS=1` | Supported if Squads mod enabled | Squads can reuse linked state/helpers without linked movement option. | Compatibility risk: shared helper defects affect squads |
| `LINKED_MOVEMENT=1`, `SQUADS=1` | Technically allowed, risky | Both features share linked state; followers can be affected by both linked and squad rules. | Compatibility risk; requires runtime validation before support |
| Option toggled across save/load with linked state present | Unsupported/risky | DLL should clear or validate stale linked IDs before use. | Missing validation |
| External Lua calls linked helpers while `LINKED_MOVEMENT=0` | Unsupported/risky | Ideally no state mutation, or explicit documented support. | Compatibility risk |

- `LINKED_MOVEMENT` remains the standalone Link Units feature and is enabled by the modmod SQL.

- `QUICK_GROUP_MOVEMENT` is inserted by modmod SQL with value `0`.

- `INTERFACEMODE_MOVE_TO_ALL` is only handled by copied WorldView resources when `QUICK_GROUP_MOVEMENT=1`, and DLL interface availability is gated by `MOD_QUICK_GROUP_MOVEMENT`.

## Observability and validation obligations

Add diagnostic-only logging or debug assertions around `LinkUnits`, `UnlinkUnits`, `MoveLinkedLeader`, `DoGroupMovement`, `CvUnit::move` linked failure, and post-load linked ID validation. Log unit owner/ID, leader ID, follower IDs, target plot, current plot, `MOD_LINKED_MOVEMENT`, `MOD_SQUADS`, and whether each referenced unit resolved.

| Scenario | Required checks |
| --- | --- |
| Normal escort movement | Link military+civilian on same plot; move leader several tiles; verify same destination, moves, wake state. |
| Blocked escort | Leader can enter next tile, follower cannot; verify no state mutation if policy is all-or-nothing. |
| Blocked civilian | Civilian blocked by danger/stacking/domain; verify selected policy and UI feedback. |
| Unequal movement points | Fast escort + slow civilian and reverse; verify lowest-budget or approved policy. |
| Embarkation | Link land unit/civilian before embark, while embarked, and during disembark; verify legality and no pre-failure embark mutation. |
| Zones of control | Move linked combat/civilian around enemy ZOC; verify path legality and movement budget. |
| Combat and attacks | Attack with linked escort/leader; verify whether link breaks or persists per approved policy. |
| Capture and death | Kill/capture linked leader and follower; verify no stale ID crash and intended survivor state. |
| Upgrade and conversion | Upgrade linked military, convert/capture/gift linked unit; verify preservation or cleanup policy. |
| Save/load | Save linked group before move, mid-route, after member loss, with option toggles; verify validation. |
| Simultaneous Squads and linked use | Unit in squad, linked escort, squad movement with escort option, manual link command; verify no double movement. |
| Multiplayer determinism | Verify all human movement/link actions use network-safe paths or are disabled in MP if not deterministic. |

1. `LINKED_MOVEMENT=1`, `SQUADS=0`: link two same-domain combat units, attempt a leader move where the follower cannot enter the destination. Verify no unit changes plot, leader moves are unchanged, and embark state is unchanged.

2. `LINKED_MOVEMENT=1`, `SQUADS=0`: link three units on one plot. Verify each non-leader receives sleep/fortify and no unrelated stack member receives repeated missions.

3. `LINKED_MOVEMENT=1`, `SQUADS=0`: run `DoGroupMovement` at map edge where a relative destination projects off-map. Verify fallback behavior and no crash.

4. Save linked units, reload, kill/disband/capture one member, then move/unlink/end turn with the survivor. Verify no null dereference and state clears.

5. Use the normal unit action/UI path for `INTERFACEMODE_MOVE_TO_ALL`. Verify whether the action appears, whether it links/groups units, and whether more than the selected head unit moves.

6. Repeat tests 1 through 4 with `SQUADS=1` and `LINKED_MOVEMENT=0`, then with both enabled.

Add diagnostic-only logging/assertions around `CvUnit::LinkUnits`, `CvUnit::UnlinkUnits`, `CvUnit::MoveLinkedLeader`, `CvUnit::DoGroupMovement`, `CvUnit::move` linked validation failure, `CvUnit::read` post-load linked state, and `CvPlayer::EndTurnsForReadyUnits`.

Log owner, unit ID, plot, target plot, linked leader ID, linked follower IDs, resolved pointer state, `MOD_LINKED_MOVEMENT`, `MOD_SQUADS`, move cost, and embark-state changes.

- Static inspection with `rg` for linked dereference sites, option gating, UnitPanel button wiring, and Quick Group Movement hooks.

- Static inspection confirmed UnitPanel no longer directly calls `unit:LinkUnits()` or `unit:UnlinkUnits()`.

- Static inspection confirmed `COMMAND_LINK_UNITS` / `COMMAND_UNLINK_UNITS` route through `GAMEMESSAGE_DO_COMMAND` and `CvUnit::doCommand`.

- `git diff --stat` review.

- `git diff --check` passed; Git reported LF-to-CRLF warnings only.

- No DLL build was run; this environment does not provide the Windows build toolchain.

- No Lua bytecode check was run; `lua` and `luac` are not installed.

## Version differences, risks and unresolved questions

### Recovered finding: Confirmed defect: failed linked leader move still mutates leader state

`CvUnit::move` changes embark/disembark state before linked followers are validated at `CvGameCoreDLL_Expansion2/CvUnit.cpp:5467` through `CvUnit.cpp:5484`. It then checks each linked follower at `CvUnit.cpp:5487` through `CvUnit.cpp:5502`. If any follower cannot enter the target plot, `bCanDoLinkedMove` becomes false and no unit is moved, but movement cost is still subtracted at `CvUnit.cpp:5521`.

Current call path: `WorldView.lua` or other mission caller -> `CvGame::selectionListGameNetMessage` -> `CvDllNetMessageHandler::ResponsePushMission` -> `CvUnit::PushMission` -> `CvUnitMission::ContinueMission` -> `CvUnit::UnitPathTo` / `CvUnit::UnitMove` -> `CvUnit::move`.

Intended behavior: linked movement should be atomic for the leader and followers for a single step, or should cancel without changing leader state.

Observed/static failure: when a follower blocks the linked step, the leader can lose moves without moving. If the attempted target crosses embark state, the leader may also embark/disembark before the failure.

Why current architecture exposes it: current `CvUnit::move` performs embark-state mutation and final cost accounting outside the linked all-or-nothing success branch.

Smallest safe correction: validate linked follower pointers and `canMoveInto` before any embark/disembark or cost mutation, and return/cancel the move when the linked move cannot be performed.

Regression risks: squad movement also uses the same branch under `MOD_SQUADS`; changing failure behavior may alter squad rerouting and "continue to closest plot" handling.

Runtime validation: link two land units, attempt to move the leader into a tile the leader can enter but a follower cannot, then confirm leader plot, embark state, and moves are unchanged.

### Recovered finding: Confirmed defect: `LinkUnits` queues follower sleep/fortify on the wrong unit

`CvUnit::LinkUnits` iterates `v_unitvector` with local `pUnit`, but the follower mission block uses stale loop variable `pLoopUnit` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:14875` through `CvUnit.cpp:14879`.

Current call path: `CvLuaUnit::lLinkUnits` -> `CvUnit::LinkUnits`; or `CvLuaUnit::lDoGroupMovement` -> `CvUnit::DoGroupMovement` -> `CvUnit::LinkUnits`; or squad escort setup at `CvGameCoreDLL_Expansion2/CvUnit.cpp:15306`.

Intended behavior: each non-leader linked follower should be sent fortify/sleep so it stops asking for direct orders.

Observed/static failure: the last plot-iteration unit receives repeated fortify/sleep pushes; the current follower `pUnit` may receive none.

Why current architecture exposes it: the helper mixes two different iteration variables after collecting the vector. No later code normalizes follower mission queues.

Smallest safe correction: replace `pLoopUnit` with `pUnit` in the `canFortify` and `PushMission` calls.

Regression risks: this changes follower activity immediately after linking; test worker-leader priority and mixed combat/civilian stacks.

Runtime validation: link three same-domain units on a plot, inspect each unit's linked flags and head mission/activity after `LinkUnits`.

### Recovered finding: Confirmed defect: `DoGroupMovement` can dereference an off-map ideal plot

`CvUnit::DoGroupMovement` computes `pFirstTargetPlot` with `GC.getMap().plot(...)` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:14970`, then immediately dereferences it in `pUnit->canMoveInto(*pFirstTargetPlot, ...)` at `CvUnit.cpp:14976`.

Current call path: `CvLuaUnit::lDoGroupMovement` -> `CvUnit::DoGroupMovement`.

Intended behavior: if the ideal relative destination is invalid, fall back to approximate ring movement or exclude that unit from the group.

Observed/static failure: near map edges or with invalid Lua target input, `pFirstTargetPlot` can be null and the helper can crash.

Why current architecture exposes it: `DoGroupMovement` is Lua-exposed but performs no target validation beyond `pCurrentPlot`.

Smallest safe correction: null-check `pDestPlot` and `pFirstTargetPlot`; only test the ideal plot when non-null, otherwise try ring fallbacks.

Regression risks: edge-map grouped movement may choose fallback plots more often than before.

Runtime validation: call group movement from a unit at the map edge where at least one adjacent unit's ideal offset projects outside the map.

### Recovered finding: Likely defect: stale linked IDs can crash cleanup and movement

Linked ID consumers dereference `GET_PLAYER(...).getUnit(id)` without null checks in kill cleanup (`CvGameCoreDLL_Expansion2/CvUnit.cpp:2587`), leader unlink (`CvUnit.cpp:14690`), follower unlink (`CvUnit.cpp:14900`), linked leader movement (`CvUnit.cpp:14911`), linked leader step execution (`CvUnit.cpp:5494`), and squad end handling (`CvUnit.cpp:15481`).

Current call path: save/load or unit death/ownership transition leaves stale linked IDs -> any unlink, move, kill, or squad end path reads the ID -> null dereference.

Intended behavior: invalid linked references should be ignored and the remaining linked state should be cleared.

Observed/static failure: no validation prevents dereferencing null unit pointers.

Why current architecture exposes it: linked references are raw per-owner unit IDs stored in the unit save archive; there is no load-time repair pass or central link registry.

Smallest safe correction: add null checks at consumers and clear the local linked state when the leader/follower cannot be found; optionally add a post-load validation helper.

Regression risks: silent unlinking may mask data corruption; logging/assertions should accompany the repair in debug builds.

Runtime validation: save linked units, remove or kill one member through combat/disband/capture after reload, then move/unlink/end turn with the remaining member.

`LINKED_MOVEMENT` is not ready for normal use from the audited repository UI. The helper code is active and partially authoritative, but normal UI reachability is not verified, and there are confirmed state-mutation and null/target validation defects.

### Option B: Separate Links Using The Squads Movement Engine

Recommended.

| Factor | Assessment |
| --- | --- |
| Files affected | Likely `CvUnit.h`, `CvUnit.cpp`, `CvUnitMission.cpp`, `CvPlayer.cpp`, `CvLuaUnit.cpp`, `NewCustomModOptions.xml` only if option policy changes, plus a small resource/UI file for link button if bundled. |
| Expected code volume | Medium: harden links first, then extract a small movement-planning helper from Squads if needed. |
| Reused code | Squads mission/reroute/end handling and assignment concepts; existing linked fields for save compatibility. |
| Duplicated logic | Some membership logic remains separate by design. |
| Regression risk | Moderate; shared `CvUnit::move` and `CvUnitMission` paths affect Squads. |
| Save-format impact | Can avoid new fields initially by using existing linked serialization. Version change may not be required if no fields/enums are added. |
| Impact on Squads | Low-to-moderate if helper extraction preserves current call behavior. |
| Maintainability | Best near-term balance: linked units stay a small feature, movement safety comes from Squads path. |
| Migration path | Harden old linked state, then route movement through extracted squad-like planning/mission helpers. |
| Runtime testing burden | High for movement/lifecycle, but smaller than a generic backend rewrite. |
| Files affected | `CvUnit` squad/link fields, Squads UI Lua, possibly DB/text/options. |
| Expected code volume | Medium if hidden squad type is added; low only if existing public squads are reused. |
| Reused code | Maximum direct reuse of squad number, movement assignment, `DoSquadMovement`. |
| Duplicated logic | Low, but at cost of semantic coupling. |
| Regression risk | Moderate/high: squad UI may expose or mutate linked escort groups. |
| Save-format impact | Existing squad number cannot distinguish ordinary squad from link without extra flag or reserved squad range. |
| Impact on Squads | High unless hidden group type is carefully isolated. |
| Maintainability | Good only if Squads maintainers accept link-as-squad semantics. |
| Migration path | Create hidden/reserved squad subtype or extra group type flag. |
| Runtime testing burden | High, especially overlap with existing squads and UI. |
| Files affected | Broad: `CvUnit`, possible new group manager, serialization, Lua APIs, Squads UI, linked UI. |
| Expected code volume | High. |
| Reused code | Existing code becomes frontends over new backend after extraction. |
| Duplicated logic | Lowest long-term. |
| Regression risk | High due to save, mission, UI, and multiplayer surface. |
| Save-format impact | Likely serialization-version change or compatibility adapter. |
| Impact on Squads | High. |
| Maintainability | Architecturally cleanest but excessive for restoring linked escorts. |
| Migration path | Requires phased compatibility bridge for existing linked and squad fields. |
| Runtime testing burden | Very high. |
| Files affected | Mostly `CvUnit.cpp`, `CvUnitMission.cpp`, `CvPlayer.cpp`, old/new UI. |
| Expected code volume | Low/medium. |
| Reused code | Little beyond selected helpers and null checks. |
| Duplicated logic | High; old group/linked movement remains parallel to Squads. |
| Regression risk | Moderate in shared linked branch; old logic remains fragile. |
| Save-format impact | Can avoid new fields. |
| Impact on Squads | Still risky because Squads already calls linked helpers. |
| Maintainability | Weak; preserves duplicate systems and old assumptions. |
| Migration path | Patch bugs, re-enable old UI. |
| Runtime testing burden | High because historic glitches remain likely. |

Rationale: the desired feature is same-plot escort linking, not full squad management. Keeping link membership separate avoids forcing hidden squad semantics onto the player, while using Squads movement concepts avoids preserving old unsafe movement logic.

### Option A: Linked Units As A Specialized Squad

Concern: civilians fit Squads as stacking units, but the ordinary Squads UI assumes named persistent control groups. A hidden link needs a distinct type/flag to prevent UI exposure and squad overlap.

### Option C: Generic Unit-Group Backend

This is attractive only if VP intends to expand multiple unit-group features, not for the smallest viable linked-unit restoration.

### Option D: Minimally Repaired Independent Linked Movement

This is the smallest patch but does not meet the maintainability goal.

Recommendation: Option B. Keep "linked units" as an explicit same-plot escort/link feature with its own membership state, but replace unsafe movement execution with hardened shared movement helpers derived from Squads.

Exact rationale:

1. The desired behavior is not full named squad management; it is quick escort linking, primarily military plus civilian on the same plot.

2. Squads already contains better movement assignment, mission sequencing, reroute continuation, and arrival cleanup concepts.

3. Existing linked fields provide a compatibility-preserving place to store membership, but their consumers need validation.

4. Treating links as hidden squads risks leaking into the Squads UI, conflicting with squad membership, and requiring new type flags.

5. A generic backend is likely correct long-term but too large for the current target.

Likely production files:

| File | Expected changes in implementation phase |
| --- | --- |
| `CvGameCoreDLL_Expansion2/CvUnit.h` | Possibly add small helper declarations; avoid new serialized fields in phase 1 if possible. |
| `CvGameCoreDLL_Expansion2/CvUnit.cpp` | Harden `LinkUnits`, `UnlinkUnits`, linked movement validation, lifecycle cleanup; optionally extract squad assignment helper. |
| `CvGameCoreDLL_Expansion2/CvUnitMission.cpp` | Preserve `KEEP_LINK`; ensure cancellation/end movement does not stale links. |
| `CvGameCoreDLL_Expansion2/CvPlayer.cpp` | Harden `EndTurnsForReadyUnits` linked follower handling if retained. |
| `CvGameCoreDLL_Expansion2/Lua/CvLuaUnit.cpp` | Gate or validate link APIs; expose any new safe helper if needed. |
| `(1) Community Patch/Core Files/Overrides/InGame.lua`, `(2) Vox Populi/Core Files/Overrides/WorldView.lua` | Only if current VP UI should own link/move actions rather than modmod UI. |
| Linked movement SQL/text/icons | Reuse link/unlink text/icons if bundled; do not revive old `WorldView.lua` wholesale. |
| Squads resources | Ideally unchanged; if overlap policy is needed, Squads UI may need to hide linked-only groups or block linked units. |

Likely resource-side files:

Serialization impact: smallest viable implementation should not add fields. It should validate existing linked fields on load/use and clear invalid links. If a distinct link type, group subtype, or old-save migration marker is added later, a serialization-version change may be required.

UI impact: provide a simple Link/Unlink command separate from Squads UI. Do not expose linked groups in the Squads panel in phase 1. Do not re-enable old `MOVE_TO_ALL` as linked-unit restoration; that is quick group movement and should remain separate.

Effect on existing Squads behavior: linked helper hardening should improve Squads escort movement because Squads already calls `CanLinkUnits`/`LinkUnits` and relies on linked leader movement. Any change to failed linked movement or arrival unlinking must be regression-tested with `SQUADS=1`.

Go for integrating linked units into current VP if the scope is "same-plot escort linking" and the first implementation hardens existing linked state before exposing UI.

Squads reuse substantially improves maintainability if reused as movement planning/sequencing infrastructure rather than by copying old `DoGroupMovement`. The current code is not yet separated into clean services, so the first implementation should extract only the minimal helpers needed.

Smallest viable implementation:

1. Keep existing linked serialized fields.

2. Fix linked helper correctness and stale-ID cleanup.

3. Make linked movement atomic for each step.

4. Expose only Link/Unlink for same-plot escorts.

5. Leave old quick group movement out of scope and plan a later temporary-squad replacement.

### Recovered finding: LM-1: linked leader failed move still mutates state

Confirmed defect. `CvUnit::move` can subtract movement cost, and can already have changed embark state, after follower validation fails. See `CvGameCoreDLL_Expansion2/CvUnit.cpp:5467`, `CvUnit.cpp:5487`, and `CvUnit.cpp:5521`.

Smallest safe correction: prevalidate linked follower existence and movement before embark/disembark and before movement-cost accounting; cancel without state mutation if validation fails.

### Recovered finding: LM-2: `LinkUnits` queues missions on stale `pLoopUnit`

Confirmed defect. `CvUnit::LinkUnits` iterates `pUnit` but queues fortify/sleep through `pLoopUnit` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:14875`.

Smallest safe correction: use `pUnit` for `canFortify` and `PushMission`.

### Recovered finding: LM-3: `DoGroupMovement` can dereference null ideal plot

Confirmed defect. `CvUnit::DoGroupMovement` computes `pFirstTargetPlot` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:14970` and dereferences it at `CvUnit.cpp:14976` without a null check.

Smallest safe correction: validate `pDestPlot` and `pFirstTargetPlot`; if the ideal plot is null, skip to ring fallback.

Likely defect: normal in-repository UI reachability is incomplete. `CvUnit::CanDoInterfaceMode` exposes `INTERFACEMODE_MOVE_TO_ALL` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:30630`, but repository `WorldView.lua` uses a generic mission handler and `CvGame::selectionListGameNetMessage` sends only the head selected unit. No repository Lua caller for `DoGroupMovement` or `MoveLinkedLeader` was found.

Missing validation: linked IDs are serialized and later dereferenced without null checks in `CvGameCoreDLL_Expansion2/CvUnit.cpp:2587`, `CvUnit.cpp:5494`, `CvUnit.cpp:14690`, `CvUnit.cpp:14900`, `CvUnit.cpp:14911`, and `CvUnit.cpp:15481`.

Compatibility risk: `MOD_SQUADS` and `MOD_LINKED_MOVEMENT` share linked state. Fixes must be validated with `SQUADS=1` even though this pass targeted `LINKED_MOVEMENT`.

Insufficient runtime evidence: no in-game runtime tests or DLL build were run in this environment.

1. Fix `CvUnit::LinkUnits` to use `pUnit` instead of `pLoopUnit` when assigning follower sleep/fortify missions.

2. Add null checks and state cleanup around all linked leader/follower ID dereferences.

3. Make linked leader movement validation atomic in `CvUnit::move`, before embark/disembark and movement-cost mutation.

4. Add `DoGroupMovement` target validation for `pDestPlot` and `pFirstTargetPlot`.

5. Decide and document the intended UI activation path: either wire current UI to the helper path, or remove/hide `INTERFACEMODE_MOVE_TO_ALL` behavior unless an external mod provides it.

`LINKED_MOVEMENT`: not ready for normal use. The source has confirmed defects and missing validation, and audited repository UI reachability is not proven.

## Source and dependency lookup

Normal mission movement path:

```text
WorldView click or Game.HandleAction
  -> CvGame::selectionListGameNetMessage (CvGameCoreDLL_Expansion2/CvGame.cpp:2751)
  -> gDLL->sendPushMission (CvGame.cpp:2796)
  -> CvDllNetMessageHandler::ResponsePushMission (CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp:848)
  -> CvUnit::PushMission (CvGameCoreDLL_Expansion2/CvUnit.cpp:30896)
  -> CvUnitMission::PushMission / ContinueMission (CvGameCoreDLL_Expansion2/CvUnitMission.cpp:99)
  -> CvUnit::UnitPathTo / UnitMove
  -> CvUnit::move (CvGameCoreDLL_Expansion2/CvUnit.cpp:5453)
```

Linked helper Lua path:

```text
Lua unit method
  -> CvLuaUnit::lLinkUnits / lMoveLinkedLeader / lDoGroupMovement
  -> CvUnit::LinkUnits / MoveLinkedLeader / DoGroupMovement
  -> CvUnit::PushMission / CvUnit::move
```

End-turn linked follower path:

```text
ActionInfoPanel.lua player:EndTurnsForReadyUnits(true)
  -> CvLuaPlayer::lEndTurnsForReadyUnits (CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:9907)
  -> CvPlayer::EndTurnsForReadyUnits(..., bLinkedUnitsOnly=true)
  -> follower fortify/sleep only when MOD_LINKED_MOVEMENT is true
```

`CvUnit::move` is still the authoritative movement-state application function for the guarded linked leader branch at `CvGameCoreDLL_Expansion2/CvUnit.cpp:5487`.

`CvUnitMission` remains the authoritative mission queue and continue path for `MISSION_MOVE_TO` in `CvGameCoreDLL_Expansion2/CvUnitMission.cpp:99` and `CvUnitMission.cpp:480`.

`CvGame::selectionListGameNetMessage` sends only the head selected unit's mission at `CvGameCoreDLL_Expansion2/CvGame.cpp:2753` and `CvGame.cpp:2796`. It does not call `LinkUnits`, `MoveLinkedLeader`, or `DoGroupMovement`.

LIKELY_DEFECT: Audited repository UI makes `INTERFACEMODE_MOVE_TO_ALL` reachable through generic interface-mode/action code, but no repository path converts that mode into linked/grouped movement. `CvUnit::CanDoInterfaceMode` returns true for `INTERFACEMODE_MOVE_TO_ALL` when `MOD_LINKED_MOVEMENT` and adjacent owned units exist at `CvGameCoreDLL_Expansion2/CvUnit.cpp:30630`; `WorldView.lua` maps that mode to a generic mission click handler at `(2) Vox Populi/Core Files/Overrides/WorldView.lua:457` and `WorldView.lua:475`; that handler sends only the DB mission for the interface mode. The normal net path then sends only the selected head unit. Repository search found no current Lua caller for `DoGroupMovement` or `MoveLinkedLeader`.

INFERRED: `LINKED_MOVEMENT` may have depended on older or external UI that called the Lua helpers directly. That dependency is not present in the audited repository, so normal in-repo reachability is insufficiently validated.

With `LINKED_MOVEMENT=0` and `SQUADS=0`, the guarded runtime branches in `doTurn`, `move`, `maxMoves`, `ReadyToMove`, `CanDoInterfaceMode`, `CvUnitMission`, and `CvPlayer::EndTurnsForReadyUnits` do not execute. Lua helpers remain callable.

With `LINKED_MOVEMENT=1`, non-leader linked units are suppressed in `CvUnit::doTurn` (`CvGameCoreDLL_Expansion2/CvUnit.cpp:3212`) and `CvUnit::ReadyToMove` (`CvUnit.cpp:29800`), linked/grouped max moves use `GetLinkedMaxMoves` (`CvUnit.cpp:14607`), and linked leaders attempt to move followers in `CvUnit::move` (`CvUnit.cpp:5487`).

With `SQUADS=1`, many of the same linked helpers and states are used even if `LINKED_MOVEMENT=0`, including linked leader movement at `CvGameCoreDLL_Expansion2/CvUnit.cpp:5487` and max move override at `CvUnit.cpp:14607`.

Local `git log -S'MOD_LINKED_MOVEMENT'` and targeted `git log -G'LinkUnits|MoveLinkedLeader|DoGroupMovement|IsLinked|IsGrouped|MOD_SQUADS'` over linked movement files find production-source provenance at `54dee87` (`5.2.6 Release`) only. The local history is grafted/compressed, so this audit cannot identify the original rewrite that introduced the defects.

INSUFFICIENT_RUNTIME_EVIDENCE: The environment has Visual Studio solution/project files (`CvGameCoreDLL.vs2010.sln`, `VoxPopuli_vs2013.sln`, and `CvGameCoreDLL_Expansion2/*.vcxproj`) but no `msbuild`, `xbuild`, or `dotnet` command available. No DLL build was run.

Planning and architecture audit only. No production source, modmod resources, existing reports, commits, or pushes were modified.

Required context read: `codex/AGENTS.md`, `codex/docs/modmod-audit/00-baseline.md`, `01-feature-inventory.md`, `dependency-graph.md`, `test-matrix.md`, `features/linked-movement.md`, `groups/movement-systems.md`, and the relevant event/flagship reports.

The old linked/group movement modmod is `codex/modmod_resources/Linked Unit Movement for VP-EUI (v 4)`. Its modinfo identifies `Linked and Group Unit Movement for VP-EUI`, author `nobubblegums`, version `4`, `AffectsSavedGames=0`, `SupportsMultiplayer=0`, dependency `(1) Community Patch` minversion `108`, `(2) Vox Populi` minversion `15`, and EUI compatibility files minversion `1`.

Old linked/group movement resource files:

| Kind | Files |
| --- | --- |
| Activation/data | `LinkUnits.sql`, `Linked Unit Movement for VP-EUI (v 4).modinfo` |
| UI/Lua overrides | `Core Files/CoreLua/InGame.lua`, `Core Files/CoreLua/WorldView.lua`, `LUA/UnitPanel.lua`, `LUA/UnitPanel.xml`, `LUA/UnitPanel_small.xml` |
| Icons | `Art/Icons.xml`, `Art/Link*.dds`, `Art/Unlink*.dds`, `Art/GroupMove*.dds` |
| Activation/data | `(4a) Squads for VP/DB/Squads.sql`, `(4a) Squads for VP/(4a) Squads for VP (v 1).modinfo` |
| UI/Lua | `UI/Squads.lua`, `UI/Squads.xml`, `UI/SquadNames.lua`, `UI/SquadsOptions.lua`, `UI/SquadsUtil.lua` |
| Text/art/project | `XML/Squads_EN_US.xml`, `Art/SquadsIcon.dds`, `Squads.civ5proj`, `Squads.civ5sln` |
| Bundled helpers referenced by modinfo | `ThirdParty/PlotIterators.lua`, `ThirdParty/TableSaverLoader.lua`; these files are referenced in modinfo and included from Lua but were not present in the visible `find` output. |

`LinkUnits.sql` enables `CustomModOptions.Name='LINKED_MOVEMENT'`, adds `TXT_KEY_ACTION_LINK_UNITS` and `TXT_KEY_ACTION_UNLINK_UNITS`, and exposes `INTERFACEMODE_MOVE_TO_ALL` as "Group Move" on `KB_X`.

The official Squads modmod is `(4a) Squads for VP`. Its modinfo identifies `(4a) Squads for VP`, teaser `(5.3.3) RTS Style Control Groups and Group Movement`, author `Vern`, special thanks `Gwennog`, version `1`, `SupportsMultiplayer=1`, dependency `(1) Community Patch` minversion `150`, and reference `(2) Vox Populi` minversion `17`.

Squads resource files:

`DB/Squads.sql` enables `CustomModOptions.Name='SQUADS'` and inserts `INTERFACEMODE_SQUAD_UNIT_MANAGEMENT`, `INTERFACEMODE_SQUAD_BASE_MANAGEMENT`, and `INTERFACEMODE_SQUAD_MOVEMENT`.

Current VP integration points include `(1) Community Patch/Database Changes/NewCustomModOptions.xml` rows for `SQUADS` and `LINKED_MOVEMENT`; `CvGameCoreDLL_Expansion2/CustomMods.h`/`.cpp` macros and cache; `CvGameCoreDLL_Expansion2/CvUnit.h`, `CvUnit.cpp`, `CvUnitMission.cpp`, `CvPlayer.cpp`, `CvPlayer.h`; Lua wrappers in `CvLuaUnit.cpp`, `CvLuaPlayer.cpp`; and UI handlers in `(1) Community Patch/Core Files/Overrides/InGame.lua` and `(2) Vox Populi/Core Files/Overrides/WorldView.lua`.

No old external DLL `.cpp`/`.h` snapshot or patch file for linked movement or Squads was found under `codex/modmod_resources/`. The old DLL-side implementation exists in current `CvUnit`/`CvUnitMission` guarded code. The resource versions above are the only explicit VP-version notes found: linked movement targets CP `108`/VP `15`; Squads targets CP `150`/VP `17` and advertises `(5.3.3)`.

The player-facing link action is a unit-panel button added by the old modmod UI. `LUA/UnitPanel.xml` and `UnitPanel_small.xml` add `LinkUnitsButton`; `LUA/UnitPanel.lua` calls `unit:UnlinkUnits()` if already linked, otherwise `unit:LinkUnits()` when `unit:CanLinkUnits()` is true. The button is hidden unless the selected unit is linked or can link and has moves left.

`CvLuaUnit.cpp` exposes `IsLinked`, `IsLinkedLeader`, `CanLinkUnits`, `LinkUnits`, `UnlinkUnits`, `MoveLinkedLeader`, and `DoGroupMovement` regardless of runtime option gating. Existing reports classify this as a compatibility risk because external Lua can mutate linked state when `LINKED_MOVEMENT=0`.

`CvUnit::CanLinkUnits` selects same-owner, non-delayed-death, non-trade, non-air units on the same plot. Units are compatible if the selected unit is "on sea" (`DOMAIN_SEA` or embarked) and the other unit is sea or embarked, or if the selected unit is land and the other unit is land and not embarked. This supports civilian escorts because it excludes trade/air but not civilians.

Membership is stored per unit, not in a central registry: `m_bIsLinked`, `m_bIsLinkedLeader`, `m_iLinkedLeaderID`, `m_LinkedUnitIDs`, `m_bIsGrouped`, and `m_iLinkedMaxMoves` in `CvUnit.h`. Linked state is reset in `CvUnit::reset` and serialized in `CvUnit::Serialize` plus sync archive variables.

There is a leader. `CvUnit::LinkUnits` chooses a land worker on the stack as leader if found; otherwise the ordering unit becomes leader. Followers store `m_iLinkedLeaderID`; the leader stores follower unit IDs. All linked members get the lowest current moves and lowest max moves from the group.

Movement is initiated from two paths:

```text
Old UnitPanel link button
  -> CvLuaUnit::lLinkUnits / lUnlinkUnits
  -> CvUnit::LinkUnits / UnlinkUnits
  -> later ordinary leader MISSION_MOVE_TO
  -> CvUnit::move linked-leader branch
```

```text
Old WorldView right-click when selected unit is a linked follower
  -> unit:MoveLinkedLeader(plot)
  -> leader PushMission(MISSION_MOVE_TO, destination)
  -> CvUnit::move linked-leader branch
```

`CvUnit::move` implements linked step execution. If `MOD_LINKED_MOVEMENT || MOD_SQUADS` and the moving unit is a linked leader, it checks each follower with `canMoveInto(targetPlot, MOVEFLAG_DESTINATION)`, moves the leader with direct `setXY`, then pushes a follower `MISSION_MOVE_TO` to the same target plot with `MOVEFLAG_DESTINATION | MOVEFLAG_KEEP_LINK`.

Links are removed by `CvUnit::UnlinkUnits`, by `SetIsLinkedLeader(false)`, by some command/interface paths that call `UnlinkUnits`, and by Squads arrival cleanup. There is no central ownership/lifecycle repair pass.

Lua/UI interaction is direct Lua-to-DLL helper calls. No custom mission ID is used for link/unlink. Movement still relies on ordinary `MISSION_MOVE_TO`, mission queues, and the `CvUnit::move` hook.

Known unsafe behavior:

| Area | Finding |
| --- | --- |
| Direct `setXY` | Leader movement uses direct `setXY` in `CvUnit::move`; follower movement is then pushed as ordinary missions, so callbacks can observe a partially moved linked group. |
| Failed movement | Existing report confirms leader embark/disembark and movement-cost mutation can occur before follower validation failure is handled. |
| Stale IDs | `GET_PLAYER(...).getUnit(id)` is dereferenced without null checks in kill cleanup, unlink, linked movement, and Squads end handling. |
| Wrong mission target | `LinkUnits` queues fortify/sleep through stale `pLoopUnit` instead of the current `pUnit`. |
| Illegal followers | Validation checks a one-step `canMoveInto` target, but does not prove the follower path, zones of control, embark state, or movement budget matches the leader's whole path. |
| Domains/embarkation | The sea/land compatibility test treats embarked land with sea, but a group can still diverge when embark/disembark requirements differ. |
| Partial group callbacks | Leader moves first; follower missions execute later. Lua/game callbacks, enemy reveal aborts, or combat/capture during movement can see incomplete group state. |
| Save/load | Linked IDs are serialized unconditionally and not validated on load or option changes. |

Lifecycle coverage:

| Lifecycle path | Current evidence |
| --- | --- |
| Death/deletion | `CvUnit::kill` tries to clear linked state, but follower path dereferences leader without null check. |
| Upgrade | `CvUnit::upgrade` copies squad number when `MOD_SQUADS`, but no equivalent explicit linked-state migration was found. `convert()` later kills the old unit, so old linked IDs can stale unless cleanup succeeds. |
| Conversion/capture/gift | `convert()` copies many unit fields, but no verified linked-state transfer or cleanup policy exists. Capture/gift paths create/convert units and can leave old IDs stale. |
| Ownership transfer | Linked IDs are owner-local unit IDs. Any transfer across owners is unsafe unless links are explicitly broken first; no central validation was found. |
| Embark/disembark | `CvUnit::move` may mutate embark state before linked failure. Compatibility checks are based on current domain/embark state only. |
| Teleport/rebase/airlift | Rebase is air-domain and `CanLinkUnits` excludes air. Airlift and other teleports use direct `setXY` outside the linked leader branch, so links are not guaranteed to move together. |
| Save/load | Linked state serializes; no post-load repair. |
| Player elimination | No linked-specific elimination cleanup was found beyond normal unit death/deletion paths. |

The old "Group Move" is distinct from persistent linked units. `LinkUnits.sql` exposes `INTERFACEMODE_MOVE_TO_ALL`, and old `Core Files/CoreLua/WorldView.lua` calls `pHeadSelectedUnit:DoGroupMovement(plot)` for that mode.

`CvUnit::DoGroupMovement` first calls `LinkUnits()` for same-plot units, then scans ring 0 through ring 1 around the selected unit. It selects same-owner, non-delayed-death units that are not ordinary civilians, except Great Generals and Great Admirals are allowed. Domain/embark compatibility mirrors linked movement.

Original relative positions are represented transiently as `iXDiff = selected.getX() - pUnit.getX()` and `iYDiff = selected.getY() - pUnit.getY()`. The ideal target is `pDestPlot - diff`. If the ideal plot fails, it tries `MOVEFLAG_APPROX_TARGET_RING1`, then ring 2. Membership does not persist as a separate group; units are marked `IsGrouped` only for movement/max-move override and then cleared by mission handling.

It uses ordinary `MISSION_MOVE_TO` and pathfinding flags, not a custom mission. It does not force all units through one atomic planner. It can be quicker than Squads because it needs no assignment UI, no persistent squad numbers, and no separate management mode: select a unit, press `X`, click destination.

Historical problems from design:

| Problem | Cause |
| --- | --- |
| Edge/null crash | `pFirstTargetPlot` is dereferenced without null check when the projected relative tile is off-map. |
| Partial movement | Each adjacent unit gets an independent mission and can fail independently. |
| Formation drift | Ring fallback preserves only approximate grouping, not a stable formation model. |
| Unsafe reuse of linked state | `LinkUnits()` is called as part of group movement, so old linked-state bugs also affect quick group movement. |
| Civilian mismatch | Ordinary civilians are excluded from adjacent group movement, while linked units are explicitly useful for civilian escorts. |

Squads activation is `CustomModOptions.Name='SQUADS'`, defined in `NewCustomModOptions.xml`, cached by `CustomMods`, and enabled by `(4a) Squads for VP/DB/Squads.sql`.

User workflow:

```text
Squads panel / hotkeys
  -> management mode (button or CTRL+number)
  -> LCLICK unit/flag or box-select to assign/remove units
  -> movement mode (button or double-tap number)
  -> RCLICK destination
```

The UI calls `unit:AssignToSquad(currentSquadNumber)`, `unit:RemoveFromSquad()`, `player:UnitsInSquad(number)`, `unit:GetSquadMovementPreview(plot)`, `unit:SetSquadEndMovementType(...)`, and `unit:DoSquadMovement(plot, bEscortNonCombatUnits)`.

Membership representation is per-unit `m_iSquadNumber`; valid UI numbers are `1..10` with `0` mapped to `10`. There is no dedicated squad object in C++. Iteration is via `CvPlayer::firstUnitInSquad` / `nextUnitInSquad`, exposed to Lua as `UnitsInSquad`.

Squads has no persistent leader. The first unit found in the squad is used as a caller/anchor for `DoSquadMovement`; `GetSquadCenterOfMass` computes an anchor from all member positions, with wrap handling.

Formation is generated at movement time, not stored as offsets. `DoSquadPlotAssignments` partitions units into stacking units, eligible land combat units, and eligible sea combat units. Stacking/non-combat units are assigned to the destination plot. Land and sea combat units are assigned to nearby eligible plots by `DoSquadPlotAssignmentsByDomain`, prioritizing distant units and plots near squad center of mass.

Escort support is implemented by reusing linked movement. If the `escort` option is true, non-combat/stacking units on the same plot as an eligible land/sea combat unit cause that combat unit to call `CanLinkUnits()` and `LinkUnits()`. `DoSquadMovement` skips sending an independent mission for linked followers and sends missions for unlinked units or linked leaders.

Squad movement uses ordinary networked `sendPushMission(MISSION_MOVE_TO, target, MOVEFLAG_ABORT_IF_NEW_ENEMY_REVEALED | MOVEFLAG_CONTINUE_TO_CLOSEST_PLOT | MOVEFLAG_KEEP_LINK)`. `CvUnitMission::ContinueMission` has `MOD_SQUADS` branches to continue along rerouted cached paths and call `TryEndSquadMovement`.

Partial-failure behavior is permissive. Units unable to `canMoveInto(*pDestPlot)` are excluded from the assignment map. Units that can move may continue, using closest-plot continuation logic if the exact target becomes blocked. `TryEndSquadMovement` handles arrival/alert/sleep behavior based on `SquadsEndMovementType`.

Serialization: `m_iSquadNumber`, `m_iSquadDestinationX`, `m_iSquadDestinationY`, and `m_SquadEndMovementType` serialize in `CvUnit::Serialize`. Squad names are saved separately by `UI/SquadNames.lua` using `TableSaverLoader` into save data. User options are stored with `Modding.OpenUserData("Squads Options", 1)`, not in the DLL save stream.

Unit upgrade copies `GetSquadNumber()` to the new unit when `MOD_SQUADS` is enabled. No similar verified support was found for capture/gift/conversion beyond normal unit creation/kill cleanup.

Maintenance boundary: the DLL owns squad membership integer, unit iteration, movement planning, destination state, and movement-end behavior. The official modmod owns interface modes, panel UI, hotkeys, names, options, highlights, and activation.

Reusable infrastructure:

| Directly reusable | Caveat |
| --- | --- |
| `DoSquadPlotAssignments` and `DoSquadPlotAssignmentsByDomain` | Assumes squad membership and combat/stacking categories. |
| `DoSquadMovement` mission sequencing | Assumes persistent squad number and destination shared across all squad members. |
| Escort link reuse | Already solves "combat escorts non-combat on same plot", but via old linked helpers with inherited defects. |
| `MOVEFLAG_CONTINUE_TO_CLOSEST_PLOT` handling | Useful for coordinated movement; not a standalone service. |
| `TryEndSquadMovement` arrival cleanup | Tied to squad number/destination/end-movement policy. |

| Concern | Old linked units | Old quick group movement | Current Squads |
| --- | --- | --- | --- |
| Persistent membership | Yes, per-unit linked IDs | No; grouped flag only during move | Yes, per-unit squad number |
| Temporary membership | No separate temp link type | Yes | Movement assignments are temporary; squad membership persists |
| Leader/anchor | Explicit leader | Selected unit is anchor | No leader; caller plus center of mass |
| Civilian support | Yes, non-trade/non-air same plot | Mostly no, except generals/admirals | Yes for stacking/non-combat and escort option |
| Military escorts | Primary use case | Adjacent combat group | Explicit escort option uses linked helpers |
| Mixed military/civilian | Yes on same plot if domain-compatible | Not generally | Yes as squad members; escorts via links |
| Relative formation | Same plot following only | Attempts relative offsets | Dynamic target-area assignment |
| Movement-point differences | Lowest current/max moves applied | Lowest current/max moves applied | Individual paths; linked escorts inherit linked min-move behavior |
| Domain differences | Same land/sea/embarked bucket | Same land/sea/embarked bucket | Land/sea partitioned; stacking units assigned to destination |
| Embarkation | Risky; state can mutate before failure | Risky via linked and per-unit missions | Better partitioning, but escort links inherit risk |
| Blocked followers | Can block leader but still mutate state | Unit dropped from group if cannot move | Blocked units excluded or rerouted to closest plot |
| Partial movement | Possible via follower missions/callbacks | Expected | Expected/permissive |
| Pathfinding | One-step linked check plus ordinary missions | Ordinary move missions with approximate flags | Assignment plus ordinary pathing and closest-plot continuation |
| Mission queues | Link queues sleep/fortify; movement queues followers | Clears/sets grouped movement missions | Clears moving units and sends networked missions |
| Unit lifecycle | Weak; stale IDs | Weak | Better upgrade copy; other transfer cleanup still weak |
| Ownership changes | Unsafe owner-local IDs | Not persistent | Squad number can copy/drop; no cross-owner policy |
| Serialization | Linked fields unconditional | Grouped fields unconditional | Squad fields serialized; names in Lua save data |
| Lua/UI dependencies | Old EUI UnitPanel/WorldView overrides | Old WorldView override | Official Squads UI and VP WorldView/InGame hooks |
| AI support | Human-only linking check | Human-oriented | UI-driven human feature; DLL methods are generic but no AI assignment policy found |
| Multiplayer determinism | Old mod says unsupported | Old mod says unsupported | Official mod says supported; uses `sendPushMission` |
| Save compatibility | Existing serialized fields | Existing grouped fields | Existing serialized squad fields |

Duplicated functionality: membership flags, leader/follower movement suppression, shared minimum moves, target assignment around a destination, and UI-triggered coordinated movement all overlap. The old quick group movement is most duplicated by Squads. The linked escort concept is partly duplicated by Squads' escort option but implemented by calling old linked helpers rather than a cleaner service.

Old code that should not be carried forward unchanged: direct `setXY` linked movement after partial validation; stale-ID dereferences; `DoGroupMovement` relative-offset crash path; `pLoopUnit` follower mission bug; option-ungated Lua mutation; and save/load without validation.

Can reuse directly after hardening:

| Code | Use |
| --- | --- |
| `CvUnit::CanLinkUnits` eligibility shape | Starting point for same-plot escort link eligibility. |
| `CvUnit::LinkUnits` data fields | Existing serialized compatibility path, but needs null/mission bug fixes and policy clarification. |
| Squads assignment functions | Movement planning for area-based groups and future quick group movement. |
| Squads mission flags and `CvUnitMission` branches | Safer group movement sequencing/reroute behavior than old `DoGroupMovement`. |
| Lua wrappers | Existing UI bridge, but should be option-gated or documented. |

Requires extraction:

| Code | Required extraction |
| --- | --- |
| `DoSquadPlotAssignments` | Separate "given unit list, destination, escort policy -> assignment map" from squad-number lookup. |
| Escort link setup | Separate link creation from Squads membership and destination state. |
| `TryEndSquadMovement` | Separate linked-arrival cleanup from squad end policies. |
| Linked movement validation | Make an atomic helper that validates all followers before embark/cost/setXY mutation. |

Should retire or leave unused:

| Code | Reason |
| --- | --- |
| Old `DoGroupMovement` as primary implementation | Duplicates Squads with weaker assignment and known null/partial-failure issues. |
| Old EUI `WorldView.lua` movement override | Stale relative to current VP/Squads interface-mode routing. |
| Old direct `MOVE_TO_ALL` UI exposure | Audited repository path does not call helper unless old override is used. |

These require user approval before implementation:

| Decision | Existing behavior/evidence | Needs approval |
| --- | --- | --- |
| Link persistence | Old links persist until unlink/lifecycle break. | Keep persistent until explicit remove, or auto-dissolve on move/separation? |
| Include all same-plot eligible units | Old `LinkUnits` includes all compatible same-plot units. | Keep automatic all-eligible behavior? |
| Multiple military units | Old link can include multiple compatible units. | Allow or restrict to one escort? |
| Multiple civilian units | Old link can include non-trade civilians. | Allow or restrict to one civilian? |
| Leader/anchor | Old code uses worker-first else ordering unit. | Keep, use selected unit, or explicit leader? |
| Movement authority | Old follower can call `MoveLinkedLeader`. | Movement follows selected unit or fixed leader only? |
| Any-member block policy | Old intent seems atomic per step, implementation flawed. | Stop all if any member cannot move, or allow partial continuation? |
| Separation policy | Old state can persist across separation. | Auto-dissolve if members separate? |
| Upgrade/capture/gift/save survival | Old save survives, transfer policy unclear. | Which lifecycle transitions preserve links? |
| Escort attacks | Old linked escort can receive ordinary missions if unlinked/commanded. | Does attack break link, block attack, or allow attack and keep link? |
| Movement-point differences | Old code clamps to lowest moves. | Keep strict lowest-budget behavior? |
| Squads UI | Old linked UI separate; Squads UI separate. | Should linked groups appear in Squads UI? |
| System overlap | Current code allows risky overlap. | Can a unit be both linked and in a squad? If yes, which movement command owns it? |

Minimal linked-units integration:

1. Fix linked null dereferences and stale-ID cleanup at all consumers.

2. Fix `LinkUnits` follower mission variable bug.

3. Make linked leader movement prevalidate followers before embark/disembark, `setXY`, and movement-cost mutation.

4. Gate or document Lua link mutation under `MOD_LINKED_MOVEMENT || MOD_SQUADS`.

5. Add a current VP link/unlink UI action without importing old stale `WorldView.lua`.

Hardening:

1. Add load/use validation for linked IDs and owner consistency.

2. Define lifecycle policies for upgrade, conversion, capture, gift, deletion, teleport, and save/load.

3. Add debug logging/assertions around link creation, movement, failure, unlink, and cleanup.

4. Validate Squads escort behavior with `SQUADS=1`, `LINKED_MOVEMENT=0`, and both enabled.

UI integration:

1. Decide whether the command is a unit-panel button, action button, or interface mode.

2. Add text/tooltips that distinguish "Link escort" from "Squad movement".

3. Avoid exposing hidden linked groups in Squads panel unless explicitly approved.

Optional future quick group movement:

1. Do not restore old `DoGroupMovement` as-is.

2. Implement a quick temporary-squad command that builds a transient unit list from adjacent eligible units and calls extracted Squads movement assignment/sequencing.

3. Add preview/highlight only after core movement is stable.

Recommend option 3: replace it later with a quick temporary-squad command.

Do not integrate old group movement now. It is distinct from linked escorting, has known unsafe behavior, and is mostly superseded by Squads. Do not remove the code during the linked-unit pass unless cleanup is explicitly in scope; leave old `DoGroupMovement` dormant or retired behind non-exposed UI until a temporary-squad replacement is designed.

Old linked membership exists in serialized `CvUnit` fields: linked flags, leader ID, follower ID vector, grouped flag, and linked max moves. This means old linked saves can be detected by scanning loaded units for `IsLinked()`, `IsLinkedLeader()`, non-empty `GetLinkedUnits()`, or `IsGrouped()`.

Current Squads saves serialize `m_iSquadNumber`, `m_iSquadDestinationX/Y`, and end movement type. Existing Squads saves should remain compatible if those fields are not reordered/removed and no new required invariants are imposed.

INFERRED: Practical old linked-save migration should be "validate and clear or repair" rather than full semantic migration. Owner-local unit IDs after capture/gift/upgrade are not reliable enough for rich reconstruction.

Option naming: keep `LINKED_MOVEMENT` only if the feature remains separate. If it becomes a Squads subtype later, consider aliasing old `LINKED_MOVEMENT` to the new link command but not to old `MOVE_TO_ALL`.

Simultaneous enablement: current code technically allows `SQUADS=1` and `LINKED_MOVEMENT=1`, but existing reports classify the interaction as risky. Recommended policy is to support simultaneous enablement only after overlap rules are approved. In phase 1, allow Squads escort internals to use hardened links while preventing manual linked groups from being assigned to ordinary squads unless explicitly allowed.

Serialization version: no version change is required for the smallest viable hardening if fields are unchanged. A version change is likely required for a generic backend, hidden group type, or new link subtype flag.

Subsystem: Movement systems

Features audited in this pass: `LINKED_MOVEMENT`

`LINKED_MOVEMENT` is a runtime `CustomModOptions` switch defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:1140`, cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:573`, and wrapped by `MOD_LINKED_MOVEMENT` in `CvGameCoreDLL_Expansion2/CustomMods.h:1022`.

The linked state and helpers live on `CvUnit`: declarations in `CvGameCoreDLL_Expansion2/CvUnit.h:844`, fields in `CvUnit.h:2320`, serialization in `CvGameCoreDLL_Expansion2/CvUnit.cpp:28429`, and helper implementations in `CvUnit.cpp:14640` through `CvUnit.cpp:14993`.

`MOD_SQUADS` shares this linked state and several helper paths, especially `CvUnit::move` at `CvGameCoreDLL_Expansion2/CvUnit.cpp:5487` and `CvUnit::maxMoves` at `CvUnit.cpp:14607`.

The authoritative movement application branch is still `CvUnit::move`, reached through mission processing in `CvUnitMission.cpp` and network dispatch in `CvDllNetMessageHandler::ResponsePushMission`.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `LINKED_MOVEMENT` | Link adjacent combat units, move leader, fail pathing, clear missions, test follower ready-to-move state. | Save linked units before and after movement; reload and check linked IDs/grouped flags. | `CvUnit::LinkUnits`, `CvUnit::MoveLinkedLeader`, `CvUnit::DoGroupMovement`, unit serialization. | `CvUnit.cpp:14640`, `CvUnit.cpp:5487`, `CvUnit.cpp:28430` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `LINKED_MOVEMENT=1`, `SQUADS=0` linked leader blocked by follower | Link two same-domain combat units; choose a destination the leader can enter but the follower cannot. | Save before the move, reload, execute the blocked linked move. | No unit should move; leader movement points and embark/disembark state should remain unchanged. | Confirmed failed-move mutation in `CvUnit.cpp:5467`, `CvUnit.cpp:5487`, and `CvUnit.cpp:5521`. |
| `LINKED_MOVEMENT=1`, `SQUADS=0` follower sleep/fortify assignment | Link three units on one plot. | Save immediately after linking; reload and inspect linked flags and head missions. | Every non-leader follower should have the intended sleep/fortify mission; no unrelated last iterated unit should receive repeated missions. | Confirmed stale `pLoopUnit` bug in `CvUnit.cpp:14875`. |
| `LINKED_MOVEMENT=1`, `SQUADS=0` group movement at map edge | Call `DoGroupMovement` where one follower's relative target would be off-map. | Save before group movement and reload before executing. | The null ideal plot should be skipped and fallback placement attempted without crash. | Confirmed null dereference risk in `CvUnit.cpp:14970` and `CvUnit.cpp:14976`. |
| `LINKED_MOVEMENT=1`, `SQUADS=0` stale linked ID cleanup | Link units, save, reload, then kill/disband/capture one linked member before unlink/move/end-turn. | Save at each stage: linked, after reload, after member removal. | Remaining units should not dereference null linked IDs; linked state should clear or repair. | Missing validation at `CvUnit.cpp:2587`, `CvUnit.cpp:5494`, `CvUnit.cpp:14690`, `CvUnit.cpp:14900`, `CvUnit.cpp:14911`, and `CvUnit.cpp:15481`. |
| Normal UI path with `INTERFACEMODE_MOVE_TO_ALL` | Enable `LINKED_MOVEMENT`, select a combat unit adjacent to owned units, invoke the visible move-all action and click a destination. | Save before action; reload and repeat. | Either units should link/group and move together, or the action should not be exposed. | Likely UI reachability defect: `CvUnit.cpp:30630`, `WorldView.lua:457`, `WorldView.lua:475`, and `CvGame.cpp:2796`. |
| `LINKED_MOVEMENT=0`, external Lua helper calls | With option disabled, call `unit:CanLinkUnits()` and `unit:LinkUnits()` from Lua. | Save after helper call and reload. | DLL should either prevent state mutation or document that helpers are available independent of option. | Compatibility risk: Lua wrappers at `CvLuaUnit.cpp:2554` through `CvLuaUnit.cpp:2626` are not runtime-gated. |
| `LINKED_MOVEMENT=0`, `SQUADS=1` | Run squad escort and movement using linked escorts. | Save before squad move and after squad arrival. | Squad movement should remain stable despite linked helper fixes and should clean linked state on arrival. | Shared-state compatibility with `CvUnit.cpp:15306`, `CvUnit.cpp:15355`, and `CvUnit.cpp:15464`. |
| `LINKED_MOVEMENT=1`, `SQUADS=1` | Run linked units inside a squad and move the squad. | Save before movement, during movement, and after arrival. | Followers should not be double-suppressed or left with stale linked state. | Unsafe interaction between `MOD_LINKED_MOVEMENT` and `MOD_SQUADS` shared state. |

- DLL/source:
  - `CvGameCoreDLLUtil/include/CvEnums.h`
  - `CvGameCoreDLL_Expansion2/Lua/CvLuaEnums.cpp`
  - `CvGameCoreDLL_Expansion2/CustomMods.h`
  - `CvGameCoreDLL_Expansion2/CustomMods.cpp`
  - `CvGameCoreDLL_Expansion2/CvUnit.h`
  - `CvGameCoreDLL_Expansion2/CvUnit.cpp`
  - `CvGameCoreDLL_Expansion2/CvUnitMission.cpp`

- Current-base copied VP integration resources:
  - `codex/modmod_resources/vp_edits/(3a) VP - EUI Compatibility Files/LUA/UnitPanel.lua`
  - `codex/modmod_resources/vp_edits/(3a) VP - EUI Compatibility Files/LUA/UnitPanel.xml`
  - `codex/modmod_resources/vp_edits/(3a) VP - EUI Compatibility Files/LUA/UnitPanel_small.xml`
  - `codex/modmod_resources/vp_edits/(2) Vox Populi/LUA/UnitPanel.lua`
  - `codex/modmod_resources/vp_edits/(2) Vox Populi/LUA/UnitPanel.xml`
  - `codex/modmod_resources/vp_edits/(2) Vox Populi/LUA/UnitPanel_small.xml`
  - `codex/modmod_resources/vp_edits/(2) Vox Populi/Core Files/Overrides/WorldView.lua`
  - `codex/modmod_resources/vp_edits/(1) Community Patch/Core Files/Overrides/InGame.lua`

- Updated modmod resources:
  - `codex/modmod_resources/Linked Unit Movement for VP-EUI (v 4)/...`
  - `codex/modmod_resources/Linked Unit Movement for VP-noEUI (v 3)/...`

- `COMMAND_LINK_UNITS` and `COMMAND_UNLINK_UNITS` were appended to `CommandTypes` and registered to Lua.

- UnitPanel Link/Unlink buttons now call `Game.SelectionListGameNetMessage(GAMEMESSAGE_DO_COMMAND, CommandTypes.COMMAND_LINK_UNITS/COMMAND_UNLINK_UNITS, ...)`.

- The synchronized path is:
  - UnitPanel button
  - `Game.SelectionListGameNetMessage`
  - `gDLL->sendDoCommand`
  - `CvDllNetMessageHandler::ResponseDoCommand`
  - `CvUnit::doCommand`
  - `CvUnit::LinkUnits` / `CvUnit::UnlinkUnits`

- No DB `Commands` rows were required for the custom button path because the UI uses the registered enum directly and does not need these commands to appear as generic `GameInfoActions`.

- Link creation sorts same-plot eligible units by unit ID before leader selection and follower ID storage. Worker-first leader semantics are preserved, with deterministic tie-breaking.

- Squads still reuses `CanLinkUnits`, `LinkUnits`, `UnlinkUnits`, linked leader state, `MOVEFLAG_KEEP_LINK`, and the linked movement branch in `CvUnit::move`.

- The linked null checks and atomic step validation apply to Squads escort movement as well.

- Manual Link/Unlink UI is gated by `LINKED_MOVEMENT`, but Squads can still call linked helpers when `SQUADS=1` and `LINKED_MOVEMENT=0`.

- No squad save fields or linked save fields were added, removed, or reordered.

- Normal linked escort movement.

- Blocked escort.

- Blocked civilian.

- Unequal movement points.

- Embark/disembark.

- Zones of control.

- Combat/attack while linked.

- Capture/death.

- Upgrade/conversion.

- Save/load.

- Simultaneous Squads and linked use.

- Quick Group Movement rigid formation.

- Quick Group Movement blocked member.

- Quick Group Movement slowest-unit pace.

- Quick Group Movement default-off behavior.

- Non-EUI UnitPanel link button.

- EUI UnitPanel link button.

- Multiplayer/network determinism for host/client Link/Unlink commands.

- Quick Group Movement still uses the existing Lua helper entry point; movement missions are network-dispatched, but grouped-state setup should be runtime-tested before treating Quick Group Movement as multiplayer-safe.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
