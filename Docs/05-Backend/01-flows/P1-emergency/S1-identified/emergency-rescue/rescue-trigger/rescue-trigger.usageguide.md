# Rescue Trigger Usage Guide

Audience: mobile/web client developers integrating emergency dispatch.

## 0. Phase Context

This guide reflects current `RT-1` behavior (Global Phase 1).
Roadmap: `../emergency-rescue.roadmap.md`

Current matching path is PostGIS-based from persisted rescuer location.
`Redis-first` matching described in roadmap belongs to `RT-2` and is not active yet.

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

Use `is_success` and `status_code` for client control flow.

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

### Success response (`data` excerpt)
```json
{
  "id": "incident-guid",
  "userId": "member-guid",
  "locationCoordinates": {
    "latitude": 10.762622,
    "longitude": 106.660172
  },
  "status": 0,
  "currentSessionNumber": 1,
  "currentRadiusKm": 10,
  "incidentOccurredAt": "2026-02-06T12:00:00Z",
  "sessionId": "session-guid",
  "sessionNumber": 1,
  "radiusKm": 10,
  "rescuersPinged": 3
}
```

## 3. Raise Search Range Manually

### Endpoint
- Method: `POST`
- URL: `/api/incidents/{incidentId}/raise-range`
- Auth: required

### Current behavior note
Current implementation creates a new session record and updates incident radius/session counters.
It does not trigger broadcast automatically for that session.
Treat this endpoint as internal/admin/debug until service path is completed.

## 4. Incident Detail and Symptom Update

### Get incident detail
- Method: `GET`
- URL: `/api/incidents/{incidentId}`

### Update symptoms
- Method: `PUT`
- URL: `/api/incidents/{incidentId}/symptoms-tracking`

Request body follows `UpdateSymptomReportRequest` contract.

## 5. Rescuer SignalR Integration

### Hub
- URL: `/rescuer-hub`

### Client -> Server methods
- `JoinAsRescuer(string userId)`
- `AcceptRequest(Guid requestId, Guid rescuerId)`
- `RejectRequest(Guid requestId)`
- `UpdateLocation(string userId, double latitude, double longitude)`
- `GetConnectedRescuers()`

### Server -> Client events (production path)
- `Joined`
- `NewRescueRequest`
- `RequestAccepted`
- `RequestTaken`
- `RequestCancelled`
- `RequestRejected`
- `RequestError`
- `LocationUpdated`
- `ConnectedRescuers`

### `NewRescueRequest` payload shape
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

## 6. Monitoring Endpoints

Both endpoints require auth:
- `GET /api/monitoring/session-timeout-status`
- `GET /api/monitoring/health/session-timeout`

Use these for operational visibility of in-memory timeout scheduler.

## 7. Important Integration Caveats

1. `UpdateLocation(...)` currently does not persist rescuer coordinates to database.
2. Hub currently does not enforce authorization attributes at endpoint level.
3. `RequestExpired` notification method exists in service, but production timeout path currently does not push this event.
4. FCM fallback is not integrated yet in production path.

## 8. Planned Changes in RT-2 (Not Active Yet)

1. Candidate lookup will move to `Redis-first`.
2. `PostGIS-fallback` will be used when Redis data is unavailable.
3. Endpoint contracts are expected to remain stable, but matching latency and candidate freshness should improve.

## 9. Error Cases to Handle

- `400`: invalid state transition (for example accepting expired/taken request).
- `401`: unauthorized API access.
- `404`: incident/request/session not found.
- `500`: unexpected server error.

Always show `message` to user and log `error.errorCode` when available.
