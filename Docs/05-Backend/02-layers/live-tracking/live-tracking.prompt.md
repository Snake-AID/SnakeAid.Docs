# Live Tracking Prompt

Use this prompt when implementing the next live-tracking iteration in SnakeAid backend.

## Prompt

You are implementing Live Tracking for SnakeAid backend on top of existing rescue-trigger flow.

### Goal
Deliver production-safe tracking behavior without breaking current rescue dispatch.

### Must preserve
1. `POST /api/incidents/sos` existing behavior and response envelope.
2. Session timeout and acceptance race semantics.
3. Existing entity state transitions for incident/session/request/mission.

### Mandatory fixes in this iteration
1. Unify session expansion logic so manual raise-range path also broadcasts and schedules timeout.
2. Enforce authorization on rescuer hub and derive rescuer identity from token claims.
3. Persist location updates to `RescuerProfile.LastLocation` and `LastLocationUpdate`.
4. Add snapshot endpoint for active session tracking.

### Nice-to-have if scope allows
1. Add history endpoint for location playback.
2. Add fallback notification strategy abstraction for critical events.
3. Add request-expired notification fanout in timeout handling.

### Implementation constraints
1. Keep API response structure (`status_code`, `is_success`, `message`, `data`, `error`).
2. Keep domain ownership:
   - durable state in PostgreSQL,
   - realtime transport in SignalR,
   - optional volatile acceleration store only via backend service layer.
3. Do not introduce direct client access to infrastructure stores.
4. Keep solution monolith-friendly and incremental.

### Deliverables
1. Updated controllers/services/interfaces/entities/configuration as needed.
2. Migrations if schema changes are required.
3. At least:
   - unit tests for state transition and timeout behavior,
   - integration tests for SOS -> dispatch -> accept flow,
   - integration tests for snapshot contract.
4. Updated docs:
   - `live-tracking.sourcecode.md`
   - `live-tracking.usageguide.md`
   - `rescue-trigger.sourcecode.md` if flow contract changes.

### Validation checklist
1. Manual raise-range now triggers actual dispatch actions.
2. Unauthorized hub clients cannot impersonate rescuer IDs.
3. Location publish updates persisted data used by dispatch queries.
4. Snapshot endpoint supports reconnect/resume map state.
5. Existing emergency flow regression tests pass.

