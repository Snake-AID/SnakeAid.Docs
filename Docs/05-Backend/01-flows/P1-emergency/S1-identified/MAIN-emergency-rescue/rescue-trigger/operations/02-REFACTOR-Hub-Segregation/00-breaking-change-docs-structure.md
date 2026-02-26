---
doc_role: operation
operation_id: REFACTOR-split-mission-hub
type: REFACTOR
status: active
created_at: 2026-02-26
---

# Documentation Structure for Hub Separation Breaking Change

Due to the complex nature of separating `RescuerHub` and `MissionHub`—which involves SignalR state, distributed network resilience, and complex transitions—this operation deviates from the standard linear `plan.md` defined in `AGENTS.md`.

To ensure safe implementation and proper risk assessment, this operation is structured into specialized focus documents:

## Document Hierarchy

1. **`00-breaking-change-docs-structure.md`** (This file)
   - Defines why and how documents are organized for this specific refactor.

2. **`01-architecture-decision.md` (ADR)**
   - Discusses the timing of transitioning users between hubs.
   - Evaluates options and trade-offs.
   - Records the final decision ("Asymmetric Connection") and mitigations.

3. **`02-state-machine.md`**
   - Defines the valid states for Member and Rescuer.
   - Matrices for state transitions (Cancel, Accept, Timeout).
   - Establishes the hard rule separating "SignalR Presence" from "Database State".

4. **`03-sequence-flows.md`**
   - Mermaid.js diagrams visualizing the exact API and SignalR interactions for key scenarios.
   - Provides developers a visual timeline of events to prevent race conditions.

5. **`04-plan.md`**
   - The standard `AGENTS.md` execution plan.
   - Maps the designs in files 01, 02, and 03 into specific class, controller, and database changes.
   - Serves as the source for `prompt.md`.

6. **`05-prompt.md`**
   - Generated from `04-plan.md` for AI execution.

7. **`decision-log.md`**
   - Historical record of the multi-agent brainstorming session used to stress-test the ADR.
