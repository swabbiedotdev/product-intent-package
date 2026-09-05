# Sequences

## SEQ-001 Increment once

This sequence applies [RULE-001](../behavior/rules.yaml) to
[DATA-001 and DATA-002](../data/data-model.md) during
[FLOW-001](../experience/user-flows.md#flow-001-read-and-increment-the-counter).

**DCL:** 4 (product default)

```mermaid
---
config:
  theme: dark
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

This sequence loads [DATA-001](../data/data-model.md) initially and resolves an
unknown [SEQ-001](#seq-001-increment-once) outcome through DATA-002 during
[FLOW-001](../experience/user-flows.md#flow-001-read-and-increment-the-counter).

**DCL:** 4 (product default)

```mermaid
---
config:
  theme: dark
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
    Note right of D: READ · DATA-001 counter<br/>KEY · id <- COUNTER_ID product constant
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
    Note over B: PRESERVE · retain request_key when reconciling<br/>Retry repeats the failed read, not the increment
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
