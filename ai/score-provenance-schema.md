# Score and control-input provenance schema

## Model and identifiers

A decision is an ordered, conditional graph of **terms**, not a bag of additive bonuses. A term is a sourced input, calculation, predicate, state update or selection operation. It may return a number, boolean, enum/rejection code, candidate set, ordering or chosen object. Every contributing value, including non-score control inputs, must resolve to a leaf origin or an explicitly unresolved dependency.

Decision/process IDs come from the [registry](decision-registry.md). Within an owning document use stable term IDs `t001`, `t002`, etc.; qualify references as `decision-id:t001`. Give term sections explicit anchors such as `<a id="t001"></a>`. Revisions do not silently reuse an ID for another meaning. Flavor paths and observability rows use the identifiers defined in their registries. Evidence IDs may be local, e.g. `e001`, qualified by their document.

Use a small pipeline table (order, term ID, inputs, output, branch, next consumer) plus detailed term records. Repeated instances use an explicit index domain, e.g. one term per `(candidate, flavor)` and a separate reduction term. Do not write one row per runtime instance or duplicate shared producer formulas.

## Term record

Use this field/value format under each term anchor, or an equivalent table with exactly these meanings. Group fields for readability; no contributing field may disappear into unexplained prose.

| Field | Required content |
|---|---|
| Term ID / name | Stable qualified ID and readable purpose |
| Decision / stage / order | Owning decision/process; lifecycle stage; evaluation order and branch |
| Source function / symbol | Exact repository-relative path, qualified symbol and location, tied to snapshot/evidence |
| Expression / calculation | Actual expression or faithful pseudocode; initialization, assignments and evaluation order; not merely "situational modifier" |
| Operation | One of the operation kinds below; split sequential operations when their intermediates matter |
| Inputs | Ordered, named edges to term IDs or explicit leaf records; types, indices and branch dependencies; no unexplained variables |
| Provenance class | Immediate origin class below; inherited origins remain visible through input edges |
| Exact origin | DB table/column/row keys and loader/cache/getter; Define name, fallback and overrides; constant declaration/literal; runtime field/producer; or shared output contract and its detailed owner |
| Scope | Player/city/object/candidate/role/operation/flavor indices and control mode; distinction between actual, cached and hypothetical state |
| Conditions / gates | Exact predicates, option/compile gates and parameter variants; short-circuit order; whether excluded candidates are never generated or later rejected |
| Scale / units / type | Numeric or boolean/enum type, score units, percent convention, signedness, expected range, integer truncation, rounding, overflow-sensitive behavior and sentinel values where material |
| Lifetime | Initialization, update/reset/invalidation, observation timing, cache age, persistence/save-load and reversal if applicable |
| Result / consumer | Named output/version and its consumers; acceptance/rejection code, early exit, state mutation or next calculation |
| Current logging coverage | Link to observability row(s), `FULL/PARTIAL/NONE/UNKNOWN`, and relevant dimension; no new logging implied |
| Source evidence | Claim-level evidence IDs resolving to the named verification baseline; current-target claims require registered exact snapshot + audit pass and source; separate DB/runtime corroboration and applicability |
| Notes / uncertainty | Unknown leaves, inferred links, excluded variants and impact on coverage; use explicit `UNKNOWN`, not an invented value |

A compact sourced leaf can be recorded in an input table with these fields; repeated metadata may be explicitly inherited from a document header or indexed contract. State which fields are inherited. Values with different sources/lifetimes need separate records. An unavailable effective DB value stays unknown even if the consuming expression is verified.

## Operation vocabulary

| Kind | Meaning and details that must be recorded |
|---|---|
| `read` | Source/state/cache read or alias; exact layer and time |
| `gate` | Predicate with pass/fail/early-return effects and reason code |
| `set` | Initialization, overwrite, override, reset or conditional assignment; previous value handling |
| `add`, `subtract` | Ordered arithmetic; accumulation/reduction domain and initial value |
| `multiply`, `divide` | Ordered arithmetic, factors, denominator and zero behavior; include nested multipliers explicitly |
| `ratio` | Numerator/denominator definitions, population/domain, zero fallback and result scale |
| `percentage` | Percent representation, base, denominator, application order and truncation |
| `root` | Square/integer/other root, degree, pre-scaling, rounding and domain |
| `clamp` | Bounds, timing and whether stored state or only reads are clamped |
| `min`, `max` | Compared values, domain and tie/empty behavior |
| `duration` | Full turn/cost transform and its component terms; no opaque "duration penalty" |
| `threshold` | Exact comparator, cutoff origin, relative/absolute scale, inclusivity and rejected set |
| `order` | Sort/rank and stability, enumeration order and tie handling |
| `weighted-select` | Eligible ordered set, weight interpretation/normalization, sampling rule and zero/negative/empty behavior |
| `random-draw` | Purpose of the draw; range/distribution; deterministic seed/context where applicable; call/evaluation order; mapping from draw to result; fallback behavior; tie-breaking rules when that is the use |

These are normalized documentation names, not proposed APIs. For a compound helper, index its contract but decompose the underlying calculation into these operations. An unknown helper body is an unresolved dependency, not complete provenance. Separate gate outcomes from numeric rejection sentinels so negative scores cannot silently mean the wrong thing.

The normalized vocabulary is preferred, not a closed enum. If current source contains a materially distinct operation or provenance origin that cannot be represented without semantic loss, extend the central taxonomy rather than coercing it into the nearest existing category. Add the demonstrated category here first, before using it in individual decision documents.

## Value provenance classes

Classify the immediate source once; use input edges to retain mixed upstream origins. Operation and provenance are independent: a hard-coded threshold is `threshold` + `CODE_CONSTANT`; a legal-query predicate is `gate` + `LEGALITY`; a random draw can be `random-draw` + `SELECTION_CONTROL` with constant/DB seed inputs.

| Class | Distinction preserved |
|---|---|
| `FLAVOR_PERSONALITY` | Base runtime personality flavor; trace table/default, initialization/randomization and clamps |
| `FLAVOR_ACTIVE_PLAYER` | Active player flavor state after its actual mutations; not assumed equal to personality or city state |
| `FLAVOR_RECIPIENT_LATEST` | Recipient-local/latest flavor; identify city versus non-city recipient and read semantics |
| `FLAVOR_GRAND_STRATEGY` | Grand-strategy-derived flavor transformation/read, including its specific blending mode |
| `FLAVOR_STRATEGY_DELTA` | Economic/military/city strategy flavor delta; producer, target layer, start/end symmetry |
| `FLAVOR_SPECIALIZATION_DELTA` | City-specialization flavor delta, with its own producer/lifetime |
| `FLAVOR_OBJECT_ROW` | Object flavor row, e.g. `Unit_Flavors`; never a runtime personality layer |
| `DB_VALUE` | Other DB fields/rows, including handicap or formation data; exact keys and effective-data evidence |
| `DB_DEFINE` | Named global Define, with source fallback, load path and effective override uncertainty |
| `CODE_CONSTANT` | Hard-coded literal, named constant or fixed enum/control mapping |
| `RUNTIME_STATE` | Direct live/cached game state not covered by a more specific class |
| `DERIVED_METRIC` | Computed count, ratio, score or composite; every constituent has an input edge |
| `TRAIT_POLICY_RELIGION` | Direct trait, policy, belief or religion input; require subkind and exact state/effect field |
| `MANAGER_STATE` | Diplomacy/economic/military/strategy manager output; trace producer, inputs and freshness |
| `LEGALITY` | Native eligibility/legal query result; parameters, conditions and underlying predicate contract |
| `SELECTION_CONTROL` | Derived selection/random/threshold control state; trace its source inputs rather than hiding constants/DB fields here |

Flavor classes specify semantic layers even when their ultimate leaves are DB values. Record those leaves, getters and transforms as dependencies. A direct personality read is `FLAVOR_PERSONALITY` plus a **direct/bypass access path** in the flavor registry; it is not an active/latest signal. Runtime state, manager output and trait/religion effects do not become flavors merely because they influence the same score.

`UNRESOLVED` may mark an origin while investigating; it is not a provenance class and blocks complete provenance. Known origin with unknown effective value must say so separately. Do not conflate a data default, a proven effective value and a value actually consumed in a recorded runtime decision.

## Dependency and completeness rules

1. Name score versions after each ordered mutation. For example, write `s_after_t010 = s_before_t010 * factor`, with edges to both inputs, rather than implying commutative addition.
2. Explicitly connect gates and branch outcomes to candidate set changes, skipped calculations, overrides and fallback selection. Account for short-circuit, empty lists, zero/negative values, ties and application revalidation.
3. Decompose nested expressions into named intermediates. A per-flavor object/runtime product feeds a reduction; wrapper constants and demand then feed later transforms. Preserve every factor and its units.
4. Shared producers have an exact output contract: owning document/term, snapshot, type/scale, update timing, conditions and consumers. A manager accessor name alone is insufficient. Follow derived manager metrics and legality predicates to their contributing inputs, or record the unresolved boundary and keep provenance partial.
5. Cached/stateful dependencies are time-indexed: initialization and transitions produce the state read by a later decision. Describe the transition and baseline, including reversal/reset/load behavior where material. Do not force a temporal feedback loop into a cyclic same-invocation calculation graph.
6. Source/formula completeness and runtime reconstructability are separate. A fully traced expression can have no useful logging. Each term links [observability](logging-coverage.md); each flavor term also links its [signal path](flavor-signal-registry.md).
7. No contributing unknown leaf, unexplained broad modifier, unexamined branch or opaque shared producer permits score provenance `COMPLETE` for that scope. Explicit deferral describes an incomplete boundary; it does not certify it away.

The [schema representation checks](infrastructure/schema-dry-run.md) exercises this model without supplying current mechanics or values. Use the [decision template](templates/decision.md) for actual audit documents.
