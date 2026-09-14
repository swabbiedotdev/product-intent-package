# PIP Use and Alignment Checks

These are instructions for people and agents using a PIP. They are not PIP
content. Never create package status, readiness, signature, approval, gate,
handoff, checklist, or review-result fields or files because of these checks.

## Change discipline

For a material product-intent change:

1. Identify the owning fact and release, then verify the requester's identity
   and current editing authority from `governance.yaml` or unambiguous project
   guidance.
2. Use module-bounded review below where boundaries exist; otherwise follow
   direct links and obvious semantic dependents. Determine whether every
   required edit in the complete change fits that authority.
3. If the new end state is not already an unambiguous authorized decision, or
   any required dependent edit falls outside a scoped editor's access, create
   one complete isolated PIP fork rather than editing the canonical package.
   Never apply only an in-scope fragment that leaves canonical intent
   contradictory, unstable, or incomplete.
4. When authority covers the complete change, update the owner first, then
   affected flows, behavior, data, architecture,
   sequences, constraints, and acceptance.
5. Write affected content as present-tense facts or timeless requirements for
   the resulting product, not instructions to change it. Refresh `Current
   rationale` with only the active reasons for that design.
6. Resolve contradictions outside the package, then adopt one coherent end
   state through an editor whose authority covers the complete change and the
   team's normal Git process. Do not create a per-change signoff or approval
   record; Git owns commit history.

Preserve a stable cross-file ID when meaning is unchanged. Do not maintain a
global dependency graph, per-artifact version counter, package hash, change log,
or product history inside the PIP.

## Module-bounded review

Use this procedure for changes and scoped audits in a modular PIP. It narrows
the reasoning scope, not the obligation to preserve a coherent product. A user
request for a full-package audit still requires that scope. All logic owners
are modules, including orchestration and shared-policy owners. In a legacy
nonmodular package, report the structural gap and inspect the necessary current
owners without assuming modular isolation or silently migrating unrelated scope.

1. Read root product context and the public contracts of modules owning
   applicable product-wide constraints, the
   changed or audited module's public boundary and affected internals, and the
   public contracts of its direct dependencies that the work consumes. Do not
   recursively open dependency internals by default.
2. Check that internal logic fulfills the public promise and local invariants.
   Compare old and new meaning, not merely file paths or signatures. Exported
   meaning includes result semantics, errors, authorization, side effects,
   privacy/retention, freshness, compatibility, atomicity, recovery, and stated
   quality/cost bounds.
3. Search direct operation links and obvious semantic consumers for assumptions
   the boundary omits, including shared reads/writes and integration invariants.
   This is impact discovery, not a review of every consumer's internals. Missing
   links or unchanged contract text do not prove that no consumers are affected.
4. If exported guarantees and applicable cross-module invariants remain valid,
   stop at the boundary. If meaning changes, inspect the directly affected
   consumer call sites and outcome handling; propagate farther only where their
   own guarantees or shared invariants are affected. Additive changes still
   require checking assumptions such as exhaustive outcome handling.
5. Review the integration sequence in its owning domain or orchestration module
   and affected state owners whenever
   shared transactions, mutation ownership, locks, cancellation, or other
   cross-module invariants change. Product-wide policy changes may genuinely
   require broad review. If a contract is incomplete or a hidden dependency
   appears, inspect the necessary owners and report the boundary gap; do not
   invent a guarantee to stop early.

For example, changing ranking weights need not reopen an unchanged raw-scoring
algorithm, but does require reviewing affected selection outcomes. Changing
score meaning or evidence compatibility reaches consumers relying on those
semantics. Changing atomic publication and debit behavior reaches both state
owners and their shared transaction, even if every function signature stays
the same.

Report the review scope, reasons for stopping or expanding it, and verification
limits in the task response or existing working notes, not a new PIP artifact.
YAML, links, and rendering checks establish structural validity, not behavioral
compatibility. Test plans and execution results remain outside the PIP.

## Implementation alignment

At the start of PIP-governed implementation or audit, note the canonical PIP Git
revision and release in the existing task or working context. Compare product-
significant behavior, design, schema, migration, query, policy, and data changes
with that baseline and later direct instructions from a verified authority whose
scope covers the change.

For an audit, a ticket may guide which implementation area, code, and PIP
records to inspect. It is not the conformance target. Derive expected behavior,
acceptance, constraints, diagrams, data rules, and design targets from the
canonical PIP owners. Apply module-bounded review where boundaries exist;
otherwise follow direct links and obvious semantic dependents needed to assess
the requested scope. A completed ticket, checked box, passing test, or
faithful-looking paraphrase does not establish PIP alignment.

Implementation findings stay in the task, audit, or code review. Do not add an
as-built lane, implementation observation, deviation status, or implementation
DCL to the PIP. If the implementation is wrong, fix it toward the PIP. If the
team wants different product intent, create a coherent PIP fork and route that
product change separately. Report ticket-to-PIP drift separately from
implementation-to-PIP drift so neither is mistaken for product authority.

Apply the semantic-expansion boundary in
[Authority and Evidence](authority-and-evidence.md#semantic-expansion-boundary)
before persisting a new classification, splitting a population, moving a rule
between processing stages, adding maintained derived state or a broad backfill,
or encoding product policy in a database predicate.

## Minimal implementation tasks

This section defines the boundary with companion tasks, not a task-management
system or required workflow. Use the team's existing tracker or working notes.

The PIP describes the intended product. An implementation ticket in Linear,
Notion, ClickUp, GitHub, or another task system describes only the shortest
practical route from the codebase to that product. It is an execution overlay,
not a second specification. Keep tasks, assignment, order, progress, and
verification outside the PIP. Identify the canonical PIP release or revision
and link directly to the relevant records or files.

Do not copy, summarize, reinterpret, or rewrite product behavior, acceptance,
constraints, diagram logic, data rules, or design targets into a ticket. Even a
faithful paraphrase creates another version that can drift or be mistaken for
authority. A task may narrow implementation scope but cannot revise the linked
PIP. When the task and PIP conflict, follow the PIP and correct or clarify the
task. When the PIP lacks needed product meaning, resolve that gap through the
PIP authority process rather than making the ticket authoritative.

Create or mutate an external task only when authorized. Otherwise return a
draft. Do not introduce a PIP task file, task registry, or parallel tracker.

A task should contain no more than the implementer needs to act without product
guesswork:

- a concise implementation objective and direct PIP links or record IDs;
- the execution boundary: what code, schema, configuration, or surfaces to
  touch and what adjacent areas not to touch;
- reuse/modify directions for verified code or design owners when they matter;
- only essential ordering or dependency information;
- codebase-specific tips, known hazards, assumptions to verify, or blockers;
- assignment, progress, and links to implementation evidence when useful; and
- the smallest relevant verification or completion evidence.

State completion as alignment with the linked PIP plus task-specific evidence.
Do not duplicate PIP acceptance as a ticket checklist.

Keep one product outcome together even when it touches several files or layers.
Split only for independent ownership, dependency order, material risk, release
scope, or work that can genuinely ship or be reviewed separately. Do not split
by file, component, layer, diagram, scenario, or checklist item merely to make
progress easier to count.

For each task, ask:

1. Does every instruction directly align the codebase with the PIP?
2. Does it link to product truth instead of restating or interpreting it?
3. Can any instruction or task be removed or merged?
4. Does it reuse existing code, tests, tools, and project process where useful?
5. Does every requested test, gate, proof, report, or review protect a core
   outcome or dangerous edge case?

Remove ceremony that does not survive those questions.

### Verify in proportion to consequence

Implementation verification should cover the core path and observable done
condition, material failure or recovery behavior required by the PIP, and
dangerous edge cases relevant to the change.

Use [outcome-focused correctness](development-complexity.md#outcome-focused-correctness-not-byte-identity):
compare required semantics and invariants, not incidental bytes. Before adding
costly byte-for-byte snapshots, historical replay, canonicalization, or broad
data reconciliation, identify the actual consumer or failure they protect.
If none requires exact identity, use the simpler semantic check and continue
delivery. Lower DCL should reduce unnecessary implementation and verification
work, not weaken real safeguards. Keep exact comparisons where required or
already useful and cheap; do not invent tolerances or silently relax the PIP.

An edge case is dangerous when a plausible failure could cause an authorization,
security, or privacy breach; incorrect money movement; data loss or corruption;
an unsafe schema or migration result; a destructive or irreversible side
effect; or comparable product or operational harm. Prefer existing focused
checks. Do not enumerate every hypothetical case, chase blanket coverage, or
require new test harnesses, proof documents, screenshots, gates, or reports
without a specific risk or project requirement.

Before expanding design, implementation, or verification for a rare or complex
edge case the PIP does not resolve, ask the product manager, originator, or
other human product authority whether current intent is automatic handling,
safe manual review or recovery, visible failure or deferral, or exclusion from
the release. Do not infer the answer from DCL. Record the adopted behavior in
its owning PIP artifact before adding it to an implementation task. A manual
resolution path must still prevent dangerous effects from occurring before
review.

For a manual fallback, first verify that a responsible admin or operator will
actually become aware of the incident through a monitored path and receive
enough context to identify and investigate it. Do not require bespoke admin
screens, queues, or action controls unless the intended response cannot be
performed safely with existing tools or real frequency, urgency, or volume
justifies them. Visibility is the default requirement; a full control surface
is not.

## Four lightweight checks

Run these as review questions. Do not store their answers in the PIP.

### 1. Coherent current intent

Confirm that the release, outcome, boundary, actors, capabilities, exclusions,
measures, and acceptance are understandable and consistent. The canonical PIP
must describe one current end state, without alternatives, statuses,
implementation findings, or historical narration.

Read affected YAML, diagram labels, supporting notes, and acceptance as a
description of the intended product. Would they still make sense without
knowing the previous design or pending work? Apply the
[end-state wording guidance](product-intent-package-standard.md#write-the-end-state-not-the-change):
keep runtime behavior and intended owners, move construction steps and progress
outside the PIP, and do not treat present-tense intent as evidence of delivery.
Preserve standing maintenance, deployment, recovery, and other action rules
that still govern the next applicable situation after being followed once.
Imperative wording or ordered actions alone do not make a rule a task.

If a material product question remains, resolve it outside the package. Keep the
canonical PIP unchanged and use an isolated PIP fork for a concrete alternative.

### 2. Enough product and process meaning

Check for orphan logic in the affected scope. For each rule or shared operation,
locate its actual process application: a local diagram step or an explicit
caller with input provenance and result handling. Follow nested calls far
enough to identify the documented entry trigger. Module placement, inbound
links, a public contract, an `applies_to` list, or a cycle of helper calls are
not sufficient. Check intended application sites, not merely whether one
arbitrary consumer exists. A data invariant or quality bound must constrain
an identified process step; it need not be a separate function call.

Flag missing application, an unused returned decision, or an undefined effect
as an intent gap. Document existing intended integration, or ask about the
missing product choice. Do not silently delete the rule or invent consumers,
checks, or jobs to make the package look connected. Link validation proves
references resolve, not that a process meaningfully applies the rule.

Hide the narrative prose and inspect the rendered diagrams plus their directly
linked structured inputs. Can a reader trace each affected rule, gate, priority,
calculation, timeout, and failure/recovery path to its material outcome? Flag
logic found only in prose, YAML `when`/`then` lists, Mermaid comments, or rationale.
Check that attached notes are concise and branch-specific, shared rules resolve
to diagram owners, and no removed prose condition disappeared during migration.
An accessibility equivalent or explicit compact-matrix exception must not add
unrepresented or competing logic. Preserve intent when correcting placement.

Confirm that the package explains applicable:

- actor-visible surfaces, states, paths, failure, and recovery;
- lifecycle states and the high-level interaction of consequential processes;
- detailed process ordering, input provenance, retry, fallback, and recovery in
  sequences;
- physical systems, responsibility, owned state, deployment, and external
  boundaries;
- product-significant data relationships, constraints, indexes, and concurrency;
  and
- material quality, security, privacy, compatibility, operational, and cost
  bounds.

Use optional artifacts only where they add distinct meaning. An absent optional
artifact is not a failure.

Check ownership before accepting a proposed organization: every behavioral
diagram, workflow, shared policy, data invariant, contract, and quality
constraint must have a named module owner. Root architecture or experience
overviews may link owners but cannot contain independent rules. A tree with
feature modules plus root workflows, sequences, data, or quality specifications
fails this check. Moving all leftovers into an undifferentiated shared module
also fails to establish review isolation. Do not require a new file for every
concern when a coherent existing module can own it.

Check the reader's path as well as file ownership: can someone starting with a
module find its stack context, shared infrastructure, connections, and owning
contracts without reconstructing them from technology-layer inventories?
Strongly prefer module-named sections or diagram groups in overall documents,
with consistent names and direct links. Any different primary organization
should have a concrete reason and still provide that module-oriented path.
Shared services retain one physical identity; module grouping must not invent
deployment or trust boundaries.

For an existing-product or design-led implementation, also check that:

- sequences name intended code owners and their runtime responsibilities,
  preserving suitable existing owners without construction-task labels;
- consequential inputs state their source;
- consequential database steps name each physical table, view, or collection/document path and operation,
  reference the canonical index badge when applicable, or show exact key fields
  when no canonical index applies;
- user-flow surface boundaries identify what needs mockups;
- exact linked mockups and compatible example code are followed; and
- every diagram's current rationale succinctly covers the active reasons for
  its design without recounting history.

For product-significant database design, check that:

- document-store relationships remain visible without foreign keys; exact
  document keys/path components are distinct from stored fields, and edges name
  matching endpoints, both-end cardinality, and the relationship kind;
- consistency owners and lifecycle rules distinguish database enforcement from
  application invariants, permit intentionally missing targets explicitly, and
  never imply cascading deletion from containment alone;
- custom table nodes contain the owning entity's attribute badges and attached
  index/coordination compartments, unless a project-requested format or observed
  viewer limitation requires the documented fallback;
- badges use brackets and explicit key/coordination role suffixes, match their
  compartment entries, and account for compound and conditional uniqueness as
  well as single-column constraints;
- persisted columns or document fields that determine product behavior appear individually with
  exact physical names, types, and material constraints rather than synthetic
  grouped rows, while abbreviated cross-diagram references are clearly marked;
- each physical index has exactly one badge and one complete `INDEXES`
  compartment entry; indexes are never grouped under one badge or abbreviated
  as `same key`, and every attribute badge matches its key order, predicate,
  expression, or included-column role without a redundant same-index `·where`;
- routine primary-key indexes remain `PK` entity facts unless their particular
  physical definition has an independently product-significant purpose;
- each index states its product or process reason;
- an index or predicate supports an independently stated product rule rather
  than inventing a classification;
- any explicit lock or serialization protects a named invariant, is no broader
  or longer than needed, and cannot be replaced by a simpler narrow mechanism;
- persisted lease fields use matching ERD coordination badges and a
  `COORDINATION` compartment, while runtime acquisition, renewal, expiry,
  fencing, release, and recovery stay in the owning sequence;
- every index or coordination badge sits on the exact physical column, document
  field, or explicitly labeled native index metadata and type,
  never a grouped field or abbreviated reference projection;
- sequence access annotations agree with the linked ERD table names, index
  badges, and key fields without duplicating full index definitions or promising
  query-planner behavior;
- a coordination overlay is present only when multiple contenders or mechanisms
  need a contention map, and it references module-owned sequences and data
  rather than defining coordination rules at the root;
- connection design considers aggregate fan-out and combines process-local
  clients or pools where that preserves effective concurrency and session
  needs; and
- the design does not require oversized infrastructure to compensate for
  avoidable contention or connection amplification.

When DCL is used, check that the product default and any narrow override are
based on current users, interactions, wait path, failure consequences, recovery,
credible load, and material risk. Do not require DCL on every record or treat a
number as permission to add or remove mechanisms. Confirm that rare or complex
edge-case handling reflects an explicit automatic, manual, visible fallback, or
excluded product choice rather than an assumption derived from DCL. When a
manual fallback exists, confirm that it provides monitored admin awareness
without inventing controls the actual response does not need.

### 3. Observable acceptance and engineering discretion

Confirm that every in-scope capability has recognizable success and material
failure in its owning diagrams. Inspect `product.yaml` and `acceptance.yaml`
specifically for regression: keep only necessary product context, direct links,
or unique non-diagrammable acceptance cases with a concrete reason. Do not accept
logic hidden under `outcome`, `boundary`, `success_measures`, or given/when/then.
More detail, complexity, or convenience does not justify textual acceptance.
An absent acceptance file is normal; do not require a YAML checklist when the
diagram already expresses the outcome. Preserve all meaning when moving text.

An exact current mockup target must cover its required visible states and
interactions. Do not duplicate every visual detail into YAML.

Unspecified internals remain with engineering when choices stay inside PIP
behavior and constraints. This discretion does not permit a parallel replacement
for a stated code owner or a silent change to current mockups, views,
components, states, or interactions.

### 4. Consistency, format, and implementation alignment

Confirm that direct links resolve, linked artifacts do not contradict their
owners, YAML parses with unique keys, changed Mermaid renders when practical,
and `git diff --check` is clean when Git is used.

Separately compare implementation with the PIP when implementation is in scope.
Report `aligns`, `deviates`, or `unclear` in the task or audit response and name
only material differences. Use a ticket to guide audit scope, never as the
expected product. Do not persist that result in the package, and do not change
the PIP to make a divergent ticket or implementation appear compliant.

Use existing tools and proportional manual checks. A missing renderer should be
reported, not solved by adding Python, a package hash, snapshot tests, a new
dependency, or a full-package validator.

## Report

Return a concise human-readable result: what PIP content changed, the release
boundary, optional artifacts added or removed, unresolved product questions
kept outside the PIP, and—when implementation was reviewed—material alignment
findings and minimal follow-up tasks. Do not generate a persistent handoff or
readiness artifact.
