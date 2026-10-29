# Decision registry

This is the single catalog and completeness/control surface for the maintained AI reference. Recovered decision knowledge is registered with its older verification baseline; it is neither an empty planning catalog nor verification against an unselected modern target. Domain owners are linked below. The [Task 1U boundary](units/task-1u-scope.md) remains demand-to-training, not all unit AI.

## Row schema and version applicability

| Field | Meaning |
|---|---|
| ID / question / kind | Stable semantic ID; DECISION, PROCESS or AREA. An unsplit AREA cannot be complete |
| Domain / detailed owner | Unique topic document/section; multiple bounded decisions can share a clearly divided owner |
| Disposition | PLANNED, ACTIVE, DEFERRED or RETIRED; not verification |
| Knowledge basis | REPORTED_SOURCE_AUDIT (substantial recovered source-audit reference), PRECISE_SOURCE_AUDIT (identified modern evidence), or UNCHECKED_NOTE (not established as a source-backed reference) |
| Last verification baseline | Recovered baseline or precise snapshot/pass, with section/profile limits |
| Lifecycle / score / logging mapping | Independent UNMAPPED, PARTIAL or COMPLETE axes, explicitly attached to that baseline and scope |
| Target applicability | UNSELECTED, NOT_REVALIDATED, PARTIAL or REVALIDATED; version applicability is independent of documentary coverage |
| Target source verification | UNCHECKED, PARTIAL or VERIFIED for the selected target only; never inferred from age or richness of the reference |
| Dependencies / deferrals | Producer contracts, shared consumers, configuration variants, unresolved paths and next owner |

The recovered rows below use `legacy-vp526`, with full identity limits in [source baselines](source-snapshots.md#recovered-baseline). Their basis is REPORTED_SOURCE_AUDIT, target applicability UNSELECTED and target source verification UNCHECKED. No current-source pass was performed by migration. A richer old reference is distinguished from an unchecked note without fabricating a precise old commit or current verification.

## Coverage semantics

Lifecycle `COMPLETE` means every applicable lifecycle stage and materially different branch is mapped, including empty/failed/fallback paths, application failure and downstream state; each omitted stage has a reason. Score provenance `COMPLETE` means every contributing numerical/boolean input, operation, gate and selection transform resolves through the dependency graph to an identified origin. A decision with no numeric score uses the same axis for its predicates and control inputs. An opaque manager getter or untraced formula keeps that axis `PARTIAL`.

Logging mapping `COMPLETE` means every applicable reconstruction dimension has a supported assessment and every gap is recorded. It can coexist with `NONE` observability. Unknown relevant log paths keep mapping `PARTIAL`; `UNKNOWN` is honest while investigating. See [logging coverage](logging-coverage.md).

These meanings are unchanged from Task 0; the added baseline/applicability dimensions say **which source state** a coverage assessment concerns. Recovered narrative/helper coverage is generally PARTIAL under the exhaustive schema: substantial mappings exist, but not all branches, leaf inputs, effective data and observability dimensions were enumerated. This is not equivalent to UNMAPPED.

Target `VERIFIED` requires registered exact identity and claim-level source evidence across the stated target scope. Complete documentation at an older baseline does not imply REVALIDATED. Deferred dependencies remain visible; an untraced producer cannot close provenance merely because it has a link. Retired/merged IDs retain redirects rather than disappearing.

## Canonical decision and process index

Every row in this table is ACTIVE as a maintained reference, with the inherited basis/applicability above. L/S/O denote lifecycle, score provenance and logging mapping **at the recorded older baseline**, not modern validation. Individual parameterized families must split when source shows materially different decisions.

| ID | Kind / domain | Detailed owner / decision question | L / S / O | Principal remaining dependency or limit |
|---|---|---|---|---|
| `units.city-training` | DECISION / units | [Unit production](units/production.md): recommendation and training candidate scoring | PARTIAL / PARTIAL / PARTIAL | Exhaustive sanity terms, legality and competing non-unit provenance |
| `units.operation-demand` | DECISION / units | [Operation demand](units/operation-demand.md): missing roles becoming city requests | PARTIAL / PARTIAL / PARTIAL | Normal-queue commitment/completion and standing-army ownership |
| `infra.unit-flavor-paths` | PROCESS / infrastructure | [Flavor propagation](infrastructure/flavors.md): paths consumed by units | PARTIAL / PARTIAL / PARTIAL | Exact modern layer/update/dependency verification |
| `production.city-choice` | DECISION / production | [Shared city selection](production/city-selection.md): mixed buildable competition | PARTIAL / PARTIAL / PARTIAL | Shared selector branches and full per-class score provenance |
| `production.building-value` | PROCESS / buildings | [Building valuation](buildings/valuation.md): shared sanity/estimator output | PARTIAL / PARTIAL / PARTIAL | Full building field/formula enumeration and consumer contexts |
| `production.purchase-choice` | DECISION / acquisition | [Purchasing](production/purchasing.md): economic/military/tactical/religious acquisition paths | PARTIAL / PARTIAL / PARTIAL | Distinct branch owners, legality/application differences and log gaps |
| `beliefs.category-choice` | DECISION / beliefs | [Beliefs](beliefs/selection.md): six category chooser families | PARTIAL / PARTIAL / PARTIAL | Field-by-field belief scoring and final legality/event variants |
| `policies.policy-choice` | DECISION / policies | [Policies](policies/selection.md): policies, normal branches and tenets | PARTIAL / PARTIAL / PARTIAL | Exact direct attribute terms, prerequisite propagation and effects |
| `policies.ideology-adoption` | DECISION / policies | [Policies](policies/selection.md): first ideology branch | PARTIAL / PARTIAL / PARTIAL | Forced/team/vassal/option variants and priority calculations |
| `policies.ideology-switch` | DECISION / policies | [Policies](policies/selection.md): later ideology switching | PARTIAL / PARTIAL / PARTIAL | Human-vassal exception, opinion/happiness/threshold inputs |
| `wonders.desired` | DECISION / wonders | [Wonders](wonders/selection.md): desired wonder | PARTIAL / PARTIAL / PARTIAL | Local race/strategy terms and team/limited wonder semantics |
| `wonders.best-city` | DECISION / wonders | [Wonders](wonders/selection.md): city assignment | PARTIAL / PARTIAL / PARTIAL | Reported null-result finding and caller/design intent |
| `strategies.pressure` | PROCESS / strategies | [Managers](strategies/managers.md): represented outputs and deltas | PARTIAL / PARTIAL / PARTIAL | Not a complete inventory of all manager decisions |
| `operations.creation` | DECISION / operations | [Planning](operations/planning.md): operation requests/preconditions/recruitment | PARTIAL / PARTIAL / PARTIAL | Operation-type-specific target and precondition coverage |
| `units.role-strategies` | PROCESS / specialized profile | [Role strategies](units/role-strategies.md): optional UPAI role pressure | PARTIAL / PARTIAL / PARTIAL | Base-source versus effective VP+UPAI profile; missing CSV evidence |
| `infra.flavor-propagation` | PROCESS / infrastructure | [Flavors](infrastructure/flavors.md): shared producers/recipients/bypass reads | PARTIAL / PARTIAL / PARTIAL | All-consumer exhaustive term graph remains incomplete |
| `infra.weighted-selection` | PROCESS / infrastructure | [Weighted selection](infrastructure/weighted-selection.md): cutoff/ordering/draw helpers | PARTIAL / PARTIAL / PARTIAL | Exact helper/caller contracts and random context per decision |
| `infra.turn-order` | PROCESS / infrastructure | [Turn lifecycle](infrastructure/turn-lifecycle.md): triggers and producer ordering | PARTIAL / PARTIAL / PARTIAL | Not every caller/automation/interrupt variant is mapped |

## Known deferred areas

The [adjacent-area index](units/task-1u-scope.md#adjacent-area-index) preserves Task 1U's deferred unit areas: purchases, upgrades, promotions, tactical actions, homeland assignments, worker improvements, settler destinations, great-person actions, other missions/roles and detailed non-unit valuation. They remain outside Task 1U except its stated interfaces, even when a canonical reference now contains partial knowledge of them.

Additional partial interfaces represented in flavor/strategy/event references include technology valuation, diplomacy/espionage and league proposal/vote hooks. These are known AREA/UNCHECKED_NOTE inventory gaps, not complete decision audits. Record newly discovered bounded decisions and their exact owners during later tasks. Neither the recovered catalog nor this seeded registry proves the global AI inventory exhaustive.

## Task 1U current-pass overlay

The earlier table retains `legacy-vp526` scope. Current pass `audit-20261029-task1u-01` uses [registered precise sources](source-snapshots.md#task-1u-selected-source-pass). These overlays supersede only the bounded subjects below, not untouched decisions or historical evidence.

| IDs | Knowledge / applicability / source verification | Lifecycle / score / logging mapping | Limits |
|---|---|---|---|
| units.city-training | PRECISE_SOURCE_AUDIT / PARTIAL / PARTIAL | PARTIAL / PARTIAL / PARTIAL | Direct sanity inventory current; unresolved producer/legality/extra caller dependencies |
| units.operation-demand | PRECISE_SOURCE_AUDIT / PARTIAL / PARTIAL | PARTIAL / PARTIAL / PARTIAL | Current queue/direct-commit distinction; completion and creation predicates incomplete |
| infra.unit-flavor-paths, infra.flavor-propagation | PRECISE_SOURCE_AUDIT / PARTIAL / PARTIAL | PARTIAL / PARTIAL / PARTIAL | Unit consuming layers current; full initialization/strategy propagation not complete |
| production.city-choice, infra.weighted-selection | PRECISE_SOURCE_AUDIT / PARTIAL / PARTIAL | PARTIAL / PARTIAL / PARTIAL | Current unit integration/control; non-unit outputs and CvSeeder dependency incomplete |
| strategies.pressure | PRECISE_SOURCE_AUDIT / PARTIAL / PARTIAL | PARTIAL / PARTIAL / PARTIAL | Recommended forces/soft cap inspected; downstream input producers unresolved |
| operations.creation | REPORTED_SOURCE_AUDIT / NOT_REVALIDATED / UNCHECKED | PARTIAL / PARTIAL / PARTIAL | Current bridge evidence does not certify operation-type creation decisions |
| units.role-strategies | PRECISE_SOURCE_AUDIT / PARTIAL / PARTIAL | PARTIAL / PARTIAL / PARTIAL | Separate UPAI artifact/data intentions; no matching effective DB/compatibility proof |

Additional bounded dependency identified: `production.process-unit-proxy` (PROCESS, ACTIVE, PRECISE_SOURCE_AUDIT of call-site existence only, target applicability PARTIAL/source verification PARTIAL, L/S/O PARTIAL): CvProcessProductionAI.cpp444 calls unit sanity and feeds process competition. Owned by [city selection](production/city-selection.md#task-1u-current-unit-selection); its complete calculation remains unresolved. Existing purchase/advisor/free-unit/religious callers indexed in unit owner are boundary coverage, not complete acquisition audits.

## Task 1U-B current applicability and boundaries

Snapshot src-20261029-vp-548 / audit-20261029-task1u-b is sole current upstream target; earlier c285a18 overlay retained as history. All listed current knowledge basis PRECISE_SOURCE_AUDIT. Direct43 records preserved; relevant discovered paths/callers have explicit dispositions in scope owner.

### Task 1U-B final coverage decision

**TASK_1U_COMPLETE** for the bounded source/interface audit. This does not mean every AI manager's internals, every effective mod stack or complete city-choice score ancestry is audited. Criteria accept precise independent producer contracts and bounded UNRESOLVED_EXTERNAL evidence; those limits remain visible rather than converting all score axes to COMPLETE.

| Current bounded subject | Lifecycle / score provenance / logging mapping | Basis |
|---|---|---|
| units.city-training | COMPLETE / PARTIAL / COMPLETE | Immediate direct expressions/branches plus known input contracts mapped; independent/native rules attribution and unavailable effective profile constrain full transitive provenance |
| units.operation-demand | COMPLETE / PARTIAL / COMPLETE | Creation/request interface through queue/token/reserve/readiness feedback; strategic request and pathfinding internals bounded |
| infra.unit-flavor-paths | COMPLETE / PARTIAL / COMPLETE | Symbolic unit signal routing/reset/cache/reversal/bypass complete; strategy decision internals and loaded row values external |
| production.process-unit-proxy | COMPLETE / PARTIAL / COMPLETE | Exact unit reward proxy/selection contribution mapped; league eligibility and other reward/competitor scores independent |
| infra.weighted-selection (unit normal/hurry consumers) | COMPLETE / COMPLETE / COMPLETE | Exact source seed/hash/mix/range/cutoff/ordered mapping/overrides; no runtime draw observed |
| units.role-strategies | COMPLETE / PARTIAL / COMPLETE | Optional dispatch/data-role/package interfaces/test evidence mapped; effective loaded compatibility external |
| production.city-choice (whole) | PARTIAL / PARTIAL / PARTIAL | Nonunit full valuation deliberately not Task1U; unit-side common selection/log dimensions mapped |

All other broader registry subjects retain partial/historical scope. PARTIAL provenance here is an explicit wider transitive/interface limit, not an omitted direct unit branch or an unclassified relevant caller. Source/profile/lifecycle completion and whole-system provenance remain separate assessments.

`units.inputs` is registered as PROCESS / ACTIVE, detailed owner [unit producer contracts](strategies/managers.md#task-1u-b-producer-contracts-on-548), PRECISE_SOURCE_AUDIT at src-20261029-vp-548 / audit-20261029-task1u-b. p001–p016 identify time-indexed producer contracts consumed by t001–t043. Current lifecycle COMPLETE at the declared interfaces, full transitive score PARTIAL, logging mapping COMPLETE for these input families. Independent planner/rules decisions and effective profile limits remain explicit; this process is not a new unit scorer.
