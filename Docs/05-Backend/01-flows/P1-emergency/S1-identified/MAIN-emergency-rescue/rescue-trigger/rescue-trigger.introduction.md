---
doc_role: baseline
module: rescue-trigger
kind: flow
status: active
last_updated: 2026-02-26
owners: [backend-team]
---

# Rescue Trigger Module - Introduction

## Roadmap Position

This domain follows:

- `RT-1` (Global Phase 1): stabilize dispatch core.
- `RT-1.1`: Hub Segregation (RescuerHub vs MissionHub) - **RESOLVED**.
- `RT-2` (Global Phase 4): switch matching to `Redis-first` with `PostGIS-fallback`.

Roadmap source:

- `../emergency-rescue.roadmap.md`

## Purpose

`rescue-trigger` is the backend flow that turns an emergency report into active rescuer dispatch:

1. Create a `SnakebiteIncident`.
2. Create dispatch session(s) with progressive radius.
3. Fan out requests to eligible rescuers over SignalR (`RescuerHub`).
4. Resolve the first successful acceptance into a `RescueMission`.
5. Transition Member and Assigned Rescuer to a dedicated `MissionHub` for coordination.

This module is the current operational bridge between emergency API and real-time rescuer notifications.

## Architecture: Segregated Hubs

The module uses two specialized SignalR hubs to minimize race conditions and state complexity:

### 1. RescuerHub (`/hubs/rescuer`)

- **Audience**: Available/Idling rescuers.
- **Purpose**: Broadcasts new `RescueRequest` pings.
- **Scope**: Ends once a rescuer accepts a mission and transitions to the execution phase.

### 2. MissionHub (`/hubs/mission`)

- **Audience**: Member (Incident Creator) and Assigned Rescuer only.
- **Purpose**: Active mission coordination.
- **Features**: Location tracking of active rescuer, arrival notifications, mission cancellation/completion events.
- **Security**: Strict authorization in `OnConnectedAsync` (verifying `IncidentId` membership) and per-operation verification via `Context.Items` persistence to prevent ID spoofing.
- 3. **Persistence**: Locations sent via `MissionHub.UpdateLocation` are persisted to the database for tracking.
- 4. **Reliability**: Notification delivery is decoupled from database transactions. A failure in the real-time notification layer will not cause a rollback of the underlying business operation. All notification services are hardened with error handling to ensure service continuity.

## Current Scope In Code

### Entry point

- `POST /api/incidents/sos` (authorized) creates incident then immediately starts session 1 dispatch.

### Session model

- Radius progression: `10 -> 20 -> 30` km.
- Maximum sessions: `3`.
- Per-session timeout: `60` seconds.
- Timeout handling runs in background (`SessionTimeoutBackgroundService`) and auto-expands radius while incident is still `Pending`.

### Current matching mode (RT-1)

- Candidate lookup is PostGIS-based from `RescuerProfile.LastLocation`.
- No production `Redis GEO` matching path yet.
- Live accuracy depends on whether location ingestion is implemented by `live-tracking` LT-1.

### Rescuer eligibility

A rescuer is considered dispatchable only when all conditions are true:

- `RescuerProfile.IsOnline == true`
- `RescuerProfile.LastLocation != null`
- `RescuerProfile.Type` is `Emergency` or `Both`
- Distance from incident is within current session radius
- Rescuer has active SignalR connection in `SignalRRescueNotificationService.ConnectedRescuers`

### Resolution rule

- First valid `AcceptRequest` wins.
- Winning request becomes `Accepted`.
- Other pending requests in the same session become `Taken`.
- Session becomes `Completed`.
- Incident is assigned and `RescueMission` is created.

## State Overview

### Incident (relevant states)

- `Pending` -> dispatching/awaiting acceptance
- `Assigned` -> rescuer mission created
- `Finished` -> mission completed
- `Cancelled` -> cancelled by user/system action
- `NoRescuerFound` -> max sessions exhausted

### Session

- `Active` -> waiting rescuer responses
- `Completed` -> one rescuer accepted
- `Failed` -> timeout reached with no accepted request
- `Cancelled` -> cancelled before resolution

### Request

- `Pending`, `Accepted`, `Rejected`, `Taken`, `Cancelled`, `Expired`

## Reality Check (Important)

Current implementation is functional for dispatch and follows a segregated hub architecture for scaling:

- `POST /api/incidents/{incidentId}/raise-range` currently creates a next-session record.
  - It does not automatically trigger broadcast logic or timeout scheduling; these are orchestrated by the background session manager.
- `RescuerHub` only handles broad-range discovery and request signals.
- `MissionHub` handles all active mission coordination, including location streaming and status lifecycle.
- `MissionHub` enforces strict `[Authorize]` at connection time, scoped to the specific `incidentId`.
- API services (`RescueMissionService`, `RescueRequestSessionService`) utilize `IMissionNotificationService` to target the appropriate hub for specific event types.
- **Reliability Design**: All SignalR notifications are decoupled from database transactions (dispatched after `Commit`). Notification service implementations are hardened with `try-catch` to ensure SignalR availability issues never interrupt core business logic.

## Dependency on Live Tracking

- `rescue-trigger` reads location for matching.
- `live-tracking` LT-1 is responsible for making that location feed truly live (persisting rescuer updates).
- `rescue-trigger` RT-2 will consume Redis geo pipeline when LT-2 contracts are ready.

## Cross References

- Source state: `rescue-trigger.sourcecode.md`
- Implementation prompt: `rescue-trigger.prompt.md`
- Plan: `rescue-trigger.plan.md`
- Integration contract: `rescue-trigger.usageguide.md`
- Related live tracking direction: `../live-tracking/live-tracking.architecture.md`
