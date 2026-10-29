# Policy adoption and ideology application

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

How do policy and ideology choices mutate player state?

Policy legality/application, branch unlocks, switching and their effects are shared mechanics. AI selection/scoring remains in the policy decision owner.

## Reference synthesis

The policy selector and ideology decision owners hand off to shared adoption, unlock and switch paths. Those routines change ownership/branch state, process policy effects and can affect events, free choices, city/player caches and serialization-sensitive state. Human/UI choice is separate from that application. The recovered references are not an exhaustive policy-effects field catalog.

Canonical related owners: [Policies, branches, tenets and ideology](<../../ai/policies/selection.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Application and state

Normal branch application is `CvPlayerPolicies::DoUnlockPolicyBranch`. It validates with `CanUnlockPolicyBranch`, calls `SetPolicyBranchUnlocked`, calls `DoSwitchToPolicyBranch`, spends culture or a free policy, updates next policy cost, dirties UI, triggers trait great engineer/merchant behavior through `CvPlayerTraits::GetPolicyGEorGM`, grants instant yields and great-person progress, dirties city citizens, and fires Lua hook `PlayerAdoptPolicyBranch`.

Ideology branch application is `CvPlayerPolicies::SetPolicyBranchUnlocked`, called directly by `CvPolicyAI::DoChooseIdeology`, vassal forcing in `CvPlayer::doTurn`, and ideology-switch code. For purchase-by-level branches, it grants free tenets through `CvPlayer::ChangeNumFreeTenets`, includes `CvPlayerTraits::GetExtraTenetsFirstAdoption` on first adoption, notifies other major civs, grants the branch free policy with `CvPlayer::setHasPolicy`, invokes `GAMEEVENT_IdeologyAdopted` when not a revolution, records the first ideology turn in `CvPlayerCulture::SetTurnIdeologyAdopted`, and completes `ACCOMPLISHMENT_CHOOSE_IDEOLOGY` for Freedom/Autocracy/Order.

Policy and tenet application is `CvPlayer::doAdoptPolicy`. It validates `CvPlayer::canAdoptPolicy`, spends a free tenet, free policy, or culture, calls `CvPlayer::setHasPolicy`, updates next policy cost, ensures the policy branch is unlocked with `SetPolicyBranchUnlocked`, dirties UI, applies trait great engineer/merchant behavior, grants instant yields and great-person progress, dirties city citizens, fires Lua hook `PlayerAdoptPolicy`, and calls `CvPlayer::updateYield`.

`CvPlayer::setHasPolicy` calls `CvPlayerPolicies::SetPolicy` and `CvPlayer::processPolicies`. `CvPlayerPolicies::SetPolicy` updates owned/free policy arrays, policy counts, modifier cache, difficulty bonuses, recent-branch history, branch-finished state, and finisher policy grants/removals. `CvPlayer::processPolicies` applies or removes the actual `CvPolicyEntry` effects across player, policy modifier cache, corporations, cities, units, promotions, resources, other players, trade routes, religion-related modifiers, and one-shot effects.

Ideology switching application is `CvPlayerPolicies::DoSwitchIdeologies`. It fires `GAMEEVENT_IdeologySwitched`, computes retained free tenets after `SWITCH_POLICY_BRANCHES_TENETS_LOST`, clears the old branch with `ClearPolicyBranch`, toggles old/new branch unlocked state, recomputes public opinion, records switch turn, zeroes culture, grants retained free tenets, fires `PlayerAdoptPolicyBranch`, destroys buildings enabled by the old ideology branch or old-branch policies, and dirties the policies UI for the active player.

Reformation is a policy side effect, not a policy-selection input. `CvPlayer::processPolicies` checks `CvPolicyEntry::IsAddReformationBelief`; if the player owns a religion and has not added a reformation belief, non-human players call `CvReligionAI::ChooseReformationBelief` and apply with `CvGameReligions::AddReformationBelief`.

- `CvPlayer::doAdoptPolicy(PolicyTypes)` applies chosen policies/tenets for AI and non-AI callers after validation.

- `CvPlayer::processPolicies(PolicyTypes, int)` applies/removes broad `CvPolicyEntry` effects and can trigger AI reformation choice.

- `CvPlayerPolicies::SetPolicyBranchUnlocked(...)`, `DoUnlockPolicyBranch(...)`, and `DoSwitchIdeologies(...)` in `CvPolicyClasses.cpp` apply branch/ideology state.

RISK: These final application functions are broad side-effect hubs. They interact with notifications, Lua/game events, free policies/tenets/techs, city/UI dirty bits, religion state, trait reinitialization, unit creation, building effects, caches, and serialization-backed state.

- `CvPlayerPolicies::DoPolicyAI()` loops while the player has enough culture, a free policy, or a free tenet. If the returned integer is below the branch count it applies with `CvPlayerPolicies::DoUnlockPolicyBranch(PolicyBranchTypes)`; otherwise it subtracts the branch count and applies with `CvPlayer::doAdoptPolicy(PolicyTypes)`.

### First Ideology Branch Choice

- `CvPolicyAI::DoChooseIdeology(...)` applies the chosen ideology directly with `CvPlayerPolicies::SetPolicyBranchUnlocked(eChosenBranch, true, false)`, then logs the branch choice, handles `CvPlayerTraits::GetPolicyGEorGM()`, and fires ideology-adoption script/game hooks later in the function. This bypasses `CvPlayerPolicies::DoUnlockPolicyBranch(...)` and therefore does not use normal branch-spending validation or the normal weighted branch vector.

### Ideology Switching

- switch application is `CvPlayerPolicies::DoSwitchIdeologies(PolicyBranchTypes)`. `CvPolicyAI::DoConsiderIdeologySwitch(...)` first sets anarchy with `CvPlayer::SetAnarchyNumTurns(SWITCH_POLICY_BRANCHES_ANARCHY_TURNS)`, then calls `DoSwitchIdeologies(...)` and sends anarchy notifications.

## Version differences, risks and unresolved questions

- Human/UI split: human policy-choice players are routed to notifications and net/UI policy adoption. `CvDllNetMessageHandler::ResponseAdoptPolicy(...)` and `ResponseAdoptPolicyBranch(...)` in `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp` call the same final application functions (`CvPlayer::doAdoptPolicy(...)`, `CvPlayerPolicies::DoUnlockPolicyBranch(...)`, and `CvPlayerPolicies::SetPolicyBranchUnlocked(...)`) but do not call `CvPolicyAI::ChooseNextPolicy(...)`.

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
