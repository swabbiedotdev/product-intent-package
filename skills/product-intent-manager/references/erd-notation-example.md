# Worked ERD notation

This is a formatting reference, not a default PIP file or a proposed application
schema. It demonstrates ordinary, composite-unique, and partial-unique indexes,
multiple badges on one field, and a persisted lease in one entity. Adapt only
the presentation needed for the product's existing intent. Do not add these
indexes, columns, or a lease merely because they appear here.

Keep each quoted HTML table label on one source line: some Mermaid renderers
turn embedded newlines into unwanted blank space above the table. Use explicit
`<br/>` inside cells for intended line breaks. Keep `themeCSS` readable on
separate lines; `theme: dark` alone does not style custom HTML. Each badge has
text, a tinted fill, and a bright border.

## Data view

```mermaid
---
config:
  theme: dark
  flowchart:
    htmlLabels: true
    wrappingWidth: 760
  themeVariables:
    edgeLabelBackground: '#1f2c38'
  themeCSS: |
    .pip-entity { border-collapse: collapse; background: #111827; color: #e5edf5; text-align: left; }
    .pip-entity th, .pip-entity td { border: 1px solid #344556; padding: 8px 12px; text-align: left; }
    .pip-entity th { background: #263544; color: #e5edf5; }
    .pip-entity code { color: #e5edf5; font-family: monospace; }
    .badge { display: inline-block; white-space: nowrap; border: 1px solid; border-radius: 4px; padding: 2px 5px; margin: 2px; }
    .badge.index { background: #162c48; border-color: #6ea8ff; color: #c6ddff; }
    .badge.unique { background: #163c2a; border-color: #6fd38a; color: #b8f5cb; }
    .badge.partial { background: #302047; border-color: #c58aff; color: #e4caff; }
    .badge.lease { background: #15393e; border-color: #67cfd7; color: #baf4f7; }
---
flowchart TB
  RESOURCE["<table class='pip-entity'><tr><th colspan='4'>DATA-001 · resource</th></tr><tr><th>ATTRIBUTE</th><th>TYPE</th><th>KEY / RULE</th><th>INDEX / COORDINATION</th></tr><tr><td><code>id</code></td><td><code>UUID</code></td><td>PK</td><td></td></tr></table>"]
  JOB["<table class='pip-entity'><tr><th colspan='4'>DATA-002 · export_job</th></tr><tr><th>ATTRIBUTE</th><th>TYPE</th><th>KEY / RULE</th><th>INDEX / COORDINATION</th></tr><tr><td><code>id</code></td><td><code>UUID</code></td><td>PK</td><td><span class='badge index'>[I1·3]</span> <span class='badge lease'>[LEASE1·scope]</span></td></tr><tr><td><code>resource_id</code></td><td><code>UUID</code></td><td>FK → resource.id; not null</td><td><span class='badge unique'>[U1·1]</span> <span class='badge partial'>[P1·1]</span></td></tr><tr><td><code>request_key</code></td><td><code>UUID</code></td><td>Not null; exact request identity</td><td><span class='badge unique'>[U1·2]</span></td></tr><tr><td><code>state</code></td><td><code>TEXT</code></td><td>Not null; queued, running,<br/>complete, or failed</td><td><span class='badge index'>[I1·1]</span> <span class='badge partial'>[P1·where]</span></td></tr><tr><td><code>created_at</code></td><td><code>TIMESTAMPTZ</code></td><td>Not null</td><td><span class='badge index'>[I1·2]</span></td></tr><tr><td><code>lease_owner</code></td><td><code>UUID</code></td><td>Current execution; nullable</td><td><span class='badge lease'>[LEASE1·owner]</span></td></tr><tr><td><code>lease_until</code></td><td><code>TIMESTAMPTZ</code></td><td>Expiry; nullable</td><td><span class='badge lease'>[LEASE1·until]</span></td></tr><tr><td><code>lease_version</code></td><td><code>BIGINT</code></td><td>Not null; nonnegative</td><td><span class='badge lease'>[LEASE1·fence]</span></td></tr><tr><th colspan='4'>INDEXES</th></tr><tr><td><span class='badge index'>[I1]</span></td><td colspan='3'><code>export_job_state_created_idx</code><br/>BTREE (state ASC, created_at ASC, id ASC)<br/>Supports SEQ-002: oldest-first selection within a state</td></tr><tr><td><span class='badge unique'>[U1]</span></td><td colspan='3'><code>export_job_resource_request_key</code><br/>UNIQUE BTREE (resource_id ASC, request_key ASC)<br/>Supports SEQ-001: one durable job per resource/request pair</td></tr><tr><td><span class='badge partial'>[P1]</span></td><td colspan='3'><code>export_job_one_active_per_resource_idx</code><br/>UNIQUE BTREE (resource_id ASC)<br/>WHERE state IN ('queued', 'running')<br/>Supports SEQ-001: at most one active export per resource</td></tr><tr><th colspan='4'>COORDINATION</th></tr><tr><td><span class='badge lease'>[LEASE1]</span></td><td colspan='3'>SCOPE · id<br/>OWNER · lease_owner<br/>EXPIRES · lease_until<br/>FENCE · lease_version<br/>PROTECTS · current execution's result publication<br/>PROCESS · SEQ-002 claim and recovery</td></tr></table>"]
  RESOURCE -->|"one resource owns zero or more export jobs; each job has one resource"| JOB
  classDef entity fill:#111827,stroke:#8ea0b3,color:#e5edf5
  class RESOURCE,JOB entity
```

Legend: blue `I` = ordinary non-unique index; green `U` = ordinary unique index;
purple `P` = partial/expression/specialized index (which can also be unique);
cyan `LEASE` = persisted coordination. Numeric suffixes mark ordered keys.
`·where` marks a predicate-only column. Lease suffixes identify stored roles.

Related process owners: [SEQ-001 admission](#seq-001-admission) and
[SEQ-002 claim and recovery](#seq-002-claim-and-recovery). These short notes give
the illustration's references a destination; a real package links its actual
sequence owner instead.

## SEQ-001 Admission

Admission owns request replay and the active-export guard. Its database notes
reference `DATA-002 export_job [U1]` and `[P1]`; their complete physical
definitions remain in the entity. Neither key of the compound `[U1]` is
individually unique.

## SEQ-002 Claim and recovery

Claiming owns oldest-first selection through `DATA-002 export_job [I1]`.
Recovery owns lease acquisition, renewal, expiry, and stale-publication
rejection through `[LEASE1]`. A sequence uses these identifiers and the exact
table name rather than copying the entity's full definitions.

## Why the layout matters

- `resource_id` has two independent index badges because two definitions use it.
- `state` has a direct-key badge for `[I1]` and a predicate-only badge for
  `[P1]`; a field's role is evaluated separately for each index.
- `id` remains a routine `PK`. Its `[I1·3]` badge belongs to the separate
  ordering index, not a newly documented primary-key index.
- The lease's scope, owner, expiry, and fence are attached to their exact
  physical rows. They are not ambiguous `C1` comments or floating prose.
- A diagram export retains the definitions and their purposes inside the entity.
  The process links resolve detailed runtime behavior without repeating it.

For an included column use `[I1·inc]`; for an indexed expression use the owning
index's `·expr` badge on its source field and show the full expression in the
index definition. Add only roles that the actual definition contains. See
[the complete suffix rules](artifact-responsibilities.md#product-significant-index-notation).
