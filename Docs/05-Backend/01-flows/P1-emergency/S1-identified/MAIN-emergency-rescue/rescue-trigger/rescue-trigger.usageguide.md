---
doc_role: baseline
module: rescue-trigger
kind: flow
status: active
last_updated: 2026-02-26
owners: [backend-team]
---

# Rescue Trigger Module - Usage Guide

Audience: mobile/web client developers integrating emergency dispatch.

## 0. Phase Context

This guide describes the **Segregated Hubs** architecture for SOS and active mission coordination.
Roadmap: `../emergency-rescue.roadmap.md`

## 1. Response Envelope

HTTP APIs in this module return:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {},
  "error": null
}
```

## 2. Trigger SOS Dispatch

### Endpoint

- Method: `POST`
- URL: `/api/incidents/sos`
- Auth: required (`Bearer` token)

### Request body

```json
{
  "lng": 106.660172,
  "lat": 10.762622
}
```

## 3. Raise Search Range (Manual)

### Endpoint

- Method: `POST`
- URL: `/api/incidents/{incidentId}/raise-range`
- Auth: required

### Behavior

- Creates a new dispatch session for the incident.
- **Note**: In current `RT-1` code, this does not manually trigger a broadcast or timeout schedule—it relies on the background service to pick up the next state if the previous one failed.

## 4. Incident Detail

### Endpoint

- Method: `GET`
- URL: `/api/incidents/{incidentId}`
- Auth: required

### Response Data

Includes `RescueMission` details if a rescuer has accepted.

## 5. Rescuer Connection (`RescuerHub`)

Available rescuers should connect to this hub to receive job pings.

- URL: `/hubs/rescuer` (Requires Auth)

### Common Methods

- `JoinAsRescuer(string userId)`: Start receiving rescue requests.
- `AcceptRequest(Guid requestId, Guid rescuerId)`: Accept a pending request.
- `RejectRequest(Guid requestId)`: Explicitly decline a request.

### Inbound Events

- `NewRescueRequest`: A new incident is nearby.
- `RequestAccepted`: You successfully won the mission. **Client: Close this connection and switch to MissionHub immediately.**
- `RequestTaken`: Another rescuer accepted first.
- `RequestExpired`: The session timed out.

## 6. Mission Coordination (`MissionHub`)

Once a mission is assigned, both Member and Rescuer must connect to this dedicated hub for end-to-end coordination.

- URL: `/hubs/mission?incidentId={GUID}` (Requires Auth)

### Rescuer Methods

- `UpdateLocation(Guid incidentId, double latitude, double longitude)`: Pushes live updates to the Member and persists coordinates.

### Shared Events

- `LocationUpdated`: `{ userId, latitude, longitude, updatedAt }`
- `RescuerArrived`: Rescuer has reached the scene.
- `MissionCompleted`: Mission finished successfully.
- `MissionCancelled`: `{ reason }`: Rescuer aborted or User cancelled.
- `SessionExpired`: (Sent to Member) No rescuers were found after all search phases.

## 7. Monitoring Endpoints

### Session Timeout Queue

- URL: `/api/monitoring/session-timeout-status`
- Purpose: Inspect current in-memory timeout queue (for debugging).

### Health Check

- URL: `/api/monitoring/health/session-timeout`
- Purpose: Returns `200 OK` if the background worker is running.

## 8. Important Implementation Details

1. **Hub Transition**: After receiving `RequestAccepted` on `RescuerHub`, the mobile app must immediately connect to `MissionHub` using the `incidentId`.
2. **Authorization**: `MissionHub` enforces that only the incident's creator or the assigned rescuer can connect. It further verifies that all method calls (like `UpdateLocation`) target the specific `incidentId` authorized during connection.
3. **Persistance**: Locations sent via `MissionHub.UpdateLocation` are persisted to the database for tracking.

## 9. Error Cases to Handle

- `400`: invalid state transition (e.g., accepting expired request).
- `401`: unauthorized access (e.g., trying to join a MissionHub for an incident you don't own).
- `404`: incident not found.
- `500`: unexpected server error.
