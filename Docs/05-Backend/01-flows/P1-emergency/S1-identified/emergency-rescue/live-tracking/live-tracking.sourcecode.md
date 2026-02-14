---
doc_role: baseline
module: live-tracking
kind: flow
status: active
last_updated: 2026-02-15
owners: [backend-team]
---

# Live Tracking Module - Source Code

## Roadmap Alignment
- Domain phase mapping:
  - `LT-1` (Global Phase 2): ingestion foundation.
  - `LT-2` (Global Phase 3): full live tracking pipeline.
- Roadmap source: `../emergency-rescue.roadmap.md`

## Status Snapshot (Current Codebase)

Last verified: 2026-02-13.

| Capability | Status | Notes |
|---|---|---|
| Session-based rescue dispatch | Implemented | SOS creates incident and starts session broadcast |
| Radius progression with timeout | Implemented | `10 -> 20 -> 30`, 60s/session |
| Acceptance race winner handling | Implemented | First accepted request wins, others become `Taken` |
| Mission creation after acceptance | Implemented | `RescueMission` created, incident becomes `Assigned` |
| Realtime rescuer offer delivery | Implemented (SignalR) | `NewRescueRequest` sent to connected rescuers |
| Realtime map tracking for patient/admin | Missing | No session tracking group and no tracking events for viewers |
| Persisted rescuer live location ingestion | Implemented | Hub update now persists `LastLocation` and `LastLocationUpdate` via `RescuerLocationService` |
| Snapshot/history tracking API | Missing | No `/tracking/snapshot` or `/tracking/history` endpoints |
| FCM fallback delivery | Missing | Notification service is SignalR-only |
| Redis NOW state | Missing | No Redis integration in current implementation |

## Phase Readiness Matrix

| Domain phase | Current status | Notes |
|---|---|---|
| LT-1 ingestion foundation | Complete | Location publish path now persists profile location with throttling |
| LT-2 full pipeline | Not started | No viewer stream, no snapshot/history, no Redis NOW path |

## Modules and Responsibilities

### Incident orchestration
- File: `SnakeAid.Service/Implements/SnakebiteIncidentService.cs`
- Responsibilities:
  - Create incident (`CreateIncidentAsync`)
  - Start initial rescue session (`StartRescueAsync`)
  - Manual range raise (`RaiseSessionRangeAsync`)
  - Symptom updates and incident detail read

### Session orchestration
- File: `SnakeAid.Service/Implements/RescueRequestSessionService.cs`
- Responsibilities:
  - Create session
  - Broadcast rescuer requests
  - Handle timeout
  - Handle accept/reject
  - Expand and create next session
  - Handle mission abort re-dispatch

### Timeout scheduler
- File: `SnakeAid.Service/Implements/SessionTimeoutBackgroundService.cs`
- Responsibilities:
  - In-memory timeout schedule queue
  - Periodic wake-up and expired session processing
  - Health and queue status for monitoring

### Realtime transport
- Files:
  - `SnakeAid.Api/Hubs/RescuerHub.cs`
  - `SnakeAid.Api/Services/SignalRRescueNotificationService.cs`
- Responsibilities:
  - Rescuer registration in connection dictionary
  - Receive accept/reject hub calls
  - Send request/taken/cancelled notifications

### Mission state
- File: `SnakeAid.Service/Implements/RescueMissionService.cs`
- Responsibilities:
  - Create mission when rescuer wins request
  - Update mission status
  - Handle user cancellation / rescuer abort

### Location Ingestion (New in LT-1)
- File: `SnakeAid.Service/Implements/RescuerLocationService.cs`
- Responsibilities:
  - Validate location/throttling strategy
  - Persist `RescuerProfile.LastLocation` (PostGIS)
  - Update `RescuerProfile.IsOnline` via Hub disconnection flow

## Current Dispatch Sequence

1. `POST /api/incidents/sos`
2. Incident created with point geometry (`LocationCoordinates`)
3. Session 1 created with radius 10 km and timeout scheduled
4. Candidate rescuers selected from `RescuerProfile`:
   - online
   - location not null
   - emergency-capable type
   - within radius by PostGIS `Distance`
   - currently connected in SignalR map
5. `RescuerRequest` rows inserted and `NewRescueRequest` pushed
6. Accept/reject handled via hub methods
7. Timeout marks session failed and auto-expands while session count < 3

## Data Surfaces Used by Dispatch

### Incident location
- `SnakebiteIncident.LocationCoordinates` (`geometry(Point, 4326)`)

### Rescuer location used for matching
- `RescuerProfile.LastLocation` (`geometry(Point, 4326)`)
- `RescuerProfile.LastLocationUpdate`

### Session records
- `RescueRequestSession` with:
  - `SessionNumber`
  - `RadiusKm`
  - `Status`
  - `TriggerType`
  - `RescuersPinged`

### Request records
- `RescuerRequest` with:
  - per-rescuer state (`Pending`, `Accepted`, `Rejected`, `Taken`, `Cancelled`, `Expired`)
  - timeout fields (`ExpiredAt`)

## Live Tracking Gap Analysis

### Gap 1: Location publish path is non-persistent (RESOLVED)
`RescuerHub.UpdateLocation(...)` now calls `RescuerLocationService` to persist `LastLocation`.
This gap is closed in LT-1.

### Gap 2: No session-viewer realtime channel
There is no `JoinSession/LeaveSession` contract for patient/admin watchers.
Current hub is rescuer-centric only.

### Gap 3: No tracking read APIs
No endpoint for:
- current snapshot for reconnect,
- historical path retrieval.

### Gap 4: Fallback and durable notification path missing
`IRescueNotificationService` currently has SignalR implementation only.
No fallback channel is used for delivery guarantees.

### Gap 5: Manual range raise is not equivalent to timeout expansion
`RaiseSessionRangeAsync` creates session row but does not broadcast/schedule timeout.
This diverges from `TryExpandAndCreateNewSessionAsync`.
This gap belongs to `rescue-trigger` RT-1 and is a dependency for stable full tracking rollout.

## Operational Interfaces

### Monitoring
- `GET /api/monitoring/session-timeout-status`
- `GET /api/monitoring/health/session-timeout`

### Hub path
- `/rescuer-hub`

## Notes for Next Iteration

1. Execute LT-1 first (real location ingestion for dispatch freshness).
2. Execute LT-2 after LT-1 is stable and RT-1 dispatch behavior is aligned.
3. Keep transport and business state ownership separated.
4. Ensure docs are updated when contracts change:
   - `live-tracking.usageguide.md`
   - `rescue-trigger.usageguide.md`
