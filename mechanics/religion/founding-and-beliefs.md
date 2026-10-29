# Religion founding and belief application

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How are selected beliefs applied to religion, cities, traits and prophets?

The shared religion application path is separate from AI belief selection. Human/network validation, religion mutation, hooks, notifications and prophet consumption belong here.

## Reference synthesis

Belief selection returns IDs. Shared founding/enhancement/reformation routines mutate CvReligion/CvReligionBeliefs, initialize traits, update cities/player religion state, dispatch hooks and notifications, and manage faith thresholds or prophet state where applicable. Human/network paths perform their own final validation before the same application.

The exact final legality and secondary belief-data fields were not fully enumerated by the recovered audit. AI category triggers, scores and category seeds remain with belief selection rather than being duplicated as general mechanics.

Canonical related owners: [Belief selection](<../../ai/beliefs/selection.md>), [Settler-carried religion](<settler-religion.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Candidates and gates

- Upstream legality inputs: `CvGameReligions::CanCreatePantheon(...)` gates pantheon creation and uses `GAMEEVENT_PlayerCanFoundPantheon` or Lua `PlayerCanFoundPantheon`. `CvUnit::CanFoundReligion(...)` requires `CvUnitEntry::IsFoundReligion()`, unused prophet spread charges, owned-team city plot, non-minor team, not delayed death, religion slots available unless `CvPlayerTraits::IsAlwaysReligion()`, no currently owned religion, event/Lua permission, and at least one available founder and follower belief. `CvUnit::CanEnhanceReligion(...)` requires a founding unit at the owned religion's holy city, no prior enhancement, and at least one available enhancer and follower belief.

## Application and state

Pantheon application is `CvGameReligions::FoundPantheon`. It creates a `CvReligion` with `RELIGION_PANTHEON`, adds the belief through `CvReligionBeliefs::AddBelief`, pushes it into `m_CurrentReligions`, reinitializes player traits, may trigger free tech from `CvPlayerTraits::IsAdoptionFreeTech`, updates religion state, subtracts pantheon faith cost, increments the next pantheon faith threshold, fires `PantheonFounded` game/Lua hooks, spreads pantheon pressure to all player cities, updates all cities for the religion, sends notifications/replay messages, logs, and dirties city UI.

Religion founding application is `CvGameReligions::FoundReligion`. It copies any pantheon beliefs from the player's pantheon, adds selected beliefs, stores the custom name, pushes the religion to `m_CurrentReligions`, calls `CvCityReligions::DoReligionFounded` on the holy city, applies population-boost spread trait behavior, reinitializes traits, updates religion state, resets founding state, updates prophets to the new religion, fires `ReligionFounded` game/Lua hooks, sends notifications/replay, logs, and dirties city UI.

Enhancement application is `CvGameReligions::EnhanceReligion`. It finds the existing religion or pantheon, adds one or two beliefs, marks non-pantheon religions enhanced when requested, reinitializes traits, updates all cities and player religion state, fires `ReligionEnhanced` hooks, sends notifications when requested, logs, and dirties city UI.

Reformation application is `CvGameReligions::AddReformationBelief`. It finds the owned religion, adds the selected belief, reinitializes traits, marks `m_bReformed`, updates all cities and player religion state, fires `ReligionReformed` when mod events are enabled, sends notifications, logs, and dirties city UI.

`CvUnit::DoFoundReligion` and `CvUnit::DoEnhanceReligion` in `CvGameCoreDLL_Expansion2/CvUnit.cpp` consume or partially consume the prophet after application, with special handling for `CvPlayerTraits::IsProphetFervor`.

- `CvGameReligions::FoundPantheon(...)`, `FoundReligion(...)`, `EnhanceReligion(...)`, and `AddReformationBelief(...)` in `CvReligionClasses.cpp` apply belief choices for AI and human/UI/network paths.

- `CvPlayer::foundCity(...)` and `CvPlayer::processPolicies(...)` are player-level triggers into pantheon and reformation application.

- pantheon application is `CvGameReligions::FoundPantheon(PlayerTypes, BeliefTypes)`: creates a `CvReligion` with `RELIGION_PANTHEON`, calls `CvReligionBeliefs::AddBelief(...)`, pushes to `m_CurrentReligions`, reinitializes traits, may grant a free tech through `CvPlayerTraits::IsAdoptionFreeTech()`, updates religion state, subtracts pantheon faith cost, increments the next pantheon threshold, fires `PantheonFounded` game/Lua hooks, adds pantheon pressure to all player cities, updates all cities, sends replay/notifications, logs, and dirties city UI. founding application is `CvGameReligions::FoundReligion(...)`: copies existing pantheon beliefs, adds selected beliefs, sets religion identity/name, pushes to `m_CurrentReligions`, calls `CvCityReligions::DoReligionFounded(...)`, reinitializes traits, updates player/religion state, updates prophets to the new religion, fires `ReligionFounded`, sends messages, logs, and dirties UI. enhancement application is `CvGameReligions::EnhanceReligion(...)`: adds one or two beliefs, marks non-pantheon religions enhanced when requested, reinitializes traits, updates cities/player religion state, fires `ReligionEnhanced`, sends optional notifications, logs, and dirties UI. reformation application is `CvGameReligions::AddReformationBelief(...)`: adds the belief, reinitializes traits, sets `m_bReformed`, updates all cities/player religion, fires `ReligionReformed` under mod events, sends notifications, logs, and dirties UI.

- Human/UI split: human players are routed to notifications/net/UI choice in `CvGameReligions::DoPlayerTurn(...)`, `CvPlayer::foundCity(...)`, `CvUnit::DoFoundReligion()`, `CvUnit::DoEnhanceReligion()`, and `CvPlayer::doAdoptPolicy(...)`; AI selection is skipped for those human religion-choice paths. final application is shared: `CvDllNetMessageHandler::ResponseFoundReligion(...)` and `ResponseEnhanceReligion(...)` in `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp` validate with `CvGameReligions::CanFoundReligion(...)` / `CanEnhanceReligion(...)`, then call `FoundReligion(...)` / `EnhanceReligion(...)`.

- chosen beliefs alter religion state through `CvReligionBeliefs::AddBelief(...)`, city religion pressure/followers, trait initialization, faith thresholds/costs, prophet state, notifications/replay, and game/Lua events. `CvUnit::DoFoundReligion()` and `CvUnit::DoEnhanceReligion()` expend or partially expend the prophet after application, with special handling for `CvPlayerTraits::IsProphetFervor()`. INFERRED because `CvPlayerTraits::InitPlayerTraits()` is called after applying beliefs, belief effects can affect systems outside religion through trait/religion-derived player and city modifiers.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
