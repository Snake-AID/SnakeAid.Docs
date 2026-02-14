---
doc_role: operation
operation_id: phase-2-ingestion-foundation
generated_from: plan.md
status: done
created_at: 2026-02-15
---

# Agent Prompt: Live Tracking Implementation (Phase 2)

Use this prompt when implementing live-tracking tasks in `emergency-rescue` roadmap.

Roadmap reference:
- `../emergency-rescue.roadmap.md`
- `live-tracking.plan.md`

## Prompt

You are implementing Live Tracking for SnakeAid backend on top of existing rescue-trigger flow.

### Goal
Deliver production-safe tracking behavior without breaking current rescue dispatch.

### Select execution phase first
1. `LT-1`: ingestion foundation.
2. `LT-2`: full tracking pipeline.

Do not mix both phases in one PR unless explicitly requested.

### Must preserve
1. `POST /api/incidents/sos` existing behavior and response envelope.
2. Session timeout and acceptance race semantics.
3. Existing entity state transitions for incident/session/request/mission.

### Mandatory fixes in this iteration
1. If implementing `LT-1`:
   - Persist location updates to `RescuerProfile.LastLocation` and `LastLocationUpdate`.
   - Enforce identity-safe location writes from authenticated rescuer.
   - Add validation/throttle/stale handling for location ingestion.
2. If implementing `LT-2`:
   - Add session viewer realtime stream.
   - Add snapshot/history read APIs.
   - Add Redis NOW-state and fallback delivery integration.

### Nice-to-have if scope allows
1. Add richer telemetry on location publish rate and stream lag.
2. Add retention tooling for history compaction.
3. Add graceful degraded mode when Redis is unavailable.

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
   - `LT-1`: tests for location persistence, validation, identity checks.
   - `LT-2`: tests for snapshot/history and viewer stream behavior.
4. Updated docs:
   - `live-tracking.introduction.md`
   - `live-tracking.sourcecode.md`
   - `live-tracking.usageguide.md`
   - `rescue-trigger.sourcecode.md` if dispatch dependency changed.

### Validation checklist
1. `LT-1`: dispatch can query fresh persisted location.
2. `LT-1`: unauthorized clients cannot write location for other rescuers.
3. `LT-2`: snapshot + stream supports reconnect/resume map state.
4. `LT-2`: patient/admin viewer receives session location updates.
5. Existing emergency flow regression tests pass.
