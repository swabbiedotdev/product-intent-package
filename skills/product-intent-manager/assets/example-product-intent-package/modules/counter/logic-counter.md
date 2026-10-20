# Counter logic

Counter's sequences, rule, lifecycle, and quality targets live together so one
process reads in one file.

## SEQ-001 Increment once

This diagram owns the increment and replay logic of
[RULE-001](#rule-001-persist-and-present-the-counter-safely) for
[DATA-001 and DATA-002](data-counter.md) during
[FLOW-001](experience-counter.md#flow-001-read-and-increment-the-counter).

**DCL:** 4 (product default)

```mermaid
---
config:
  theme: base
  sequence:
    noteAlign: left
    noteFontSize: 14
  themeCSS: |
    .noteText { font-size: 14px !important; }
  themeVariables:
    noteBkgColor: '#0E1B33'
    noteTextColor: '#EFE7D2'
    background: '#121314'
    primaryColor: '#0E1B33'
    primaryBorderColor: '#536F98'
    primaryTextColor: '#EFE7D2'
    textColor: '#EFE7D2'
    lineColor: '#536F98'
    actorBkg: '#0E1B33'
    actorBorder: '#536F98'
    actorTextColor: '#EFE7D2'
    actorLineColor: '#536F98'
    signalColor: '#536F98'
    signalTextColor: '#EFE7D2'
    noteBorderColor: '#536F98'
    labelBoxBkgColor: '#0E1B33'
    labelBoxBorderColor: '#536F98'
    labelTextColor: '#EFE7D2'
    loopTextColor: '#EFE7D2'
---
sequenceDiagram
  autonumber
  participant U as ACTOR-001 User
  participant B as ARCH-001 Browser application
  participant S as ARCH-002 Serverless API
  participant D as ARCH-003 Supabase Postgres

  U->>B: Press Increment or Retry
  Note over U,B: trigger <- user action on SCREEN-001
  alt New action or retry after a confirmed precommit failure
    B->>B: Generate and retain request_key for this attempt
    Note over B: INPUT · request_key from browser-generated UUID
  else Retry after reconciliation confirmed no receipt
    Note over B: INPUT · request_key retained from the original SEQ-001 attempt
  end
  Note over B: INPUT · increment amount from product constant 1
  B->>S: API-001 Increment(request_key)
  Note over S: OWNER · API-001 owns increment and replay handling
  S->>D: Begin short transaction and find the increment receipt
  Note right of D: READ · DATA-002 counter_increment_receipt<br/>ACCESS · [U1] counter_increment_request_key<br/>INPUT · request_key <- API-001 parameter
  alt Receipt already exists
    D-->>S: Recorded value_after and state_after
    S-->>B: Return recorded result without mutation
    B-->>U: Display recorded progress or completion
  else Request key is new and counter is open
    S->>D: Atomically increment, complete at target, and record the receipt
    Note right of D: READ/UPDATE · DATA-001 counter<br/>KEY · id from COUNTER_ID product constant<br/>GUARD · persisted state is open, value below target<br/>WRITE · value + 1, state from resulting value and persisted target<br/>updated_at from database time<br/>INSERT · DATA-002 counter_increment_receipt<br/>INPUT · counter_id from counter.id, request_key from API-001<br/>WRITE · value_after and state_after from the counter update<br/>id from generated UUID, created_at from database time<br/>CONSTRAINT · DATA-002 counter_increment_receipt [U1]<br/>TRANSACTION · counter change and receipt commit together
    D-->>S: Commit value_after and state_after
    S-->>B: Return committed result
    B-->>U: Display progress or Target reached
  else Counter is already complete
    Note right of D: READ · DATA-001 counter<br/>KEY · id from COUNTER_ID product constant<br/>GUARD · persisted state is complete<br/>NO WRITE · value and state remain unchanged
    D-->>S: No mutation
    S-->>B: Return complete counter
    B-->>U: Show Target reached without Increment
  else Failure is confirmed before commit
    D-->>S: Roll back without receipt or counter change
    S-->>B: Confirmed failure
    B-->>U: Show unchanged value and Retry
  else Client cannot determine the outcome
    Note over S,D: The transaction may or may not have committed
    S--xB: No conclusive result
    B-->>U: Show reconciling and prevent a new request
    Note over B,D: Continue with SEQ-002 using the same request_key
  end
  Note over S,D: TRANSACTION · every return closes the transaction<br/>CONCURRENCY · a competing same-key receipt insert rolls back this attempt's counter change,<br/>API-001 reads and returns the committed DATA-002 counter_increment_receipt via [U1]
```

### Current rationale

- The browser creates one request key for a new action and retains it through
  reconciliation because a network failure must not turn uncertainty into a
  second product action. When reconciliation finds no receipt, Retry uses the
  original key; a known precommit failure may start a new attempt with a new key.
- The API checks `[U1]` before mutation and returns an existing receipt because
  replaying a recorded request is a read, not another increment.
- The counter update, completion transition, and receipt insert share one short
  transaction because partial commit would make progress, state, and replay
  protection disagree.
- A failure known to occur before commit may offer Retry because the value and
  receipt are known to be unchanged.
- An unknown outcome does not create a new request because the first transaction
  may already have committed; `SEQ-002` reconciles the original key.

## SEQ-002 Load or reconcile progress

This sequence loads [DATA-001](data-counter.md) initially and resolves an
unknown [SEQ-001](#seq-001-increment-once) outcome through DATA-002 during
[FLOW-001](experience-counter.md#flow-001-read-and-increment-the-counter).

**DCL:** 4 (product default)

```mermaid
---
config:
  theme: base
  sequence:
    noteAlign: left
    noteFontSize: 14
  themeCSS: |
    .noteText { font-size: 14px !important; }
  themeVariables:
    noteBkgColor: '#0E1B33'
    noteTextColor: '#EFE7D2'
    background: '#121314'
    primaryColor: '#0E1B33'
    primaryBorderColor: '#536F98'
    primaryTextColor: '#EFE7D2'
    textColor: '#EFE7D2'
    lineColor: '#536F98'
    actorBkg: '#0E1B33'
    actorBorder: '#536F98'
    actorTextColor: '#EFE7D2'
    actorLineColor: '#536F98'
    signalColor: '#536F98'
    signalTextColor: '#EFE7D2'
    noteBorderColor: '#536F98'
    labelBoxBkgColor: '#0E1B33'
    labelBoxBorderColor: '#536F98'
    labelTextColor: '#EFE7D2'
    loopTextColor: '#EFE7D2'
---
sequenceDiagram
  autonumber
  participant U as ACTOR-001 User
  participant B as ARCH-001 Browser application
  participant S as ARCH-002 Serverless API
  participant D as ARCH-003 Supabase Postgres

  Note over S: OWNER · API-002 owns initial reads and receipt reconciliation
  alt Initial load or retry of a failed initial read
    U->>B: Open Counter or choose Retry
    Note over U,B: trigger <- user action on SCREEN-001
    B->>S: API-002 Current progress
    S->>D: Read current value, target, and state
    Note right of D: READ · DATA-001 counter<br/>KEY · id <- COUNTER_ID product constant<br/>NO WRITE · value, target, state, and receipts remain unchanged
  else Unknown increment outcome or retry of its failed reconciliation read
    Note over B: request_key <- retained input from SEQ-001
    B->>S: API-002 Reconcile(request_key)
    S->>D: Find the original increment receipt
    Note right of D: READ · DATA-002 counter_increment_receipt<br/>ACCESS · [U1] counter_increment_request_key<br/>INPUT · request_key <- retained SEQ-001 value
  end
  alt Current progress or receipt is found
    D-->>S: Persisted progress and state
    S-->>B: Persisted progress and state
    B-->>U: Display progress or Target reached
  else Receipt is not found after the unknown outcome
    D-->>S: No recorded request
    S-->>B: No receipt visible, outcome remains unconfirmed
    B-->>U: Show Outcome unconfirmed and Retry
    Note over B: Retry uses the retained request_key
  else Read fails
    D-->>S: Read failure
    S-->>B: Read failed, increment outcome is not determined
    B-->>U: Show Retry and keep Increment unavailable
    Note over B: PRESERVE · retain request_key when reconciling<br/>Retry repeats the failed read, not the increment<br/>GUARD · no Dismiss while the outcome is unresolved<br/>Do not claim the increment failed or left progress unchanged
  end
```

### Current rationale

- Initial load and reconciliation share one read process because each must
  return authoritative progress without creating a new product action.
- Reconciliation looks up the original request receipt rather than inferring
  success from the latest counter value because other users may have advanced
  the counter afterward.
- A missing receipt permits Retry only with the retained request key because a
  concurrent original transaction and its retry must still resolve to one
  `[U1]` receipt.
- Increment remains unavailable after a read failure because the browser cannot
  safely present or mutate progress it has not reconciled.

## RULE-001 Persist and present the counter safely

`RULE-001` applies to `CAP-001`. Its logic is owned by the steps where it runs:
the request-key replay check, atomic increment and completion, and receipt
insert in [SEQ-001](#seq-001-increment-once), and the initial load and
original-key reconciliation in
[SEQ-002](#seq-002-load-or-reconcile-progress). The visible results appear in
[FLOW-001](experience-counter.md#flow-001-read-and-increment-the-counter).

## SM-001 Counter lifecycle

```mermaid
---
config:
  theme: base
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
stateDiagram-v2
  [*] --> Open: Seed below the fixed target
  Open --> Complete: SEQ-001 commits the increment that reaches the target

  note right of Open
    Accepted increments below the target remain Open.
    Their request, replay, and recovery logic belongs to SEQ-001 and SEQ-002.
  end note

  note right of Complete
    Complete is terminal because target changes and reset are excluded.
  end note
```

### Current rationale

- `Open` and `Complete` are separate durable states because they allow different
  product actions: Open permits Increment, while Complete does not.
- Increments below the target are intentionally absent as self-loops because
  they do not change the stable lifecycle state; `SEQ-001` owns their detailed
  request, transaction, replay, and failure logic.
- Completion is terminal because changing or resetting the fixed target is
  outside this release.

## Quality targets

| ID | Applies to | Measure | Target |
| --- | --- | --- | --- |
| `QC-001` | `CAP-001`, [SEQ-001](#seq-001-increment-once) | Accepted increment latency, receipt uniqueness, and persistence consistency under concurrent load | p95 latency is no more than 500 milliseconds at 10 requests per second, with no lost or duplicate accepted increments |
| `QC-002` | `CAP-001`, [DATA-001 and DATA-002](data-counter.md) | Retention of persisted counter and receipt state across application deployment and rollback | Deployment and rollback do not reset persisted counter or receipt state; backups are maintained for recovery |
