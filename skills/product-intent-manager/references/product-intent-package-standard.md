# Product Intent Package Standard

## Purpose

A Product Intent Package (PIP) is the smallest current description of what
product to build, how people use it, how its consequential processes work, and
which outcomes and constraints matter. It is not a project plan, implementation
log, decision history, or proof that every possible artifact exists.

## Format 7.0

This standard defines format `7.0.0`. Format 7 makes the PIP a pure current-
intent package:

- the default package has three files;
- product context stays minimal; textual acceptance is exceptional;
- package and item statuses, readiness fields, signatures, confirmation
  records, implementation observations, and handoff records are removed;
- alternative intent lives in an isolated PIP fork rather than beside canonical
  intent;
- DCL is an optional product-wide default with narrow overrides; and
- each diagram file succinctly explains the active reasons for its current
  design without recounting product history.

The default package is:

| File | Responsibility |
| --- | --- |
| `product.yaml` | Minimal product context: name, release, outcome, boundary, actors, capabilities, exclusions, measures, optional DCL, and direct links; no behavioral logic |
| `architecture/stack-context.md` | Physical clients, services, managed platforms, stores, external systems, responsibility, owned state, deployment placement, and connections |
| `experience/user-flows.md` | Actor goals, actions, surface topology, visible states, choices, failure, recovery, and outcomes |

Populate these three files. Do not pre-create optional directories or empty
placeholders. Add another artifact only when it communicates distinct
information needed to understand, build, or recognize the intended product.

Use these conventional paths when an optional artifact is needed:

| Optional artifact | Conventional path |
| --- | --- |
| Unique acceptance cases that cannot be meaningfully diagrammed | `acceptance.yaml`, only under the exception below |
| Editing authority | `governance.yaml` |
| Intended journey | `experience/journeys/JOURNEY-*.md` |
| Screen detail or local mockup | `experience/screens.yaml`, `experience/mockups/` |
| Product-specific design patterns | `experience/design-system.md` or a link to the authoritative design system |
| Shared rule/decision diagrams or state machines | `behavior/rules.md`, `behavior/state-machines.md`; `behavior/decision-tables.md` only for the compact-matrix exception below |
| Data model or product-significant schema | `data/data-model.md`, `data/schema.yaml` |
| Shared, external, or product-significant contracts | `contracts/contracts.yaml` or `contracts/openapi.yaml` |
| Runtime sequences | `sequences/sequences.md` |
| Measurable quality constraints | `quality/constraints.yaml` |
| Complex deployment topology | `architecture/deployment.md` |
| Complex coordination topology | `architecture/coordination.md` |
| Capability module boundary and internal owners | `modules/<capability>/boundary.md` and only the needed local artifacts |
| Cross-module runtime composition | `workflows/<process>.md`, or an existing focused sequence |

See [Artifact Responsibilities](artifact-responsibilities.md) for the trigger
and ownership of each artifact.

## Capability modules

Use capability modules when intertwined logic prevents useful local review.
Keep one canonical PIP and one minimal root `product.yaml`; small products keep
the three-file default. The paths above are conventions, not a requirement to
keep all sequences or data records in global type-based directories. Existing
paths may remain when direct boundaries already provide isolation.

A module groups a coherent responsibility whose internal logic can be reviewed
against an explicit public promise. Choose boundaries around reusable behavior
and state ownership, not simply actors, screens, file types, or deployment
units. Reusable calculations can sit inside state-owning domains; workflows
compose their operations. A module does not imply a separate deployed service,
repository, package, database, or independently versioned product. A PIP
reorganization alone does not authorize changing those implementation boundaries.

### Public behavioral boundary

Public means consumable by other PIP modules, not internet-accessible. The
boundary must let a caller reason about an operation without reading its
internal algorithm. For each operation, represent the applicable:

- input meanings, provenance, caller obligations, authorization, and scope;
- successful results, their meaning, and permitted durable or external effects;
- refusal, partial failure, cancellation, and recovery outcomes, including
  preserved state and responsibility for retries;
- privacy and retention, freshness, compatibility, ordering, idempotency,
  atomicity, and stated quality or resource guarantees on which callers rely.

Do not invent guarantees or fill every category mechanically. Preserve current
intent and resolve a consequential gap before asserting independence. Required
side-effect differences, such as ephemeral processing versus persisted results,
must be explicit operations or already-intended modes, not a hidden default.

The public promise belongs in a focused diagram or boundary section of an
existing owning diagram. Use sequence, decision, or state notation appropriate
to the meaning; prose explains scope and rationale. Schemas may own exact shapes
and parameters, but YAML and prose are not alternate behavioral specifications.
Internal diagrams own how the promise is fulfilled and link to exported rules
instead of defining them again. If an existing rule diagram already supplies a
complete public operation, use it directly rather than creating a second copy.

Each consuming call names and directly links the public operation, supplies
inputs from named sources, and shows how its outcomes affect the caller. Keep
the callee's internal predicates, formulas, storage mechanics, and recovery
loops in the callee. A generic call is sufficient only when its linked boundary
actually defines the result and effects the caller relies on.

### Internal ownership and composition

Group internal sequences, shared decisions, relevant state/data views, and
module-local experience detail under the capability when useful. Add only the
artifacts the module needs: a small stateless module may fit in one file. Do not
add per-module `product.yaml`, acceptance files, versions, or review ledgers as
modularity machinery. The root retains product context, physical architecture,
and cross-module experience; it links local detail instead of duplicating it.

Each entity has one full schema/ERD owner. Other views use linked reference
projections. If several modules legitimately share a physical record, make
field-level mutation ownership and the shared invariants explicit without
copying the full entity or pretending the database is separated. An external
reader relies on an exported read/projection contract; an external writer uses
an owned mutation operation or the explicit shared integration boundary.

Cross-module workflows own call order, orchestration decisions, and handling
of public outcomes. They do not repeat the called algorithms. They retain
detailed logic that genuinely belongs to the composition, especially shared
transactions, lock order, revalidation, cancellation races, or compensation.
Keep one owning integration sequence for each such invariant, naming the
participating module operations, physical state, and commit/failure behavior.
Do not split an atomic operation into independently committing calls merely to
make modules appear independent. Constant reliance on each other's internals
is a reason to revise the boundary or review the coupled owners together.

Keep physical systems as sequence lifelines; label logical operation calls and
link their boundaries. A small, explicitly logical module-interaction view may
accompany architecture when it adds useful composition context, but it must not
depict modules as deployed services or become an exhaustive dependency registry.
Direct consumer-to-operation links are enough to derive incoming references
during a review; do not maintain duplicate reverse edges.

### Reorganizing an existing package

Start with one or two reusable boundaries and their real consumers before a
large file move. Separate public promises from internal logic, split broad
records by ownership, and preserve behavior, stable IDs, and direct links.
Keep a concise pointer at an old path when external references still need it;
do not leave a second specification there. Migration plans and review results
stay outside the canonical PIP.

Exercise [module-bounded review](change-and-handoff.md#module-bounded-review)
on a representative change: can the reviewer establish why consumers remain
valid without opening unaffected internals? A necessary excursion identifies
an incomplete contract or real coupling, not permission to ignore the detail.
Folder structure and link checks alone do not prove semantic isolation.

## Diagram-first product logic

Strongly prefer diagrams as the canonical representation of behavioral logic.
They are the specification of that logic, not an overview of a specification
hidden in paragraphs. Rules, gates, permissions, eligibility, validation,
priorities, calculations, transitions, timeouts, retries, fallbacks, and recovery
belong in rendered nodes, labeled edges, guards, branches, and attached notes.
Prose alone is not the default acceptable owner of these facts, even when it is
in the same Markdown file as a diagram. Mermaid source comments do not count:
the logic must survive rendering/export.

Put process-local conditions in sequence `alt`/`else`, `opt`, `loop`, or `break`
blocks. Show the condition, allowed action, refusal/failure result, and what is
preserved or not written where consequential. A note may define a predicate or
formula; it must attach to the relevant step, not replace branching with a wall
of text. Use a focused decision flowchart for reusable rule selection or complex
precedence, linked from consuming sequences. Keep lifecycle guards on state
transitions, data invariants in ERD rows/edges or attached notes, and visible
choices and outcomes in user flows. Do not move internal gates into user flows
or ordered execution into ERDs just to make everything visual.

Name exact inputs, operators, units, thresholds or parameter references, and
outcomes. “Check eligibility,” “apply RULE-001,” and “retry if needed” are not
complete logic unless they link to a diagram that owns that detail. Show
evaluation precedence and otherwise/no-match outcomes when they affect behavior.
Do not invent an unspecified branch outcome; resolve that product choice first.

Keep one maintained owner. A shared `RULE-*` may be a heading/anchor containing
a decision diagram or identify logic owned by a sequence; it need not be a YAML
rule record. Consumers name the rule, link to its exact diagram, and use its
result without copying its predicates. Existing rule IDs and externally used
paths can retain concise pointers to their diagram owners. Do not add a rule
registry just to inventory diagrams.

Prose remains appropriate for product purpose, boundary, definitions, current
causal rationale, and navigation. Structured records remain appropriate for
schema and contract shapes, exact parameter/configuration values, measurable
quality bounds, and editing authority. Textual acceptance is limited to the
[Acceptance](#acceptance) exception. Structured records must not
become alternate prose/YAML homes for branching behavior: the consuming diagram
shows when and how those values or contract results change an outcome.

A compact decision matrix is a narrow exception when a genuinely tabular rule
is materially clearer there than in a branching graph. Prefer a rendered table
node in the owning diagram; a standalone structured matrix is acceptable when
that is the clearer exact representation. State its condition columns,
precedence/match policy, and outcomes, and link the consuming diagram directly.
Do not use this exception for paragraphs or `when`/`then` bullet lists. An
accessible text equivalent may accompany a diagram but must introduce no new
logic. An observed viewer limitation may use the documented fallback without
discarding requirements; a missing local renderer is not such a limitation.

For updates, inspect prose and YAML around the affected diagrams for hidden
logic. Move each rule to its appropriate diagram owner, preserve its meaning
and references, and remove duplicate normative prose after confirming it is
represented. Split oversized diagrams by responsibility rather than shrinking
text or exporting the hard parts to prose. This is a scoped authoring rule, not
authorization to rewrite unrelated PIPs or a requirement for extra artifacts.

Review question: with narrative prose hidden, can a reader follow the affected
conditions to every material outcome using the diagrams and their explicitly
linked structured inputs? If not, the logic representation is incomplete.

## Canonical current intent

The PIP at the project's canonical location is the current intended product.
Its contents need no `status`, `build_ready`, `confirmed`, `proposed`,
`observed`, `blocked`, or `stale` fields. The package does not contain
signatures, approvals, readiness evidence, implementation observations, review
results, or handoff records. It just describes the current end state.

Optional `governance.yaml` is the narrow exception: it may record current
editing authority so an agent can verify whether a requester may change the
canonical package. It does not approve individual changes or become product
decision history. Git records who committed each edit.

Keep these outside the canonical PIP:

- discovery evidence and source inventories;
- conversations, questions, alternatives, and product-leader review;
- implementation plans, tickets, assignments, progress, and receipts;
- as-built findings, deviations, audit results, and implementation DCL; and
- superseded requirements and ordinary decision history.

Git records ordinary changes. The task system or concise working notes record
implementation work. Adjacent research, design, operations, schema, or contract
sources remain authoritative for their own detail; link them when needed rather
than copying them into the package.

### Write the end state, not the change

Use present-tense statements of intended fact or timeless directives. Both
“The sign-in surface uses the provider's native component” and “Use the
provider's native component for sign-in” describe the required product without
implying a previous design or an implementation task. This convention applies
throughout YAML, diagram labels, supporting notes, rationale, and acceptance.
It specifies intent, not evidence that the code already implements it.

| Avoid in PIP content | State the intended product instead |
| --- | --- |
| Reuse the existing sign-in component instead of the old custom view. | The shared sign-in surface uses the provider's native sign-in component. |
| Modify `loadProgress()` to reconcile uncertain requests. | `loadProgress()` owns initial loading and reconciliation of uncertain requests. |
| We added retries; next, implement the failure message. | A failed read offers Retry and keeps Increment unavailable. |
| Add graceful shutdown in the next deployment. | On each deployment, drain in-flight work before stopping the worker. |

Keep work order, reuse/modify/new labels, one-time migration steps,
implementation gaps, completion reports, and before/after comparisons in tasks
or working notes outside the PIP. Keep ordinary history in Git. Do not merely
change “will add” to “adds”: rewrite the sentence around the resulting behavior
or constraint.

Do not remove technical design meaning to achieve this. Required components,
function ownership, input provenance, detailed runtime steps, retries, data
rules, and compatibility constraints remain in their owning artifacts. “A retry
uses the original request key” describes runtime behavior, not construction
work. Current release identifiers and supported schema or protocol versions
also remain valid facts; a chronology of version changes does not.

Standing instructions for maintenance, deployment, recovery, or other recurring
conditions are also valid when they define product-significant operating
behavior. For example, “During maintenance, preserve saved work and show the
unavailable state” remains applicable after any one maintenance event.
Distinguish these by meaning, not verb tense: after an action is performed,
does the rule still govern the next applicable situation? A standing rule does
not become stale merely because it has been followed; a one-time change task
becomes obsolete when completed. Preserve the former in its owning PIP artifact
and keep the latter in companion tasks. Incidental runbook commands still
belong outside the PIP.

### Alternative intent uses a PIP fork

When a different end state is being proposed, create an isolated PIP fork in a
branch, worktree, or separate proposal location. The fork must describe one
coherent intended product, not a collection of options or status-labeled
fragments. Its location and review context identify it as noncanonical; do not
add proposal status fields inside it.

Do not place conflicting alternatives in the canonical PIP. Resolve product
questions outside the package. When a requester whose editing authority covers
the complete change adopts the fork, update the affected canonical intent
together through the normal Git process. A direct, unambiguous authorized
instruction may update canonical intent without a separate signature, approval,
or signoff record.

## Product record

Keep `product.yaml` minimal, flat, and outward-facing. It supplies product
context and direct links, not a growing inventory of rules or acceptance cases.
Do not use any field as a loophole for logic that belongs in a diagram.

```yaml
schema_version: 7.0.0
name: Example product
release: 1.0.0
outcome: The outcome this release creates for its users.
boundary: What is included in this product and release.

dcl:
  level: 4
  basis: >-
    Users rely on the core path in production, common failures need recovery,
    and current load does not justify generalized scale machinery.

actors:
  - id: ACTOR-001
    name: User
    goal: Complete the primary job.

capabilities:
  - id: CAP-001
    name: Complete the primary job
    actor_ids: [ACTOR-001]
    outcome: The user can recognize that the job completed.

exclusions:
  - A clear consequential exclusion.

success_measures:
  - An observable product measure when one is useful.
```

`outcome` is the user or product result this release creates. `boundary` is the
included and excluded product scope needed to interpret that result. Keep
algorithms, queue behavior, runtime topology, database rules, deployment
settings, retry logic, and similar implementation mechanics in their owning
artifacts. Link those owners when needed instead of repeating their contents in
the root record.

The `dcl` mapping is optional. Omit any other empty field. Add an ID only when
another file or external system references the record. Do not add a package ID
unless something genuinely refers to the package by that ID. Do not duplicate
capability inclusion lists or broad `related_ids` inventories; use direct,
purpose-specific links where a reader needs them.

## Acceptance

Observable outcomes in the owning diagrams provide acceptance meaning. Do not
copy their branches, rules, gates, or failure/recovery behavior into
`product.yaml` or `acceptance.yaml`, even as given/when/then examples. Neither
file is a secondary behavioral specification or an exhaustive test catalogue.

Omit `acceptance.yaml` unless a unique acceptance case cannot be represented
meaningfully in a diagram or its attached notes. Keep only the irreducible case,
a brief `why_not_diagram` explanation, and a direct `verifies` link to the
relevant owner. The explanation must identify a real representation limitation,
not merely say that prose is easier. An isolated qualifying case may stay inline
on its capability; a separate file is warranted only when qualifying cases need
their own small owner. Do not repeat the same case in both files.

More scenarios, cross-capability scope, complex gates, or a crowded diagram do
not qualify. Extend or split the owning diagrams. Exact quality values and
contract shapes keep their existing structured owners; their presence does not
justify another acceptance copy. Necessary non-diagrammable evaluation context
may be textual, but its runtime conditions and behavior remain diagram-owned.

Before adding text to either file, ask whether it is essential product context
or a genuinely non-diagrammable acceptance case. Otherwise move it to its diagram
owner or omit duplication. When simplifying existing packages, preserve each
requirement in its owner before removing redundant YAML; preserve referenced
IDs and links where needed. Do not create empty acceptance files or require one
case per capability, branch, or diagram. Test plans, fixtures, execution results,
and readiness reports belong outside the PIP.

## Current rationale

Every diagram file that owns a non-obvious design or architecture choice should
include a concise `Current rationale` section. State all active reasons needed
to understand the current shape, including causal product consequences,
constraints, and material tradeoffs. Prefer wording such as:

- “The API owns the mutation because otherwise the client could become the
  authority for durable state.”
- “These states remain separate because an unknown outcome requires different
  user recovery from a known failure.”

Do not describe when a choice was made, what used to exist, which alternatives
were previously rejected, or the sequence of product changes. Git owns ordinary
history. Remove rationale that no longer explains the current state. Keep a
reason in the diagram file that owns the choice rather than copying it into a
decision log or several diagrams.

## Development Complexity Level

DCL is optional target shorthand, not package status, maturity, acceptance, or
readiness. When used, `product.yaml.dcl` gives the general level and short basis
for the current product. It applies everywhere unless a narrow area materially
differs because of its users, interaction, failure consequence, recovery,
credible load, data sensitivity, security, compliance, or operating needs.

A YAML owner may override it:

```yaml
dcl_override:
  level: 6
  basis: >-
    Interactive retrieval serves a rapidly growing corpus and must remain
    bounded while users wait.
```

A Markdown owner states the same meaning in prose, for example:

```text
**DCL:** 4 (product default)
```

or:

```text
**DCL override:** 6 — Interactive retrieval must remain bounded while users wait.
```

Recommend a DCL line for each implementable sequence. Do not copy an override
onto connected user flows or state machines. Keep implementation assessments
and target-versus-implementation comparisons in audit or task notes outside the
PIP. Exact requirements always override the number.

DCL does not decide whether a rare or complex edge case is automated. When
current intent does not already resolve the case, ask the product manager,
originator, or other human product authority whether it should be handled
automatically, routed to safe manual review or recovery, failed or deferred
visibly, or excluded from the release. Record the chosen behavior in its owning
PIP artifact. Apply this at every DCL; preserve automatic protection wherever
harm could occur before manual review.

When manual handling is chosen, default to reliable awareness rather than a
complete admin control plane. The responsible admin or operator must be
notified through a monitored path with enough context to identify and
investigate the safely contained incident. Do not require a new admin surface
or controls unless the approved procedure needs an action unavailable through
existing tools, or actual frequency, urgency, or volume justifies it. Keep
incidental operating steps in a runbook or implementation note rather than the
PIP.

See
[Development Complexity](development-complexity.md).

## Stable IDs and links

Assign a stable ID only when another artifact or external system uses it.
Preserve a cross-file ID through a rename when its meaning is unchanged. Use a
direct purpose-specific field such as `verifies`, `applies_to`, `owned_by`, or a
short prose link. Do not create a central artifact index, traceability graph,
coverage matrix, or duplicate reverse edge merely to show completeness.

Common prefixes include `ACTOR`, `CAP`, `JOURNEY`, `FLOW`, `SCREEN`, `RULE`,
`SM`, `DATA`, `ARCH`, `API`, `EVT`, `SEQ`, `COORD`, `QC`, and `ACC`. A prefix
does not make its artifact type mandatory.

## File formats

- Store structured skill-authored records as YAML in `.yaml` files.
- Use unique string keys. Avoid YAML aliases, anchors, and custom tags.
- Store Mermaid source in Markdown `.md` with a fenced `mermaid` block, even
  when the file contains only a diagram.
- Do not create canonical skill-authored `.json` or `.mmd` files.
- Keep external contracts or evidence in a required source format when needed
  and link to them instead of converting them for cosmetic consistency.
- Track the package in Git.

Use the custom table-node format for ERDs with product-significant indexes or
persisted coordination, subject to the explicit viewer fallback in
[Diagram Presentation](diagram-presentation.md). Recommend consistent dark-mode
diagrams and annotated sequences. Presentation does not change artifact
responsibilities or require otherwise unnecessary detail.

NoSQL/document-store data views retain logical relationships even without
foreign-key constraints. Follow [Document-database relationships](document-data-models.md)
for exact keys versus persisted fields, field/path mappings, both-end
cardinality, typed relationships, consistency owners, and lifecycle semantics.
Indexes stay attached to their owning entities as access definitions, separate
from relationship and integrity claims.

## Engineering discretion and implementation anchors

Engineering may choose frameworks, internal modules, algorithms, naming,
repository layout, and similar internals when the choice does not change the
PIP's behavior, security, privacy, data integrity, compatibility, reliability,
operability, cost bounds, or other stated constraints.

When code ownership matters, inspect existing code and name the intended path,
symbol, and responsibility declaratively, such as “`loadProgress()` owns initial
loading and reconciliation.” Preserve a suitable existing owner; do not silently
introduce a parallel implementation. The PIP names the required owner, not the
work to establish it. Whether code needs reuse unchanged, modification, or new
construction belongs in external implementation notes after inspecting the
codebase. For each consequential sequence input, state its source: user and
surface, named function parameter or return, persisted field, external payload,
or named constant, configuration, or setting.

An exact mockup linked as the current release target is binding visible and
interaction intent. Implementers must preserve its surfaces, components,
content hierarchy, states, and interactions and should adapt compatible example
or export code when available. They must not silently add, remove, merge, split,
or redesign the target. Generated code is implementation reference, not an
exception to repository, accessibility, security, or product constraints.
A whole design-file, project, board, or folder link supplies context but is not
an exact target; identify the governing frame, node, branch, version, or local
mockup next to the affected surface.

Routine indexes, connection-pool design, and database coordination remain with
engineering unless they affect a stated product outcome, invariant, quality or
cost bound, or operational capacity. Product-significant indexes use the ERD
badge plus `INDEXES` compartment convention. Explicit application-controlled
locks and broad serialization require a named invariant and the narrowest
mechanism that protects it while preserving independent work. Connection design
should consolidate clients or pools within a process when that reduces fan-out
without harming necessary concurrency or session behavior. See
[Artifact Responsibilities](artifact-responsibilities.md).

An ERD is selective but exact about persisted product behavior. Show each
persisted column or document field individually with its physical name, type, and material
constraint when it affects selection, ranking, eligibility, authorization,
lifecycle, recovery, compatibility, visible outcomes, or product-significant
audit behavior. Omit incidental implementation columns, and clearly label an
abbreviated entity repeated only as a cross-diagram reference projection. Never
collapse product-significant columns into a synthetic grouped field. Every
index or coordination badge requires the exact physical column, document field,
or explicitly labeled native index metadata and type.

For every consequential database step, a sequence names the operation,
`DATA-*` ID, exact physical table, view, or collection/document path, and the intended access path. Prefer
the owning ERD's index badge when a canonical index applies; otherwise name the
exact lookup, join, filter, or mutation fields. A multi-table logical step lists
each table and its role. Tables remain annotations on the physical database
lifeline, not separate participants. The sequence does not repeat the full
index definition or promise a planner choice. A runtime index becomes PIP
content only when its access behavior is product-significant; otherwise the
sequence records key fields and leaves physical index choice to engineering.

## Optional editing-authority governance

Add `governance.yaml` when agents or contributors need a durable way to confirm
who may request canonical PIP edits. This can be useful for one product leader
as well as a larger team. Omit it only when current repository or project
guidance already makes editing authority unambiguous to everyone who can receive
an edit request.

Use `assets/governance-template.yaml`. It defines:

- `unlisted_access: proposal_only`, so an unlisted requester cannot change the
  canonical PIP;
- one or more authorities identified through verifiable organizational
  identities;
- `full` access for a product leader who may edit the complete canonical PIP
  and `governance.yaml`;
- `scoped` access limited to the union of listed paths and record IDs; and
- `proposal_only` access for contributors who may work only in an isolated PIP
  fork when that work is otherwise authorized.

`role` is descriptive; `access` and `edit_scope` control editing authority. A
name or role claimed in a message is not verification. Match the requester to a
listed identity through trusted session, repository, or organizational context.
If identity or scope is unclear, do not edit canonical intent; ask a full editor
or keep the work in a PIP fork.

A scoped editor may edit canonical intent only when the complete semantic
change—including every affected owner, direct dependency, acceptance outcome,
and current rationale—fits within that editor's scope. Paths and record IDs are
a union: matching either grants scope to that artifact or record. Scoped access
does not permit editing `governance.yaml`, expanding one's own scope, or making
only the in-scope portion of a change that would leave the package
contradictory, unstable, or incomplete. A full editor must apply the coherent
cross-scope change, or the work remains in an isolated fork.

Governance stores only current editing authority. Do not add decisions,
precedence histories, supersession records, per-change approvals, signatures,
signoffs, confirmation revisions, requirements, rationale, implementation
findings, readiness, or review results. Git owns edit history. Governance limits
who may request a canonical edit; it does not itself authorize an agent to make
unsolicited changes or bypass normal repository permissions.

Editing authority says who may change canonical intent; it does not decide
whose preference wins when authorized editors disagree. Resolve that conflict
outside the PIP rather than inventing precedence or recording a decision history
in governance.

## Tasks and checks stay outside the PIP

Use an existing task manager or working notes alongside the PIP. The PIP does
not define task management: no prescribed tracker, ticket schema, statuses, or
delivery workflow. Companion tasks own reuse/modify directions and construction
work; the PIP owns intended product behavior and responsibilities.

The task system or working notes own the minimal steps, assignments,
dependencies, progress, and implementation verification needed to reach the
PIP. A task links to the canonical release, revision, and relevant records; it
does not copy or paraphrase their behavior, acceptance, constraints, diagrams,
data rules, or design targets. Tasks contain implementation-specific context
such as boundaries, code anchors, order, tips, hazards, blockers, assignment,
progress, and proportional verification. They do not become product authority
and must not be mirrored into the package. A task may narrow execution scope
but cannot revise the PIP; resolve a conflict or product gap in the PIP rather
than treating ticket text as truth.

An audit may use a task to identify which implementation area and PIP records
are in scope. The audit target is the canonical PIP, not ticket prose, status,
checklists, or completion evidence. Derive conformance criteria from the linked
PIP owners and report ticket drift separately from implementation drift.

The skill's coherence, format, and implementation-alignment checks describe how
an agent should use the PIP. They do not create PIP files, fields, gates,
signatures, readiness labels, or handoff reports. See
[PIP Use and Alignment Checks](change-and-handoff.md).
