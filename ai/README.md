# Maintained Vox Populi AI knowledge

This is a **decision-first rather than file-first** technical reference: the best established understanding of each represented decision and its supporting state. Begin with the existing topic owner, identify its verification baseline, then revalidate relevant changed or uncertain paths against the task-selected source. Update that owner instead of creating another generation of reports.

The conceptual lifecycle remains:

```text
trigger → candidate generation → eligibility/gates → scoring → selection
        → application → side effects/state → logging/observability
```

Omitted stages need a reason. Gates, deterministic rules and state propagation need provenance even when there is no numeric score. The recovered references are substantial lifecycle/scoring knowledge, but are not exhaustive term-by-term score records.

## Source qualification and reuse

A maintained audit is versioned knowledge, not a claim about every later VP release. The recovered references use the qualified [legacy baseline](source-snapshots.md#recovered-baseline): VP 5.2.6 release family, resolved release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`, uncertain per-audit verification worktrees, and distinct later repair/modmod profiles. No newer source target is selected by this migration. The civ5-live VP 5.4.8 profile does not select one.

Use the four lightweight provenance fields and [incremental revalidation workflow](source-snapshots.md#incremental-revalidation). Older source-audit findings remain the starting technical reference for their baseline; they are not automatically `VERIFIED_IN_SOURCE` for a different target. Current task-relevant source wins in a conflict. When the old exact commit is unknown, the first modern pass must re-establish the relevant paths precisely; apparent stability cannot prove equivalence.

Use the consuming project’s instructions and tools for procedure. Recovered policy files have no procedural authority. Evidence labels retain their original scope: source proof, effective DB observations, runtime checks, guide patterns, inference and mismatches are distinct. Migration performed no new mechanical/source, database or runtime verification.

## Canonical topic owners

| Domain | Start here |
|---|---|
| Unit training and demand | [Production](units/production.md), [operation demand](units/operation-demand.md), [role strategies / VP+UPAI](units/role-strategies.md) |
| City production and acquisition | [Shared city selection](production/city-selection.md), [AI purchasing](production/purchasing.md), [building valuation](buildings/valuation.md), [wonders](wonders/selection.md) |
| Beliefs and policies | [Belief selection](beliefs/selection.md), [policies/tenets/ideologies](policies/selection.md) |
| Strategic and operational pressure | [Strategy managers](strategies/managers.md), [operation planning](operations/planning.md) |
| Shared AI infrastructure | [Turn lifecycle](infrastructure/turn-lifecycle.md), [flavors](infrastructure/flavors.md), [weighted selection](infrastructure/weighted-selection.md), [observability](infrastructure/observability.md) |
| Shared mechanics | [Queue/application](../mechanics/production/queue-and-creation.md), [religion application](../mechanics/religion/founding-and-beliefs.md), [policy application](../mechanics/policies/adoption-and-ideology.md) |
| Shared architecture | [Operation/army state](../architecture/operations-armies.md), [DB loading](../architecture/database-loading.md), [optional features](../architecture/optional-features.md), [event dispatch](../architecture/event-dispatch.md) |

Detailed mechanics have one owner; AI decisions link that contract rather than maintain competing copies. Specialized optional-feature and later-repair knowledge is version/profile-qualified under its topic, not quarantined as a separate documentation generation. Source and function lookup detail stays with the decision/contract it supports.

## Control surfaces

- [Decision registry](decision-registry.md): one catalog, canonical ownership, baseline-qualified coverage, target applicability and deferrals.
- [Score provenance schema](score-provenance-schema.md): normalized terms, operations, origins and nested dependencies.
- [Flavor signal registry](flavor-signal-registry.md): layer-specific producer-to-consumer paths.
- [Logging coverage](logging-coverage.md): reconstruction dimensions and gaps, separate from assessment completeness.
- [Decision template](templates/decision.md): create a missing bounded decision or extend a canonical owner where clearer; do not force every subsystem into identical boilerplate.
- [Task 1U scope](units/task-1u-scope.md): the preserved demand-to-training boundary, starting from the migrated unit references.
- [Schema representation checks](infrastructure/schema-dry-run.md): examples, not new source verification.

Maintain registry rows and owning documents together. Coverage `COMPLETE` always refers to a named baseline, scope and dependency surface; known gaps and deferred adjacent decisions remain visible. This reference corpus does not certify the entire AI inventory as exhaustive.
