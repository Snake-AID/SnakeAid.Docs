---
doc_role: operation
operation_id: 03-REFACTOR-operator-realtime-and-duty-snapshot
type: REFACTOR
status: done
created_at: 2026-03-14
affects:
  - SnakeAid.Api/Hubs/RescuerHub.cs
  - SnakeAid.Api/Services/SignalROperatorRealtimeNotificationService.cs
  - SnakeAid.Api/Controllers/RescuerController.cs
  - SnakeAid.Api/Controllers/ShiftController.cs
  - SnakeAid.Service/Implements/OperatorSnapshotService.cs
---

# Plan - REFACTOR operator realtime and duty snapshot

## 1. As-Is

- Operator workflow needs live awareness of incidents and rescuers
- Rescuer availability also depends on shift status and current location

## 2. Gap Analysis

- Automatic dispatch-era docs did not center operator visibility
- Operator candidate selection needed shift-aware snapshot data
- Operators needed realtime rescuer presence and dispatch feedback

## 3. To-Be Design

- Add operator realtime group
- Broadcast incident and rescuer dispatch events to operators
- Expose on-duty rescuer snapshot API
- Introduce shift CRUD and assignment lifecycle APIs
- Support operator dashboard actions and migration planning toward dedicated `OperatorHub`

Sprint trace from commit analysis:

- `c00db438177d3f7a56f04b387a11e9e93bf4b00b`
- `db1d8f603cd538e39354074b36d5d7952cf293f0`
- `792526d90e78b3da0c5f16bbf2148d95f8f3620f`

## 4. Impacted Components

- operator realtime events
- on-duty rescuer snapshot
- shift template and assignment APIs

## 5. Risks & Constraints

- no standalone `OperatorHub` yet
- realtime events must stay consistent with HTTP state transitions
- snapshot filtering still depends on current availability flags

## 6. Validation Plan

- verify `GET /api/rescuers/on-duty`
- verify shift CRUD and assignment endpoints
- verify operator realtime events for claim, dispatch, rescuer accept/decline, and presence
- verify sprint-added operator events such as false alarm, no answer, cancel, and rescuer abort are represented in the notification contract
