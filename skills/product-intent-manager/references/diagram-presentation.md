# Diagram Presentation

Use the ERD format rules and recommended sequence style below for application packages.
They make the data and execution context visible at the point where it matters.
Keep sources in Markdown with fenced Mermaid. Use the same presentation across
related files, adding only product-significant detail. These are presentation
conventions, not new artifact types or a requirement to document every column,
call, or branch. [Artifact Responsibilities](artifact-responsibilities.md) owns
the meaning and selection rules.

## Recommended dark-mode style

Default to dark-mode diagram styling unless the project requests another theme.
Pick one exact palette and apply it to every diagram rather than relying on a
general instruction to use dark mode. Give overview diagrams and detail
diagrams distinct box colors so a reader can tell the zoom level at a glance.
This palette has held up across a package of several hundred diagrams:

| Element | Color |
| --- | --- |
| Canvas and edge-label background | `#121314` |
| All text | `#EFE7D2` |
| Overview diagrams: box fill / borders and arrows | `#3B301E` / `#B69A61` |
| Detail diagrams, including ERD rows and sequence notes: box fill / borders and arrows | `#0E1B33` / `#536F98` |
| Index badge accents: I / U / P | `#6ea8ff` / `#6fd38a` / `#c58aff` |

Keep semantic legend colors, such as index badges, as they are. Keep text labels
alongside every color distinction.

Start every Mermaid block with `theme: base` and explicit `themeVariables`
rather than `theme: dark`, so each diagram type renders the same colors. A
detail-diagram configuration:

```yaml
config:
  theme: base
  themeVariables:
    background: "#121314"
    primaryColor: "#0E1B33"
    primaryBorderColor: "#536F98"
    primaryTextColor: "#EFE7D2"
    textColor: "#EFE7D2"
    lineColor: "#536F98"
    edgeLabelBackground: "#121314"
    actorBkg: "#0E1B33"
    actorBorder: "#536F98"
    actorTextColor: "#EFE7D2"
    signalColor: "#536F98"
    signalTextColor: "#EFE7D2"
    noteBkgColor: "#0E1B33"
    noteBorderColor: "#536F98"
    noteTextColor: "#EFE7D2"
```

An overview diagram uses the same block with the brass fill and line colors.
Style custom HTML table cells and badges explicitly because they may not inherit
the Mermaid theme. Check text, arrowheads, edge labels, notes, and exports
against the actual background.

Do not set a custom `fontFamily`. When the named font is missing from a viewer,
Mermaid sizes boxes for one font and draws them in another, clipping labels; the
default font renders correctly.

A bulk recolor must change only styling. A color named inside product content,
such as an image placed on a dark `#222222` square, is product intent and must
never be touched. Separate configuration and `classDef` styling from labels
before replacing colors.

This preference governs documentation diagrams, not the product application's
UI theme.

## Overview first, then lettered detail

A large record is easier to read as an overview followed by zoomed-in parts.
Keep a small record as one diagram.

- Open the record with `### <ID> overview`: a diagram of 4–10 boxes, or a few
  summary messages for a sequence, in the overview palette. Put intro lines such
  as the runtime, the owning `FLOW-*`, and the DCL line above it.
- Give each part of the overview a lettered ID (`SEQ-014a`, `SEQ-014b`, …) and
  follow with one detail diagram per part under `### SEQ-014a <name>`, in the
  detail palette.
- Mark each boundary between parts with a short stub node or note, such as
  `→ SEQ-014b Publish`.
- When an `alt` block spans two parts, keep the `else` line in the earlier part
  with a stub note and move the branch body into the next part. Never reword a
  branch condition while splitting.
- Data files follow the same pattern: an entity-only overview ERD, then the
  detailed table diagrams as lettered parts.
- A file with five or more records opens with a `### Map` that has one box per
  record and edges only for real relations between them.
- Place `### Current rationale` after the record's last detail diagram.

Make overviews per record, not per file. A file-level overview whose parts are
lettered versions of the file would renumber every record and every external
reference to it, including code comments.

When splitting existing diagrams, lose nothing: every line of the old detail
diagram must appear in exactly one lettered part. For a package-wide split,
compare the committed diagrams with the new ones mechanically and list any
missing lines before accepting the change.

## Custom table-shaped ERDs

For an ERD with product-significant indexes or persisted coordination, start
with Mermaid `flowchart` and HTML table labels. This is the expected format,
not an optional embellishment after a plain `erDiagram`. Follow the fallback
below only for a project-requested format or an observed viewer limitation.
Read the [worked notation example](erd-notation-example.md) before authoring;
adapt its table structure and CSS, not its illustrative schema or mechanisms.
For document stores, also read [Document-database relationships](document-data-models.md)
for the path headings, typed relationship legend, and worked connected groups.

- A prominent entity heading and four columns: `ATTRIBUTE`, `TYPE`,
  `KEY / RULE`, and `INDEX BADGE` (or `INDEX / COORDINATION` when both apply).
  Put the `DATA-*` owner and exact physical table name or document key/path in the entity heading,
  not just a field comment. Qualify references by table when one DATA owner
  covers several physical entities.
- One row per product-significant physical column or document field. Separate
  path components and native index metadata from persisted attributes. Use monospace for field
  names, types, and index names; keep explanatory rules readable and wrapped.
- An `INDEXES` compartment within the same entity, below its attributes. Each
  entry has one base badge, its complete definition, and its process or product
  purpose. Add `COORDINATION` below it only for relevant persisted locks/leases.
- Matching textual and colored badges: blue `I`, green `U`, purple `P`, with
  separately distinguishable lease/lock badges when needed. Include a compact
  legend. Color supplements the notation; it never replaces it.
- Labeled relationships with explicit cardinality and direction of meaning.
  A flowchart line alone does not express ERD cardinality. Use compact dashed
  `REFERENCE · DATA-* · entity` nodes for entities owned elsewhere and link to
  their owners rather than duplicating their fields.
  When field-level attachment is unavailable, label edges with exact,
  endpoint-qualified field/path mappings and cardinality/optionality at both
  ends. Arrowheads never imply execution order or enforcement by the database.

Use restrained borders, contrasting section headers, left-aligned cells, and
adequate spacing. Apply the recommended dark palette consistently. Keep
long definitions wrapped and avoid shrinking text to fit an oversized canvas.
Split by coherent data responsibility when necessary, preserving references.

### Build the entity as one visual

1. Collect the product-significant physical fields and the required index and
   persisted coordination definitions from their owners. Check compound and
   conditional uniqueness stated in prose too. Do not guess a missing physical
   definition or add a mechanism merely to fill a compartment.
2. Put each field in its own attribute row. Keep type and key/rule in separate
   cells. Put each badge in a styled `<span>` in the last cell; several badges
   on one field remain separate spans. Text such as `U1.1` in a quoted Mermaid
   field comment is not a substitute for a badge column.
3. Append `<tr><th colspan='4'>INDEXES</th></tr>` inside that same `<table>`.
   Each entry places its base badge in one cell and its physical name, complete
   definition, and purpose in a `colspan='3'` cell. Use `<br/>` to separate those
   lines. Include uniqueness, method, ordered keys/directions, expressions,
   included columns, and predicate wherever present. Do not summarize a
   compound key as `UNIQUE` on individual fields or a partial index as just
   "unique when active".
4. Append `COORDINATION` the same way, only when needed. Map the actual persisted
   scope, owner/attempt, activity/expiry, and fence fields to their badges.
   Include the protected resource and a direct link to the process owner.
   Describe structure here; acquisition, renewal, expiry, retries, and stale-
   owner rejection remain in the linked sequence. Do not turn a transient row
   lock into an invented stored lock entity.
5. Match badge colors between fields and compartments and include a textual
   legend. Use the exact bracket/middle-dot notation: `[U1·1]`, `[P2·where]`,
   `[LEASE1·owner]`. The
   [index](artifact-responsibilities.md#product-significant-index-notation) and
   [coordination](artifact-responsibilities.md#product-significant-lock-and-lease-notation)
   rules own suffix meanings. `UK` is a constraint marker, not an index badge.

`INDEXES` and `COORDINATION` are attached sections of their owning entity, not
separate graph nodes, floating notes, or one Markdown table for several
entities. Do not repeat full definitions in prose below the rendered entity.
Use nearby Markdown links when the viewer cannot preserve links inside HTML
labels; this does not require duplicating the definition. A diagram export
should retain the fields, badges, definitions, and role mappings together.

The [Counter data model](../assets/example-product-intent-package/modules/counter/data/data-model.md)
demonstrates the complete dark table style, per-attribute badges, an attached
index compartment, and explicit cardinality. Its
[sequences](../assets/example-product-intent-package/modules/counter/sequences/sequences.md)
reference that same index without copying its definition. No lease or lock
compartment is needed because the example has no such persisted mechanism.

The established index suffix and one-badge-per-index rules still apply. Qualify
cross-diagram badges by `DATA-*` and table because `[U1]` may occur on several
entities. Keep routine primary keys as `PK` unless their physical index has an
independent product-significant purpose.

### Viewer fallback and rendering check

Check the rendered result, not just Mermaid parsing: table cells, dark text
contrast, badge colors, attached compartments, relationship labels, and clipped
or overlapping content. A missing local renderer means rendering is unverified;
it does not establish that the target viewer rejects HTML. Keep the expected
source format and report that limit. Do not enable relaxed security settings
just to display a diagram.

If the project explicitly requests plain ERDs, or the intended viewer actually
rejects table labels, use ordinary Mermaid entities with the same exact textual
badges and per-entity Markdown `INDEXES` / `COORDINATION` sections directly
adjacent. Clearly identify the owning table on every section and preserve full
definitions and role mappings. If only color/CSS is unsupported, keep the table
structure and textual badges. State the chosen fallback and reason in the task
response, not as implementation-status metadata inside the PIP. Keep one
maintained definition; do not ship competing HTML and plain-ERD copies.

## Annotated sequences

Use Mermaid `sequenceDiagram`, `autonumber`, and left-aligned notes. Put the
process title, direct related-record links, and applicable DCL above the diagram.
Use physical participants and clear actor roles. Keep each message focused on
the logical action; attach the extra information next to the participant that
owns it, inside the relevant `alt`, `opt`, `loop`, or `break` branch.

Choose only relevant note labels:

| Label | Detail |
| --- | --- |
| `OWNER` / `CALL` | Intended function, routine, or operation and responsibility |
| `READ`, `INSERT`, `UPDATE`, `DELETE`, `JOIN` | DATA owner, exact table/view or collection/document path, and operation |
| `ACCESS` / `CONSTRAINT` | Table-qualified ERD badge and index name or enforcing constraint |
| `KEY` / `INPUT` | Consequential fields and where their values originate |
| `WRITE` / `RETURN` | Changed fields or returned facts that determine subsequent behavior |
| `GUARD` | Condition that permits or refuses the step |
| `TRANSACTION` | Which changes commit together and where the transaction ends |
| `PRESERVE`, `FREEZE`, `NO WRITE` | Material immutability or mutation boundary |
| `LEASE` / `CONCURRENCY` | Linked mechanism, scope, owner check, or contention behavior |

```mermaid
---
config:
  theme: base
  themeVariables:
    background: "#121314"
    primaryColor: "#0E1B33"
    primaryBorderColor: "#536F98"
    primaryTextColor: "#EFE7D2"
    lineColor: "#536F98"
  sequence:
    noteAlign: left
    noteFontSize: 14
    noteFontFamily: Arial
  themeCSS: |
    .noteText { font-family: Arial !important; font-size: 14px !important; }
  themeVariables:
    noteBkgColor: '#242626'
    noteTextColor: '#e5edf5'
---
sequenceDiagram
  autonumber
  participant API as Application API
  participant DB as PostgreSQL
  API->>DB: Reserve the report request
  Note right of DB: OWNER · reserve_report_request()<br/>INSERT/JOIN · DATA-002 · report_request<br/>CONSTRAINT · [U1] report_request_key<br/>INPUT · request_key from API parameter<br/>account_id from authenticated account<br/>GUARD · account may request this report<br/>RETURN · stable request id
  alt Same request already exists
    DB-->>API: Return the existing request id
  else Request is admitted
    DB-->>API: Return the committed request id
  end
```

Use `Note over` for a boundary spanning participants, such as external work
occurring outside a database transaction. Use short line breaks to keep notes
scannable. Put behavioral detail in rendered branches and attached notes; split
into linked diagrams when needed. Only non-normative explanation and causal
rationale move below the diagram. Keep code ownership visible at its step and
use nearby links for exact code destinations.

Arrows and notes together describe current intended runtime behavior. Preserve
the PIP's end-state wording: no reuse/modify tasks or implementation progress.
Do not copy complete index definitions into sequences or fill every step with
every label. Detailed notes earn their place by resolving a real ambiguity.

## Rules and gates as visible decisions

Use process-local sequence branches for gates that belong to one operation.
For shared selection logic, use a focused Mermaid decision flowchart with exact
predicates, precedence, and outcomes. A box saying “apply rules” is a link to
another diagram, not a substitute for one. A paragraph pasted into a giant note
is not a decision diagram: put consequential alternatives on distinct paths.
Calculations and invariants that do not branch can use concise attached notes
or entity rows. Keep failure/refusal consequences visible beside their condition.

### Worked rule: publication decision

This synthetic rule demonstrates notation, not required product behavior.
`RULE-201` owns selection only; its consuming sequence owns fetching inputs,
authorization enforcement, transaction boundaries, and publication. The diagram
includes input provenance, priority, the exact limit comparison, and each result.

```mermaid
---
config:
  theme: dark
  flowchart:
    wrappingWidth: 280
---
flowchart TD
  INPUT["RULE-201 · Publication decision<br/>editor: current membership.role = editor<br/>state: persisted document.state<br/>used: current period's published count<br/>limit: PUBLISH_LIMIT setting, same period"]
  AUTH{"editor = true?"}
  STATE{"state = draft?"}
  CAPACITY{"used &lt; limit?"}
  DENIED["DENY · no publication or count change"]
  INVALID["REFUSE_STATE · no publication or count change"]
  FULL["LIMIT_REACHED · no publication or count change"]
  ALLOW["ALLOW · consuming sequence may publish"]
  INPUT --> AUTH
  AUTH -->|No| DENIED
  AUTH -->|Yes| STATE
  STATE -->|No| INVALID
  STATE -->|Yes| CAPACITY
  CAPACITY -->|No, including equality| FULL
  CAPACITY -->|Yes| ALLOW
```

The ordering makes refusal precedence inspectable. Consuming sequences link to
this diagram and branch on its returned result without repeating the predicates.
An input lookup failure belongs to the consuming sequence's failure path; it
must not silently become `ALLOW`. For a real product, use its actual rule and
sequence owners rather than importing this example's roles or quota.
