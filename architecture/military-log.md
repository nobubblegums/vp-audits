# Military event log and Lua contract

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How are human-visible military events captured, exposed and retained?

The base audit and a reported 2026-06-30 repair describe different revisions. The repaired report includes null-safe Lua access, bounded nuclear summaries and versioned save payloads; the old missing-coverage/non-serialized findings describe the pre-repair revision.

## Revision reconciliation

The base group audit reports null-unsafe Lua access, absent nuclear entries and a non-serialized ring. The feature report's **2026-06-30 repair update** reports those three areas changed: unavailable logs return an empty table, nuclear paths emit bounded summaries, and a version-gated logical payload restores retained entries. These are distinct recovered revisions; pre-repair findings below are not assertions that the repaired revision still has those defects.

The repair report specifies effective capacity 49 of 50 slots, marker `0x4d4c4f47`, format version 1 and `SAVE_VERSION_MILITARY_LOG = 1`. Runtime fog-of-war, eligible human-slot and old/new-save compatibility remained untested. A UI integration proposal does not establish an implemented notification tab.

Canonical related owners: [Optional event and mission dispatch](<event-dispatch.md>), [Military log UI integration reference and design](<military-log-ui.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Reported revision changes

Current option locations remain `(1) Community Patch/Database Changes/NewCustomModOptions.xml`, `CvGameCoreDLL_Expansion2/CustomMods.h`, and `CvGameCoreDLL_Expansion2/CustomMods.cpp`. The supplied `UI - Military Log` resource is present under `codex/modmod_resources/Merged UI Mods/MilitaryLog/`; `XML/UIMilitaryLog.xml` enables `CustomModOptions.Name='WH_MILITARY_LOG'`, and `UI/MilitaryLogPopup.lua` calls `Players[Game.GetActivePlayer()]:GetMilitaryLog()`.

The supplied Lua UI expects an array table whose entries contain `Turn`, `Message`, `OtherPlayer`, `Plot`, and `Data1` through `Data4`, then iterates from `#playerLog` down to `1`. Returning an empty table for an unavailable log is compatible with this path because the loop body is skipped.

REPORTED_REPAIR: `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp::CvLuaPlayer::lGetMilitaryLog` now returns the already-created empty table when `CvPlayer::GetMilitaryLog()` is null. The method remains registered and callable with `MOD_WH_MILITARY_LOG` disabled, for ineligible players, and before any entry exists. It does not allocate a log merely because Lua queried it.

REPORTED_REPAIR: `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp` now writes bounded military-log entries for nuclear combat. A fully intercepted Atomic Bomb logs one intercepted entry for the attacker and intercepting city owner using the existing `TXT_KEY_ATOMIC_BOMB_INTERCEPTED*` localization. A successful detonation logs one attacker summary in `ResolveNuclearCombat` and one affected-player summary through the existing affected-player notification path in `ApplyNuclearExplosionDamage`. No per-unit, per-city, or per-plot blast details were added.

Nuclear logging remains guarded by `MOD_WH_MILITARY_LOG` and uses existing nuclear notification localization keys. The added successful-detonation messages use the target plot already used by nuclear notifications. Intercepted-launch messages use the intercepting city plot, which is known to the attacker and city owner. Runtime fog-of-war tests are still required for human observers who are neither attacker nor affected owner.

REPORTED_REPAIR: Retained military-log entries are now serialized through `CvEventLog::Write` / `CvEventLog::Read` and `CvPlayer::Write` / `CvPlayer::Read`. The global save header version in `CvGlobals::SaveVersionTags` was advanced from initial version `0` to `SAVE_VERSION_MILITARY_LOG = 1`; `CvPlayer::Read` consumes military-log bytes only when `GC.getSaveVersion() >= SAVE_VERSION_MILITARY_LOG`, so older saves created without the log payload still load with an empty log.

The serialized military-log format is logical and bounded: marker `0x4d4c4f47`, format version `1`, entry count, then each retained entry in chronological display order. Each entry stores turn, already-rendered `CvString` message, other player, plot coordinates, and four integer data fields. Raw pointers, vector capacity, unused slots, and ring-buffer indices are not serialized.

On read, the log is initialized before restoration; negative counts are treated as empty; oversized counts assert but still consume every serialized entry to keep the stream aligned; only the newest entries up to current effective capacity are restored. Entries are restored directly rather than via `Add`, so load does not rewrite turns or duplicate entries as new combat events.

Current retained capacity remains unchanged: `CvEventLog` allocates `MAX_EVENTS = 50` slots and reserves one slot to distinguish full from empty, so the effective maximum serialized and restored entries per eligible player is `49`.

Player lifecycle after the repair: `CvPlayer::init` still allocates a military log for major-civ slots when the option is enabled; `CvPlayer::Init` initializes it for new games; `CvPlayer::Uninit` clears it; `CvPlayer::Read` initializes or deletes the pointer according to current option/player eligibility, then restores serialized entries only for save version `1+`; `CvPlayer::Write` writes either the live log or an empty log payload. The old load-time comment saying the log was not serialized was removed.

`CvEventLog::Add` still refuses to record entries for players that are not `isHuman(ISHUMAN_UI)`. Serialization writes an empty payload for player slots with no log, and preserves logs for every major-civ player slot that can legitimately accumulate entries under the existing UI-human restriction. It does not begin recording AI-only logs.

INFERRED: Save-size impact is small and bounded. Each eligible player writes at most 49 entries, each consisting of one localized string plus eight scalar fields. Historical restored strings remain in the language in which they were generated because the log stores rendered `CvString` messages, not localization keys.

REPORTED_REVALIDATION: Existing non-nuclear combat coverage still routes through the previously identified authoritative sites: melee, ranged, city ranged, city melee, air strike, interception, air sweep, paradrop interception, pillage, civilian capture, city capture, and extra/splash damage. Naval, embarked, carrier-aircraft, and coastal variants appear to route through those shared resolvers, but runtime scenarios remain required to prove ownership and fog-of-war behavior for each specialized case.

RESOURCE_FINDING: No resource-side file change was required. The supplied UI addin enables the correct database option and consumes the existing Lua table shape. Its `AddMessage` path assumes `Plot` is non-null for displayed entries, so DLL log entries should continue to provide a visible/known plot or avoid creating displayable entries for hidden plots.

Remaining runtime tests:

1. Lua safety with option disabled/enabled, eligible human, AI/ineligible player, empty log, populated log, and after load.

2. Save/load with empty log, several entries, wrapped ring, chronological order after reload, new events after reload, old version-0 save, enabled-at-save/disabled-at-load, disabled-at-save/enabled-at-load, single-player, and hotseat or multiplayer human slots.

3. Combat coverage for melee, ranged, city attacks, naval and embarked targets, submarines/invisible attackers, air strike, interception, air sweep, carrier aircraft, paradrop interception, pillage, civilian capture, city capture, extra/splash damage, fully intercepted Atomic Bomb, partially intercepted Nuclear Missile, and successful detonation.

4. Information safety for unseen attacker, unseen target, fogged plot or city, and events relevant to one human player but not another.

## Trigger and ownership

`MILITARYLOG` calls are made from unit capture/pillage paths and combat resolution paths.

Enable the option, execute combat, capture, pillage, and then query the military log from Lua/UI.

## Application and state

The downstream state is `CvMilitaryLog` held by `CvPlayer`. Lua can retrieve it through `CvLuaPlayer::lGetMilitaryLog`.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:768` with default `Value="0"`. Macro is `MOD_WH_MILITARY_LOG` in `CvGameCoreDLL_Expansion2/CustomMods.h:673`; declaration is `CustomMods.h:1724`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:501`.

The option gates military log allocation, the `MILITARYLOG` macro, and player log lifecycle.

| Configuration | Validity | Expected DLL behavior | Audit result |
| --- | --- | --- | --- |
| All five disabled | Supported default | Raw Lua fallback names still apply where present; no GameEvents hooks or military log allocation from these switches. | Working as intended, runtime unvalidated |
| `WH_MILITARY_LOG=1` only | Supported with external UI | Human-visible combat/capture/pillage/city events are stored in a small in-memory ring and exposed to Lua. | Partially ready; confirmed nuclear coverage defect and Lua null-safety defect |
| All five enabled | Supported but heavy-test configuration | Event hooks and military log coexist; combat logging is unrelated to GameEvents dispatch. | Not ready for normal use because `WH_MILITARY_LOG` coverage and Lua safety need fixes |
| Option toggled on after loading a save | Supported for event dispatch, risky for log | Event switches affect future calls; military log is non-serialized and recreated only for human players in `CvPlayer::Read`. | Compatibility risk |
| `WH_MILITARY_LOG=1`, Lua calls `Player:GetMilitaryLog()` for AI/minor/barbarian or before allocation | Unsupported but reachable through public binding | DLL should return an empty table or nil safely. | Confirmed defect: null dereference risk |

## Observability and validation obligations

Log allocation in player init/read, every `AddMilitaryEvent`, Lua getter entry, and null pointer state.

8. `WH_MILITARY_LOG=1`: run melee land attack, ranged attack, city ranged attack, city melee attack, naval-to-land city attack, land-to-naval/embarked target, submarine/invisible attack, air strike, air interception, air sweep, carrier aircraft strike, paradrop interception, pillage, civilian capture, city capture, splash/extra damage, and nuclear attack/interception/blast.

9. `WH_MILITARY_LOG=1`: call `Player:GetMilitaryLog()` for every player slot before combat, after combat, after save/load, and with the option disabled.

10. `WH_MILITARY_LOG=1`: late-game stress test with many air/naval combats in one turn; measure turn time, log churn, and memory before/after with a 32-bit process memory monitor.

Add assertions or warnings for `WH_MILITARY_LOG` when a combat branch exits without a log entry while the active player is involved.

Add a null-safe Lua assertion path for `GetMilitaryLog()` during development builds, then return an empty table in release behavior.

## Version differences, risks and unresolved questions

Local `git log -S'MOD_WH_MILITARY_LOG'` finds production-source provenance at `54dee87` only.

No compile-time risk was found.

`CvPlayer::Read` recreates the log for human players after load when enabled at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44427`, with a comment indicating the log is not serialized. AI player logs are not restored after load.

INFERRED: `CvLuaPlayer::lGetMilitaryLog` can dereference a null log if Lua calls it for an ineligible player or option state.

### Recovered finding: EI-1: `WH_MILITARY_LOG` does not cover nuclear combat

Confirmed defect. The special rule for this audit requires all combat actions to be logged. Nuclear attack is an authoritative combat path through `CvUnitCombat::AttackNuclear` at `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:4118`, `CvUnitCombat::GenerateNuclearCombatInfo` at `CvUnitCombat.cpp:2238`, `CvUnitCombat::ResolveCombat` at `CvUnitCombat.cpp:3097`, and `CvUnitCombat::ResolveNuclearCombat` at `CvUnitCombat.cpp:2880`. Repository search finds no `MILITARYLOG` call in the nuclear generation, blast-damage, or resolution path.

Intended behavior: a military log advertised as logging all combat actions should record nuclear launch/interception/result and affected units/cities visible to the human log owner.

Observed/static failure: no military log entry is written for the nuclear combat action; normal notifications still happen separately.

Why exposed by current architecture: the military log implementation is a set of scattered per-resolution-site calls, not a centralized `ResolveCombat` observer. Nuclear combat is a separate resolution branch and was not instrumented.

Smallest safe correction: add `MILITARYLOG` calls in `ResolveNuclearCombat` for the attacker and affected human owners, with one summary entry per nuclear strike plus optional bounded detail for intercepted/affected units.

### Recovered finding: EI-2: `CvLuaPlayer::lGetMilitaryLog` dereferences a null log

Confirmed defect. `CvLuaPlayer::lGetMilitaryLog` obtains `CvEventLog* pMilitaryLog = pkPlayer->GetMilitaryLog()` at `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:12174` and immediately calls `pMilitaryLog->GetNumEvents()` at `CvLuaPlayer.cpp:12175`. `m_pMilitaryLog` is only allocated for `eID < MAX_MAJOR_CIVS` when `MOD_WH_MILITARY_LOG` is enabled at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:842`, and on load only for `isHuman()` players at `CvPlayer.cpp:44427`.

```text
Lua Player:GetMilitaryLog()
  -> CvLuaPlayer::lGetMilitaryLog
  -> CvPlayer::GetMilitaryLog
  -> null pointer possible
```

Smallest safe correction: return an empty Lua table when `pMilitaryLog == NULL`.

Runtime validation needed: call `Players[i]:GetMilitaryLog()` for human major, AI major, city-state, barbarian, observer/hotseat if available, before and after save/load, with option on and off.

1. Fix `CvLuaPlayer::lGetMilitaryLog` to return an empty table when `GetMilitaryLog()` returns null.

`WH_MILITARY_LOG`: not ready for normal use under the stated "all combat actions" requirement. Nuclear coverage and Lua null safety should be fixed first; performance risk is bounded for retained memory but needs late-game runtime measurement.

## Source and dependency lookup

Direct references are `CustomMods.h:673`, `CustomMods.h:1089`, `CustomMods.h:1724`, `CustomMods.cpp:501`, `NewCustomModOptions.xml:768`, `CvPlayer.cpp:842`, `CvPlayer.cpp:1092`, `CvPlayer.cpp:2049`, `CvPlayer.cpp:3038`, `CvPlayer.cpp:4386`, `CvPlayer.cpp:43543`, `CvPlayer.cpp:44427`, `CvUnit.cpp:3088`, `CvUnit.cpp:10466`, `CvUnit.cpp:10474`, `CvUnit.cpp:20216`, `CvUnitCombat.cpp:390`, `CvUnitCombat.cpp:397`, `CvUnitCombat.cpp:421`, `CvUnitCombat.cpp:436`, `CvUnitCombat.cpp:471`, `CvUnitCombat.cpp:478`, `CvUnitCombat.cpp:884`, `CvUnitCombat.cpp:916`, `CvUnitCombat.cpp:956`, `CvUnitCombat.cpp:1015`, `CvUnitCombat.cpp:1099`, `CvUnitCombat.cpp:1109`, `CvUnitCombat.cpp:1231`, `CvUnitCombat.cpp:1252`, `CvUnitCombat.cpp:1259`, `CvUnitCombat.cpp:1273`, `CvUnitCombat.cpp:1280`, `CvUnitCombat.cpp:1295`, `CvUnitCombat.cpp:1302`, `CvUnitCombat.cpp:1678`, `CvUnitCombat.cpp:1685`, `CvUnitCombat.cpp:1695`, `CvUnitCombat.cpp:1702`, `CvUnitCombat.cpp:1732`, `CvUnitCombat.cpp:1737`, `CvUnitCombat.cpp:1746`, `CvUnitCombat.cpp:1752`, `CvUnitCombat.cpp:1780`, `CvUnitCombat.cpp:1787`, `CvUnitCombat.cpp:1797`, `CvUnitCombat.cpp:1804`, `CvUnitCombat.cpp:1840`, `CvUnitCombat.cpp:1847`, `CvUnitCombat.cpp:2108`, `CvUnitCombat.cpp:2115`, `CvUnitCombat.cpp:2140`, `CvUnitCombat.cpp:2154`, `CvUnitCombat.cpp:2184`, `CvUnitCombat.cpp:2191`, `CvUnitCombat.cpp:3038`, `CvUnitCombat.cpp:3492`, `CvUnitCombat.cpp:3499`, and `CvUnitCombat.cpp:4413`.

`CvPlayer::init` allocates `m_pMilitaryLog` for major civs when enabled at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:842`. `CvPlayer::Uninit` clears it at `CvPlayer.cpp:1092`; `CvPlayer::Init` initializes it at `CvPlayer.cpp:2049`; `CvPlayer::AddMilitaryEvent` writes entries at `CvPlayer.cpp:43541`.

`CvLuaPlayer::lGetMilitaryLog` exposes the log to Lua in `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:12167`.

GUIDE_PATTERN: The XML comment indicates this option requires `UI - Military Log`. Audited repository search did not find a bundled UI implementation beyond DLL Lua exposure.

INFERRED: The feature assumes UI/Lua callers only query the log for players that have a non-null `m_pMilitaryLog`.

INFERRED: If current UI no longer includes the military log screen, logging can still occur in DLL but be invisible.

Save after log entries exist, reload, and verify expected non-serialized behavior and Lua null safety.

`NOT_READY_BASELINE_DEFECT_REPORTS`

The log container is bounded, in-memory, and non-serialized. `CvEventLog` defines `MAX_EVENTS 50` at `CvGameCoreDLL_Expansion2/CvEventLog.cpp:7`, allocates that vector in `CvEventLog::Init` at `CvEventLog.cpp:35`, and drops the oldest entry when `end + 1 == begin` at `CvEventLog.cpp:155`. Because one slot is reserved to distinguish full from empty, effective retained capacity is 49 entries.

`CvEventLog::Add` refuses to record for non-human UI players at `CvGameCoreDLL_Expansion2/CvEventLog.cpp:62`. `CvPlayer::Read` recreates a log only for human players when enabled at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44427`.

Confirmed defect: `CvLuaPlayer::lGetMilitaryLog` dereferences `pMilitaryLog` without a null check at `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:12174` and `CvLuaPlayer.cpp:12175`. The Lua method is registered regardless of option state, while `m_pMilitaryLog` is option- and player-kind-dependent. Smallest safe correction is returning an empty table when `pMilitaryLog == NULL`.

Confirmed defect: nuclear combat is not logged. The authoritative path is `CvUnitCombat::AttackNuclear` at `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:4118`, `CvUnitCombat::GenerateNuclearCombatInfo` at `CvUnitCombat.cpp:2238`, `CvUnitCombat::ResolveCombat` at `CvUnitCombat.cpp:3097`, and `CvUnitCombat::ResolveNuclearCombat` at `CvUnitCombat.cpp:2880`; no `MILITARYLOG` call exists in that path.

Covered combat resolution functions include melee at `CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:307`, ranged unit attack at `CvUnitCombat.cpp:831`, city ranged attack at `CvUnitCombat.cpp:1069`, city melee at `CvUnitCombat.cpp:1161`, air attack at `CvUnitCombat.cpp:1589`, air sweep at `CvUnitCombat.cpp:2042`, paradrop interception at `CvUnitCombat.cpp:2954`, and extra unit damage at `CvUnitCombat.cpp:4381`.

INFERRED: Naval-to-land attacks, embarked target attacks, coastal city attacks, and carrier aircraft attacks should route through the covered melee/ranged/city/air resolution functions, but runtime tests are required to prove message ownership and visibility are correct for each scenario.

Performance estimate: retained memory is bounded and likely small for a 32-bit process because at most 50 `EventLogEntry` objects are allocated per major-civ log and actual retained entries are human-only through `CvEventLog::Add`. CPU and transient allocation cost matter more: with the option enabled, combat paths format localized `CvString` messages and write ring entries. Late-game air/naval wars can churn the 49-entry ring quickly, so any patch adding nuclear or splash detail must remain bounded and avoid one message per affected plot unless explicitly desired.

Smallest safe corrections:

1. Return an empty table from `CvLuaPlayer::lGetMilitaryLog` if `GetMilitaryLog()` is null.

2. Add visibility-safe, bounded `MILITARYLOG` entries in `CvUnitCombat::ResolveNuclearCombat`.

3. Add diagnostic assertions/logging around combat branches that affect the active human but do not write a military log entry.

Runtime scenario needed: melee land attack, ranged attack, city ranged attack, city melee attack, naval-to-land/city attack, embarked target attack, submarine/invisible attack, air strike, interception, air sweep, carrier aircraft strike, paradrop interception, pillage, civilian capture, city capture, extra damage/splash, nuke interception, and nuclear blast.

Features audited in this pass: `EVENTS_REBASE`, `EVENTS_CUSTOM_MISSIONS`, `EVENTS_RESOLUTIONS`, `EVENTS_UNIT_CREATED`, `WH_MILITARY_LOG`

| Feature | Row | Macro | Cache |
| --- | --- | --- | --- |
| `WH_MILITARY_LOG` | `(1) Community Patch/Database Changes/NewCustomModOptions.xml:768` | `CvGameCoreDLL_Expansion2/CustomMods.h:673` | `CvGameCoreDLL_Expansion2/CustomMods.cpp:501` |

`WH_MILITARY_LOG` allocates `CvEventLog` for major-civ player slots at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:842`, initializes it at `CvPlayer.cpp:2049`, writes through `CvPlayer::AddMilitaryEvent` at `CvPlayer.cpp:43541`, exposes it to Lua through `CvGameCoreDLL_Expansion2/Lua/CvLuaPlayer.cpp:12167`, and writes from `MILITARYLOG` macro sites in `CvUnit.cpp`, `CvPlayer.cpp`, and `CvUnitCombat.cpp`.

Covered authoritative combat resolution functions include melee (`CvGameCoreDLL_Expansion2/CvUnitCombat.cpp:307`), ranged unit vs unit/city target (`CvUnitCombat.cpp:831`), city ranged attack (`CvUnitCombat.cpp:1069`), city melee (`CvUnitCombat.cpp:1161`), air unit vs unit/city (`CvUnitCombat.cpp:1589`), air sweep (`CvUnitCombat.cpp:2042`), paradrop interception (`CvUnitCombat.cpp:2954`), and extra unit damage (`CvUnitCombat.cpp:4381`) through existing `MILITARYLOG` call sites.

`CvEventLog` is bounded by `MAX_EVENTS 50` at `CvGameCoreDLL_Expansion2/CvEventLog.cpp:7`; because the ring treats `end + 1 == begin` as full at `CvEventLog.cpp:155`, effective retained entries are 49. Entries are not serialized; `CvPlayer::Read` recreates only human logs at `CvGameCoreDLL_Expansion2/CvPlayer.cpp:44427`.

INFERRED: Memory impact is small and bounded for normal play: each allocated major-civ log stores 50 `EventLogEntry` objects, each with a `CvString`, player ID, turn, coordinates, and four data integers. Since `CvEventLog::Add` drops non-human UI players at `CvGameCoreDLL_Expansion2/CvEventLog.cpp:62`, active message storage is effectively human-only, but allocation still occurs for major-civ slots when the option is enabled.

INFERRED: CPU/string-allocation impact is more important than retained memory. Many call sites format localized text before calling `MILITARYLOG`, but most are already inside `if (MOD_WH_MILITARY_LOG)` branches. With the option enabled, every logged combat result allocates/transcodes localized strings and may add ring entries. The impact should be modest in normal combat, but late-game air/naval wars can generate enough messages that duplicate/detail-heavy fixes for nuclear or splash damage must stay bounded.

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `WH_MILITARY_LOG` | Enable option, execute combat, capture, pillage, and query log from Lua/UI. | Save after log entries, reload, verify documented non-serialized behavior and Lua null safety. | `CvPlayer::AddMilitaryEvent`, `CvLuaPlayer::lGetMilitaryLog`, `MILITARYLOG` macro sites. | `CvPlayer.cpp:43541`, `CvLuaPlayer.cpp:12167` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `WH_MILITARY_LOG=1` Lua null safety | Call `Player:GetMilitaryLog()` for every player slot with option on and off, before combat and after load. | Save before and after entries exist; reload and call getter for every slot. | Getter should return an empty table for players without a log instead of dereferencing null. | Confirmed defect at `CvLuaPlayer.cpp:12174` and `CvLuaPlayer.cpp:12175`. |
| `WH_MILITARY_LOG=1` exhaustive combat coverage | Execute melee land, ranged, city ranged, city melee, naval-to-land/city, embarked target, submarine/invisible, air strike, interception, air sweep, carrier aircraft, paradrop interception, pillage, civilian capture, city capture, extra damage, and nuclear attack. | Save before each combat and after log entries; reload and compare log behavior. | Every human-relevant combat action should create bounded, visibility-safe log entries. | Confirmed nuclear gap and coverage validation for `CvUnitCombat.cpp` combat paths. |
| `WH_MILITARY_LOG=1` late-game performance | Run a late-game war turn with many air/naval combats and log polling from UI. | Save before stress turn; reload and replay with logging enabled/disabled. | Retained memory should remain bounded by the 49-entry ring; turn-time and transient string allocation cost should be measured. | Performance/OOM risk in 32-bit process; `CvEventLog.cpp:7`, `CvEventLog.cpp:40`, and `CvEventLog.cpp:155`. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
