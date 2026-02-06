# Rescue Trigger Source Code

## Roadmap Alignment
- Domain phase mapping:
  - `RT-1` (Global Phase 1): dispatch core stabilization.
  - `RT-2` (Global Phase 4): dynamic locator switch (`Redis-first`, `PostGIS-fallback`).
- Roadmap source: `../emergency-rescue.roadmap.md`

## Status
- Module status: implemented for session-based rescue dispatch.
- Baseline commit analyzed: `f6aca477d58cdae30b1c1d952aecbd10bf54f378`.
- Current branch also includes follow-up changes (for example `GetIncidentDetail` endpoint in later commit).
- Last verified against code: 2026-02-06.

## Function Implementation Status (Agent Guardrail)

Use this section before coding to avoid re-implementing existing functions.

### A. Implemented and in active production path

| Layer | Function | Status | Notes |
|---|---|---|---|
| Controller | `CreateSnakebiteIncident` | Implemented | Main SOS entry (`POST /api/incidents/sos`) |
| Controller | `RaiseSessionRange` | Implemented | Endpoint exists, but behavior gap noted in section C |
| Controller | `GetIncidentDetail` | Implemented | Read incident detail |
| Controller | `UpdateSymptomReport` | Implemented | Updates symptom report |
| Service | `CreateIncidentAsync` | Implemented | Creates incident record |
| Service | `StartRescueAsync` | Implemented | Calls session service to start dispatch |
| Session Service | `CreateSessionAsync` | Implemented | Creates session + schedules timeout |
| Session Service | `BroadcastRequestsAsync` | Implemented | Creates rescuer requests + pushes SignalR |
| Session Service | `HandleSessionTimeoutAsync` | Implemented | Expires pending requests + marks session failed |
| Session Service | `TryExpandAndCreateNewSessionAsync` | Implemented | Auto expansion path after timeout |
| Session Service | `StartRescueSessionAsync` | Implemented | Initial session bootstrap |
| Session Service | `AcceptRequestAsync` | Implemented | Winner flow + mission create |
| Session Service | `RejectRequestAsync` | Implemented | Reject pending request |
| Timeout Service | `ScheduleSessionTimeout` | Implemented | In-memory schedule |
| Timeout Service | `CancelSessionTimeout` | Implemented | Cancel schedule |
| Timeout Service | `GetQueueStatus` | Implemented | Monitoring support |
| Timeout Service | `IsHealthy` | Implemented | Monitoring support |
| Mission Service | `CreateMissionAsync` | Implemented | Creates mission and assigns incident |

### B. Implemented but not in active production call path

| Layer | Function | Status | Notes |
|---|---|---|---|
| Incident Service | `TriggerRescueAsync` | Implemented (not used) | Validation/response only, not used by controller path |
| Incident Service | `AcceptRescueAsync` | Implemented (not used) | Validation/response wrapper, real accept is in session service |
| Incident Service | `RejectRescueAsync` | Implemented (not used) | Validation/response wrapper, real reject is in session service |
| Session Service | `HandleMissionAbortAsync` | Implemented (conditional path) | Used when mission abort flow triggers retry |
| Session Service | `CancelSessionAsync` | Implemented (conditional path) | Not part of primary SOS path |

### C. Implemented but behavior incomplete (do not duplicate, fix existing)

| Layer | Function | Status | Missing part |
|---|---|---|---|
| Incident Service | `RaiseSessionRangeAsync` | Partial | Creates session row only; missing broadcast + timeout scheduling |
| Hub | `UpdateLocation` | Partial | Echoes location only; missing persistence to `RescuerProfile` |
| Notification Service | `NotifyRequestExpiredAsync` | Partial wiring | Method exists, timeout flow does not call it |

### D. Not implemented yet (expected future work)

| Area | Missing function/capability | Status |
|---|---|---|
| Locator abstraction | `IRescuerLocator` (or equivalent) for strategy-based candidate lookup | Missing |
| Redis-first candidate query | `Redis GEO` matching path in production flow | Missing |
| Fallback strategy | `PostGIS-fallback` under unified locator | Missing |
| Tracking read APIs | Snapshot/history endpoints for session tracking | Missing |

## Key Files

### API
- `SnakeAid.Api/Controllers/SnakebiteIncidentController.cs`
- `SnakeAid.Api/Hubs/RescuerHub.cs`
- `SnakeAid.Api/Controllers/MonitoringController.cs`
- `SnakeAid.Api/Services/SignalRRescueNotificationService.cs`
- `SnakeAid.Api/Program.cs`

### Service
- `SnakeAid.Service/Implements/SnakebiteIncidentService.cs`
- `SnakeAid.Service/Implements/RescueRequestSessionService.cs`
- `SnakeAid.Service/Implements/RescueMissionService.cs`
- `SnakeAid.Service/Implements/SessionTimeoutBackgroundService.cs`

### Contracts and Domains
- `SnakeAid.Service/Interfaces/ISnakebiteIncidentService.cs`
- `SnakeAid.Service/Interfaces/IRescueRequestSessionService.cs`
- `SnakeAid.Service/Interfaces/IRescueMissionService.cs`
- `SnakeAid.Service/Interfaces/IRescueNotificationService.cs`
- `SnakeAid.Service/Interfaces/ISessionTimeoutService.cs`
- `SnakeAid.Core/Domains/SnakebiteIncident.cs`
- `SnakeAid.Core/Domains/RescueRequestSession.cs`
- `SnakeAid.Core/Domains/RescuerRequest.cs`
- `SnakeAid.Core/Domains/RescueMission.cs`

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

## SignalR Contract (Current)

Hub route:
- `/rescuer-hub`

Client -> Server methods:
- `JoinAsRescuer(string userId)`
- `AcceptRequest(Guid requestId, Guid rescuerId)`
- `RejectRequest(Guid requestId)`
- `UpdateLocation(string userId, double latitude, double longitude)`
- `GetConnectedRescuers()`

Server -> Client events used in production flow:
- `Joined`
- `NewRescueRequest`
- `RequestAccepted`
- `RequestTaken`
- `RequestCancelled`
- `RequestRejected`
- `RequestError`
- `LocationUpdated` (echo to caller from `UpdateLocation`)
- `ConnectedRescuers`

`NewRescueRequest` payload currently includes:
- `requestId`
- `sessionId`
- `incidentId`
- `radiusKm`
- `expiredAt`
- `requestSentAt`

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

2. Location update is not persisted
- `RescuerHub.UpdateLocation(...)` does not write `RescuerProfile.LastLocation`.
- Matching still depends on `RescuerProfile.LastLocation` in database.

3. No production request-expired notification fanout
- `IRescueNotificationService.NotifyRequestExpiredAsync(...)` exists.
- Timeout flow in `HandleSessionTimeoutAsync(...)` does not call it.

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
