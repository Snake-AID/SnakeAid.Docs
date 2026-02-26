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
- **Security**: Strict authorization in `OnConnectedAsync` verifying User is part of requested `IncidentId`.

## Current Scope In Code

### Entry point

- `POST /api/incidents/sos` (authorized) creates incident then immediately starts session 1 dispatch.

### Session model

- Radius progression: `10 -> 20 -> 30` km.
- Maximum sessions: `3`.
- Per-session timeout: `60` seconds.
- Timeout handling runs in background (`SessionTimeoutBackgroundService`) and auto-expands radius while incident is still `Pending`.

### Rescuer eligibility

A rescuer is considered dispatchable only when all conditions are true:

- `RescuerProfile.IsOnline == true`
- `RescuerProfile.LastLocation != null`
- `RescuerProfile.Type` is `Emergency` or `Both`
- Distance from incident is within current session radius
- Rescuer has active SignalR connection in `RescuerHub`.

### Resolution rule

- First valid `AcceptRequest` wins.
- Winning request becomes `Accepted`.
- Other pending requests in the same session become `Taken`.
- Mission created; Rescuer and Member connect to `MissionHub`.

## Reality Check (Important)

Current implementation status:

- `RescuerHub.UpdateLocation(...)` has been **removed** and moved to `MissionHub`.
- `MissionHub` enforces authorization at connection time using `incidentId`.
- API services (`RescueMissionService`, `RescueRequestSessionService`) use `IMissionNotificationService` to push events to the correct hub.

## Dependency on Live Tracking

- `rescue-trigger` reads location for matching.
- `live-tracking` LT-1 (via `MissionHub`) is responsible for making that location feed truly live during an active mission.

## Cross References

- Source state: `rescue-trigger.sourcecode.md`
- Implementation plan: `REFACTOR-Hub-Segregation/04-plan.md`
- Integration contract: `rescue-trigger.usageguide.md`
