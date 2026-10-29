# Puppet faith purchasing

Knowledge status: Maintained reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Optional switches, supplied modmods and later local repairs are distinct applicability/revision conditions. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

What permits puppet faith purchases and which UI/AI paths expose them?

The DLL option, FaithCost/trait belief availability, UI filtering and resource overrides have different scopes. A broad popup change was reportedly reverted in 2026-06-29; the UI concern remains a validation/design question, not a confirmed defect.

## Revision reconciliation

The recovered feature/group audits distinguish broad DLL faith-building eligibility from narrower UI usefulness checks and selected unit PuppetPurchaseOverride resources. A **2026-06-29 report** says a temporary popup scan over every FaithCost-positive building was rejected and reverted. The enduring UI question is resource/runtime validation, not a confirmed source defect.

Do not confuse general DLL purchase authority with UI display, AI preferences or the list of resources intentionally made purchasable. The paired settler-religion repair is a separate mechanic.

Canonical related owners: [AI purchasing and emergency acquisition](<../../ai/production/purchasing.md>), [Production queues and creation](<queue-and-creation.md>), [Optional feature activation and dependency map](<../../architecture/optional-features.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Reported revision changes

Revalidated against audited source and supplied modmod resources. `codex/modmod_resources/Merged Gameplay Mods/FaithPurchases/GameDefines.sql` enables `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` and marks `UNIT_MISSIONARY` and `UNIT_INQUISITOR` with `PuppetPurchaseOverride = 1`. `codex/modmod_resources/Various Bal Tweaks (v 2)/PuppetCivilianPurchase.sql` also enables the same DLL option and marks workers/workboats with `PuppetPurchaseOverride = 1`.

The resource-side `PuppetPurchaseOverride` updates help unit purchase reachability. Current `CvGameCoreDLL_Expansion2/CvCity.cpp:30545` still permits puppet faith-building checks when the option is enabled, while `(1) Community Patch/LUA/ProductionPopup.lua:880` through `ProductionPopup.lua:900` intentionally prefilters puppet popup access to buildings with `AllowsPuppetPurchase` or `PuppetPurchaseOverride` and units with `PuppetPurchaseOverride`.

CHANGE_REVERTED: No UI source change is made for this feature. A temporary `FaithCost > 0` popup scan was rejected as overbroad because most VP buildings can have faith costs and the mod resource title is not sufficient evidence that all such buildings should become a popup-opening condition.

RECOVERED_CONCLUSION: The DLL path already worked for direct, Lua/network, AI, and purchase execution calls. The older audit's UI concern remains a runtime validation item, not a confirmed source defect. Current VP did not supersede the DLL implementation. The supplied resources provide explicit unit-side popup reachability through `PuppetPurchaseOverride`; building-side UI behavior should be validated against the intended resource design before patching.

REMAINING_RUNTIME_TESTS: Enable `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1`, create non-Venice puppet cases for the intended resource set, and compare direct `pCity:IsCanPurchase(..., YIELD_FAITH)` results with production popup availability. Repeat with the option disabled to confirm standard puppet behavior remains unchanged except for existing Venice, `AllowsPuppetPurchase`, and `PuppetPurchaseOverride` cases.

## Trigger and ownership

Lua `CvLuaCity::lIsCanPurchase` calls this city method in `CvGameCoreDLL_Expansion2/CvLuaCity.cpp:2233`. UI purchase checks call `IsCanPurchase` from `(2) Vox Populi/Core Files/Overrides/ProductionPopup.lua:89`, `ProductionPopup.lua:136`, `ProductionPopup.lua:210`, and `ProductionPopup.lua:880`.

AI religion faith spending calls `CvCity::IsCanPurchase(..., YIELD_FAITH)` in `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp`, including `CvReligionClasses.cpp:6644`, `CvReligionClasses.cpp:6687`, `CvReligionClasses.cpp:7298`, `CvReligionClasses.cpp:7307`, `CvReligionClasses.cpp:7337`, `CvReligionClasses.cpp:11467`, and `CvReligionClasses.cpp:11488`.

Attempt faith building purchase in a puppet city with and without the option enabled, including UI and AI faith-spending paths.

## Application and state

The guarded code affects purchase eligibility. Actual purchase execution remains downstream in `CvCity::Purchase`.

`CvCityReligions::AdoptReligionFully` erodes other pressure, adds pressure for the target religion, and recomputes followers at `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp:5011` through `CvReligionClasses.cpp:5014`, so capital-religion or pantheon pressure added earlier in `CvCity::init` is not the last write when `eInitialReligion` is provided.

```text
GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS
UI/Lua/AI purchase query
  -> CvCity::IsCanPurchase slow overload
  -> CvCity::IsCanPurchase fast overload
  -> puppet gate exception
  -> faith-building eligibility checks
  -> CvCity::PurchaseBuilding
  -> CreateBuilding / faith cost / CityConstructed hook
```

The slow `CvCity::IsCanPurchase` overload builds total building counts and delegates to the fast overload at `CvGameCoreDLL_Expansion2/CvCity.cpp:30231` through `CvCity.cpp:30242`. The guarded puppet exception is in the fast overload at `CvCity.cpp:30256`, and the puppet rejection gate uses it at `CvCity.cpp:30272`.

Faith-building purchase validation continues through `GetFaithPurchaseCost`, belief-unlock checks, `canConstruct(..., bWillPurchase=true)`, and duplicate-building checks at `CvGameCoreDLL_Expansion2/CvCity.cpp:30520` through `CvCity.cpp:30544`. `CvCity::PurchaseBuilding` rechecks `IsCanPurchase` at `CvCity.cpp:30871` before creating the building and charging faith at `CvCity.cpp:30946` through `CvCity.cpp:30986`.

## Data and configuration

Defined in `(1) Community Patch/Database Changes/NewCustomModOptions.xml:282` with default `Value="0"`. Macro is `MOD_GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` in `CvGameCoreDLL_Expansion2/CustomMods.h:237`; declaration is `CustomMods.h:1567`; cache load is `CvGameCoreDLL_Expansion2/CustomMods.cpp:344`.

The cached option enables a puppet exception in `CvCity::IsCanPurchase`.

`MOD_GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` is declared as `gCustomMods.isGLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS()` in `CvGameCoreDLL_Expansion2/CustomMods.h:237`, declared in `CustomMods.h:1567`, and cached in `CvGameCoreDLL_Expansion2/CustomMods.cpp:344`.

No invalid DLL combination was found. The operational case requiring validation is relying on human UI access for puppet faith purchases beyond the existing Venice, `AllowsPuppetPurchase`, and `PuppetPurchaseOverride` popup paths.

## Observability and validation obligations

Log `CvCity::IsCanPurchase` parameters, puppet state, `bPuppetExceptionFaithBuilding`, and final purchase result.

4. `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=0`: non-Venice puppet with eligible belief building and sufficient faith. Expected: `CvCity::IsCanPurchase(..., YIELD_FAITH)` returns false unless override/allow-puppet mechanisms apply.

5. `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1`: same city/building. Expected: DLL `IsCanPurchase` returns true and `PurchaseBuilding` succeeds if all later faith-building checks pass.

6. `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1`: human UI attempts to open puppet purchase popup for intended resource-set purchases. Expected: popup availability should match the design path, such as Venice, `AllowsPuppetPurchase`, `PuppetPurchaseOverride`, or a future explicit helper if one is intentionally added.

7. Save/load after a puppet faith-building purchase. Expected: building presence, faith balance, and purchased markers/events are stable.

- In `CvUnit::initWithNameOffset`, log option state, city ID, candidate religion follower weights, total weight, seed inputs, chosen religion, and whether `m_Religion.SetReligion` was called.

- In `CvCity::IsCanPurchase`, log puppet flags, faith-building exception, purchase yield, faith cost, belief unlock result, `canConstruct` result, and final result.

- In the production popup or future DLL helper, log why a puppet city is considered to have or not have any purchaseable item.

## Version differences, risks and unresolved questions

Local `git log -S'MOD_GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS'` finds production-source provenance at `54dee87` only.

No compile-time risk was found.

INFERRED: Feature can be masked by other puppet purchase permissions, Venice/no-annex behavior, or purchase override data.

Classification: missing validation.

Exact location: `(1) Community Patch/LUA/ProductionPopup.lua:866` through `ProductionPopup.lua:900`.

```text
City banner / production popup open check
  -> MayPurchaseOrProduceAnything(pCity)
  -> puppet shortcut scan
  -> returns false unless Venice, AllowsPuppetPurchase, PuppetPurchaseOverride, or an override item can be purchased
```

Intended behavior: If a resource set intends a puppet purchase to be exposed to the human production popup, it should either satisfy the existing popup prefilter or provide a narrower UI/DLL affordance for that intended category.

Observed or statically inferred behavior: The UI shortcut scans buildings with `AllowsPuppetPurchase = 1 OR PuppetPurchaseOverride = 1` and units with `PuppetPurchaseOverride = 1`. This is narrower than the DLL option, which can allow faith purchases for any building that passes the later faith-purchase checks. A blanket UI scan over every `FaithCost > 0` building is not justified by static audit alone because VP assigns faith costs to many building categories, not just religious or belief buildings.

Why a rewrite caused or exposed it: INFERRED: Local history is grafted at `54dee87`; no later local UI rewrite is visible. The architecture has a broad DLL eligibility rule and a narrower UI optimization, but source inspection alone does not prove that the optimization is wrong for the intended resource set.

Current revalidation update 2026-06-29: The supplied `codex/modmod_resources/Merged Gameplay Mods/FaithPurchases/GameDefines.sql` and `codex/modmod_resources/Various Bal Tweaks (v 2)/PuppetCivilianPurchase.sql` enable the DLL option and add `PuppetPurchaseOverride` for selected units. A temporary popup scan over `FaithCost > 0` buildings was rejected as overbroad and reverted. The UI concern remains a runtime/resource validation item, not a confirmed source defect.

Possible regression risks: Scanning all faith-cost buildings can expose much more than the modmod intended and increase UI popup-open cost. A resource marker or a filtered DLL helper would be safer if broader building UI access is desired.

Runtime scenario needed: Enable `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1`, create non-Venice puppet cases for the intended resource set, and compare direct `pCity:IsCanPurchase(..., YIELD_FAITH)` results with production popup availability.

Classification: compatibility risk.

The puppet exception at `CvGameCoreDLL_Expansion2/CvCity.cpp:30256` checks only building presence and `YIELD_FAITH`, not whether the building is a belief building. Later checks still enforce faith cost, belief unlock, `canConstruct`, and duplicates at `CvCity.cpp:30520` through `CvCity.cpp:30544`.

INFERRED: This matches the XML wording "buildings that have a Faith cost" rather than only religious buildings. It can still surprise modmods that expected only belief buildings.

AI religion purchase paths use `CvCity::IsCanPurchase` and `PurchaseBuilding` directly, including `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp:6644`, `CvReligionClasses.cpp:7298`, `CvReligionClasses.cpp:7307`, `CvReligionClasses.cpp:7337`, and `CvReligionClasses.cpp:11488`.

INFERRED: Once the DLL option is enabled, AI can spend faith in puppet cities through the same eligibility path. No additional source check was found that intentionally deprioritizes puppets for this option.

Classification: working as intended.

Lua `CvLuaCity::lPurchase` calls all three purchase methods, including `PurchaseBuilding`, at `CvGameCoreDLL_Expansion2/Lua/CvLuaCity.cpp:2251` through `CvLuaCity.cpp:2262`. `CvLuaGame::lCityPurchaseBuilding` sends purchases through `CvGame::CityPurchase` at `CvGameCoreDLL_Expansion2/Lua/CvLuaGame.cpp:645` through `CvLuaGame.cpp:651`; `CvGame::CityPurchase` sends the network purchase at `CvGameCoreDLL_Expansion2/CvGame.cpp:2878` through `CvGame.cpp:2893`; `CvDllNetMessageHandler::ResponseCityPurchase` calls `PurchaseBuilding` at `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp:211` through `CvDllNetMessageHandler.cpp:223`.

`git log -- CvGameCoreDLL_Expansion2/CvUnit.cpp CvGameCoreDLL_Expansion2/CvCity.cpp CvGameCoreDLL_Expansion2/CvPlayer.cpp CvGameCoreDLL_Expansion2/CvReligionClasses.cpp` shows production-source provenance only at `54dee87`.

INFERRED: This repository cannot prove whether upstream rewrites before the graft exposed either issue. Findings are based on current architecture, not a visible local rewrite.

2. No source patch was made for `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS`; the broad `FaithCost > 0` UI scan was rejected as over-scoped and reverted.

| Feature | Ready for normal use? | Reason |
| --- | --- | --- |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` | Runtime validation still required | DLL purchase eligibility is authoritative. Human popup behavior is resource- and design-dependent; no source defect is confirmed by static audit alone. AI/direct DLL paths remain statically plausible. |

## Source and dependency lookup

Direct references are `CustomMods.h:237`, `CustomMods.h:1567`, `CustomMods.cpp:344`, `NewCustomModOptions.xml:282`, and `CvCity.cpp:30256`.

`CvCity::IsCanPurchase(bool, bool, UnitTypes, BuildingTypes, ProjectTypes, YieldTypes, int)` computes `bPuppetExceptionFaithBuilding` at `CvGameCoreDLL_Expansion2/CvCity.cpp:30256`. The puppet purchase block at `CvCity.cpp:30272` allows the check to continue for faith building purchases when the option is enabled.

The candidate must still be a valid faith-purchasable building under the later cost, prerequisite, and purchase checks in `CvCity::IsCanPurchase` and `CvCity::Purchase`.

INFERRED: UI and AI callers that query `IsCanPurchase` are expected to use the same eligibility path. Direct purchase calls that skip this check could bypass the option.

Puppet purchase override data exists independently through `PuppetPurchaseOverride` columns in `(2) Vox Populi/Database Changes/Buildings/BuildingTableChanges.sql:132` and `(2) Vox Populi/Database Changes/Units/UnitTableChanges.sql:36`, and through `CvCity::IsAllowPuppetPurchase`.

Save with a puppet city eligible for a faith building purchase, reload, and confirm UI and AI eligibility remain consistent.

`ACTIVE_AND_REACHABLE`

The authoritative DLL eligibility path is `CvCity::IsCanPurchase`. The slow overload delegates to the fast overload at `CvGameCoreDLL_Expansion2/CvCity.cpp:30231` through `CvCity.cpp:30242`; the guarded puppet exception is computed at `CvCity.cpp:30256` and used by the puppet gate at `CvCity.cpp:30272`.

The exception only bypasses the initial puppet rejection. Faith-building candidates must still pass `GetFaithPurchaseCost`, belief-building checks, `canConstruct(..., bWillPurchase=true)`, duplicate-building checks, and faith affordability at `CvGameCoreDLL_Expansion2/CvCity.cpp:30520` through `CvCity.cpp:30544`.

`CvCity::PurchaseBuilding` rechecks `IsCanPurchase` at `CvGameCoreDLL_Expansion2/CvCity.cpp:30871`; faith purchase then creates the building, fires `CityConstructed`, charges faith, logs religion purchase, and triggers instant yields at `CvCity.cpp:30946` through `CvCity.cpp:31018`.

Direct Lua and network purchase paths still end at `PurchaseBuilding`: `CvGameCoreDLL_Expansion2/Lua/CvLuaCity.cpp:2251`, `CvGameCoreDLL_Expansion2/Lua/CvLuaGame.cpp:645`, `CvGameCoreDLL_Expansion2/CvGame.cpp:2878`, and `CvGameCoreDLL_Expansion2/CvDllNetMessageHandler.cpp:211`.

MISSING_VALIDATION: Human UI reachability is resource- and design-dependent. `(1) Community Patch/LUA/ProductionPopup.lua:866` through `ProductionPopup.lua:900` uses `MayPurchaseOrProduceAnything` to reject non-Venice puppets unless production is not being opened, the player may not annex, an `AllowsPuppetPurchase` building is present, or a `PuppetPurchaseOverride` item can be purchased. A broad scan over all `FaithCost > 0` buildings is not justified by static audit alone because VP assigns faith costs to many building categories.

COMPATIBILITY_RISK: The DLL option applies to any faith-cost building, not only belief-unlocked religious buildings. This matches `(1) Community Patch/Database Changes/NewCustomModOptions.xml:281`, but can surprise modmods expecting only belief buildings.

MISSING_VALIDATION: AI faith-building purchase paths use this same DLL eligibility through `CvGameCoreDLL_Expansion2/CvReligionClasses.cpp:6644`, `CvReligionClasses.cpp:7298`, `CvReligionClasses.cpp:7307`, `CvReligionClasses.cpp:7337`, and `CvReligionClasses.cpp:11488`. No additional source check was found that intentionally deprioritizes puppets when the option is enabled.

No source correction made: Keep `CvCity::IsCanPurchase` as the purchase eligibility authority. Any UI expansion should be based on an explicit resource-side marker or a narrower DLL/UI helper, not a blanket `FaithCost > 0` scan.

Runtime validation required: Enable the option, create a non-Venice puppet with sufficient faith and an eligible faith building lacking `PuppetPurchaseOverride`, verify `pCity:IsCanPurchase(..., YIELD_FAITH)` returns true, and verify the human purchase UI opens and can execute `PurchaseBuilding`.

Readiness: DLL direct/AI paths are statically plausible. Human-facing behavior requires runtime validation with the intended resource set.

The repository contains Visual Studio solution/project files including `CvGameCoreDLL.vs2010.sln`, `VoxPopuli_vs2013.sln`, and `CvGameCoreDLL_Expansion2/CvGameCoreDLL_Expansion2.vcxproj`.

No local `msbuild`, `xbuild`, `devenv`, or `cl` executable was found on `PATH`, so no compile was attempted in this Linux workspace. This is not runtime validation.

With `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` disabled, non-Venice puppet cities without `PuppetPurchaseOverride` or `IsAllowPuppetPurchase()` fail the puppet purchase gate at `CvGameCoreDLL_Expansion2/CvCity.cpp:30272`.

With `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` enabled, non-Venice puppet cities can pass that gate for any `eBuildingType != NO_BUILDING` and `ePurchaseYield == YIELD_FAITH`, but must still pass all later faith-building purchase checks.

- Should AI faith-purchase selection intentionally deprioritize puppets when `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` is enabled, or is equal treatment intended?

| Feature | Required trigger scenario | Required save/load scenario | Recommended logging points | Expected authoritative path |
| --- | --- | --- | --- | --- |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS` | Attempt faith building purchase in puppet, normal city, Venice/no-annex city, and override-purchase city. | Save with puppet eligible to faith-purchase, reload, verify UI and AI eligibility unchanged. | `CvCity::IsCanPurchase`, `CvCity::Purchase`, Lua `CvLuaCity::lIsCanPurchase`. | `CvCity.cpp:30245` |

| Feature/configuration | Trigger scenario | Save/load scenario | Expected behavior | Finding guarded |
| --- | --- | --- | --- | --- |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=0` | Non-Venice puppet with sufficient faith and eligible belief building lacking puppet overrides. | Save before attempted purchase. | `CvCity::IsCanPurchase(..., YIELD_FAITH)` should return false unless another override applies. | Disabled behavior parity. |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1` DLL path | Same puppet/building with option enabled. | Save before and after purchase. | `IsCanPurchase` should return true if later faith-building checks pass; `PurchaseBuilding` should create building and charge faith. | DLL authority at `CvCity.cpp:30256`, `CvCity.cpp:30520`, and `CvCity.cpp:30871`. |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1` human UI path | Human attempts to open production/purchase UI for a puppet whose only legal purchase is an ordinary faith-cost building. | Save before opening UI and after purchase. | UI should open and expose the legal faith purchase. | Likely UI reachability defect in `ProductionPopup.lua:866`. |
| `GLOBAL_PURCHASE_FAITH_BUILDINGS_IN_PUPPETS=1` AI path | AI has a puppet with a legal faith building and enough faith. | Save before `CvReligionAI` faith purchase processing; reload and process. | AI should use `IsCanPurchase` and `PurchaseBuilding`; if undesired, a policy decision is needed. | Missing validation of AI puppet prioritization. |

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
