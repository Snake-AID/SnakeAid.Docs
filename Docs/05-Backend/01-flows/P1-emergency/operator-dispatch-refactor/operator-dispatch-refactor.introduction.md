---
doc_role: baseline
module: operator-dispatch-refactor
kind: flow
status: active
last_updated: 2026-03-14
owners: [backend-team]
---

# Operator Dispatch Refactor - Introduction

## Purpose

`operator-dispatch-refactor` documents the new emergency dispatch flow where SOS incidents are handled by an operator-first dispatch center instead of the legacy ping-and-radius matching model.

This module exists to preserve the context of the architectural shift itself:

1. Member creates SOS.
2. Operator claims and verifies the incident.
3. Operator selects an on-duty rescuer.
4. Rescuer accepts or declines the dispatch.
5. Accepted dispatch opens a rescue mission and hands execution to mission tracking.

## Why this flow is documented separately

`P1-emergency/rescue-trigger` still captures the older session-based dispatch lineage and related live-tracking evolution.

This module is intentionally separate because the refactor changes:

- the business ownership of dispatch
- the state model of emergency rescue
- the realtime audiences and event semantics
- the operational dashboard expectations for operators

Keeping this as a dedicated flow avoids mixing legacy dispatch context with the new operator-centered model.

## Current status in codebase

This refactor is partially implemented in production backend code.

Current estimate:

```text
~60% complete versus the target design captured by the refactor documents.
```

What already exists:

- operator claim / confirm / dispatch APIs
- rescuer accept / decline dispatch flow
- `OperatorProfile`, `WorkShift`, `ShiftAssignment`
- `IncidentCallLog` schema/model scaffold
- operator realtime group and rescuer on-duty snapshot APIs
- shift CRUD and check-in / check-out APIs

What is still incomplete:

- full false-alarm flow
- redispatch endpoint and operator-driven reassignment flow
- operator disconnect release logic
- pending-incident escalation background worker
- final state-machine alignment with the target design

## Refactor operations and current completion

This flow records the refactor as explicit operations so the module can show both the new intended direction and the current implementation status.

| Operation | Purpose | Current status |
| --- | --- | --- |
| `01-INIT-operator-dispatch-refactor` | Bootstrap the dedicated flow docs and preserve the new narrative separately from legacy emergency docs | `done` |
| `02-REFACTOR-core-operator-dispatch` | Move emergency intake to operator claim / confirm / dispatch with rescuer accept-or-decline before mission creation | `done` |
| `03-REFACTOR-operator-realtime-and-duty-snapshot` | Add operator-facing realtime and shift-aware rescuer snapshot support | `done` |
| `04-REFACTOR-gap-closure-and-state-alignment` | Close remaining gaps such as false alarm, redispatch, recovery paths, and state alignment | `draft` |

## Boundaries

This module focuses on the emergency dispatch refactor only.

In scope:

- incident handling by operators
- rescuer dispatch and acknowledgement
- shift-aware rescuer availability
- operator-facing realtime visibility

Out of scope:

- snake identification subflow details
- full live-tracking pipeline internals
- consultation flow
- snake-catching flow

## Relationship to neighboring modules

- `../rescue-trigger/`
  - historical and legacy dispatch lineage
- `../live-tracking/`
  - location ingestion and mission-time location streaming
- `../snake-detection/`
  - species identification and first-aid support

## Documentation model

This module follows the Backend Documentation Protocol:

- baseline files describe current code truth
- operation folders capture refactor mutations and history

The baseline here reflects the current implemented operator-dispatch path, including known gaps.
