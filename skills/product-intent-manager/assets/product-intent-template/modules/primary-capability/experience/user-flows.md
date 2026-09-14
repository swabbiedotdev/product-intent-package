# User flows

This file belongs to the initial capability module. Rename its module directory
to the actual responsibility. Keep its public behavior here; add internal
diagrams within the same module only when needed. Shared or cross-module
behavior also needs a module owner, not a root workflow or sequence file.

Show actor actions, choices, navigation, and visible product outcomes. Use one
focused flow per actor goal, including consequential failure and recovery
paths. This view owns the user-visible path, not internal execution.

Link system work to its `SEQ-*`, `SM-*`, or rule diagram. Do not draw service
calls, database reads, authorization checks, or query mechanics here.

Every rule must belong to a documented process or be explicitly called by one.
For shared internal logic, the owning sequence shows invocation, input sources,
and outcome handling; a link from this user flow is not a substitute. Keep
purely visible conditions at their actual interaction step rather than in an
isolated rule list.

Visually group actions and states by consequential user-visible surface. Label
each boundary with `SURFACE · Name`, or `SCREEN-* · Name` when another artifact
references it. These boundaries form the mockup inventory; individual flow
nodes do not each require a mockup. A project may consistently distinguish
surface types with labels such as `VIEW`, `COMPONENT`, `DIALOG`, and `EXTERNAL`.

For each surface governed by a current release mockup, add the exact frame or
node, version or branch when available, and any companion example or export-code
reference. The linked mockup is the implementation target, not a visual
attachment. Do not add, remove, merge, split, or materially alter its views,
components, or states unless the product or design leader changes the target.

Use labeled edges for navigation choices and visible permission, availability,
or validation outcomes. Use a diamond only for a question visibly presented to
the actor. When several conditions route to different surfaces, use one compact
rectangular condition node. Keep runtime selection logic in a linked decision
diagram or sequence, not prose or YAML rule lists. Render visible conditions,
refusal, and recovery paths here rather than adding them only below the diagram.

```mermaid
---
config:
  theme: dark
---
flowchart TD
  %% Label the flow with its FLOW-* ID. Use ACTOR-* and SCREEN-* IDs where
  %% applicable, and phrase each node as an actor action or visible product
  %% response. Reserve diamonds for questions visibly presented to the actor.
  %% Use labeled subgraphs to bound user-visible surfaces and their states.
```

## Current rationale

- State each active reason for the surface boundaries, visible state
  distinctions, navigation, or recovery experience.
- Explain current cause and effect; do not narrate prior designs, superseded
  decisions, dates, or product history.
