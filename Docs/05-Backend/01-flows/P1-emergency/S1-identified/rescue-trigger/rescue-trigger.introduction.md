# Rescue Trigger Introduction

## Purpose
`rescue-trigger` is the backend flow that turns an emergency report into active rescuer dispatch:

1. Create a `SnakebiteIncident`.
2. Create dispatch session(s) with progressive radius.
3. Fan out requests to eligible rescuers over SignalR.
4. Resolve the first successful acceptance into a `RescueMission`.

This module is the current operational bridge between emergency API and real-time rescuer notifications.

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
Current implementation is functional for dispatch, but not fully aligned with full live-tracking architecture yet:

- `POST /api/incidents/{incidentId}/raise-range` currently creates a new session record only.
  - It does not call broadcast logic.
  - It does not schedule timeout monitoring for the new session.
- `RescuerHub.UpdateLocation(...)` only sends `LocationUpdated` back to caller.
  - It does not persist `RescuerProfile.LastLocation`.
  - It does not broadcast to patient/admin.
- `NotifyRequestExpiredAsync` exists in notification service but is not invoked in production timeout flow.
- Hub does not enforce `[Authorize]` at class/endpoint level in current code.

## Cross References
- Source state: `rescue-trigger.sourcecode.md`
- Integration contract: `rescue-trigger.usageguide.md`
- Related direction doc: `../../../../02-layers/live-tracking/live-tracking.architecture.md`

