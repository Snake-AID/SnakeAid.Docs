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

# Architecture Decision - Core Operator Dispatch

## Problem

The older emergency flow was built around broad rescuer pinging and progressive search expansion.
The new requirement shifts dispatch control to operators.

The system needs a new control point where:

- operator owns incident intake
- operator validates before dispatch
- rescuer acknowledgement opens the mission

## Options considered

### Option A - Keep pure session/radius dispatch as the main path

Rejected because:

- dispatch ownership stays with automatic matching
- poor fit for operator call-and-verify workflow
- difficult to express manual rescuer selection

### Option B - Replace dispatch core with operator-centered workflow

Chosen because:

- matches the new rescue center operating model
- allows explicit claim / confirm / dispatch transitions
- makes operator visibility a first-class concern

## Chosen direction

Adopt operator-centered dispatch as the new emergency control flow:

1. Member creates incident
2. Operator claims
3. Operator confirms
4. Operator dispatches rescuer
5. Rescuer accepts/declines
6. Acceptance creates mission

## Current implementation reality

This decision is only partially realized.

Implemented:

- operator claim
- operator confirm
- operator dispatch
- rescuer accept / decline
- mission creation from dispatch acceptance

Not fully realized yet:

- false alarm path
- redispatch flow
- operator disconnect release behavior
- final target state naming parity
