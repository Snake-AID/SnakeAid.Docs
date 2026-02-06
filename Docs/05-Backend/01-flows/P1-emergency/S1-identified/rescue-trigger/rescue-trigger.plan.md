# Rescue Trigger Plan

## Objective
Stabilize and complete rescue-trigger flow so manual and automatic dispatch paths behave consistently.

## Current State

Implemented:
1. SOS creates incident and starts first dispatch session.
2. Timeout scheduler auto-expands session radius up to max sessions.
3. Accept/reject race handling and mission creation.
4. Monitoring endpoints for timeout service.

Gaps:
1. Manual `raise-range` path does not broadcast or schedule timeout.
2. Rescuer location update path does not persist matching coordinates.
3. Hub identity security is weak (client-provided userId).
4. Request-expired event fanout is not emitted in production timeout path.

## Work Items

### Work Item 1: Unify Session Expansion Path

Goal:
- Ensure manual `raise-range` path calls the same orchestration used by timeout expansion.

Changes:
- Refactor `SnakebiteIncidentService.RaiseSessionRangeAsync(...)` to delegate to `IRescueRequestSessionService`.
- Keep response contract unchanged.

### Work Item 2: Harden Hub Identity and Authorization

Goal:
- Prevent rescuer impersonation in SignalR methods.

Changes:
- Add authorization requirement on `RescuerHub`.
- Resolve rescuer identity from authenticated claims.
- Validate request `rescuerId` against claim identity.

### Work Item 3: Persist Rescuer Location

Goal:
- Make `UpdateLocation` meaningful for matching.

Changes:
- Write `LastLocation` and `LastLocationUpdate` to `RescuerProfile`.
- Add throttling/rate limit for location writes.

### Work Item 4: Complete Timeout Notification Semantics

Goal:
- Notify pending rescuers when requests expire due to timeout.

Changes:
- Add notification fanout from timeout handler for expired requests.

## Validation Strategy

1. Integration scenario:
- `sos` -> request broadcast -> no accept -> timeout -> auto-expand.

2. Integration scenario:
- `sos` -> manual raise-range -> verify request broadcast actually occurs.

3. Integration scenario:
- two rescuers accept same request concurrently -> only one mission created.

4. Security scenario:
- unauthorized hub client cannot join/accept as arbitrary rescuer.

## Done Criteria

1. Manual and automatic expansion produce equivalent dispatch behavior.
2. Location updates affect matching query data source.
3. Timeout expiration notifies affected rescuers.
4. Rescue-trigger docs remain synchronized after changes.

