---
name: product-intent-manager
description: Create, reconstruct, simplify, or update an explicitly requested Product Intent Package (PIP), or preserve its current intent while planning, implementing, or auditing work governed by it. Use when the user asks to work on a PIP or explicitly implement or audit against one; do not activate for ordinary product planning, diagramming, coding, or project documentation with no PIP.
---

# Product Intent Manager

## First principle: all product logic is module-owned

Every PIP defines its product logic inside named `modules/<module>/` owners.
This applies to small and large products, not only complex or reusable features.
Include orchestration, authorization, private data handling, reconciliation,
data invariants, quality constraints, shared infrastructure, and user-visible
behavior. Cross-cutting logic is not an exception: give it a coherent module
owner and an explicit public boundary. Do not leave a parallel top-level
`workflows/`, `sequences/`, `data/`, `behavior/`, `contracts/`, `quality/`,
or normative `experience/` tree outside modules.

The root owns product context, physical topology, optional editing authority,
and navigation—not behavioral rules. Root overview diagrams link module owners
and introduce no independent logic. Modules are required; extra files and
subdivisions are optional. A small product can have one small module.
**Strongly prefer module-first organization in overall documents too.** Readers
enter a PIP through a module, so organize `stack-context.md` and other existing
overviews by the same module names, with direct owner links. A compact whole-
product map can orient the reader, but a technology-layer or service inventory
should not be the only reading path. Keep shared physical infrastructure clear
without duplicating services or implying one deployment per module. See
[Module-first overview documents](references/artifact-responsibilities.md#module-first-overview-documents).
Read [Capability modules](references/product-intent-package-standard.md#capability-modules)
before proposing a layout or authoring product logic. Existing nonmodular PIPs
remain authoritative for behavior; report structural gaps without silently
migrating unrelated scope or treating folder moves as proof of isolation.

## Read this first: diagram responsibilities

**No orphan logic or rules.** Every rule lives within an actual documented
process or is explicitly invoked by one, directly or through documented calls.
A module owner, standalone rule diagram, exported operation, or generic link is
not enough. The process shows the application point, input sources, and how the
result affects its next action, state, or outcome. Trace the call chain to a
concrete user action, request, event, schedule, or operating trigger; mutually
referencing unused rules do not qualify. This describes intended processes,
not a requirement that they are already implemented. Read
[Process-connected logic](references/product-intent-package-standard.md#process-connected-logic-no-orphan-rules)
before adding or reviewing shared rules.

**Diagrams are the default home for product logic, not illustrations of prose.**
Put rules, gates, eligibility, permissions, validation, calculations, precedence,
state changes, retries, and failure/recovery behavior in the owning rendered
diagram. Show conditions and their consequences as branches, guards, transitions,
or concise attached notes; a generic “validate” box linked to a prose rule is
not enough. Do not hide a behavioral requirement in prose, YAML `when`/`then`
lists, Mermaid comments, or rationale beneath the diagram.

Use prose for purpose, scope, terminology, causal rationale, and direct links.
Structured schemas and parameter values may stay in their appropriate records;
textual acceptance is limited to the exceptions below. The diagram names inputs and shows the logic
that uses them. Prefer splitting a crowded diagram into linked focused diagrams
over moving its logic into prose. Each fact still has one owner—do not maintain
a second prose specification. Read the
[diagram-first logic standard](references/product-intent-package-standard.md#diagram-first-product-logic)
when authoring, updating, or simplifying behavioral intent.

Keep these boundaries clear from the start:

- **Sequence diagrams hold detailed process logic:** ordered calls and events,
  input sources, intended code ownership, decisions, durable changes,
  retries, fallbacks, timeouts, partial failures, recovery, polling, heartbeats,
  lease renewal, and the tables and access paths used by consequential database
  steps.
- **State machines show high-level process interaction:** stable lifecycle states
  and the transitions produced by the processes represented in sequence
  diagrams. They intentionally omit each process's internal logic and routine
  self-loops that do not change a stable lifecycle state.
- **User flows show the user experience and mockup inventory:** actor actions,
  clearly bounded user-visible surfaces and states, navigation, visible failure
  and recovery, and outcomes. They intentionally omit internal system logic. A
  binding design target identifies the exact frame, node, branch, version, or
  local mockup; a link to an entire design file is context only.
- **Decision diagrams hold shared rule logic:** exact conditions, priority,
  calculations, and allowed/refused outcomes that need a reusable owner. Put a
  process-local gate directly in its sequence instead of creating another file.

Missing sequence-level detail in a state machine or user flow is correct. Link
to the owning sequence instead of copying its internals into those views.

Recommend a dark-mode style for PIP diagrams: dark backgrounds, light readable
text, subdued panels, and contrasting lines and badges. Apply it consistently
to ERDs and sequence diagrams unless the project requests another theme.
For ERDs with product-significant indexes or persisted coordination, use custom
Mermaid `flowchart` table nodes: physical attribute, type, key/rule, and badge
columns, with `INDEXES` and applicable `COORDINATION` compartments inside each
owning entity. Plain `erDiagram` plus detached prose or index tables is not the
default for these cases. Use exact badges such as `[U1·1]`, `[P2·where]`, and
`[LEASE1·owner]`, not `U1.1` or an unexplained `C1` in a field comment.
Before authoring an ERD, read [Diagram Presentation](references/diagram-presentation.md)
and its [worked notation example](references/erd-notation-example.md). That
reference defines the supported-viewer fallback; do not assume HTML is unsupported.
Simple relationship-only ERDs may stay plain when no custom detail is needed.
For NoSQL/document stores, also read
[Document-database relationships](references/document-data-models.md).
Show logical relationships derived from document paths, reference fields, and
product invariants even without foreign keys. Name exact path/field mappings,
both-end cardinality, consistency owners, and lifecycle semantics; indexes are
access paths, not evidence of relationships or referential integrity.
For sequences, prefer numbered action arrows and adjacent, left-aligned notes
for consequential execution details; read Diagram Presentation before authoring.

## The canonical PIP is current intent

The canonical PIP is the product's current intended end state. It does not need
status fields, readiness labels, signatures, confirmation records, handoff
records, implementation observations, or proposal markers. It simply states
what the product is meant to be. Git records ordinary history.

Write every PIP artifact as present-tense product facts or timeless requirements:
“The sign-in surface uses the provider's native component” or “Use the provider's
native component for sign-in.” Describe the intended result, not the one-time
work to reach it or what it replaces. Never put reuse/modify directions,
“replace the old,” “will add,” one-time migration steps, or completed-work
narration in the PIP; those belong only in external tasks or notes.
This applies to diagram labels, supporting notes, and acceptance too. Detailed
runtime logic and required components or code owners still belong in the PIP;
construction instructions do not. Present-tense intent is not a claim that the
implementation already matches it.

Standing action rules are valid PIP content, including product-significant
maintenance, deployment, and recovery instructions. Ask: “After this action is
completed once, does the rule still govern the next applicable situation?”
If yes, it is ongoing intent, not a completed task. Do not remove a valid rule
merely because it uses an imperative or describes ordered actions.

Keep conversations, evidence, unresolved questions, implementation findings,
tasks, and review results outside the canonical package. If someone wants to
propose a different end state, create an isolated PIP fork in a branch,
worktree, or separate proposal location. Make that fork internally coherent as
one complete intended end state; do not mix competing alternatives into the
canonical package. Its location makes it noncanonical, so its records need no
`proposed` status. Adopt it by replacing the affected canonical intent through
the team's normal authorized-editor and Git process.

Add `governance.yaml` when agents or contributors need a durable way to confirm
who may request canonical PIP edits. It records current editing authority only:
at least one product leader with `full` access, plus any `scoped` or
`proposal_only` contributors. It contains no decisions, approvals, signoffs,
change records, requirements, rationale, or history; Git records who committed
each change. A `scoped` editor may not leave canonical intent partial or
contradictory. If a coherent change needs dependent edits outside that scope, a
`full` editor must make the complete change or the work stays in an isolated
PIP fork. Read [Authority and Evidence](references/authority-and-evidence.md)
before accepting or applying a canonical edit request.

Use the team's existing task manager or implementation notes alongside the PIP.
The PIP does not define task management, a ticket format, or a delivery workflow.
Tickets in Linear, Notion, ClickUp, GitHub, or another task system are execution
overlays, not secondary product specifications. A ticket may narrow what an
implementation pass will touch, but it must point to the
canonical PIP for product behavior, acceptance, constraints, diagrams, and
design targets rather than paraphrasing them. If a ticket conflicts with its
linked PIP, the PIP governs and the ticket must be corrected or clarified.

Audit against the canonical PIP, not against a ticket's wording. A ticket may
identify which implementation area and PIP records to inspect, but the linked
PIP supplies the expected behavior, acceptance, constraints, and design. Ticket
status, checklists, tests, or paraphrased requirements do not prove PIP
alignment. Report ticket drift separately from implementation drift.

## Keep the package proportional

The smallest starter has three files, with behavior already inside a module:

```text
product.yaml
architecture/stack-context.md
modules/primary-capability/experience/user-flows.md
```

**Keep `product.yaml` and `acceptance.yaml` minimal. Neither is a home for logic.**
`product.yaml` contains product context: identity, release, outcome, boundary,
actors, concise capabilities, exclusions, measures, optional DCL, and direct
links. Do not accumulate rules, gates, conditions, algorithms, retry policies,
or scenario lists in any field, including `outcome`, `boundary`, `acceptance`,
and `success_measures`. Diagrammed outcomes already provide acceptance meaning;
do not restate them as YAML checklists.

Rename `primary-capability` to the actual capability. Keep the module's public
boundary in its owning diagram when that suffices; do not require a separate
boundary file or every artifact type.

Omit `acceptance.yaml` by default. Retain only unique acceptance cases that
cannot be represented meaningfully in a diagram or its attached notes, with a
brief reason why and a direct link to the relevant owner. The same exception
applies wherever acceptance is written; keep a qualifying exception in its
module, beside its owner or in a small module-local acceptance file, not in the
root product record. Complexity, many branches, cross-capability scope, or convenience
are not reasons to put logic in either file. Split or extend diagrams instead.
Before adding a line, ask: “Is this necessary product context, or genuinely
non-diagrammable acceptance?” If neither, put it in the owning diagram or omit
it as duplication. Preserve existing meaning when moving it; never just delete
a requirement because its current placement is wrong. Read the
[Package Standard](references/product-intent-package-standard.md) before
creating or migrating package structure and
[Artifact Responsibilities](references/artifact-responsibilities.md) before
choosing or separating diagrams.

Keep the root product record outward-facing. `outcome` states the user or
product result. `boundary` states what this product and release include and
exclude. Do not turn either into an inventory of algorithms, queues, database
mechanics, deployment settings, or concurrency rules; put those facts in their
owning artifacts and link them when context is necessary.

Use one owner for each fact and direct links where another artifact needs
context. Assign a stable ID only when another file or external system refers to
the item. Do not create artifact indexes, traceability graphs, coverage
matrices, change logs, readiness ledgers, or placeholder files.

## Modular PIPs and bounded review

Organize one canonical package around modules with explicit public behavioral
boundaries, internal diagrams, and owned state. All logic has a module owner;
choose the smallest coherent number of modules rather than a module per file,
table, or operation. A module is not automatically a service, library package,
or separate product. Do not create per-module product records, acceptance
checklists, or dependency ledgers.

Read [Capability modules](references/product-intent-package-standard.md#capability-modules)
before defining or reorganizing these boundaries. Keep the public promise
diagram-owned, internal logic separate, and cross-module workflows inside their
owning domain or orchestration module. Preserve shared transactions and other
real coupling explicitly; an integration sequence also needs a module owner.
For updates and scoped audits, apply
[Module-bounded review](references/change-and-handoff.md#module-bounded-review):
review changed internals and consumed public contracts, then expand to consumers
only where exported meaning or a cross-module invariant is affected. An
unchanged signature or an incomplete contract is not evidence of isolation.

## Explain the current design, not its history

In the diagram file that owns a design or architecture choice, add a short
`Current rationale` section. State every active reason needed to understand why
the current design is shaped that way, using causal language such as “X is
necessary because otherwise Y can occur.” Include product consequences,
constraints, and material tradeoffs; keep it concise.

Describe the reasons for the current state, not the chronology of how it was
reached. Do not recount former designs, superseded decisions, dates, or a series
of changes. Remove a reason when it no longer explains the current design. Git
owns ordinary history; optional governance records only current editing
authority.

## Development Complexity Level

Development Complexity Level (DCL) is an optional shorthand for the general
engineering and operational stage the current product intent requires. When it
is useful, put one product-wide default in `product.yaml`:

```yaml
dcl:
  level: 4
  basis: >-
    Users depend on this path in production, common failures need automatic
    recovery, and current load does not justify generalized scale machinery.
```

The default applies everywhere unless a narrowly defined area has materially
different user needs, failure consequences, load, security, compliance, or
operational demands. Give that owner a `dcl_override` with its own `level` and
`basis`. Recommend a visible DCL line above each implementable sequence, either
`DCL: 4 (product default)` or `DCL override: 6 — <current reason>`.

DCL describes target intent only. Keep implementation DCL assessments and
target-versus-implementation comparisons in audit or task notes outside the
PIP. It is not a readiness score, acceptance gate, or reason to add machinery.
Use the lowest level that safely fits actual users, interactions, recovery,
risk, and credible load. A low DCL never weakens authorization, security,
privacy, money safety, data integrity, or destructive-operation protections.

**Build for required outcomes, not implementation or data perfection.**
Byte-identical results are usually not a product requirement, especially at
lower DCLs and in early-stage products. Prefer semantic correctness and the
smallest useful verification over reproducing incidental serialization,
generated bytes, internal representations, or every historical implementation
detail. Do not let speculative determinism, exhaustive equivalence checks, or
unnecessary reconciliation machinery delay a working early product.
Data integrity means preserving the actual required identities, values,
relationships, and effects—not making every representation byte-for-byte equal.
Require exact bytes only for a concrete contract or correctness dependency;
keep that requirement narrow. Do not silently relax an explicit PIP requirement
or a security, money, or destructive-operation safeguard. Apply
[Outcome-focused correctness](references/development-complexity.md#outcome-focused-correctness-not-byte-identity)
when choosing implementation and verification scope, even when DCL is omitted.

DCL never requires automatic handling of every edge case. When the PIP does not
already resolve a rare or complex case, ask the product manager, originator, or
other human product authority before designing, implementing, or extensively
testing it. Clarify whether current intent is automatic handling, manual review
or recovery, visible failure or deferral, or exclusion from the release. Do
this at every DCL; a high DCL does not make costly rare-case automation
automatically worthwhile. Record the chosen behavior in the owning PIP artifact
before implementation. A manual path must still contain the case safely until
review; it cannot defer an authorization, security, privacy, money,
data-integrity, or destructive effect until after harm occurs.

For an intentional manual fallback, prioritize reliable admin awareness over a
complete admin control surface. The owning PIP behavior should require the
responsible admin or operator to be notified with enough context to identify the
incident, understand its safe current state, and find the existing place or
procedure for investigation. When a sequence exists, show that notification or
handoff. Use an existing monitored channel when it is sufficient; a passive log
no one is expected to watch is not visibility. Do not invent dashboards,
queues, retry buttons, editors, overrides, or other controls for every possible
fallback. Add a dedicated control only when the chosen manual procedure
actually requires that action and existing tools are insufficient, or when
frequency, urgency, or volume justifies it.

Read [Development Complexity](references/development-complexity.md) before
assigning or changing a level.

## Prevent implementation drift

When the PIP governs implementation or an implementation-oriented sequence:

- **Name intended code ownership.** Inspect existing code and identify the
  function, handler, job, or module that owns the process, for example
  “`path::function` handles this request.” Preserve suitable existing owners
  instead of creating parallel implementations. Keep whether to reuse, modify,
  or add code in external implementation notes, not PIP action labels.
- **Name input provenance.** For each input that affects a branch, durable
  change, visible outcome, or acceptance, state whether it comes from a user
  and surface, a named function parameter or return, a persisted field, an
  external payload, or a named constant, configuration, or setting.
- **Follow exact mockups.** A mockup linked as the current release target is the
  required surface, component, state, content hierarchy, and interaction
  design. A whole design-project or file link is useful context but is not an
  exact implementation target; identify the governing frame, node, branch,
  version, or local mockup next to the affected surface. Use compatible example
  or exported code when available. Do not add, remove, merge, split, or
  materially redesign views or components without the product or design leader
  changing the target. Raise feasibility, accessibility, security, or
  repository conflicts instead of silently deviating.

Engineering owns unspecified internals when they do not change current
behavior, security, privacy, data integrity, compatibility, reliability,
operability, cost bounds, or other stated constraints.

For an audit, use tickets only to locate the requested scope, affected code,
and implementation evidence. Reopen the canonical PIP release or revision and
derive every product-alignment criterion from its owning records. Use the
module-bounded review rule where boundaries exist; otherwise follow direct
links and obvious semantic dependents needed to judge that scope. Do not turn
ticket acceptance or completion status into the audit target.

## Declare the end state; derive the changes

A product is maintained from an explicitly declared end state, not from a
sequence of changes. The PIP declares intended product state. The repository
declares the implemented target state in declarative source files. Migrations,
patches, and apply plans are derived from the difference between that declared
target and the current state. A history of change files alone does not define
the product, and it is not an adequate owner for review, audit, or later change.

This rule applies to SQL schemas and migrations:

- Write the target SQL schema directly in declarative schema files in the
  repository, for example `supabase/schemas/*.sql`: tables, columns, types,
  constraints, indexes, views, functions, triggers, policies, and grants.
- Change the declared schema first. Generate each migration from the diff
  between the declared schema and the current migration state, for example
  `supabase db diff -f <name>`, then review the generated file.
- Do not write a migration file directly unless the project explicitly defines
  the target schema that the migration moves toward. If the project has only
  migrations, stop and propose a declared schema, for example one generated
  from the current database, before authoring schema changes.
- Hand-edit a generated migration only for what a diff cannot express, such as
  data movement or a rename that the diff shows as drop-and-create. The declared
  schema still defines the result. Review generated drops for data loss.
- Keep the declared schema consistent with the PIP data models. The PIP names
  the product-significant subset; the schema files hold the complete physical
  definition.

Apply the same rule to other stateful surfaces with a declarative option:
infrastructure, deployment configuration, access policies, document-store
indexes and rules, and external API schemas. The PIP never contains the
generated changes or one-time migration steps.

## Data access and concurrency

Record database mechanics only when they are product-significant:

- An ERD may omit incidental implementation columns, but it must show each
  persisted column or document field that directly determines product behavior
  as its own row with the exact physical name, type, and product-significant
  constraint. This includes facts that affect selection, ranking, eligibility, authorization,
  lifecycle, recovery, compatibility, visible outcomes, or product-significant
  audit behavior. Do not hide them in a synthetic row such as
  `fitness_controls SMALLINT × 5`. A clearly labeled cross-diagram reference
  projection may stay abbreviated and link to the owning `DATA-*` entity.
- In each consequential database step in a sequence, name the operation,
  `DATA-*` ID, and exact physical table, view, or collection/document path.
  Default to the owning ERD's index badge when that index is the intended access
  path; otherwise list the key lookup, join, filter, or mutation fields. When one
  logical step uses several entities, name each and its read, join, write, or constraint role.
  Keep the database service as the lifeline, link rather than repeat the ERD's
  full index definition, and do not claim the database planner is guaranteed to
  choose an index unless that guarantee actually exists.
- For a consequential index, show a textual, color-matched badge on every
  affected ERD attribute and repeat the badge in an `INDEXES` compartment below
  the entity. One base badge such as `[I1]` identifies exactly one complete
  physical index definition; never group two indexes under one badge, even when
  their keys or purpose overlap. Give indexes with different predicates,
  expressions, included columns, uniqueness, methods, directions, or key order
  separate badges. For one index, use a numeric suffix for a direct ordered key,
  `·expr` for a separate indexed expression, `·inc` for an included column, and
  `·where` only for a predicate-only column. Do not add `·where` to a column
  already represented as that index's direct key or expression. The compartment
  owns each full definition and its product or process reason. Keep an ordinary
  primary key as `PK` in the entity and omit its index badge unless the physical
  index has an independently product-significant purpose. Omit routine
  implementation indexes. Every index or coordination badge must sit on the
  exact physical column, document field, or explicitly labeled native index
  metadata it describes, never on a grouped or synthetic row. Do not invent
  stored ID fields, SQL constraints, or SQL index methods for document stores.
- Before allowing several database clients or pools in one process, check
  whether a bounded shared pool can combine them without serializing genuinely
  independent transactions, causing head-of-line blocking, breaking session
  semantics, or reducing effective product performance. Consider total fan-out
  across replicas, workers, and overlapping jobs; do not invent numeric limits.
- Treat application-controlled locks, stronger-than-normal isolation,
  singletons, and similar restrictions on otherwise independent work as
  exceptional. Use one only for a named correctness invariant or material
  capacity bound that a narrower constraint, atomic statement, optimistic
  check, idempotency rule, short transaction, or partitioned design cannot
  protect. Do not require oversized database infrastructure to compensate for
  avoidable contention.
- When several processes or coordination mechanisms make contention hard to
  understand, add a coordination overlay to `architecture/stack-context.md`.
  It maps contenders and protected resources to module-owned mechanisms without
  defining expiry, fencing, or other rules outside those modules. Keep one straightforward
  lock or lease in its sequence and, for persisted lease fields, its ERD. Split
  the overlay into `architecture/coordination.md` only when stack context would
  otherwise become unreadable.

See [Artifact Responsibilities](references/artifact-responsibilities.md) for
the diagram conventions.

## Boundary with companion implementation tasks

The PIP owns the end state. Companion tickets or working notes own the smallest
practical execution context needed to reach it, including reuse/modify
directions. The following guidance keeps those notes subordinate to the PIP;
it does not prescribe a task-management process. Identify the canonical PIP
release or revision and link directly to the relevant record
IDs or files. Do not copy, summarize, reinterpret, or rewrite the PIP's product
behavior, acceptance, constraints, sequence logic, data rules, or mockup
requirements into the ticket; parallel wording can be mistaken for authority
and drift from its owner.

A useful ticket contains implementation-specific information: the execution
boundary—including what to touch and what not to touch—verified code or design
anchors, essential order, codebase-specific tips and hazards, blockers,
assignment or progress, and proportional verification evidence.

Use a concise implementation objective or PIP IDs for the ticket title. Define
completion as alignment with the linked PIP plus task-specific evidence; do not
duplicate PIP acceptance as a ticket checklist. If the PIP lacks required
product meaning, resolve or update the PIP through its authority process rather
than filling the gap in a ticket. Split only for independent ownership,
dependency order, material risk, or separately shippable scope. Do not create a
ticket per file, layer, diagram, scenario, or implementation step. Read
[PIP Use and Alignment Checks](references/change-and-handoff.md#minimal-implementation-tasks)
when creating or reviewing implementation tasks.

For each task, ask whether every instruction directly aligns the codebase with
the PIP, whether it can be removed or merged, whether existing code and tests
can be reused, and whether each requested test, gate, proof, or review protects
a core outcome or dangerous edge case. Verify the core path, PIP-required
failure or recovery behavior, and dangerous edge cases; do not demand exhaustive
permutations, blanket coverage, new test machinery, or proof artifacts.
Before expanding a task around a rare or complex edge case the PIP does not
resolve, confirm the intended automatic, manual, visible fallback, or excluded
behavior with a human product authority rather than inferring it from DCL.

## Workflow

1. Read the canonical product context and scope-relevant owners, using module
   boundaries where available, plus `governance.yaml` or the repository's current
   editing-authority guidance, and the relevant references below. Confirm that
   the requester has authority for the complete coherent change. For a new
   package, copy `assets/product-intent-template/`.
2. Establish the release, outcome, boundary, actors, capabilities, exclusions,
   measures, and optional product-wide DCL.
3. Express observable outcomes in the owning diagrams. Keep `product.yaml`
   minimal; add textual acceptance only for unique non-diagrammable cases under
   the exception above, never as another version of diagrammed behavior.
4. Draft the root physical stack context and module-owned user-visible flows. Add state, data,
   sequence, rule, contract, journey, screen, quality, or deployment detail only
   when it resolves a real ambiguity; all logic and constraints stay inside
   modules. Move behavioral logic from prose into its owning diagram, including
   refusal and failure paths, without changing intent.
5. Add concise current rationale to each owning diagram file. Link related
   records directly instead of copying their content.
6. Use an isolated PIP fork for an unadopted alternative. Keep reconstruction
   evidence, questions, and implementation comparison outside the package.
7. Apply the lightweight checks in
   [PIP Use and Alignment Checks](references/change-and-handoff.md). Those
   checks govern how people use the PIP; they never create fields, files, gates,
   signatures, or reports inside the PIP.

For create, reconstruct, update, implementation-audit, and simplification
details, read [Workflows](references/workflows.md). For evidence and product-
authority boundaries, read
[Authority and Evidence](references/authority-and-evidence.md).

## Rules

- Use YAML in `.yaml` files for structured records.
- Store Mermaid in fenced `mermaid` blocks in Markdown `.md` files, including
  files that contain only a diagram. Do not create skill-authored canonical
  JSON or `.mmd` files.
- Keep the canonical PIP free of status, readiness, proposal, implementation,
  handoff, review, signoff, and decision-history metadata. Optional
  `governance.yaml` contains current editing authority only.
- Track the package in Git. Use Git history instead of duplicating change
  history inside the PIP.
- Keep task state and implementation notes outside the package.
- Check changed YAML, direct links, Mermaid output, and whitespace in
  proportion to the change. Do not add a validator, package hash, snapshot
  suite, or readiness gate for ordinary documentation work.
- This skill does not itself authorize product implementation, external writes,
  task mutations, commits, or pushes.

## Deliver

Return the requested PIP update or implementation-alignment result with a short
summary of the current release boundary, material package changes, optional
artifacts added or removed, and any product question that remains outside the
canonical PIP. When implementation was requested, report material deviations
and the minimal task or verification scope separately from the PIP.
