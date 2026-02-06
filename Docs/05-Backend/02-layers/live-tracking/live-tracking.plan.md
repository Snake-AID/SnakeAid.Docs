# Live Tracking Plan

## Objective
Close the gap between current rescue dispatch implementation and full live-tracking architecture.

## Baseline
- Dispatch sessions and acceptance race are already implemented.
- Timeout scheduler is implemented in-process.
- SignalR rescuer hub exists.
- Tracking for patient/admin map and fallback delivery are missing.

## Implementation Phases

### Phase 1: Contract Hardening and Consistency

Deliverables:
- Make one canonical path for session expansion and broadcasting.
- Remove or wire currently-unused service methods.
- Enforce authorization on `RescuerHub` and bind rescuer identity to token claims.

Code targets:
- `SnakeAid.Api/Hubs/RescuerHub.cs`
- `SnakeAid.Service/Implements/SnakebiteIncidentService.cs`
- `SnakeAid.Service/Implements/RescueRequestSessionService.cs`

Acceptance criteria:
- Manual raise-range creates/broadcasts/schedules timeout consistently.
- Hub methods reject mismatched user identity.

### Phase 2: Real Location Ingestion

Deliverables:
- Persist rescuer location updates to `RescuerProfile.LastLocation` + `LastLocationUpdate`.
- Add validation and throttling for location publish frequency.
- Keep current dispatch query compatible with persisted location.

Code targets:
- `SnakeAid.Api/Hubs/RescuerHub.cs`
- `SnakeAid.Service` location update service (new or existing module)
- `SnakeAid.Core/Domains/RescuerProfile.cs` usage paths

Acceptance criteria:
- Dispatch query uses recent location written by hub path.
- Location staleness policy is enforced.

### Phase 3: Session Tracking Read APIs

Deliverables:
- `GET /api/sessions/{id}/tracking/snapshot`
- `GET /api/sessions/{id}/tracking/history`
- Session role authorization for patient/rescuer/admin viewers.

Code targets:
- New tracking controller/service under `SnakeAid.Api` and `SnakeAid.Service`
- Domain read models from `SnakebiteIncident`, `RescueRequestSession`, `RescueMission`

Acceptance criteria:
- Patient/admin can restore map state after reconnect.
- Snapshot returns last known rescuer position and session metadata.

### Phase 4: Realtime Fan-out for Patient/Admin

Deliverables:
- Add session group model in SignalR (`JoinSession`, `LeaveSession`).
- Broadcast rescuer location updates to session viewers.
- Keep payload minimal and sequence-aware.

Code targets:
- `SnakeAid.Api/Hubs/RescuerHub.cs` (or dedicated tracking hub)
- Tracking service module

Acceptance criteria:
- Patient/admin receive `LocationUpdated` events during active mission.
- Reconnect flow remains stable (snapshot first, then stream).

### Phase 5: Reliability and Fallback

Deliverables:
- Integrate notification fallback channel for critical events (`NewOffer`, cancellation, timeout).
- Retry strategy and idempotent notification logging.
- Decide production strategy for distributed timeout scheduling (if multi-instance).

Code targets:
- Notification module abstraction and concrete provider.
- Timeout scheduler strategy in `SessionTimeoutBackgroundService` or replacement.

Acceptance criteria:
- Critical dispatch events are delivered even when realtime channel is unavailable.

### Phase 6: Analytics and Retention

Deliverables:
- Location history storage for audit/replay.
- Aggregated geospatial endpoints for admin heatmap/polygon overlays.
- Retention policy for high-volume location points.

Acceptance criteria:
- Admin receives map analytics payload without reading full raw point stream.

## Cross-Cutting Risks

1. In-memory timeout scheduler is single-process oriented.
2. Hub currently trusts client-provided user identity.
3. Tracking write volume can grow quickly without throttling/retention.
4. Manual raise-range path currently bypasses broadcast scheduler.

## Test Strategy

1. Unit tests:
- session expansion logic
- accept race winner semantics
- timeout transition rules

2. Integration tests:
- SOS -> dispatch -> accept -> mission creation
- timeout -> auto expansion
- reconnect snapshot + stream continuity

3. Load tests:
- concurrent location publish
- concurrent rescuer acceptance race

## Definition of Done (Layer)

1. Patient can open map and follow rescuer in near real time.
2. Rescuer dispatch remains stable under timeout and acceptance race.
3. Critical events have fallback delivery path.
4. Ops can monitor scheduler health and tracking flow health.
5. Sourcecode and usageguide docs match actual contracts.

