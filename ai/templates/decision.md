# Decision: {{question}}

Template instructions: extend the existing canonical topic owner first; for a missing decision copy into `<domain>/`, or `infrastructure/` for a shared process, then adjust relative links. Keep applicable sections; replace inapplicable sections with a short, reasoned omission. Do not silently omit a lifecycle stage. Register the ID and detailed owner in the [decision registry](../decision-registry.md). This template contains no audited behavior. An older maintained reference is the starting knowledge, with its baseline qualification retained.

## Source identity / verification

```text
Knowledge status:
Original audit baseline:
Last source verification:
Applicability / known version limits:
```

Use the [lightweight provenance and incremental workflow](../source-snapshots.md); a partial revalidation does not recertify untouched sections.

| Field | Record |
|---|---|
| Decision/process ID and kind | {{stable ID; DECISION or PROCESS}} |
| Domain / detailed owner | {{domain and this document/section}} |
| Disposition / scope | {{PLANNED/ACTIVE/DEFERRED/RETIRED; exact covered variants and exclusions}} |
| Knowledge basis / last baseline | {{REPORTED_SOURCE_AUDIT, PRECISE_SOURCE_AUDIT or UNCHECKED_NOTE; inherited baseline limits}} |
| Source snapshot / audit pass | {{registered exact target IDs; NONE until selected}} |
| Target applicability | {{UNSELECTED/NOT_REVALIDATED/PARTIAL/REVALIDATED; distinct from documentation coverage}} |
| Target source verification | {{UNCHECKED/PARTIAL/VERIFIED}} |
| Lifecycle mapping | {{UNMAPPED/PARTIAL/COMPLETE}} |
| Score provenance | {{UNMAPPED/PARTIAL/COMPLETE; includes booleans and gates}} |
| Logging mapping | {{UNMAPPED/PARTIAL/COMPLETE; separate from observability sufficiency}} |

Evidence index: assign local evidence IDs, each resolving to a snapshot, audit pass, repository-relative path, exact symbol/location, supported claim and applicability. Distinguish source proof from identified DB observations, runtime checks, inference and historical leads. Use the [source-identity rules](../source-snapshots.md). No registered source identity means no current-source verification claim.

## Purpose / decision question

{{Concrete question, output type and why this is a distinct choice/process. For a process, name its output contract and consuming decisions.}}

## Owner and trigger

{{Owner, callers, invocation conditions/frequency and ordering relative to state producers. Distinguish AI, human/automation, minor/barbarian and other relevant control modes; do not assume shared paths.}}

## Candidate generation

{{Enumeration source/domain and order; candidate identity including role/buildable/context where needed; injected/special candidates, duplicates and empty-set handling. For a non-selector, identify its input set and explain the omission.}}

## Eligibility and gates

| Term ID | Predicate / parameters | When evaluated | Pass/fail effect and reason | Evidence |
|---|---|---|---|---|

{{Include pre-generation exclusions, short-circuit gates, option/compile branches, legality queries and application rechecks. Boolean-only decisions still require full input provenance.}}

## Score pipeline

| Order / branch | Term ID | Input term IDs / state version | Output | Next consumer |
|---|---|---|---|---|

{{Show ordered initialization, per-candidate/per-flavor reductions, wrappers, direct changes, sanity/duration transforms, overrides, fallback and selection. Explain intentionally absent numeric scoring.}}

## Score-term provenance

Use the normalized [term schema](../score-provenance-schema.md); do not redefine its taxonomy here. A compact input/term index may precede detailed records. Repeat the following field slots for each term or indexed term family. Explicitly name inherited metadata and link shared producer contracts.

If current source demonstrates a new operation or provenance origin is needed, extend the central schema first.

```text
Term ID / name:
Decision / stage / order:
Source function / symbol:
Expression / calculation:
Operation:
Inputs:
Provenance class:
Exact origin:
Scope:
Conditions / gates:
Scale / units / type:
Lifetime:
Result / consumer:
Current logging coverage:
Source evidence:
Notes / uncertainty:
```

{{Give term sections stable explicit anchors; preserve nested input edges and state versions. Unknown helper internals or input origins remain unresolved, not complete.}}

## Flavor inputs

| Signal/path ID | Flavor / exact runtime layer and getter | Producer contract | Consuming term ID | Evidence |
|---|---|---|---|---|

{{Link rows in the flavor-signal registry. Separate personality/direct reads, active player, city/latest, grand-strategy transforms, strategy/specialization deltas and object rows. Explain if no flavors are consumed.}}

## Direct non-flavor inputs

| Input term ID | Direct field / manager / state | Producer contract and time | Consuming term ID | Evidence |
|---|---|---|---|---|

{{This is an index into the term records, not a substitute for calculations. Include constants, Defines, DB fields, derived metrics and direct trait/policy/belief/religion effects.}}

## Selection / threshold / randomness

{{Ordered eligible set, score version used, thresholds and units, deterministic overrides, sorting/ties, random seed/draw context, weight interpretation and empty/zero/negative/fallback behavior. Link selection-control terms and evidence.}}

Record RNG consumption as `random-draw`, including tie-breaking when applicable; keep `weighted-select` separate.

## Application

{{Exact applying function/call path and parameters, legality revalidation, queue/order changes, failure/retry and observable outcome. Distinguish selection from successful application and later completion.}}

## Side effects / persistent state / downstream consumers

{{Counters, demand commitments/releases, caches, update/reset timing, save/load and downstream recipients where material. Link temporal state terms and named contracts instead of duplicating another owner's formulas.}}

## Current logging and observability

{{Link observability rows and dimension assessments in logging-coverage.md. Record emitters, enablement, fields, stages, correlation limits and evidence basis. Distinguish source-assessed capability from logs actually observed at runtime.}}

## Missing logging required for reconstruction

| Gap ID | Decision/term and dimension | Missing value / timing / correlation | Reconstruction consequence | Observability row |
|---|---|---|---|---|

{{Requirements only; no logging implementation or new framework is implied. Explicitly account for selected, rejected and never-generated candidates where material.}}

## Reference lineage and incremental revalidation

{{Link canonical prior knowledge and its source-baseline record. Identify its last baseline, relevant source/data dependencies, compared identities where available, reused findings, changed/uncertain paths and revalidated scope. With an unknown old exact commit, re-establish a precise baseline rather than claiming diff equivalence. Retain profile/version distinctions and do not transfer older verification to an unchecked target.}}

## Uncertainties / deferred paths

| Unresolved question or excluded path | Affected coverage axis / consequence | Owning decision/area | Next evidence needed |
|---|---|---|---|

{{Update registry axes to match this document. A documented deferral does not make its untraced dependencies complete; do not imply whole-domain completion.}}
