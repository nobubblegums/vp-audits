# Military log UI integration reference and design

Knowledge status: Maintained design/reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. Proposed designs are not implemented behavior. See [baseline and revalidation](<../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How can military-log data be displayed alongside existing notification controls?

Observed UI/data contracts are distinct from the proposed tab integration. Ranked options, control hierarchy and implementation phases remain a design proposal, not a claim that the tab exists.

Canonical related owners: [Military event log and Lua contract](<military-log.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Data and configuration

`CvLuaPlayer::lGetMilitaryLog` is registered by `Method(GetMilitaryLog)` in `CvLuaPlayer.cpp` and declared as `LUAAPIEXTN(GetMilitaryLog, table)` in `CvLuaPlayer.h`.

`Player:GetMilitaryLog()` returns a Lua array table. Each entry has these fields:

| Field | Type | Meaning | Always present |
| --- | --- | --- | --- |
| `Turn` | integer | Game turn stored in `CvEventLog::EventLogEntry::m_iTurn` | Yes for each returned entry |
| `Message` | string | Already-rendered `CvString` message stored at event creation or restored from save | Yes for each returned entry |
| `OtherPlayer` | integer / `PlayerTypes` | Other player id stored with the event; may be `NO_PLAYER` | Yes for each returned entry |
| `Plot` | `CvPlot` userdata or nil | Plot resolved from stored x/y through `GC.getMap().plot(...)` | Field is set; value may be nil if x/y are invalid |
| `Data1` | integer | Optional event data, default `-1` | Yes |
| `Data2` | integer | Optional event data, default `-1` | Yes |
| `Data3` | integer | Optional event data, default `-1` | Yes |
| `Data4` | integer | Optional event data, default `-1` | Yes |

Ordering is chronological retained order from oldest to newest. `CvEventLog::GetTurn/GetMessage/...` read index `(begin + i) % size`; `lGetMilitaryLog` loops `i = 0 .. GetNumEvents() - 1` and writes Lua array indices starting at `1`.

Maximum retained entries are 49 because `CvEventLog` allocates `MAX_EVENTS = 50` and reserves one ring slot to distinguish full from empty.

Feature disabled, unavailable log, or ineligible/null player log now returns an empty table. The Lua getter creates the table before checking `pMilitaryLog` and returns it immediately on null.

Empty log returns an empty table because `GetNumEvents()` is `0`.

The log records only for players passing `CvEventLog::Add`'s `GET_PLAYER(m_ePlayer).isHuman(ISHUMAN_UI)` check. Current branch load logic allocates/keeps a log for major-civ slots only when `MOD_WH_MILITARY_LOG` is enabled, and deletes the pointer otherwise. AI-only logs are not recorded.

Save/load after the current branch changes preserves retained entries for save version `SAVE_VERSION_MILITARY_LOG = 1+`. Older version-0 saves load without reading military-log bytes and initialize an empty log.

INFERRED: The existing Lua API is sufficient for the proposed Combat tab. A UI can display message, turn, other-player icon, and click-to-plot behavior. DLL changes would only be needed for future features such as structured combat categories, localization-key replay, or entries without safe plot coordinates.

### Option A: integrate existing Military Log UI into a second container

Files likely changed:

- `(1) Community Patch/Core Files/Overrides/NotificationLogPopup.lua`

- `(1) Community Patch/Core Files/Overrides/NotificationLogPopup.xml`

- text XML/SQL for any new empty/disabled strings if not reusing existing text

- possibly the modmod `UIMilitaryLog.xml` if this remains modmod-owned

Reuse:

- Reuses `MilitaryLogPopup.lua`'s `RefreshList`, `AddMessage`, newest-first loop, icon, turn, and click behavior.

- Reuses `MilitaryLogPopup.xml`'s row layout concept.

Duplicated code:

- Would duplicate some notification row resizing/scroll behavior unless normalized.

- Existing military popup uses `InstanceManager`, while Notification Log currently uses CPK `MakeInstance`/`FreeInstance`; mixing both is possible but less consistent.

Regression risk:

- Medium. Directly transplanting the military popup can preserve behavior but may introduce separate conventions inside a CPK-based popup.

- Needs explicit nil-plot and `NO_PLAYER` icon guards because the old separate popup lacks them.

Maintenance burden:

- Medium. Two list implementations remain in the same popup: notification instances and military instances.

Compatibility:

- Good with disabled options if guarded by `Game.IsCustomModOption` and the null-safe Lua getter.

- The separate Military Log UI can remain installed during transition, but users would see both the integrated tab and Additional Information launcher unless the old addin is retired or suppressed later.

User experience:

- Good if visually restyled to match Notification Log. Fastest path to preserve current military-log behavior.

### Option B: rewrite only the display layer

- text XML/SQL for optional empty/disabled strings and possibly `TXT_KEY_POP_MILITARY_LOG` if base VP owns the UI

- Reuses the existing `Player:GetMilitaryLog()` API and current Notification Log instance/refresh idioms.

- Reuses VP tab patterns from `CorporationsOverview`.

- Reuses visual concepts from the old military popup without copying its popup lifecycle.

- Minimal if a small helper is added for row resizing and scroll refresh.

- Combat list still needs its own instance table because notification rows are keyed by notification id and activate notifications, while combat rows look at plots.

- Low to medium. It touches the Notification Log popup but keeps notification retrieval/filter state intact and adds a separate container.

- Lowest. One popup owns its own tab state and uses the same CPK instance style throughout.

- Best with disabled options. Combat tab can be hidden, disabled, or shown empty without loading the old addin.

- Compatible with the current separate Military Log UI while both coexist, though duplicate entry points should be addressed in a retirement phase.

- Best match for the requested true integrated tab.

### Option C: retain separate popup and add a launcher tab/button

- Notification Log Lua/XML only, if adding a button that calls the existing addin through a LuaEvent or duplicated launcher.

- Possibly Military Log popup Lua to expose an open event.

- Maximal reuse of existing separate popup.

- Low, but the integration is superficial.

- Low for Notification Log, medium for popup focus/stack behavior because two contexts interact.

- Medium. Two popups must be kept coherent.

- Depends on the old addin being installed. If integrated into base VP without the modmod addin, this option has no target popup.

- Does not satisfy the stated desired design because the Combat "tab" would not be its own content view inside Notification Log.

Recommended behavior needs a product decision:

- Hidden tab: safest for base VP if `WH_MILITARY_LOG` is disabled or absent. Users see the current Notification Log unchanged.

- Disabled tab: discoverable but needs disabled styling and tooltip/localization.

- Shown empty: easiest technically because `GetMilitaryLog()` returns `{}`, but it exposes a feature that cannot record entries and may look broken.

Existing VP Lua can inspect custom mod options with `Game.IsCustomModOption("OPTION")` where available, as seen in `InGame.lua` and `WorldView.lua`. VPUI also uses `GameInfo.CustomModOptions("Name = 'BALANCE_CORE_JFD'")().Value == 1` as a database fallback. Use `Game.IsCustomModOption("WH_MILITARY_LOG")` first and keep the getter call null-safe.

1. Preserve scroll position separately for Notifications and Combat.

2. Add empty and disabled tooltips.

3. Add turn separators if desired.

4. Add subtle Combat row icon fallback for `NO_PLAYER`.

5. Screenshot-test layout at small/large resolutions and UI scaling.

6. Consider remembering last selected top-level tab for the current session.

## Observability and validation obligations

| Area | Scenario | Expected result |
| --- | --- | --- |
| Popup baseline | Open Notification Log with `WH_MILITARY_LOG=0` | Existing notification history/settings work unchanged; Combat tab follows chosen hidden/disabled/empty behavior |
| Popup baseline | Open/close via normal notification-log popup and utmost toggle | Queue/dequeue, turn timer semaphore, and processed/shown events remain correct |
| Notifications tab | Existing notification row click | Popup closes and `UI.ActivateNotification` fires |
| Notifications tab | Existing settings view, kind filters, enable/disable all | Existing player option state changes correctly |
| Tab switching | Switch Notifications -> Combat -> Notifications | Notification subview and filter state are preserved |
| Combat empty | Enabled option, no entries | Empty-state text; no Lua errors |
| Combat populated | Several entries | Newest-first rows with turn, message, other-player icon where valid |
| Combat click | Entry with plot | Click centers map via `UI.LookAt` |
| Combat nil plot | Entry with invalid/no plot | Row displays but click is disabled or safe |
| Other player | `OtherPlayer == NO_PLAYER` | No `CivIconHookup(-1)` crash; icon hidden or fallback |
| Active player | Hotseat active player changes | Popup closes or refreshes without stale rows |
| Multiplayer | Multiple human UI players where feasible | Each local human sees only eligible log entries |
| Save/load | Reload with retained log entries | Combat tab shows restored chronological data newest-first |
| Old save | Load version-0 save | Empty Combat tab; no stream/UI errors |
| Feature toggles | Enabled-at-save/disabled-at-load and inverse | UI remains safe; no stale assumptions |
| Layout | Low resolution / UI scaling / long localized strings | No overlap, clipped rows, or unusable scrollbars |
| Coexistence | Old separate Military Log popup also installed | Both entry points work or documented duplicate is acceptable during transition |

## Version differences, risks and unresolved questions

### Compatibility risks

1. Existing separate Military Log addin: If left installed, users may have both an Additional Information dropdown entry and the integrated Combat tab. This is acceptable during transition but should be retired deliberately.

2. EUI/common UI mods: `Merged UI Mods.modinfo` depends on VP EUI compatibility files. Editing VP's override directly can conflict with any mod that also replaces `NotificationLogPopup.lua/xml`.

3. XML control IDs: Existing IDs such as `LogsView`, `OptsView`, `LogsScroll`, `LogsStack`, `CloseButton`, `EnableCurrOptsButton`, and `DisableCurrOptsButton` should remain stable to avoid breaking existing Lua.

4. Context lifetime: Notification Log context is queued/dequeued, not simply hidden. New Combat instances should be freed/rebuilt consistently to avoid stale rows across active-player changes.

5. Hotseat/active-player changes: Both current popups close or refresh on `GameplaySetActivePlayer`; integrated code must clear combat instances and refill from new `Game.GetActivePlayer()`.

6. Multiplayer: `CvEventLog::Add` uses `isHuman(ISHUMAN_UI)`. Runtime tests must confirm that each local/hotseat player sees only their own eligible log.

7. Observer mode/ineligible players: `GetMilitaryLog()` can return empty. Combat UI must tolerate empty table and nil plots.

8. Save/load: Current branch serializes retained entries. UI should not duplicate entries after load; it should just display `GetMilitaryLog()` output.

9. Localization: Restored historical military messages are already-rendered strings and may remain in the language in which they were generated.

10. UI scaling/resolution: Existing popup uses fixed `960x658` geometry. Adding a second top-level tab row plus retaining the existing notification/settings switch may require vertical space changes and screenshot validation.

11. Other-player icon: `OtherPlayer` can be `NO_PLAYER`; combat UI must not blindly call `CivIconHookup(-1, ...)`.

12. Plot click behavior: Existing separate popup assumes non-null plot. Integrated tab should disable click or omit callback when `Plot` is nil.

User approval recommended before implementation:

1. Default selected tab: preserve Notifications, or remember last selected tab during the session.

2. Combat tab unavailable behavior: hide, disable with tooltip, or show empty.

3. Ownership: integrate into base VP override, keep as `UI - Military Log` modmod patch, or maintain both.

4. Retirement: keep or remove the old Additional Information dropdown Military Log popup after integration.

5. Combat ordering: newest first to match current military UI, or oldest first to match Lua chronological order.

6. Refresh policy: refresh only on tab select/open/turn change, or add automatic refresh while the popup remains open.

7. Scroll preservation: retain separate scroll position per tab, or reset on refresh.

8. Notification settings visibility: hide existing notification settings buttons while Combat tab is selected, or move settings under a sub-tab/menu.

9. Empty-state copy and localization tag names.

10. Whether future Combat filters are out of scope permanently or merely deferred.

11. Whether popup title remains "Notification Log" or changes with selected tab.

12. Whether turn headings/separators should be added as polish.

GO for a scoped UI implementation after design decisions are made.

Reasons:

- The existing Lua API supplies all required display data for a straightforward Combat tab.

- Current branch null-safety and save/load behavior make the UI safe for disabled, empty, and restored logs.

- VP has reusable tab patterns, especially `CorporationsOverview`, that fit a true two-tab design.

- The Notification Log popup already has fixed content containers and instance-managed rows, so a separate Combat panel can be added without rewriting notification filters.

Conditions before implementation:

- Decide whether the Combat tab is hidden, disabled, or shown empty when unavailable.

- Decide ownership: base VP integration versus modmod-owned integration.

- Decide what to do with the old Additional Information dropdown popup.

- Validate layout with screenshots because the current popup already uses most of the vertical space.

## Source and dependency lookup

VP overrides the Notification Log popup with `(1) Community Patch/Core Files/Overrides/NotificationLogPopup.lua` and `(1) Community Patch/Core Files/Overrides/NotificationLogPopup.xml`. No second/base-game `NotificationLogPopup.lua` or `.xml` copy is present in the audited repository tree, so inherited behavior was inferred only from the VP override shape and the popup event names.

`NotificationLogPopup.lua` includes `CPK.lua` and `IconSupport`, aliases `Controls` to `C`, and uses `CPK.UI.Control.Show`, `Hide`, `Refresh`, `Instance.Make`, and `Instance.Free`. The supporting CPK helpers live under `(1) Community Patch/Kit/`; `CPK.UI.Control.Refresh` recalculates stack size/reanchoring and scroll-panel internal size, while `CPK.UI.Control.Instance.Make` wraps `ContextPtr:BuildInstanceForControl`.

Popup open/close flow:

- `Events.SerialEventGameMessagePopup.Add` listens for `ButtonPopupTypes.BUTTONPOPUP_NOTIFICATION_LOG`.

- The handler stores `m_info`, calls `ShowPopup(false)` for normal opens, and toggles utmost popup display when `info.Data1 == 1`.

- `ShowPopup` calls `UIManager:QueuePopup(ContextPtr, PopupPriority.NotificationLog)` or `PopupPriority.InGameUtmost`.

- `HidePopup` calls `UIManager:DequeuePopup(ContextPtr)`.

- `ContextPtr:SetInputHandler` closes on Escape or Return.

- `Events.GameplaySetActivePlayer.Add(HidePopup)` closes the popup on active-player change.

- `ContextPtr:SetShowHideHandler` increments/decrements the turn timer semaphore and emits `SerialEventGameMessagePopupShown` / `SerialEventGameMessagePopupProcessed`.

The current popup has two internal views, not true top-level tabs:

- `VIEW_LOGS`: existing notification history.

- `VIEW_OPTS`: instant-yield notification settings.

`ShowLogsView` and `ShowOptsView` hide/show `LogsView`, `OptsView`, their selected-state decor grids, and the bottom right enable/disable buttons. This is a view switch with two buttons (`ShowLogsView`, `ShowOptsView`) rather than a reusable tab framework.

Notification data sources are active-player notification methods exposed through Lua:

- `Players[m_player_id]:GetNumNotifications()`

- `GetNotificationIndex(i)`

- `GetNotificationDismissed(i)`

- `GetNotificationStr(i)`

- `GetNotificationTurn(i)`

`FillLogsView` builds notification rows by iterating indices `0 .. GetNumNotifications() - 1`. For each entry it reuses an existing instance keyed by notification id when possible, otherwise creates `LogInstance`. It sets turn text with `TXT_KEY_TP_TURN_COUNTER`, sets rendered notification text, resizes hover/button controls to the text-stack height, stores the notification id in `LogButton` void1, disables dismissed notifications, and registers click behavior through `GoToEvent`.

Notification selection behavior is direct activation: `GoToEvent(log_n_id)` hides the popup and calls `UI.ActivateNotification(log_n_id)`.

Notification list ordering is the engine's notification index order as returned by `GetNotificationIndex(i)`. There is no Lua sort for the notification list. The XML `LogsStack` uses `StackGrowth="Up"`, so visual ordering must be validated in game rather than assumed solely from loop direction.

Notification filtering in this popup is not filtering the notification history. The "settings" view filters instant-yield notification option rows by kind. `InitOptsView` categorizes `InstantYieldType` values into city, military, civilian, espionage, misc, or skip; creates kind radio buttons in `OptKindsStack`; creates option rows in `OptsStack`; and `FilterOpts(opt_kind)` hides/shows option rows.

Filter state is stored on the player via `SetInstantYieldNotificationDisabled` and read through `IsInstantYieldNotificationDisabled`. The currently selected option-kind radio state is UI-local and not explicitly persisted beyond the live popup context.

Instance managers/state:

- `m_logs` stores active notification row instances keyed by notification id.

- `m_opts` stores instant-yield option row instances keyed by instant-yield type.

- `m_player_id` tracks the active player.

- `m_turn` tracks the last refreshed game turn.

- `m_view` tracks current internal view.

- `m_info` stores the popup info from the popup event.

Refresh behavior:

- On initial show-hide init, `Init` sets the active player, fills the civ icon treatment, initializes options, fills options and logs, defaults to logs view, and registers callbacks.

- On active-player change while the context exists, the show-hide handler updates `m_player_id`, frees notification rows, refills option/log views, and refreshes the civ icon.

- On turn change, it refills only the notification log view.

XML hierarchy:

- `LogInstance`: `Button LogButton`, right-aligned `LogTurnLabel`, `LogTextStack`, `LogTextLabel`, separator, hover highlight.

- `OptInstance`: `Button OptButton`, checkbox, text/kind labels, separator, hover highlight.

- `OptKindInstance`: `RadioButton OptKindButton` in radio group `Kind`.

- Main `Grid` has civ icon treatment, two view-switch buttons, `LogsView` container with `LogsScroll`/`LogsStack`, `OptsView` container with kind radios and `OptsScroll`/`OptsStack`, close button, and option enable/disable buttons.

Screen sizing is fixed around a `960x658` grid. Both current content views occupy `952x550`, offset under the view switch. `LogsScroll` has vertical scroll controls and `AutoSize="1"`. `OptsScroll` has a slightly smaller height because the option-kind row consumes vertical space.

Localization used by the popup includes `TXT_KEY_POP_NOTIFICATION_LOG`, `TXT_KEY_NOTIFICATION_SETTINGS_TITLE`, `TXT_KEY_NOTIFICATION_SETTINGS_*`, `TXT_KEY_TP_TURN_COUNTER`, and `TXT_KEY_CLOSE`.

INFERRED: The popup does not have latent true-tab support. The existing two-button view switch can be adapted, but a true Notifications/Combat tab should be conceptually separate from the current Logs/Settings view pair. Otherwise the Combat tab risks becoming a third button beside notification settings, which would blur the requested tab model.

The supplied UI is under `codex/modmod_resources/Merged UI Mods/MilitaryLog/`.

`XML/UIMilitaryLog.xml` updates `CustomModOptions` row `WH_MILITARY_LOG` to `Value=1` and defines localization `TXT_KEY_POP_MILITARY_LOG` as "Military Log".

`Merged UI Mods.modinfo` imports `MilitaryLog/Art/MilitaryIcon.dds`, runs `UpdateDatabase` on `MilitaryLog/XML/UIMilitaryLog.xml`, and registers `MilitaryLog/UI/MilitaryLogPopup.xml` as an `InGameUIAddin`. The modinfo advertises single-player, multiplayer, and hotseat support and `AffectsSavedGames=0`.

`MilitaryLogPopup.lua` opens through the Additional Information dropdown, not through `ButtonPopupTypes.BUTTONPOPUP_NOTIFICATION_LOG`. It registers `LuaEvents.AdditionalInformationDropdownGatherEntries`, inserts an entry with text `TXT_KEY_POP_MILITARY_LOG`, callback `OnDiploCornerPopup`, and art `MilitaryIcon.dds`, then requests dropdown refresh.

`OnDiploCornerPopup` shows the popup by `ContextPtr:SetHide(false)`. `OnClose` hides it by `ContextPtr:SetHide(true)`. Escape/Return close it. `Events.GameplaySetActivePlayer.Add(OnClose)` closes it on active-player change.

`RefreshList` is called when the context becomes visible. It updates active-player civ icon treatment, resets the `InstanceManager` for `LogEntry`, calls `Players[Game.GetActivePlayer()]:GetMilitaryLog()`, and loops from `#playerLog` down to `1` before adding messages. This displays newest retained entry first if the Lua table is chronological oldest-to-newest.

`AddMessage(iTurn, sMsg, pPlot, iOtherPlayer)` creates a row, calls `CivIconHookup(iOtherPlayer, 32, ...)`, sets message text and turn label, recalculates text stack, resizes button/hover controls, stores plot x/y via `Button:SetVoids`, and registers click behavior. `MessageSelected` calls `UI.LookAt(Map.GetPlot(iX, iY), 0)`.

`MilitaryLogPopup.xml` uses one `LogEntry` instance with a 32px other-player icon, right turn label, wrapped message label, separator, and hover highlight. The popup has a top civ icon, `TXT_KEY_POP_MILITARY_LOG` title, one scroll panel, close button, and no filters, categories, or settings.

Empty-log behavior is implicitly an empty list: `#playerLog == 0` skips the loop and no empty-state text is shown.

Option-disabled/null-log behavior after the current branch changes is also an empty list because `Player:GetMilitaryLog()` returns `{}` if `CvPlayer::GetMilitaryLog()` is null.

The existing popup assumes active-player context and does not support inspecting another player's log. It also assumes each displayed entry has a non-null `Plot`; `pPlot:GetX()` would fail for entries with no plot.

INFERRED: Useful behavior to preserve in an integrated tab is newest-first ordering, turn label, clickable plot navigation, other-player civ icon where valid, and simple no-filter list behavior. The separate top civ treatment is already present in Notification Log and should not be duplicated inside the tab.

`CorporationsOverview` is the clearest reusable VP tab pattern. `(2) Vox Populi/Core Files/New UI/CorporationsOverview.lua` defines `g_Tabs` mapping tab names to `Panel` and `SelectHighlight`, then `TabSelect(tab)` hides all non-selected panels/highlights, stores `g_CurrentTab`, and calls selected tab `RefreshContent()`. Its XML has a top tab bar with equal-width buttons, hover highlight, selected-state `Grid9FrameTurns` grids, and separate content panels.

`EventOverview` provides an older explicit version of the same idea: each button handler hides all tab highlights and all content panels except the selected one, with tab buttons in a top panel and selected-state grids in XML.

The current Notification Log view switch has similar visual controls (`ShowLogsView`, `ShowOptsView`, `LogsViewDecor`, `OptsViewDecor`) but it represents notification history versus notification settings, not two peer data domains. It can be reused visually, but the architecture should be renamed or nested to avoid treating Combat as another notification settings view.

RECOMMENDED_PATTERN: Use the `CorporationsOverview` table-driven pattern for true top-level tabs, with separate panel containers and selected highlight controls. It is compact, already VP-native, and avoids inventing a new framework.

The least invasive hierarchy is to split the popup into top-level tabs, then keep the existing notification history/settings switch inside the Notifications tab.

Proposed XML sketch:

```xml
<Grid ID="Root" Size="960,658" ...>
  <Image ID="CivIconTreatment" ... />

  <!-- Top-level true tabs -->
  <Container ID="TopTabs" Anchor="C,T" Size="950,32" Offset="0,4">
    <Button ID="NotificationsTabButton" String="TXT_KEY_POP_NOTIFICATION_LOG" Size="475,28">
      <Grid ID="NotificationsTabHighlight" Style="Grid9FrameTurns" />
    </Button>
    <Button ID="CombatTabButton" String="TXT_KEY_POP_MILITARY_LOG" Size="475,28">
      <Grid ID="CombatTabHighlight" Style="Grid9FrameTurns" Hidden="1" />
    </Button>
    <Image Texture="HorizontalTrim.dds" ... />
  </Container>

  <Container ID="NotificationsTabPanel" Size="952,550" Offset="4,41">
    <!-- Existing Logs/Settings view switch, preferably shifted down inside this panel -->
    <Container ID="NotificationSubViewSwitch">...</Container>
    <Container ID="LogsView">...</Container>
    <Container ID="OptsView">...</Container>
  </Container>

  <Container ID="CombatTabPanel" Size="952,550" Offset="4,41" Hidden="1">
    <ScrollPanel ID="CombatScroll" Size="952,550" Vertical="1">
      <UpButton ... />
      <ScrollBar ... />
      <DownButton ... />
      <Stack ID="CombatStack" StackGrowth="Bottom" />
    </ScrollPanel>
    <Label ID="CombatEmptyLabel" String="TXT_KEY_MILITARY_LOG_EMPTY" Hidden="1" />
  </Container>

  <Container ID="BottomPanel">
    <GridButton ID="CloseButton" ... />
    <Stack ID="NotificationSettingsButtons">
      <GridButton ID="DisableCurrOptsButton" ... />
      <GridButton ID="EnableCurrOptsButton" ... />
    </Stack>
  </Container>
</Grid>

<Instance Name="CombatLogInstance">
  <Button ID="CombatButton">
    <Box ID="OtherIconBox">
      <Image ID="OtherIconBG" />
      <Image ID="OtherIconShadow" />
      <Image ID="OtherIcon" />
    </Box>
    <Label ID="CombatTurnLabel" />
    <Stack ID="CombatTextStack">
      <Label ID="CombatTextLabel" />
      <Image Texture="bar900x2.dds" />
    </Stack>
    <ShowOnMouseOver>...</ShowOnMouseOver>
  </Button>
</Instance>
```

INFERRED: The existing `LogsView` and `OptsView` may need vertical resizing if the current two-button view switch remains visible inside the Notifications tab. Alternative: keep Notification Log and Settings as two buttons where they are today, and place the top-level tabs above them or rename the existing row to top-level tabs while moving Settings elsewhere. This is a UI design decision.

Pseudocode for top-level tab switching:

```lua
local TAB_NOTIFICATIONS = "Notifications"
local TAB_COMBAT = "Combat"
local m_tab = TAB_NOTIFICATIONS
local m_combat_logs = {}

local Tabs = {
  Notifications = {
    Panel = C.NotificationsTabPanel,
    Highlight = C.NotificationsTabHighlight,
    Refresh = function()
      FillLogsView()
      -- Keep existing notification subview state; do not reset settings/logs switch.
    end
  },
  Combat = {
    Panel = C.CombatTabPanel,
    Highlight = C.CombatTabHighlight,
    Refresh = FillCombatView
  }
}

local function SelectTab(tab)
  if m_tab == tab then return end
  for id, data in pairs(Tabs) do
    local hide = id ~= tab
    data.Panel:SetHide(hide)
    data.Highlight:SetHide(hide)
  end
  m_tab = tab
  Tabs[tab].Refresh()
  C.NotificationSettingsButtons:SetHide(tab ~= TAB_NOTIFICATIONS or m_view ~= VIEW_OPTS)
end
```

Pseudocode for combat population:

```lua
local function FreeCombatLogs()
  for _, inst in pairs(m_combat_logs) do
    FreeInstance(inst)
  end
  m_combat_logs = {}
end

local function IsMilitaryLogAvailable()
  if Game.IsCustomModOption ~= nil then
    return Game.IsCustomModOption("WH_MILITARY_LOG")
  end
  local row = GameInfo.CustomModOptions and GameInfo.CustomModOptions("Name = 'WH_MILITARY_LOG'")()
  return row ~= nil and row.Value == 1
end

local function FillCombatView()
  FreeCombatLogs()

  local player = Players[m_player_id]
  local logs = player and player.GetMilitaryLog and player:GetMilitaryLog() or {}

  C.CombatEmptyLabel:SetHide(#logs > 0)
  C.CombatEmptyLabel:SetText(IsMilitaryLogAvailable()
    and L("TXT_KEY_MILITARY_LOG_EMPTY")
    or L("TXT_KEY_MILITARY_LOG_DISABLED"))

  for i = #logs, 1, -1 do
    local e = logs[i]
    local inst = MakeInstance("CombatLogInstance", C.CombatStack)
    inst.CombatTurnLabel:SetText(L("TXT_KEY_TP_TURN_COUNTER", e.Turn))
    inst.CombatTextLabel:SetText(e.Message)

    if e.OtherPlayer ~= nil and e.OtherPlayer >= 0 then
      CivIconHookup(e.OtherPlayer, 32, inst.OtherIcon, inst.OtherIconBG, inst.OtherIconShadow, false, true)
      inst.OtherIconBox:SetHide(false)
    else
      inst.OtherIconBox:SetHide(true)
    end

    Refresh(inst.CombatTextStack)
    local height = inst.CombatTextStack:GetSizeY()
    inst.CombatButton:SetSizeY(height)
    inst.CombatButton:SetDisabled(e.Plot == nil)
    if e.Plot ~= nil then
      inst.CombatButton:SetVoids(e.Plot:GetX(), e.Plot:GetY())
      inst.CombatButton:RegisterCallback(Mouse.eLClick, function(x, y)
        UI.LookAt(Map.GetPlot(x, y), 0)
      end)
    end
    table.insert(m_combat_logs, inst)
  end

  Refresh(C.CombatStack, C.CombatScroll)
end
```

RECOMMENDED: Option B.

Rationale:

- It satisfies the requirement for a true Combat tab with its own content view.

- It preserves the existing notification list and instant-yield settings logic instead of recoding it.

- It uses VP's current CPK instance helpers already used by `NotificationLogPopup.lua`.

- It keeps the DLL/Lua data contract unchanged.

- It makes option-disabled behavior straightforward because the tab can safely call `GetMilitaryLog()` and get `{}`.

Recommended visible behavior:

- Default tab: Notifications, to preserve current popup behavior.

- Combat ordering: newest first, matching the current Military Log UI and the typical "latest log first" expectation.

- Combat filters: none for baseline.

- Combat entries: turn label, message text, optional other-player icon, hover highlight, click-to-plot when `Plot` is non-nil.

- Empty state: show a centered or top label such as "No combat events recorded." when enabled but empty.

- Disabled/unavailable state: prefer showing the Combat tab disabled or hidden when `Game.IsCustomModOption("WH_MILITARY_LOG")` is false. If hidden, users without the feature see exactly today's Notification Log.

- Refresh: refresh Combat list when the Combat tab is selected, on popup open if Combat is selected, on active-player change, and on turn change while open. No live event listener is needed for same-turn refresh unless runtime testing shows combat entries can be added while the popup remains open.

- Notification settings controls: visible only on the Notifications tab and only when the existing settings subview is selected.

Required for integrated implementation:

- A VP text XML/SQL file if new strings are introduced for `TXT_KEY_POP_MILITARY_LOG`, empty state, or disabled state.

Optional or modmod-owned alternatives:

- `codex/modmod_resources/Merged UI Mods/MilitaryLog/XML/UIMilitaryLog.xml` if the modmod remains responsible for enabling `WH_MILITARY_LOG` and providing `TXT_KEY_POP_MILITARY_LOG`.

- `codex/modmod_resources/Merged UI Mods/MilitaryLog/UI/MilitaryLogPopup.lua` / `.xml` only in a later retirement phase.

- `codex/modmod_resources/Merged UI Mods/Merged UI Mods.modinfo` only if retiring the old addin entry point or changing how the modmod activates the integrated UI.

No DLL change is required for the baseline integrated Combat tab.

1. Add top-level Notifications/Combat tab controls and selected-state highlights to `NotificationLogPopup.xml`.

2. Wrap existing notification log/settings content in a `NotificationsTabPanel`.

3. Add `CombatTabPanel`, `CombatScroll`, `CombatStack`, `CombatEmptyLabel`, and `CombatLogInstance`.

4. In Lua, add tab constants/state and a table-driven `SelectTab` based on the `CorporationsOverview` pattern.

5. Add combat instance storage and `FreeCombatLogs`.

6. Add `FillCombatView` using `Players[m_player_id]:GetMilitaryLog()`, newest-first loop, nil-plot guard, and `NO_PLAYER` icon guard.

7. Gate tab visibility/disabled state with `Game.IsCustomModOption("WH_MILITARY_LOG")` plus safe fallback.

8. Update show-hide/active-player/turn-change paths to refresh the selected tab without resetting notification option state.

9. Add required localization if the implementation cannot rely on modmod-provided `TXT_KEY_POP_MILITARY_LOG`.

1. Remove or disable the `MilitaryLog/UI/MilitaryLogPopup.xml` `InGameUIAddin` entry point in the modmod.

2. Remove the Additional Information dropdown registration from the old Lua, or stop loading that context.

3. Keep `UIMilitaryLog.xml` if the modmod remains responsible for enabling `WH_MILITARY_LOG` and providing text.

4. Keep `MilitaryIcon.dds` only if reused elsewhere.

5. Document migration behavior for users who install both older and newer UI packages.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
