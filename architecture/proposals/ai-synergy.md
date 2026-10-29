# AI synergy data and community collection proposal

Knowledge status: Maintained design/reference

Original audit baseline: `legacy-vp526` — VP 5.2.6 release family; release anchor `54dee87c630406f9c36f25bbb03d7562351ede51`; per-audit checkout/worktree and later revision limits remain qualified below.

Last source verification: Recovered source-audit reports only; exact per-claim verification checkout/worktree unknown. Resolved capture commits do not constitute new mechanical verification.

Applicability / known version limits: Best available reference for its reported baseline/profile; not yet revalidated for a newer selected target. Proposed designs are not implemented behavior. See [baseline and revalidation](<../../ai/source-snapshots.md#recovered-baseline>).

## Decision or contract question

What data model was proposed for synergy rules and community claims?

This is a maintained design reference. Rule/condition/effect separation, runtime scope and collection/export design were proposals, not established VP implementation. No deployment, account integration or behavior change is authorized by this document.

Canonical related owners: [Database loading, options and effective-data provenance](<../database-loading.md>), [Flavor production, propagation and consumption](<../../ai/infrastructure/flavors.md>).

Mechanical findings below retain the recovered audit baseline, not verification against an unselected current target. Original source-verification labels are inherited at this document level; `INFERRED`, `GUIDE_PATTERN`, `REPORTED_DB_SNAPSHOT`, risks and unresolved questions retain their narrower evidence strength. Reported fixes describe recovered local revisions, not work performed by this migration.

## Candidates and gates

- decision_domain

- target_type

- condition_tags

- effect_type

- effect_amount

- notes

This is the bridge from player claims to `AI_SynergyRules`, `AI_SynergyRule_Conditions`, and `AI_SynergyRule_Effects`.

## Scoring and inputs

- agree_count

- disagree_count

- net_agreement

- avg_impact

- avg_evidence

- weighted_score

## Data and configuration

Use a generic rule model:

- `AI_SynergyRules`

- `AI_SynergyRule_Conditions`

- `AI_SynergyRule_Effects`

- optional `AI_ObjectTags`

- later optional `AI_Archetypes`

Do not create separate pairwise tables for every relationship type.

Collect player submissions as structured synergy claims, not direct DLL rules.

A claim should include:

- target choice;

- conditions;

- positive/negative direction;

- strength;

- rationale;

- VP version;

- evidence level;

- votes/disputes.

Civilization pages should be UI views over the generic claim database, not the underlying data model. Also provide generic pages for belief-policy, policy-wonder, belief-wonder, and package synergies.

Voting should track agreement, impact, and evidence separately.

```text
Review the D1 schema for the synergy collector.

Focus:
- whether invite tokens are stored safely;
- whether votes are unique per user/claim;
- whether claims can be exported cleanly;
- whether future normalized rule exports are possible;
- whether moderation statuses are sufficient;
- whether indexes are needed for claim lists and score sorting.

Return critique and minimal schema changes.
```

## Version differences, risks and unresolved questions

This file records questions from `docs-ai/04_open_questions.md` that now have source/data answers. It does not remove the need for behavior-log validation where noted.

Policy and belief decisions are low-frequency. City production is hot. Unit production is hotter and already has UPAI role logic.

City-local/map-context conditions should be deferred until they can use cached city flags.

- Should trusted users have weighted votes?

- Should comments allow markdown?

- Should duplicate claims be merged or linked?

- Should public users see submitter names?

- Should accepted claims be hidden from raw claim lists?

- Should the app later support packages/archetypes as first-class objects?

- Should there be a Discord/forum verification phrase, or are DM invite links enough?

## Source and dependency lookup

1. Where should V1 rule data be loaded and cached?

2. Which conditions are safe as cheap player-level checks?

3. Should V1 target policy first, or policy plus desired wonders?

4. Should belief scoring receive only modest synergy deltas?

5. Should unit synergies be excluded initially because UPAI already covers role composition?

6. How should community-submitted claims be normalized into rules?

7. How should vote confidence, impact, and evidence be stored?

Give AI access to human-like meta knowledge about civ, belief, policy, wonder, building, unit, terrain, and victory-path synergies without hardcoding every case in DLL logic.

Rules are named containers. Conditions allow multiple requirements for one rule. Effects allow one rule to modify one or more targets and leave room for future effect types.

Path-dependent synergy is just a rule with multiple conditions.

Example:

- condition: civ is A

- condition: has belief B

- effect: add score to policy C

Generic non-civ synergy simply omits the civ condition.

SQL is authoring/storage only.

At load time:
1. read tables;
2. resolve Type strings to integer IDs;
3. validate invalid rows;
4. create compact C++ rule objects;
5. index by decision domain + target ID;
6. runtime checks only relevant rules.

Never query SQL or compare strings inside scoring hot paths.

Use only:

- player-level conditions;

- exact target effects;

- `ADD_SCORE`;

- `SUPPRESS_SCORE`;

- applied-rule logging.

Initial targets:

1. policy scoring;

2. desired-wonder or wonder/building scoring;

3. belief scoring, lightly and cautiously.

Avoid unit-production synergies initially unless they support civ/unique-component behavior and do not duplicate UPAI.

The synergy collector is a lightweight web app for gathering Vox Populi player knowledge about AI-relevant synergies.

The goal is not to let users directly edit AI rules. The goal is to collect structured gameplay claims that maintainers can later normalize into SQL/DLL-friendly rule data.

Examples of claims:
- “This civilization strongly benefits from this belief.”
- “This belief makes this policy branch stronger.”
- “This policy and wonder work well together only under a culture-victory path.”
- “This combo is overrated unless the start has desert/coast/nearby targets.”
- “This civ should avoid this package.”

The collector should replace messy forum threads with one claim per page, plus voting and comments.

Use ChatGPT extended thinking for product/design planning:
- data model
- UX flow
- moderation model
- claim taxonomy
- export format
- prompt design for Codex
- adversarial review of edge cases

Use Codex for implementation:
- creating the web app
- database schema
- API routes
- form validation
- CSV export
- admin pages
- deployment config
- tests
- security review
- iterative patches

Reason: ChatGPT is better as the architect. Codex is better inside the repo, editing files, running local checks, and producing diffs.

Use Cloudflare Pages + Cloudflare Functions/Workers + D1.

Rationale:
- Free tier should be enough for a mod-community tool.
- Pages handles the frontend.
- Functions/Workers handle form submission, invite/session logic, voting, comments, admin actions, and CSV export.
- D1 gives a simple SQLite-like database.
- No separate server is needed.

GitHub Pages alone is not enough because the app needs a backend and database.

Use maintainer-issued invite links by DM.

Do not make users create passwords. Do not rely on GitHub accounts. Do not make public self-issued invites at first.

Flow:
1. Player asks for access on forum/Discord.
2. Maintainer creates an invite link.
3. Maintainer DMs the link.
4. User opens the link, chooses or confirms a display name.
5. The app stores a session cookie.
6. User can submit claims, vote, and comment.

This keeps friction low while reducing spam and impersonation risk.

Use three or four roles:

- `user`: can submit claims, vote, and comment.

- `trusted`: same as user, but votes may be weighted higher or shown as trusted.

- `moderator`: can edit tags/status, hide comments, merge duplicates.

- `admin`: can generate/revoke invites and export data.

Avoid complex role permissions in V1.

Fields:
- `id`
- `display_name`
- `source`, for example forum or Discord
- `source_profile`, optional
- `role`
- `status`, active/revoked
- `created_at`
- `last_seen_at`

Fields:
- `id`
- `token_hash`
- `created_by`
- `claimed_by`
- `created_at`
- `claimed_at`
- `revoked`
- `note`

Store only the token hash, not the raw token.

Fields:
- `id`
- `title`
- `direction`, positive or negative
- `target_kind`, for example civ, belief, policy, wonder, building, unit, strategy
- `target_text`
- `conditions_text`
- `rationale`
- `vp_version`
- `submitted_by`
- `status`, raw/triaged/accepted/encoded/rejected/outdated/duplicate
- `created_at`
- `updated_at`

Fields:
- `id`
- `claim_id`
- `user_id`
- `agree`, +1 / -1 / 0
- `impact`, low/medium/high, stored as integer
- `evidence`, theory/observed/tested/accepted-meta, stored as integer
- `created_at`
- `updated_at`

One vote per user per claim.

Fields:
- `id`
- `claim_id`
- `user_id`
- `body`
- `created_at`
- `hidden`

Optional but useful:
- `claim_id`
- `tag_type`

Examples:
- civilization tags
- belief tags
- policy tags
- wonder tags
- victory path tags
- map-context tags
- era tags
- package/archetype tags

The public-facing UX should not expose the eventual SQL rule schema.

Players should submit gameplay claims, not rules.

Submission form:

1. Direction:
   - makes this choice better
   - makes this choice worse / avoid

2. Affected choice:
   - civilization
   - belief
   - policy
   - policy branch
   - ideology/tenet
   - wonder
   - building
   - unit
   - strategy/package
   - other

3. Conditions:
   Free text in V1, later optionally structured chips:
   - civilization
   - has belief
   - has policy
   - has wonder
   - terrain/resource
   - era
   - victory path
   - map type
   - nearby targets
   - other

4. Rationale:
   Why this synergy matters.

5. Evidence:
   - theorycraft
   - observed in games
   - tested repeatedly
   - widely accepted meta

6. VP version.

Civilization pages should be the default entry point because players think by civilization.

But civilization pages should be views over generic claims, not separate data silos.

Recommended pages:
- `/civs`
- `/civ/:slug`
- `/claims`
- `/claims/new`
- `/claim/:id`
- `/tags/:tag`
- `/moderation`
- `/admin/invites`
- `/admin/export`

Each civ page can show:
- top synergies
- anti-synergies
- belief suggestions
- policy suggestions
- wonder/building suggestions
- unit/component notes
- package/archetype claims
- unresolved/disputed claims

Generic claims should also be visible by object/tag pages, for example all claims involving Authority or a specific belief.

Use more than simple upvote/downvote.

For each claim:
- Agree / Disagree
- Impact: low / medium / high
- Evidence: theory / observed / tested / accepted meta

Priority score can be computed later from:
- net agreement
- impact
- evidence
- trusted-user weight
- maintainer status

Do not automatically turn popular claims into AI rules. Voting is prioritization, not truth.

Claim statuses:
- `raw`: newly submitted
- `needs_clarification`
- `triaged`
- `accepted_as_guidance`
- `candidate_rule`
- `encoded`
- `rejected`
- `outdated`
- `duplicate`

Maintainer tasks:
1. merge duplicates;
2. clarify vague conditions;
3. normalize tags;
4. mark high-confidence claims;
5. export candidate rules;
6. reject outdated or unsupported claims.

V1 exports should include:

- user_id

- display_name

- source

- source_profile

- role

- status

- created_at

- last_seen_at

- claim_id

- title

- direction

- target_kind

- target_text

- conditions_text

- rationale

- vp_version

- submitted_by

- comment_id

- body

- hidden

Later, add maintainer-normalized export:

V1 should include:
- invite-link login
- claim creation
- claim list and filtering
- claim detail page
- voting
- comments
- admin invite generation
- admin status updates
- CSV export

Do not include in V1:
- real-time updates
- complex password auth
- public self-registration
- nested comments
- advanced notifications
- automatic SQL rule generation
- direct integration with VP repo
- complex recommender/ranking logic

V1 should still include:
- invite token hashing
- session cookies with reasonable expiry
- CSRF protection for form actions
- rate limiting for claim/comment/vote endpoints
- admin-only CSV export
- hidden comments rather than hard deletion
- revoke invite/session capability

Avoid collecting unnecessary personal data.

Start with a minimal Cloudflare app.

Good stack:
- SvelteKit or React/Vite frontend
- Cloudflare Pages
- Cloudflare Functions/Workers
- Cloudflare D1
- simple SQL migrations
- CSV export endpoint
- optional local dev with Miniflare/Wrangler

Use Codex to implement, but give it this handout and ask for a small MVP first.

```text
Create a new lightweight web app for collecting Vox Populi AI synergy claims.

Use Cloudflare Pages/Workers with D1 if practical. If the repo is empty, scaffold the simplest maintainable project.

Read:
- docs/web-synergy-collector-handout.md

Task:
Implement MVP V1 only.

Required features:
1. Maintainer-generated invite links.
2. Invite claim flow: user opens invite, sets display name, receives session cookie.
3. Create claim form.
4. Claim list page with basic filters.
5. Claim detail page.
6. Vote on claim: agree/disagree, impact, evidence.
7. Comment on claim.
8. Admin page for invite generation and claim status update.
9. CSV export for users, claims, claim_scores, comments.
10. Basic D1 schema/migrations.

Constraints:
- Do not implement password auth.
- Do not implement public self-registration.
- Do not implement automatic AI SQL rule generation.
- Keep UI plain and functional.
- Add README with local dev and deployment steps.
- Add minimal tests or at least a manual test checklist.

Before coding, produce a short implementation plan and file structure. Then implement.
```

```text
Review the synergy collector UX for low-friction player submissions.

Focus:
- users should not need to understand SQL rules;
- civ pages should be filtered views, not separate data stores;
- generic synergies should be discoverable;
- voting should distinguish agreement, impact, and evidence;
- maintainers should be able to triage quickly.

Return concrete UI changes only.
```

```text
Add a maintainer-normalized candidate rule export.

Do not automatically convert all claims.

Add fields/admin controls for:
- decision_domain
- normalized_target_kind
- normalized_target_type
- normalized_condition_tags
- effect_type
- suggested_effect_amount
- maintainer_notes

Export candidate_rules.csv.
```

## Reference provenance

Consolidated from historical audit material. Source paths/symbols and line numbers in this reference belong to the reported older baseline; line numbers are navigation hints, not modern-source verification.
