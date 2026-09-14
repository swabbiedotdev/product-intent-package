# Stack context

Show the physical product boundary, actors, external systems, deployable
services, owned state, and consequential connections. This view owns runtime
placement and responsibility. Include deployment here unless its topology is
too complex to remain readable.

Strongly prefer module-named sections or diagram groups as the primary reading
path. Add a compact whole-product map first only when useful. Rename the
section and module link below with the actual capability. For multiple modules,
use their existing names and show shared services once, referencing the same
physical identities from each module's context; do not imply separate services.

Do not show screen navigation, message order, entity fields, or detailed state
transitions. Link module owners for behavior, security, quality, deployment,
and recovery rules. Root topology must not become an independent logic owner.

## Module: primary-capability

Module owner: [primary capability](../modules/primary-capability/experience/user-flows.md).
Show its physical participants, state, and relevant connections here. A module
heading or diagram group is a reading lens, not a deployment or trust boundary.

```mermaid
---
config:
  theme: dark
---
flowchart LR
  %% Add only the physical participants in the current intended product.
  %% Label each node with its ARCH-* ID when cross-referenced, runtime or
  %% provider, responsibilities, and owned state. Label each edge with direction
  %% and interaction meaning.
```

### Current rationale

- State each active reason needed to understand the physical boundaries,
  responsibility placement, provider choice, or deployment shape.
- Use cause-and-effect language and material tradeoffs. Do not recount former
  designs, superseded decisions, dates, or product history.
