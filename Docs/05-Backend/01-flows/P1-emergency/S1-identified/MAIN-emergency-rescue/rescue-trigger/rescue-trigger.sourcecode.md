---
doc_role: baseline
module: rescue-trigger
kind: flow
status: active
last_updated: 2026-02-26
owners: [backend-team]
---

# Rescue Trigger Module - Source Code

## Roadmap Alignment

- Domain phase mapping:
  - `RT-1` (Global Phase 1): dispatch core stabilization.
  - `RT-1.1`: Hub Segregation (RescuerHub vs MissionHub) - **RESOLVED**.
  - `RT-2` (Global Phase 4): dynamic locator switch (`Redis-first`, `PostGIS-fallback`).
- Roadmap source: `../emergency-rescue.roadmap.md`

## Status

- Module status: implemented for session-based rescue dispatch with segregated hubs.
- Last verified against code: 2026-02-26.

## Function Implementation Status (Agent Guardrail)

Use this section before coding to avoid re-implementing existing functions.

### A. Implemented and in active production path

| Layer                | Function                            | Status      | Notes                                              |
| -------------------- | ----------------------------------- | ----------- | -------------------------------------------------- |
| Controller           | `CreateSnakebiteIncident`           | Implemented | Main SOS entry (`POST /api/incidents/sos`)         |
| Session Service      | `CreateSessionAsync`                | Implemented | Creates session + schedules timeout                |
| Session Service      | `BroadcastRequestsAsync`            | Implemented | Creates rescuer requests + pushes SignalR          |
| Session Service      | `HandleSessionTimeoutAsync`         | Implemented | Expires pending requests + Pushes MissionHub event |
| Mission Service      | `CreateMissionAsync`                | Implemented | Creates mission and assigns incident               |
| Mission Service      | `UpdateMissionStatusAsync`          | Implemented | Updates status + Pushes MissionHub event           |
| Mission Service      | `CompleteMissionAsync`              | Implemented | Completes mission + Pushes MissionHub event        |
| Notification Service | `SignalRMissionNotificationService` | Implemented | Dedicated service for MissionHub events            |

### B. Hub Responsibilities

| Hub          | Responsibility                                   | Path            |
| ------------ | ------------------------------------------------ | --------------- |
| `RescuerHub` | Broadcasting new rescue requests to rescuers.    | `/hubs/rescuer` |
| `MissionHub` | Active mission coordination & location tracking. | `/hubs/mission` |

### C. Implemented but behavior incomplete (do not duplicate, fix existing)

| Layer            | Function                 | Status      | Missing part                                                     |
| ---------------- | ------------------------ | ----------- | ---------------------------------------------------------------- |
| Incident Service | `RaiseSessionRangeAsync` | Partial     | Creates session row only; missing broadcast + timeout scheduling |
| Hub              | `UpdateLocation`         | Implemented | Persists to `RescuerProfile` via `RescuerLocationService` (LT-1) |

### D. Not implemented yet (expected future work)

| Area                        | Missing function/capability                                           | Status  |
| --------------------------- | --------------------------------------------------------------------- | ------- |
| Locator abstraction         | `IRescuerLocator` (or equivalent) for strategy-based candidate lookup | Missing |
| Redis-first candidate query | `Redis GEO` matching path in production flow                          | Missing |
| Fallback strategy           | `PostGIS-fallback` under unified locator                              | Missing |
| Tracking read APIs          | Snapshot/history endpoints for session tracking                       | Missing |

### API & Hubs

- `SnakeAid.Api/Hubs/RescuerHub.cs`
- `SnakeAid.Api/Hubs/MissionHub.cs`
- `SnakeAid.Api/Services/SignalRRescueNotificationService.cs` (Targets `RescuerHub`)
- `SnakeAid.Api/Services/SignalRMissionNotificationService.cs` (Targets `MissionHub`)
- `SnakeAid.Api/Program.cs` (Hub mappings)

### Service

- `SnakeAid.Service/Implements/RescueRequestSessionService.cs`
- `SnakeAid.Service/Implements/RescueMissionService.cs`

### Contracts

- `SnakeAid.Service/Interfaces/IRescueNotificationService.cs`
- `SnakeAid.Service/Interfaces/IMissionNotificationService.cs`

## Runtime Flow in Current Code

### 1) SOS creates incident and starts first session

`POST /api/incidents/sos`

Flow:

1. `SnakebiteIncidentController.CreateSnakebiteIncident(...)`
2. `_incidentService.CreateIncidentAsync(...)`
   - Creates `SnakebiteIncident` with:
     - `Status = Pending`
     - `CurrentSessionNumber = 0`
     - `CurrentRadiusKm = 0`
3. `_incidentService.StartRescueAsync(incidentId)`
4. `_sessionService.StartRescueSessionAsync(incidentId)`
5. `RescueRequestSessionService.CreateSessionAsync(...)` creates session 1:
   - Radius `10`
   - Trigger `Initial`
   - Status `Active`
   - Schedules timeout at `now + 60s` via `_timeoutService.ScheduleSessionTimeout`
6. `BroadcastRequestsAsync(sessionId)`:
   - Finds rescuers in radius using PostGIS `Distance(...)`
   - Filters by online/type/connected
   - Inserts `RescuerRequest` rows
   - Sends `NewRescueRequest` SignalR events
   - Updates `session.RescuersPinged`

Constants:

- `RADIUS_PROGRESSION = { 10, 20, 30 }`
- `MAX_SESSIONS = 3`
- `REQUEST_TIMEOUT_SECONDS = 60`

### Current locator mode

- Candidate lookup is PostGIS-based (`RescuerProfile.LastLocation.Distance(...)`).
- `Redis GEO` candidate lookup is not implemented yet.
- This means RT-2 is pending even when RT-1 dispatch flow is already active.

### 2) Timeout monitoring and auto-expansion

`SessionTimeoutBackgroundService` responsibilities:

- Tracks scheduled session timeout timestamps in memory.
- Wakes up based on nearest timeout.
- Calls `IRescueRequestSessionService.HandleSessionTimeoutAsync(sessionId)` for expired sessions.

`HandleSessionTimeoutAsync` responsibilities:

- Ignore sessions not `Active`.
- Mark pending requests as `Expired`.
- Mark session as `Failed`.
- Call `TryExpandAndCreateNewSessionAsync(incidentId)`.

`TryExpandAndCreateNewSessionAsync` responsibilities:

- If incident is no longer `Pending`, stop.
- If max session reached, mark incident `NoRescuerFound`.
- Else create next session with next radius and immediately broadcast requests.

### 3) Rescuer accepts request (race winner path)

Hub method:

- `RescuerHub.AcceptRequest(Guid requestId, Guid rescuerId)`

Service:

- `RescueRequestSessionService.AcceptRequestAsync(...)`

Behavior:

1. Validate request exists and belongs to rescuer.
2. Validate request is `Pending` and not expired.
3. Reject if session already `Completed` (another rescuer already won).
4. Mark winner request `Accepted`.
5. Mark other pending requests in same session `Taken`.
6. Notify other rescuers with `RequestTaken`.
7. Mark session `Completed` and cancel timeout monitoring.
8. Commit changes.
9. Create mission via `RescueMissionService.CreateMissionAsync(...)`:
   - Creates `RescueMission` (`Preparing`)
   - Updates incident to `Assigned`
   - Sets `AssignedRescuerId`, `AssignedAt`

### 4) Rescuer rejects request

Hub method:

- `RescuerHub.RejectRequest(Guid requestId)`

Service:

- `RescueRequestSessionService.RejectRequestAsync(requestId)`

Behavior:

- Validates `Pending` request.
- Marks request as `Rejected`.

## SignalR Contract (Segregated)

### 1. RescuerHub (`/hubs/rescuer`)

**Client -> Server**:

- `JoinAsRescuer(string userId)`
- `AcceptRequest(Guid requestId, Guid rescuerId)`
- `RejectRequest(Guid requestId)`

**Server -> Client**:

- `NewRescueRequest`: `{ requestId, sessionId, incidentId, radiusKm, ... }`
- `RequestAccepted`: `{ requestId, incidentId, missionId, ... }`
- `RequestTaken`: `{ requestId }`
- `RequestExpired`: `{ requestId }`

### 2. MissionHub (`/hubs/mission?incidentId={incidentId}`)

**Authorization**: User must be Incident Creator or Assigned Rescuer.

**Client -> Server**:

- `UpdateLocation(Guid incidentId, double latitude, double longitude)`

**Server -> Client**:

- `MissionStarted`: `{ status }` (sent when Rescuer starts moving)
- `RescuerArrived`: (sent when Rescuer arrives at scene)
- `MissionCompleted`: `{ missionId }`
- `MissionCancelled`: `{ reason }`
- `LocationUpdated`: `{ userId, latitude, longitude, updatedAt }`
- `SessionExpired`: (sent to member when dispatch fails)

## HTTP Endpoints in This Flow

- `POST /api/incidents/sos`
- `POST /api/incidents/{incidentId}/raise-range`
- `GET /api/incidents/{incidentId}` (detail endpoint)
- `PUT /api/incidents/{incidentId}/symptoms-tracking`
- `GET /api/monitoring/session-timeout-status`
- `GET /api/monitoring/health/session-timeout`

## Current Gaps and Inconsistencies

1. Manual `raise-range` path is incomplete

- `SnakebiteIncidentService.RaiseSessionRangeAsync(...)` creates a new session row only.
- It does not broadcast requests for that new session.
- It does not schedule timeout monitoring for that new session.

2. Location update is not persisted (RESOLVED in LT-1)

- `RescuerHub.UpdateLocation(...)` now updates `RescuerProfile.LastLocation`.
- Matching depends on `RescuerProfile.LastLocation` in database.

3. No production request-expired notification fanout (RESOLVED)

- `IRescueNotificationService.NotifyRequestExpiredAsync(...)` exists.
- Timeout flow in `HandleSessionTimeoutAsync(...)` now calls it when session times out.

4. Security hardening is incomplete on hub

- `RescuerHub` has no `[Authorize]` attribute.
- `JoinAsRescuer(userId)` trusts client-provided `userId`.

5. Some interface methods are currently not part of active call path

- `SnakebiteIncidentService.TriggerRescueAsync(...)`
- `SnakebiteIncidentService.AcceptRescueAsync(...)`
- `SnakebiteIncidentService.RejectRescueAsync(...)`

## Phase-Oriented Interpretation

### RT-1 open items

1. Unify manual raise-range with session orchestration.
2. Harden hub auth/identity.
3. Add timeout expiry notifications.

### RT-2 open items

1. Introduce locator abstraction.
2. Add `Redis-first` query path.
3. Add `PostGIS-fallback` and rollout controls.

## Related Demo Artifacts

- `SnakeAid.Api/Controllers/RescueDemoController.cs`
- `SnakeAid.Api/Pages/Demo/RescueDemo.cshtml`
- `SnakeAid.Api/Pages/Demo/RescueDemo.cshtml.cs`

These are mock/demo flow assets and should not be treated as production API contracts.
