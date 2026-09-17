# Product Intent Package

This repository contains two outward-facing product-management Agent Skills and
the Product Intent Package (PIP) format maintained by one of them.

## Agent skills

[`skills/product-management/`](skills/product-management/) provides concise,
framework-neutral product leadership for shaping plans, reviewing features,
closing material gaps, and asking useful questions of ideators, developers,
customers, and users.

[`skills/product-intent-manager/`](skills/product-intent-manager/) helps a
product agent create, reconstruct, review, simplify, or update a PIP, and helps
an implementer or auditor preserve its current intent when work is explicitly
governed by one. Reconstruction is a mode of the same skill.

The Product Intent skill contains:

- `SKILL.md`, the concise entrypoint;
- `references/`, guidance loaded only when relevant;
- `assets/product-intent-template/`, the three-file module-first starter;
- `assets/acceptance-template.yaml`, exceptional non-diagrammable acceptance;
- `assets/governance-template.yaml`, optional current editing authority;
- `assets/example-product-intent-package/`, a proportional worked example; and
- `evals/cases.yaml`, realistic behavior cases.

## Format 7.0

Format 7 treats the canonical PIP as the product's current intended end state.
It contains no package or item status, readiness label, signature, confirmation
record, implementation observation, proposal lane, handoff record, or review
result. Git records ordinary history. Tasks, conversations, evidence,
implementation findings, and audits stay outside the package.

Write PIP content as present-tense product facts or timeless requirements, not
as instructions to change an earlier implementation. Required components and
code owners belong in the PIP; reuse/modify labels and construction steps belong
in external implementation notes. Describing intent does not claim it is built.
Use the team's existing task manager alongside the PIP; the PIP does not define
task management or prescribe a delivery workflow.

The end state must also be declared in the repository, not only as a history of
changes. Write SQL schemas as declarative files (for example `supabase/schemas/`)
and generate migrations from schema diffs; do not hand-write a migration without
an explicitly defined target schema. The same rule applies to other declarable
state such as infrastructure, policies, and index configuration.

Implementation tickets in Linear, Notion, ClickUp, GitHub, or another task
system link to the applicable PIP release, revision, and records rather than
restating product behavior or acceptance. Tickets contain implementation scope,
code anchors, order, tips, hazards, blockers, progress, and proportional
verification. They may narrow execution but cannot revise the PIP.
Audits may use tickets to locate the requested scope and evidence, but audit
expected behavior and conformance against the canonical PIP—not ticket prose,
status, or checklists.

All product logic belongs to named modules. The smallest starter has three files:

```text
product.yaml
architecture/stack-context.md
modules/primary-capability/experience/user-flows.md
```

Keep `product.yaml` minimal: product context and direct links, not rules, gates,
or scenario lists. Diagrams own behavior and observable outcomes. Omit
`acceptance.yaml` unless a unique case cannot be meaningfully diagrammed;
explain why, link its owner, and retain only that irreducible case. The same
restriction applies to inline acceptance; any qualifying case stays with its
module owner, not in the root product record. Complexity or convenience is not an
exception, and diagrammed behavior must not be copied into either file.

Modules are required at every product size; rename `primary-capability` to the
actual capability and subdivide only when responsibilities justify it. All
behavior, workflows, shared infrastructure, data rules, quality constraints,
and interaction rules belong inside modules with explicit public boundaries.
Cross-module workflows belong to their responsible domain or orchestration
module, not a parallel root workflow layer. Root files provide product context,
physical topology, optional editing authority, and navigation without defining
independent logic. Small modules do not need a boilerplate bundle of files.
Strongly prefer organizing overall documents such as stack context by the same
modules, so readers can follow one module through its context and dependencies.
Keep shared infrastructure physically accurate and link owners rather than
duplicating their logic.
The [module standard](skills/product-intent-manager/references/product-intent-package-standard.md#capability-modules)
and [bounded review procedure](skills/product-intent-manager/references/change-and-handoff.md#module-bounded-review)
define where a review can stop and when shared state or changed guarantees
require wider review. Modules do not imply new services or separate PIPs.

If a different product end state needs review, create an isolated PIP fork in a
branch, worktree, or separate proposal location. Make the fork internally
coherent and keep the canonical package unchanged until a requester whose
editing authority covers the complete change adopts it. Do not mix competing
proposals into canonical intent or add proposal statuses to either package.

Add `governance.yaml` when agents or contributors need a durable way to verify
who may request canonical PIP edits. It may grant a product leader `full`
access and give other product leaders or developers `scoped` or
`proposal_only` access. A scoped editor may not apply only part of a semantic
change when required dependent edits fall outside that scope; a full editor
must make the complete coherent change or it remains in an isolated fork.
Governance stores current editing authority only—not decisions, approvals,
signoffs, requirements, rationale, or history. Git records who committed edits.

Add journeys, screen records, rules, state machines, data models, contracts,
sequences, quality constraints, mockups, or a separate deployment view only
when they add distinct product meaning. Format 7 does not require an artifact
index, coverage matrix, central traceability graph, change log, journey
registry, readiness ledger, or a formal not-applicable decision.

An optional `dcl` mapping in `product.yaml` states the product-wide Development
Complexity Level. It applies everywhere unless a narrow owner has a justified
`dcl_override`. Sequence diagrams should show whether they use the product
default or an override. Implementation DCL comparisons stay in audit or task
notes outside the PIP.

DCL does not decide whether rare or complex edge cases deserve automatic
handling. At any DCL, ask the human product authority whether the intended path
is automation, safe manual review or recovery, visible fallback, or exclusion
before expanding the design or implementation.
For a manual fallback, prioritize a monitored notification that makes the
responsible admin aware and identifies the safely contained incident. Do not
create a dedicated admin control surface unless the actual response procedure
or demonstrated operating need requires it.

Each diagram file includes concise current rationale for its non-obvious design
choices. The rationale explains all active causes, constraints, and material
tradeoffs for the present design; it does not retell product history.

Diagrams are the default home for product logic, not illustrations of prose.
Every rule must also be part of a documented process or explicitly invoked by
one, with inputs and result handling shown. A standalone diagram or module
owner is not enough; see
[process-connected logic](skills/product-intent-manager/references/product-intent-package-standard.md#process-connected-logic-no-orphan-rules).
Rules, gates, permissions, conditions, calculations, and recovery belong in
rendered branches, guards, transitions, and attached notes. Shared rule logic
uses a linked decision diagram; prose supplies context and rationale, not a
second behavioral specification. See the
[diagram-first standard](skills/product-intent-manager/references/product-intent-package-standard.md#diagram-first-product-logic)
for structured inputs, acceptance, and the narrow decision-matrix exception.

The five diagram responsibilities remain distinct:

- stack context: physical systems, responsibility, state ownership, deployment,
  and connections;
- user flow: actor actions, user-visible surfaces, visible outcomes, and
  recovery—the inventory of what needs mockups;
- state machine: valid lifecycle states and high-level process interaction;
- data model/ERD: concepts, persisted records, relationships, and product-
  significant constraints; and
- sequence: detailed ordered runtime logic, input provenance, code ownership,
  retries, fallbacks, and recovery for one consequential process.

State machines and user flows intentionally omit sequence-level detail.
Deployment normally belongs in stack context and becomes separate only when its
topology is too complex to remain readable.

Use custom table-shaped ERDs when product-significant indexes or persisted
coordination need badges and attached compartments. Recommend dark-mode diagrams
and numbered sequences with left-aligned execution notes.
See [diagram presentation](skills/product-intent-manager/references/diagram-presentation.md)
and the [worked example](skills/product-intent-manager/assets/example-product-intent-package/product.yaml).
Use simpler diagrams when extra detail adds no meaning; presentation does not
make optional artifacts or runtime mechanisms mandatory.

Document-store diagrams show logical relationships without requiring foreign
keys. The [document-database guide and worked example](skills/product-intent-manager/references/document-data-models.md)
separate paths, persisted fields, typed relationships, enforcement, retention,
and entity-owned indexes; sequence diagrams retain synchronization and order.

Structured records use YAML (`.yaml`). Mermaid sources use fenced `mermaid`
blocks in Markdown (`.md`), including diagram-only files. Copied external
sources keep their required formats.

See the
[package standard](skills/product-intent-manager/references/product-intent-package-standard.md)
for the authoritative format.
