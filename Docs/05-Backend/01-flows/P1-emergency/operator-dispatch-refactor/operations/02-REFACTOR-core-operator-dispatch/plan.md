---
doc_role: operation
operation_id: 02-REFACTOR-core-operator-dispatch
type: REFACTOR
status: done
created_at: 2026-03-14
affects:
  - SnakeAid.Core/Domains/SnakebiteIncident.cs
  - SnakeAid.Core/Domains/RescuerRequest.cs
  - SnakeAid.Service/Implements/SnakebiteIncidentService.cs
  - SnakeAid.Service/Implements/RescueMissionService.cs
  - SnakeAid.Api/Controllers/SnakebiteIncidentController.cs
---

# Plan - REFACTOR core operator dispatch

## 1. As-Is

- Emergency SOS originally centered on automatic rescuer dispatch semantics.
- Legacy docs and implementation context were not operator-first.

## 2. Gap Analysis

- No explicit operator ownership of intake and dispatch control
- Hard to represent manual verification and rescuer selection
- Realtime semantics not aligned to operator dashboard workflow

## 3. To-Be Design

- Introduce claim / confirm / dispatch flow
- Use dispatch request records for rescuer acknowledgement
- Create mission only after rescuer accepts

References:

- `analysis/01-architecture-decision.md`
- `analysis/02-state-machine.md`
- `analysis/03-sequence-flows.md`

## 4. Impacted Components

- incident states
- dispatch request records
- operator-facing incident APIs
- mission creation path

## 5. Risks & Constraints

- state naming may diverge temporarily from target design
- mission creation must not happen before rescuer acknowledgement
- operator and rescuer realtime expectations must stay aligned

## 6. Validation Plan

- verify claim / confirm / dispatch endpoints exist
- verify accept creates mission
- verify decline returns incident to redispatch-ready state
