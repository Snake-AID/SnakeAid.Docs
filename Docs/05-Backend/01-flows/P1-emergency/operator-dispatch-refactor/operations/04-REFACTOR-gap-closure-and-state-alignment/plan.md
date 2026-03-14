---
doc_role: operation
operation_id: 04-REFACTOR-gap-closure-and-state-alignment
type: REFACTOR
status: draft
created_at: 2026-03-14
affects:
  - SnakeAid.Service/Implements/SnakebiteIncidentService.cs
  - SnakeAid.Api/Hubs/RescuerHub.cs
  - SnakeAid.Api/Controllers/SnakebiteIncidentController.cs
  - SnakeAid.Api/Services/SignalROperatorRealtimeNotificationService.cs
---

# Plan - REFACTOR gap closure and state alignment

## 1. As-Is

- Core operator dispatch exists
- Realtime and shift snapshots exist
- Several target behaviors remain incomplete

## 2. Gap Analysis

- false alarm flow not fully wired
- redispatch endpoint absent
- operator disconnect recovery absent
- pending escalation background worker absent
- current state naming still diverges from target design

## 3. To-Be Design

- close parity gaps against the refactor target state
- normalize state language and transition semantics
- harden operational recovery paths

## 4. Impacted Components

- incident command APIs
- operator realtime lifecycle
- background services
- state transition rules

## 5. Risks & Constraints

- production behavior already depends on `Verified` and `Assigned`
- renaming or remapping states may impact clients and docs together
- recovery automation must not accidentally steal active incidents

## 6. Validation Plan

- verify false alarm end-to-end
- verify redispatch flow
- verify operator disconnect release semantics
- verify escalation behavior for stale pending incidents
