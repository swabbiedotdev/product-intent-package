# User flows

## FLOW-001 Read and increment the counter

The flow takes place on `SCREEN-001 Counter screen`. Increment actions and
their retries use [SEQ-001](../sequences/sequences.md#seq-001-increment-once);
loads and reconciliation reads use
[SEQ-002](../sequences/sequences.md#seq-002-load-or-reconcile-progress).
Visible outcomes obey [RULE-001](../behavior/rules.yaml).

**Exact local design target:**
[`SCREEN-001 Counter screen`](mockups/screen-001-counter.md#screen-001-counter-screen)

```mermaid
---
config:
  theme: dark
  flowchart:
    nodeSpacing: 70
    rankSpacing: 90
    curve: linear
  themeVariables:
    clusterBkg: '#18222d'
    clusterBorder: '#344556'
    edgeLabelBackground: '#1f2c38'
---
flowchart TD
  START(["ACTOR-001 opens Counter"])

  subgraph SCREEN_001["SCREEN-001 · Counter screen"]
    LOADING["Show loading"] -->|open progress loads| READY["Show current value, target, and Increment"]
    LOADING -->|completed progress loads| COMPLETE["Show target reached without Increment"]
    LOADING -->|load fails| LOAD_ERROR["Show load failure and Retry"]
    LOAD_ERROR -->|Retry| LOADING

    READY -->|Press Increment| SUBMITTING["Show increment in progress"]
    SUBMITTING -->|progress confirmed below target| READY
    SUBMITTING -->|target reached| COMPLETE
    SUBMITTING -->|failure confirmed| RETRY["Show unchanged value and Retry"]
    RETRY -->|Retry| SUBMITTING
    SUBMITTING -->|outcome unknown| RECONCILE["Show reconciling and prevent another increment"]
    RECONCILE -->|progress reconciled| READY
    RECONCILE -->|completion reconciled| COMPLETE
    RECONCILE -->|outcome still unconfirmed; retry available| UNKNOWN["Show outcome unconfirmed and Retry"]
    UNKNOWN -->|Retry| SUBMITTING
    RECONCILE -->|read fails| READ_ERROR["Show reconciliation failure and Retry; Increment unavailable"]
    READ_ERROR -->|Retry| RECONCILE
  end

  START --> LOADING
```

The flow owns actor actions and visible outcomes. Runtime calls and commit
behavior belong to the linked sequences and `RULE-001`.

## Current rationale

- One responsive, centered screen is sufficient because the product has one
  user goal and no secondary navigation or account surfaces. Its exact local
  target is linked above so the surface inventory is not mistaken for layout.
- Loading, ready, error, submitting, and reconciling remain visibly distinct
  because each state gives the user different available actions and certainty
  about the persisted value.
- The reconciling state blocks another increment because the prior request may
  already have committed; another request could create an unintended duplicate.
- Recovery distinguishes an unchanged result, an unconfirmed result, and a
  failed read because Retry must perform the appropriate operation without
  presenting an uncertain increment as a new action.
- Completion removes Increment because the fixed target is the terminal product
  outcome and changing or resetting it is outside this release.
- A native keyboard-operable button and announced state changes are necessary
  for assistive-technology use. Animation is unnecessary for understanding the
  state changes.
