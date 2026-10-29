# Logging coverage and reconstruction

This is an **audit of observability**, not a logging implementation or a new framework. It assesses what existing emitters could expose for an identified source/configuration and, separately, what a recorded runtime sample actually exposed. The maintained [observability reference](infrastructure/observability.md) consolidates recovered emitter/gap findings. This migration inspects no current logging source or runtime output; older capability references and current-target coverage are distinct.

## Status vocabulary

| Status | Meaning for one applicable dimension and stated scope |
|---|---|
| `FULL` | Existing output, with recorded prerequisites, exposes enough correctly timed and correlated information to reconstruct this dimension |
| `PARTIAL` | Some required information is exposed; list the missing values, branches, timing or correlation |
| `NONE` | Inspected relevant emitter/control paths expose none of the required information; cite the basis for absence |
| `UNKNOWN` | Relevant paths/fields have not been established; lack of an observed line alone does not prove absence |

Applicability is a separate field: mark an irrelevant dimension `not applicable` with a source-backed reason, leaving its status blank. It is not a fifth status or permission to omit unknown paths. A deterministic decision may make random draws inapplicable, while ordering and tie behavior can still matter.

Assessment completeness is the registry's `UNMAPPED/PARTIAL/COMPLETE` **logging mapping** axis. That axis says whether all applicable dimensions and gaps were audited. It does not say the logs are sufficient: mapping can be complete while coverage is `NONE`. Relevant `UNKNOWN` dimensions prevent complete mapping. Aggregate `FULL` requires every applicable component/path to be full; never average away a missing term or branch.

## Observability record and matrix

Assign stable global IDs such as `obs001`, with explicit anchors here. A record can cover a decision, a branch, a shared producer or a term family; state that granularity. Use child rows for score terms and distinct contexts whose coverage differs. Decision documents own detailed missing-information requirements; this document indexes them.

| Record field | Required content |
|---|---|
| Row ID / subject | Decision/process ID, candidate/term/path IDs and detailed owner |
| Source identity / applicability | Baseline and knowledge basis; exact snapshot + audit pass for current-target assessment; branch, actor/control mode, options and scope |
| Enablement / emitter | Compile/runtime logging switches, file routing and exact emitter/call site; unknown prerequisites remain explicit |
| Existing output | Actual field names/format, values/stages represented, omissions and precision limits |
| Correlation / timing | Player, city, turn/phase, invocation, candidate/role, score version, operation/slot and producer state identifiers as needed to join this choice; record existing capability, not a proposed protocol |
| Evidence basis | Source evidence, or identified runtime artifact/sample plus build/content identity; distinguish potential emission from observed output |
| Dimension assessments | Matrix below with applicability, status and supporting evidence per dimension |
| Gap references | Owning document's gap IDs and exact missing information/consequence |
| Notes | Unexamined paths, mismatch, aggregation limits and reconstruction verdict for stated scope |

For each row, assess:

| Dimension | Reconstruction question |
|---|---|
| `candidate-set` | What was enumerated/injected, in what identity/order, including relevant never-generated exclusions and duplicate roles? |
| `eligibility` | Which predicates passed/failed, with relevant parameters, gate order and early exits? |
| `score-terms` | Which individual contributions/transforms ran and what inputs/results did each use? |
| `flavor-values` | Which object coefficients and actual personality/active/latest/grand-strategy values or deltas were consumed? |
| `direct-inputs` | Which non-flavor heuristics, constants, Defines, manager metrics and direct state values were consumed? |
| `intermediate-score` | Can each material score version be reconstructed in evaluation order, including overwrites? |
| `final-score` | What score survived sanity, duration and other transforms, or what nonnumeric control output replaced it? |
| `threshold-cutoff` | What cutoff/comparator and reference value were used; which candidates remained? |
| `selection-behavior` | What ordered weights, tie/override/fallback rules and material random inputs/draw were used? Is the relevant seed/draw context sufficient, rather than just a later RNG state? |
| `selected-candidate` | What did the selector choose, including role/context and no-choice outcomes? |
| `rejection-reason` | Why was a materially relevant candidate rejected or bypassed, rather than merely absent? |
| `application-state` | Was the choice successfully queued/applied or later rejected, and what relevant state/demand changed? |

Use a long-form matrix to avoid an unreadable table:

| Row ID | Dimension | Applicability / reason | Status | Existing fields / evidence | Missing information / gap ID |
|---|---|---|---|---|---|

Do not infer every score term from a logged total when different input combinations can produce that total. A flavor-change line without baseline, consumer read time and layer identity may be only partial. A selected-candidate line does not prove successful application. A log inventory or a single runtime sample does not establish all-branch coverage.

## Missing-information requirements

Each gap in its detailed owner records `gap ID`, affected decision/term/dimension, missing value or event, required observation stage/correlation, and the resulting reconstruction limitation. These are requirements for a later authorized logging task, not implementation commitments. Link shared gaps rather than repeating them per consumer.

## Current assessments

Before the bounded Task 1U assessment added below, no current-target assessments existed. The rows in this historical table are **reported older-baseline capability/gap references** at `legacy-vp526`, basis REPORTED_SOURCE_AUDIT; current-target applicability UNSELECTED and status UNKNOWN. They seed revalidation rather than certify all paths. See the detailed [observability owner](infrastructure/observability.md) for emitters, enablement, field descriptions and unresolved gaps.

| Row ID | Subject / dimension | Reported baseline capability | Current-target status | Owning requirement / limit |
|---|---|---|---|---|
| <a id="obs001"></a>`obs001` | City production / candidates, intermediate/final scores, rejection and selected candidate | PARTIAL: PRE/POST vectors, invalid-item messages and final choice logs; per-term reconstruction missing | UNKNOWN | [City selection](production/city-selection.md); timing, component values and RNG context gaps |
| <a id="obs002"></a>`obs002` | Unit recommendations / role candidate lists | PARTIAL: RecommendUnit candidate-weight logs | UNKNOWN | [Unit production](units/production.md); helper totals do not reconstruct full shared selection |
| <a id="obs003"></a>`obs003` | Flavor propagation / baseline, deltas and consumed layers | PARTIAL: active/city changes; policy flavor logger reported empty | UNKNOWN | [Flavors](infrastructure/flavors.md); baseline, exact read layer and cache timing gaps |
| <a id="obs004"></a>`obs004` | Belief choice / category weights and component totals | PARTIAL: choice logs and aggregate plot/city/player components | UNKNOWN | [Beliefs](beliefs/selection.md); full field-level score provenance incomplete |
| <a id="obs005"></a>`obs005` | Operations / demand slots and city commitments | PARTIAL: setup/reserve/action messages; normal queue commitment unresolved | UNKNOWN | [Demand](units/operation-demand.md); slot identities, transitions and no-fill reasons |
| <a id="obs006"></a>`obs006` | Purchases / recommendation and applied outcome | PARTIAL: hurry/tactical/economic messages; purchase paths source-thin | UNKNOWN | [Purchasing](production/purchasing.md); per-path legality/terms/application correlation |

These are compact reference rows, not complete dimension matrices: logging mapping remains PARTIAL. Later audits expand them using the unchanged record/dimension schema above. No logging implementation or runtime test was performed.

## Task 1U current bounded assessment

<a id="obs007"></a>**obs007** — units.city-training direct scorer/recommendation and shared ChooseProduction. `src-20261029-vp-c285a18`, pass `audit-20261029-task1u-01`; source capability only, no runtime artifact. Enablement GC.getLogging && GC.getAILogging; routing uses city production log/split settings. CvUnitProductionAI::LogPossibleBuilds emits elapsed turn/civ/city, requested role, unit description and recommendation duration-weighted total. It emits surviving recommendations only after sort. Sanity body has no detailed term emitter. CvCityStrategyAI emits PRE/POST vectors, LogInvalidItem skip code and selected production before pushOrder. Correlation lacks invocation IDs, unit score version, operation/army/slot and gate/profile identity.

| Dimension | Current status / gap |
|---|---|
| Trigger | PARTIAL; turn/city known, invocation reason and early exit absent |
| Candidate set | PARTIAL; positive PRE/sorted recommendation list only; never-generated legality failures absent |
| Eligibility | PARTIAL; some sanity rejection codes, not native positive/negative predicates or same-muster silent filter |
| Base/object flavors | NONE in direct unit score emitters; coefficient products/cache origin absent |
| Runtime flavor | PARTIAL across separate flavor logs; no consumed-vector/cache-version join |
| Direct modifiers | NONE in sanity; no branch operands/ordered deltas |
| Intermediate score | PARTIAL; PRE versus duration-final POST only, no root/bonus/sanity/duration separation |
| Rejection | PARTIAL; broad skip sentinel, missing individual guard/operands and never-generated exclusions |
| Duration | PARTIAL; aggregate final total/turns, no before/divisor/Define values |
| Final score / competitor pool | PARTIAL; POST available, fallback restores PRE after POST log and detailed competitor origins absent |
| Cutoff / RNG context | NONE at inspected selector; no threshold/pool/draw/seed record |
| Selection | PARTIAL; chosen item visible, continuation/override/fallback explanation absent |
| Queue application | PARTIAL; choice emitted before pushOrder, no correlated outcome assertion |
| Operation/army fulfillment | UNKNOWN as whole lifecycle; normal completion/indirect recruitment still incomplete |

obs001/obs002 now have bounded current evidence through obs007; their untouched broader/historical paths retain prior status. obs003 flavor consumer-layer mapping partially revalidated, full producer/log join not complete. obs005 normal commit absence/direct-acquisition distinction revalidated, fulfillment still PARTIAL/UNKNOWN. obs006 purchase path mapping remains PARTIAL. No aggregate FULL or logging-mapping COMPLETE status justified. [Task 2U requirements](infrastructure/observability.md#task-2u-unit-logging-requirements) are provisional until remaining graph dependencies close.

## Task 1U-B current observation matrix

<a id="obs008"></a>obs008 — current 5.4.8 demand-to-training capability, src-20261029-vp-548 / audit-20261029-task1u-b. Source audit only, no runtime sample. obs007 remains prior capability evidence and is revalidated for unchanged scorer/selector paths. No FULL claim for aggregate unit decision.

| Graph dimension / term or transition | Status / exact reconstruction limit |
|---|---|
| Trigger / caller/control/profile | PARTIAL: city/turn descriptions, no invocation or effective gate/config snapshot, early-return causes omitted |
| Demand producer p001–p016 | PARTIAL: MilitaryStatus/Summary and optional SQLITE MilitarySummary include counts/recommended sizes/war state; Homeland diversity/best-city messages and trade/site/tactical logs provide subsets. No consistent producer-version/input-component joins |
| Flavor f001–f006 / t001–004 | PARTIAL: active/city flavor-change logs, recommendations show requested role/unit/weighted total; object products, actual cache generation and bypass-consumed values absent |
| Candidate set/pre-enumeration eligibility | PARTIAL: PRE positive candidates, not every enumerated unit or canTrain failure; native legality and Lua pass/fail not recorded per candidate |
| Direct sanity t005–043 | NONE for ordered arithmetic/branch operands inside CheckUnitBuildSanity; PARTIAL for aggregate skip sentinel and surviving totals at callers |
| Sanity rejected reason | PARTIAL: LogInvalidItem broad ProductionSkipReason, no individual predicate; same-muster silent filter and some hurry rejects missing |
| Duration / early-expansion / final score | PARTIAL: PRE/POST totals and turns, no ordered intermediates; POST emitted before fallback restoration |
| Competitor pool / process proxy | PARTIAL: candidate totals, no tier/reward/proxy/capital-sanity contribution or full competitor provenance |
| Cutoff / eligible ordering / seed / draw | NONE at inspected selection call/helper; chosen candidate does not recover weighted pool or draw; source determinism is not logged RNG context |
| Deterministic continuation/override | PARTIAL: selection visible, branch reason/reference score not joined |
| Queue intent and applied result | PARTIAL: LogCityProduction precedes pushOrder; no positive queue acceptance/result correlation, role propagation differs |
| Exact operation promise/finish/cancel | PARTIAL: operation setup/action/hurry messages, no complete token transition correlation; ordinary queue lacks token by behavior |
| Created unit→reserve→slot/stage | PARTIAL: recruitment/operation stage messages, not end-to-end city invocation/queue/unit identity and eligibility/rejection join |
| UPAI configuration/data/loader | NONE in unit-decision logs for complete effective profile or missing manifest action; runtime log files not inspected |

NONE here concerns missing conceptual trace at the inspected emitters, not a claim no other general-purpose game log exists. Missing source-inspected coverage and unavailable runtime evidence remain different. Runtime observability UNKNOWN at sample level because no game was launched. Current logging mapping includes these bounded dimensions; detailed per-producer/per-term coverage beyond this family table depends on current scope completeness. [Task2U refinement](infrastructure/observability.md#task-1u-b-task-2u-contract-disposition).
