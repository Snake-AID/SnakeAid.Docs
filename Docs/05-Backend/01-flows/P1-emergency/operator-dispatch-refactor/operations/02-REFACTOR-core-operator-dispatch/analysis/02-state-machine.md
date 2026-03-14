---
doc_role: operation
operation_id: 02-REFACTOR-core-operator-dispatch
type: REFACTOR
status: done
created_at: 2026-03-14
affects:
  - SnakeAid.Core/Domains/SnakebiteIncident.cs
  - SnakeAid.Core/Domains/RescuerRequest.cs
  - SnakeAid.Core/Domains/RescueMission.cs
---

# State Machine - Core Operator Dispatch

## Incident states in current implementation

```text
Pending
  -> OperatorContacting
  -> Cancelled

OperatorContacting
  -> Verified
  -> FalseAlarm   (target path exists in model, not fully wired end-to-end)

Verified
  -> Assigned     (rescuer accepted dispatch)
  -> Verified     (rescuer declined and operator must redispatch)

Assigned
  -> Finished
  -> Cancelled
```

## Dispatch request states

```text
Pending
  -> Accepted
  -> Declined
  -> Cancelled
```

## Mission states relevant after acceptance

```text
Preparing
  -> EnRoute
  -> MissionAborted
  -> Cancelled

EnRoute
  -> RescuerArrived
  -> MissionAborted

RescuerArrived
  -> MissionCompleted
```

## Important mismatch versus target design

Target refactor language often expects:

- `Confirmed`
- `Dispatched`

Current implemented flow primarily uses:

- `Verified`
- `Assigned`

This mismatch is one of the remaining refactor gaps and must be treated as intentional current truth, not accidental documentation drift.
