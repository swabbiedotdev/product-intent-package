# SCREEN-001 Counter screen

This low-fidelity local mockup is the exact content hierarchy and interaction
target for `SCREEN-001` in the example. It deliberately leaves typography,
color, spacing tokens, and framework choice to implementation.

```mermaid
---
config:
  theme: dark
  flowchart:
    nodeSpacing: 18
    rankSpacing: 18
  themeVariables:
    clusterBkg: '#18222d'
    clusterBorder: '#344556'
---
flowchart TB
  subgraph ENTRY["Load and recovery"]
    direction LR
    subgraph LOADING["Loading"]
      direction TB
      L_TITLE["Shared progress"] ~~~ L_VALUE["Loading…"] ~~~ L_ACTION["Increment unavailable"]
    end

    subgraph ERROR["Confirmed increment failure"]
      direction TB
      E_TITLE["Shared progress"] ~~~ E_VALUE["Progress unchanged"] ~~~ E_ACTION["Retry"]
    end

    subgraph READ_ERROR["Load or reconciliation read failure"]
      direction TB
      F_TITLE["Shared progress"] ~~~ F_VALUE["Progress unavailable"] ~~~ F_ACTION["Retry"]
    end

    subgraph UNKNOWN["Unconfirmed increment · retry available"]
      direction TB
      U_TITLE["Shared progress"] ~~~ U_VALUE["Outcome unconfirmed"] ~~~ U_ACTION["Retry"]
    end
  end

  subgraph OUTCOMES["Progress states"]
    direction LR
    subgraph READY["Ready · open"]
      direction TB
      R_TITLE["Shared progress"] ~~~ R_VALUE["4 of 10"] ~~~ R_ACTION["Increment"]
    end

    subgraph PENDING["Incrementing or reconciling"]
      direction TB
      P_TITLE["Shared progress"] ~~~ P_VALUE["Checking progress…"] ~~~ P_ACTION["Increment unavailable"]
    end

    subgraph COMPLETE["Complete"]
      direction TB
      C_TITLE["Shared progress"] ~~~ C_VALUE["10 of 10"] ~~~ C_RESULT["Target reached"]
    end
  end
```

Required variants are loading, ready/open, confirmed increment failure, read
failure, unconfirmed increment with Retry available, incrementing, reconciling,
and complete. [FLOW-001](../user-flows.md#flow-001-read-and-increment-the-counter)
owns transitions; this mockup owns the shared hierarchy and visible controls.

## Current rationale

- Every variant keeps the same title and progress position because the user is
  completing one continuous goal rather than navigating among separate views.
- Increment is unavailable while loading, incrementing, or reconciling because
  those states cannot safely accept another product action.
- Retry is the only action on failure and unconfirmed-result variants because
  recovery must not expose a fresh Increment or Dismiss before resolution.
- Complete replaces the action with `Target reached` because reset and target
  changes are outside this release.
