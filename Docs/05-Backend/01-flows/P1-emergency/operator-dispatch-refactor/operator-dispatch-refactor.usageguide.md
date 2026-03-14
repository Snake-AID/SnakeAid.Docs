---
doc_role: baseline
module: operator-dispatch-refactor
kind: flow
status: active
last_updated: 2026-03-14
owners: [backend-team]
---

# Operator Dispatch Refactor - Usage Guide

Audience: clients and internal tools integrating the current operator-dispatch emergency flow.

## 1. Create SOS

### Endpoint

- Method: `POST`
- URL: `/api/incidents/sos`

### Current behavior

- Creates a `SnakebiteIncident`
- Sets incident to `Pending`
- Broadcasts the new incident to the operator realtime group

## 2. Claim incident

### Endpoint

- Method: `POST`
- URL: `/api/incidents/{incidentId}/claim`

### Current behavior

- Only operator/admin caller should use this path
- Sets `HandlingOperatorId`
- Moves incident to `OperatorContacting`

## 3. Confirm incident

### Endpoint

- Method: `POST`
- URL: `/api/incidents/{incidentId}/confirm`

### Current behavior

- Confirms the incident after operator contact
- Current code moves incident into `Verified`

## 4. Dispatch incident to rescuer

### Endpoint

- Method: `POST`
- URL: `/api/incidents/{incidentId}/dispatch`

### Request body

```json
{
  "rescuerId": "00000000-0000-0000-0000-000000000000"
}
```

### Current validation

- caller must be the handling operator
- incident must be `Verified`
- rescuer must exist
- rescuer must be online
- rescuer must be available
- rescuer must be on duty in the active shift window

## 5. Get on-duty rescuer snapshot

### Endpoint

- Method: `GET`
- URL: `/api/rescuers/on-duty`

### Query params

- `date` optional
- `incidentId` optional
- `onlyAvailable` optional, default `true`
- `maxDistanceKm` optional

### Current behavior

- returns shift-aware rescuer snapshot for operator map / dispatch UI
- can compute approximate distance to incident when `incidentId` is provided
- current controller is role-protected for `Operator,Admin`

### Related current route

- Method: `GET`
- URL: `/api/monitoring/on-duty`

Current note:

- this route is backed by the same operator snapshot service
- despite the controller name, it currently returns on-duty rescuer snapshot data rather than operator roster data
- treat the route name as current code truth, not as ideal final naming
- unlike `/api/rescuers/on-duty`, this controller path is not currently decorated with the same role-based authorize attribute

## 6. Shift management

### Supported endpoints

- `GET /api/shifts`
- `GET /api/shifts/{id}`
- `POST /api/shifts`
- `PUT /api/shifts/{id}`
- `DELETE /api/shifts/{id}`
- `POST /api/shifts/{id}/assign`
- `PATCH /api/shifts/assignments/{assignmentId}/checkin`
- `PATCH /api/shifts/assignments/{assignmentId}/checkout`
- `GET /api/shifts/assignments?date=YYYY-MM-DD`

## 7. Rescuer realtime methods

Current SignalR hub:

- `RescuerHub`

### Relevant methods

- `JoinAsRescuer(userId)`
- `AcceptDispatchRequest(requestId)`
- `DeclineDispatchRequest(requestId, reason)`
- `UpdateLocation(latitude, longitude)`
- `JoinAsOperator(operatorId?)`
- `LeaveAsOperator()`

## 8. Operator-facing realtime events currently used

- `IncidentLocationUpdated`
- `IncidentClaimed`
- `DispatchRequested`
- `RescuerAccepted`
- `RescuerDeclined`
- `RescuerOnlineStatus`
- `RescuerIdleLocationUpdated`
- `OperatorOnlineStatus`

## 9. Current contract caveats

### State naming caveat

The target refactor language may refer to `Confirmed` / `Dispatched`.
Current code path practically uses:

- `Verified` for confirmed-and-waiting
- `Assigned` after rescuer acceptance

### Operator realtime caveat

There is no standalone `OperatorHub` yet.
Operator realtime is currently multiplexed through the shared operator group used by existing hub/service wiring.

### Gap caveat

The following are not yet stable external contracts in this module:

- false alarm endpoint
- redispatch endpoint
- operator disconnect recovery flow
- escalation flow for stale pending incidents
