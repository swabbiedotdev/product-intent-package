# Data model

`DATA-001 Counter record` owns shared progress. `DATA-002 Increment receipt`
records the result of each accepted request so a lost response can be reconciled
without applying the increment again.

The exact physical tables are `counter` for `DATA-001` and
`counter_increment_receipt` for `DATA-002`.

```mermaid
---
config:
  theme: base
  flowchart:
    htmlLabels: true
  themeCSS: |
    .pip-entity { border-collapse: collapse; background: #0E1B33; color: #EFE7D2; text-align: left; }
    .pip-entity th, .pip-entity td { border: 1px solid #536F98; padding: 8px 12px; text-align: left; }
    .pip-entity th { background: #0E1B33; color: #EFE7D2; }
    .pip-entity code { color: #EFE7D2; font-family: monospace; }
    .unique-badge { display: inline-block; background: #163c2a; color: #b8f5cb; border: 1px solid #6fd38a; border-radius: 4px; padding: 2px 5px; }
  themeVariables:
    background: '#121314'
    primaryColor: '#0E1B33'
    primaryBorderColor: '#536F98'
    primaryTextColor: '#EFE7D2'
    textColor: '#EFE7D2'
    lineColor: '#536F98'
    edgeLabelBackground: '#121314'
    clusterBkg: '#121314'
    clusterBorder: '#536F98'
---
flowchart TB
  DATA_001["<table class='pip-entity'><tr><th colspan='4'>DATA-001 · counter</th></tr><tr><th>ATTRIBUTE</th><th>TYPE</th><th>KEY / RULE</th><th>INDEX BADGE</th></tr><tr><td><code>id</code></td><td><code>UUID</code></td><td>PK</td><td></td></tr><tr><td><code>value</code></td><td><code>INTEGER</code></td><td>Not null; minimum 0</td><td></td></tr><tr><td><code>target</code></td><td><code>INTEGER</code></td><td>Not null; greater than 0</td><td></td></tr><tr><td><code>state</code></td><td><code>TEXT</code></td><td>open or complete</td><td></td></tr><tr><td><code>updated_at</code></td><td><code>TIMESTAMPTZ</code></td><td>Not null</td><td></td></tr></table>"]
  DATA_002["<table class='pip-entity'><tr><th colspan='4'>DATA-002 · counter_increment_receipt</th></tr><tr><th>ATTRIBUTE</th><th>TYPE</th><th>KEY / RULE</th><th>INDEX BADGE</th></tr><tr><td><code>id</code></td><td><code>UUID</code></td><td>PK</td><td></td></tr><tr><td><code>counter_id</code></td><td><code>UUID</code></td><td>FK → counter.id</td><td></td></tr><tr><td><code>request_key</code></td><td><code>UUID</code></td><td>UK</td><td><span class='unique-badge'>[U1·1]</span></td></tr><tr><td><code>value_after</code></td><td><code>INTEGER</code></td><td>Not null</td><td></td></tr><tr><td><code>state_after</code></td><td><code>TEXT</code></td><td>open or complete</td><td></td></tr><tr><td><code>created_at</code></td><td><code>TIMESTAMPTZ</code></td><td>Not null</td><td></td></tr><tr><th colspan='4'>INDEXES</th></tr><tr><td><span class='unique-badge'>[U1]</span></td><td colspan='3'><code>counter_increment_request_key</code><br/>UNIQUE BTREE (request_key ASC)<br/>Supports RULE-001 and SEQ-001: replay-safe increment;<br/>SEQ-002: original-request reconciliation</td></tr></table>"]
  DATA_001 -->|"counter_increment_receipt.counter_id = counter.id<br/>counter: 0..many receipts; receipt: 1 counter; database FK"| DATA_002
  classDef entity fill:#0E1B33,stroke:#536F98,color:#EFE7D2
  class DATA_001,DATA_002 entity
```

Legend: green `[U1]` identifies one unique index; `[U1·1]` marks its first
ordered key. Routine primary keys remain `PK`. Index behavior belongs to
[RULE-001](logic-counter.md#rule-001-persist-and-present-the-counter-safely),
[SEQ-001](logic-counter.md#seq-001-increment-once), and
[SEQ-002](logic-counter.md#seq-002-load-or-reconcile-progress).

There is exactly one counter record, seeded at deployment with `value: 0`, a
fixed positive target, and `state: open`. `RULE-001` owns increment and
completion behavior. The routine primary-key indexes remain implicit; `[U1]` is
shown because request-key uniqueness protects a product-significant outcome.

## Current rationale

- One singleton `DATA-001` represents the shared counter because the product
  excludes accounts, multiple counters, and multi-tenancy.
- The value, target, and state commit together because otherwise the visible
  completion state could disagree with durable progress.
- `DATA-002` commits with the accepted increment because a durable result keyed
  by the original request lets `SEQ-002` reconcile a lost response without
  creating another mutation.
- `[U1]` makes one request key identify at most one receipt because otherwise a
  replay or concurrent duplicate could advance progress more than once.
- The receipt stores the resulting value and state because reconciliation must
  return the outcome of that exact request, not merely a later counter value.
- Both records are retained for the product lifetime and protected by backups
  or exports because deployment, rollback, or infrastructure failure must not
  reset progress or erase replay protection.
