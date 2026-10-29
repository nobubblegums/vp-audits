# Flavor signal registry

This registry indexes **paths**, not one universal meaning per `FLAVOR_X`. A signal is a particular producer/layer/access path reaching a particular consumer. Two consumers reading the same named flavor through different getters may consume different values and semantics. The recovered paths below are a version-qualified starting reference. Task 1U revalidates the paths actually consumed by its unit slice and fills their exhaustive term graph; it does not start with an empty flavor model.

## Path record

Assign a stable global path ID such as `fs001`, with an explicit anchor in this document. One record may describe a parameterized family of flavors only if its domain, transformation and consumer semantics are genuinely shared. Split conditional/bypass paths when they differ.

| Field | Required content |
|---|---|
| Path ID / flavor or signal | Stable ID, exact flavor names or explicitly bounded index domain, semantic role for this consumer |
| Producer / source table | Exact source table, keys/columns, loader/getter or runtime producer; link detailed term/contract |
| Runtime layer | One of the layers below; distinguish stored value from value returned by a getter |
| Access path | Propagated/recipient read, direct personality bypass, direct grand-strategy read, object lookup or another explicitly named path; exact API and parameters |
| Mutation / transformation path | Ordered edges through initialization, randomization, deltas, blending, clamps, broadcast and cache rebuild; link term IDs rather than hide a formula here |
| Recipient | Exact manager/recipient/cache, city versus player ownership, registration and routing filters |
| Consumer / decision / term | Decision/process ID, owning document, exact consuming getter/symbol and term ID; fan-out explicitly indexed |
| Scope | Player/city/object/flavor indices and relevant actor, option and mode variants |
| Lifetime / timing | Initialization, mutation, reversal/reset, cache invalidation, read timing and persistence/load behavior where material |
| Scale / type | Stored/read units, bounds, signedness, clamping and whether an object weight or preference signal |
| Source snapshot / audit pass / evidence | Registered identity and claim-level evidence; DB freshness/runtime corroboration separately |
| Knowledge basis / source verification | Baseline-qualified inherited or precise evidence; `UNCHECKED/PARTIAL/VERIFIED` applies to the selected target, with applicability recorded separately |
| Logging path | Observability row(s), emitter/fields, baseline/delta/read coverage and correlation limits |
| Notes / uncertainty | Bypass semantics, unknown inputs and implications for consuming decisions |

## Runtime layers

| Layer | Corresponding provenance class | Distinction |
|---|---|---|
| `PERSONALITY` | `FLAVOR_PERSONALITY` | Base runtime personality; table source/default and initialization are separate dependencies |
| `ACTIVE_PLAYER` | `FLAVOR_ACTIVE_PLAYER` | Actual active player state after its mutations |
| `RECIPIENT_LATEST_CITY` | `FLAVOR_RECIPIENT_LATEST` | City-local latest state and getter-specific read semantics |
| `RECIPIENT_LATEST_NONCITY` | `FLAVOR_RECIPIENT_LATEST` | Other recipient-local latest state; never implicitly city state |
| `GRAND_STRATEGY_DERIVED` | `FLAVOR_GRAND_STRATEGY` | A specific grand-strategy transformation/read; do not assume it is broadcast to latest state |
| `STRATEGY_DELTA` | `FLAVOR_STRATEGY_DELTA` | A strategy's delta with producer, destination layer and start/end behavior |
| `SPECIALIZATION_DELTA` | `FLAVOR_SPECIALIZATION_DELTA` | City-specialization delta and its application/removal path |
| `OBJECT_ROW` | `FLAVOR_OBJECT_ROW` | Object-specific flavor coefficient such as a `Unit_Flavors` row; not personality or latest state |

A **direct personality read that bypasses propagated/latest state** uses layer `PERSONALITY` and access path `direct personality bypass`. Do not invent another personality value or assume a propagated delta reaches this consumer. A consumer callback can be triggered by one layer but recompute from another; document both the trigger and actual read.

## Path edges and closure

For each path provide a short edge table:

| Step | Input path/term | Transform term / detailed owner | Output layer/state | Recipient / next consumer | Evidence |
|---|---|---|---|---|---|

The target trace is `database/personality source → runtime transformation → recipient state → decision consumer → score term`. An object coefficient is a separate input joining that chain at multiplication/reduction. Parallel grand-strategy or direct-personality reads remain separate edges. A stateful cycle is described through time-indexed transitions, using the [score schema](score-provenance-schema.md).

Full path verification requires actual source identity and the contributing producer/read chain. A known name or matching log label is insufficient. Trace constants, randomness, strategy conditions and direct inputs into the detailed owner; unresolved producer internals keep the consuming provenance partial. Log deltas alone do not reconstruct an unknown baseline or prove which effective value a later getter consumed.

## Registered reference paths

All rows inherit `legacy-vp526`, basis REPORTED_SOURCE_AUDIT, target applicability UNSELECTED and target source verification UNCHECKED. Detailed older-baseline producer/getter/cache behavior is maintained in [flavors](infrastructure/flavors.md); exhaustive term IDs and current log evidence remain incomplete. These rows do not assert current effective values.

| ID | Signal / layer | Producer and transformation | Recipient / consumer | Logging / limits |
|---|---|---|---|---|
| <a id="fs001"></a>`fs001` | Personality / PERSONALITY | Leader/minor/default source → initialization/randomization/bounds | CvFlavorManager personality reads; direct and propagated paths split below | Baseline/initialization versus later reads require separate timing evidence |
| <a id="fs002"></a>`fs002` | Active player / ACTIVE_PLAYER | Economic/military player deltas → ChangeActivePersonalityFlavors | Non-city recipients according to actual routing | Active-flavor delta logs do not prove every consumer's effective read |
| <a id="fs003"></a>`fs003` | City/latest / RECIPIENT_LATEST_CITY | City-targeted strategy deltas, local city strategies and specialization changes | CvCityStrategyAI latest vector → production cache rebuild | Stored signed value, clamped getter and rebuild timing are distinct |
| <a id="fs004"></a>`fs004` | Grand-strategy / GRAND_STRATEGY_DERIVED | GetPersonalityAndGrandStrategy with its mode/blend/clamp inputs | Direct unit-wrapper, policy and other consumers; not assumed broadcast | Exact mode/input values and condition-specific consumers remain term work |
| <a id="fs005"></a>`fs005` | Object coefficient / OBJECT_ROW | SetFlavors / entry cache of Unit_Flavors and other object rows | AddFlavorWeights joins a specific runtime layer at the consumer | Effective DB profile and per-object/per-flavor products not established for a new target |
| <a id="fs006"></a>`fs006` | Direct personality bypass / PERSONALITY | GetPersonalityIndividualFlavor | Belief scoring helpers bypass latest/active recipient values | Latest-delta logs alone cannot reconstruct these reads |
| <a id="fs007"></a>`fs007` | Policy recomputation / PERSONALITY or GRAND_STRATEGY_DERIVED | FlavorUpdate callback → AddFlavorAsStrategies reads personality/GS plus direct situations | PolicyAI base-weight recomputation; triggered layer can differ from read layer | Reported empty policy flavor logger; parameter/era branches remain explicit |

Strategy and specialization delta layers remain separate source records within the detailed producer owner, not universal additions to every runtime vector. The [representation checks](infrastructure/schema-dry-run.md) illustrate how these paths join score terms without adding modern verification.

## Task 1U unit consumer revalidation

Pass `audit-20261029-task1u-01`, snapshot `src-20261029-vp-c285a18`, partial current applicability. Existing fs001–fs007 IDs retain their original broader scope. Unit-specific paths now verified in [current flavor contracts](infrastructure/flavors.md#task-1u-current-unit-flavor-contracts): fs003+fs005 city max(0,latest) × Unit_Flavors reduction; fs004 GS/personality OFFENSE wrappers and EXPANSION settler; fs001 direct personality NAVAL in recommended force split. Active player fs002 is not the city coefficient multiplier. Cache-zero recommendation bypasses all runtime flavor layers and sums object rows. UPAI role ratio thresholds use fs004; ±150 city deltas use fs003, ±15 player deltas use fs002. Full personality initialization/delta timing/effective-profile values remain unresolved; no path is globally COMPLETE.

## Task 1U-B current unit signal closure

src-20261029-vp-548 / audit-20261029-task1u-b supersedes earlier partial unit-consumer routing assessment only. Existing fs001–fs007 IDs preserved. fs001 personality initialized/randomized/map-adjusted per [f002](infrastructure/flavors.md#task-1u-b-flavor-lifecycle-closure); fs002 active mutations broadcast only noncity; fs003 city signed latest with nonnegative read clamp consumes city strategy/military/economic/specialization vectors; fs005 joins Unit_Flavors coefficients into reset/rebuilt cache; fs004 is independent GS-modified personality and fs001 NAVAL direct bypass. UPAI supplies separate object/personality/strategy rows, not a universal effective flavor. Recommendation zero-cache uses fs005 alone. Cache serialization and specialization's direct latest mutation without immediate rebuild are explicit temporal edges; logging must identify actual consumed cache generation. Unit-side routing mapped; loaded effective data and runtime values UNRESOLVED_EXTERNAL. All-AI flavor provenance remains PARTIAL outside this slice.
