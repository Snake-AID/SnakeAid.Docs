# Live Tracking Usage Guide

This guide documents what clients can use today and what is not available yet.

## 1. Current Integration Surface

### Available now
1. Start rescue dispatch via `POST /api/incidents/sos`.
2. Rescuer real-time offer handling via SignalR hub `/rescuer-hub`.
3. Incident detail read via `GET /api/incidents/{incidentId}`.
4. Session timeout monitoring endpoints for operations.

### Not available yet
1. Patient/admin map snapshot endpoint.
2. Patient/admin tracking stream endpoint/group contract.
3. Location history endpoint.
4. Fallback push channel for critical dispatch events.

## 2. SOS Entry for Tracking Lifecycle

### Request
- Method: `POST`
- URL: `/api/incidents/sos`
- Auth: required
- Body:

```json
{
  "lng": 106.660172,
  "lat": 10.762622
}
```

### Purpose
Starts dispatch lifecycle and creates first rescue session.
Use returned `incidentId` and `sessionId` as correlation keys in client state.

## 3. Rescuer Hub Contract

### Connect
- Hub URL: `/rescuer-hub`

### Client -> Server methods
1. `JoinAsRescuer(string userId)`
2. `AcceptRequest(Guid requestId, Guid rescuerId)`
3. `RejectRequest(Guid requestId)`
4. `UpdateLocation(string userId, double latitude, double longitude)`
5. `GetConnectedRescuers()`

### Server -> Client events
1. `Joined`
2. `NewRescueRequest`
3. `RequestAccepted`
4. `RequestTaken`
5. `RequestCancelled`
6. `RequestRejected`
7. `RequestError`
8. `LocationUpdated`
9. `ConnectedRescuers`

### `NewRescueRequest` payload example
```json
{
  "requestId": "request-guid",
  "sessionId": "session-guid",
  "incidentId": "incident-guid",
  "radiusKm": 10,
  "expiredAt": "2026-02-06T12:01:00Z",
  "requestSentAt": "2026-02-06T12:00:00Z"
}
```

## 4. Important Behavior Notes

1. `UpdateLocation(...)` currently does not persist rescuer location for matching.
2. Manual `raise-range` currently does not trigger dispatch broadcast/schedule.
3. Hub authorization hardening is not complete in current code.
4. `RequestExpired` push is not emitted in production timeout flow.

## 5. Client Strategy Until Full LLT Is Delivered

1. Treat live tracking as dispatch notifications only.
2. Do not build patient/admin map stream dependency yet.
3. Use incident detail endpoint for periodic status refresh.
4. Show graceful fallback text when true live map is unavailable.

## 6. Response Envelope Reminder

All REST endpoints use:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {},
  "error": null
}
```

Check `is_success` before using `data`.

## 7. Planned Future Contract (Not Implemented Yet)

Expected future additions:
1. `GET /api/sessions/{id}/tracking/snapshot`
2. `GET /api/sessions/{id}/tracking/history`
3. Session viewer methods:
   - `JoinSession(sessionId)`
   - `LeaveSession(sessionId)`
4. Viewer event:
   - `LocationUpdated(sessionId, locationPayload)`

Do not integrate these endpoints/events until they exist in backend source and are documented as implemented.

