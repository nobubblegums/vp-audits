# Schema representation checks against recovered scenarios

This is a **documentation-format test**, not a current VP audit. Inputs are the maintained [unit production](../units/production.md), [operation demand](../units/operation-demand.md) and [flavor propagation](flavors.md) references, derived from the recovered `legacy-vp526` baseline. They are the starting technical knowledge; this illustrative allocation adds no verification against a newer target. All cases have modern snapshot/pass `NONE`, modern source verification `UNCHECKED`, and no current logging assessment. IDs prefixed `H-` are local explanatory symbols, not registered decisions, terms or flavor signals.

No numeric constants, effective DB values or runtime observations are retained. Symbolic expressions below represent the historical descriptions for testing the schema; they do not assert current formulas. Where the history names a helper without its full arithmetic, the test records an unresolved body instead of inventing one. That is a provenance limit, not a reason to perform Task 1U now.

## Case 1: unit-production scoring

Historical symbol leads: `CvUnitProductionAI::AddFlavorWeights`, `GetWeight`, `RecommendUnit`, `CheckUnitBuildSanity`, and city-production duration/selection helpers. Actual current symbols, types, options and paths remain unverified.

| Local node | Symbolic input/output | Operation | Immediate provenance / dependency |
|---|---|---|---|
| H-U1 | `O[u,f]`: object coefficient | read | FLAVOR_OBJECT_ROW; historical `Unit_Flavors` row, loader and getter need revalidation |
| H-U2 | `C[c,f]`: effective recipient read | read | FLAVOR_RECIPIENT_LATEST; link the producer chain in case 3, not an assumed personality value |
| H-U3 | `p[u,f] = O[u,f] * C[c,f]` | multiply | DERIVED_METRIC; edges to H-U1/H-U2 |
| H-U4 | `W[u] = sum_f(p[u,f])` | set/add | DERIVED_METRIC; explicit flavor domain, initialization and accumulation order still to verify |
| H-U5 | `eligible = legal(c,u,context)` | gate | LEGALITY; exact native predicate, parameters, inputs and rejection effect required |
| H-U6 | `R = integer_root(K * W[u])` | multiply/root | DERIVED_METRIC; separate CODE_CONSTANT leaf `K`, root degree/rounding/domain records |
| H-U7 | `S1 = R + bonus` | add | DERIVED_METRIC; direct bonus has its own state/DB/trait/manager dependency record |
| H-U8 | `S2 = S1 - penalty` | subtract | DERIVED_METRIC; conditional penalty input and gate recorded separately |
| H-U9 | `S3 = (S2 * factor_a) * factor_b` | multiply/multiply | DERIVED_METRIC; explicitly ordered nested factors with independent origins |
| H-U10 | `reject(reason)` or accepted score | gate/set | LEGALITY or derived predicate; distinguish rejection enum from negative score arithmetic |
| H-U11 | Duration-adjusted accepted score | duration | Helper-body **UNRESOLVED**: exact subexpressions, turns input, origin leaves and truncation required; no claimed complete formula |
| H-U12 | Threshold-filtered ordered candidates | threshold/order | Cutoff source, comparator and reference score are separate leaves/terms |
| H-U13 | Chosen candidate or fallback | weighted-select/random-draw | Ordered weights, seed/draw/tie context, zero/empty behavior and override paths required |

H-U7 through H-U9 exercise direct arithmetic shapes; they are not an asserted sequence of current bonuses. A real audit must enumerate actual terms/branches rather than keep these generic names. Source defaults, code constants, DB fields, Defines, manager state and direct trait/policy/belief/religion effects can each be attached without being mislabeled flavors.

An example **full field allocation** for H-U3 shows that the compact pipeline does not replace detailed records:

| Term field | Dry-run allocation |
|---|---|
| Term ID / name | H-U3 / per-object per-flavor product; illustrative only |
| Decision / stage / order | Historical unit-production scenario / scoring / before reduction |
| Source function / symbol | Historical lead `CvUnitProductionAI::AddFlavorWeights`; exact current path/location UNKNOWN |
| Expression / calculation | `p[u,f] = O[u,f] * C[c,f]` |
| Operation | multiply |
| Inputs | H-U1 object-row leaf; H-U2 recipient-read chain; city/candidate/flavor indices |
| Provenance class | DERIVED_METRIC; FLAVOR_OBJECT_ROW and FLAVOR_RECIPIENT_LATEST remain visible on input edges |
| Exact origin | Product of the two named nodes; their exact keys, loader/getter and producer evidence remain unverified |
| Scope | One candidate/flavor for one city; actual actor/option variants UNKNOWN |
| Conditions / gates | Flavor-update invocation and index domain require revalidation; no assumed unconditional path |
| Scale / units / type | Object-weight × recipient-preference → contribution units; exact numeric type, bounds and overflow behavior UNKNOWN |
| Lifetime | Intermediate feeds a possible cached reduction; actual update/reset/read timing UNKNOWN |
| Result / consumer | H-U4 reduction, then wrapper/sanity paths if present |
| Current logging coverage | UNKNOWN; no current row registered; per-product/flavor reconstruction dimensions would be assessed |
| Source evidence | Historical unit-production scoring paragraph only; snapshot/pass NONE; no current verification |
| Notes / uncertainty | Allocation proves schema fit, not mechanical truth; all unknowns block current completeness |

**Observability test:** a historical claim of pre/post totals cannot by itself establish per-flavor products, individual direct bonuses, factor values or duration internals. The logging matrix can assess each dimension separately and link missing-information requirements. It must leave actual current statuses unknown until inspected.

## Case 2: operation unit demand and wrapper scoring

Historical symbol leads include formation-slot data, reserve suitability, demand lists, city/muster checks, operation recommendation helpers and `CvCityStrategyAI::ChooseProduction`. The historical adversarial review leaves queue commitment/completion questions unresolved.

| Local node | Symbolic input/output | Operation | Provenance and ownership requirement |
|---|---|---|---|
| H-O1 | Required role/slot flag and primary/secondary role | read | DB_VALUE / code mapping where applicable; exact formation keys and loading contract |
| H-O2 | Suitable reserve candidate set and suitability inputs | gate/order | LEGALITY, RUNTIME_STATE and DERIVED_METRIC records; no opaque "reserve demand" value |
| H-O3 | Unfilled required slot produces pending demand | gate/set | Boolean/control output, not a numeric score; pending/committed state transition has a lifetime |
| H-O4 | City/muster/access/role gates expose a recommendation | gate | Individual predicates and short-circuit results; edges to H-O1/H-O3 and recommendation output |
| H-O5 | `G`: grand-strategy-derived flavor; `H`: skipped-demand state | read/read | Separate FLAVOR_GRAND_STRATEGY and MANAGER_STATE nodes with producer timing |
| H-O6 | `X = G + H`; `Y = X * M`; `Z = B + Y`; `P = Z + W` | add/multiply/add/add | Four intermediate term records; `B` and `M` are DB_DEFINE leads with exact names/default/override fields, `W` links H-U4 |
| H-O7 | Conditionally multiply accepted wrapper score by `E` | gate/multiply | Separate runtime predicate and CODE_CONSTANT factor; exact value and current branch unverified |
| H-O8 | Wrapper → sanity → shared city competition | explicit term edges | Reuse case 1 contracts with context parameters; do not copy or assume identical gates |
| H-O9 | Queue/purchase branch, commit, cancel, complete, fulfill | gate/set | Application/control transitions; before/after state, legality and downstream consumer links |

The schema handles upstream demand without fabricating a numeric score. It also distinguishes direct grand-strategy flavor from cached object/latest weights within one nested wrapper. Required flags and numeric values keep separate provenance even when a branch consumes both.

**Observability test:** a final production line cannot reconstruct which slot demanded the unit, whether a reserve failed eligibility, the old skipped count, or whether a queued unit was committed/fulfilled. Distinct dimension/gap records can name these missing identities, values and transitions. Unresolved historical commitment behavior remains a question, never a completed state edge.

## Case 3: flavor propagation and bypass reads

Historical symbol leads include personality initialization/randomization, active/city flavor changes, recipient `SetFlavors`/`ChangeFlavors`/`GetLatestFlavorValue`, grand-strategy reads and production `FlavorUpdate` cache rebuilds.

| Local path/node | Representation in the schema | Required detail / unresolved boundary |
|---|---|---|
| H-F1 | DB personality row/default → randomization → clamp → `P` (FLAVOR_PERSONALITY) | Separate DB_VALUE/DB_DEFINE/CODE_CONSTANT leaves as applicable, random-draw term, bounds, exact transform and initialization lifetime |
| H-F2 | `A_next = A_prev + delta_player` (FLAVOR_ACTIVE_PLAYER) | STRATEGY_DELTA producer/conditions, start/end reversal, routing and time-indexed baseline |
| H-F3 | `C_next = C_prev + delta_city + delta_specialization` (city latest) | Separate strategy/specialization nodes, actual order, registration/new-city/load path and each recipient mutation |
| H-F4 | Stored latest state → getter-specific read/clamp → cache rebuild | Distinguish stored signed state from returned value, exact bounds, rebuild/reset/invalidation and consumers |
| H-F5 | Personality + grand-strategy inputs → derived read | Parallel FLAVOR_GRAND_STRATEGY contract, mode/scale, clamp/blend arithmetic and conditions; helper body UNKNOWN until audited |
| H-F6 | Personality getter → direct consuming term | Layer PERSONALITY + access path direct personality bypass; no invented edge through active/latest state |
| H-F7 | Latest/object path → H-U2/H-U1 → H-U3/H-U4 | Flavor registry edges end at exact score-term IDs, preserving the joining object coefficient |

These paths are separate, not a claim that city latest is always active player plus city deltas. A triggered recipient callback may read another layer; the schema records trigger and consumed value independently. Previous-state edges and initialization/reversal contracts handle temporal feedback without a cyclic same-invocation graph.

**Observability test:** delta logs without baseline, destination layer, read timing or cache-rebuild identity do not establish effective values used. A direct-personality consumer can bypass all logged latest changes. The registry's path/layer/access fields and the logging dimensions can express both gaps.

## Design-check result and limits

The three cases fit one term graph, including booleans, DB/default distinctions, object × runtime flavor, ordered nested arithmetic, direct heuristics, gates, demand state, transforms and stochastic selection. No second formula taxonomy or special unit-only schema is needed. The test also catches incomplete history: duration/blend bodies, individual bonus lists, precise gates and commitment paths remain **UNRESOLVED**, and could not be marked complete in an actual audit.

This dry run validates field coverage and explicit uncertainty handling only. It does not verify VP behavior, current logging, any numeric value, or the completeness of the historical descriptions. Actual source evidence and decomposition belong to Task 1U.
