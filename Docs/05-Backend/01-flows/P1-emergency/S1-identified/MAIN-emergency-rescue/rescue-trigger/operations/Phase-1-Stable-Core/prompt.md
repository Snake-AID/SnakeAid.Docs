---
doc_role: operation
operation_id: phase-1-stable-core
generated_from: plan.md
status: done
created_at: 2026-02-15
---

# Agent Prompt: Rescue Trigger Implementation (Phase 1)

Roadmap reference:
- `../emergency-rescue.roadmap.md`
- `rescue-trigger.plan.md`

## Prompt


## Prompt

You are improving SnakeAid rescue-trigger flow. Keep existing API contracts stable unless explicitly documented.

Before writing code, read:
- `rescue-trigger.sourcecode.md` -> section `Function Implementation Status (Agent Guardrail)`.
- Rule: prefer fixing/extending existing functions instead of creating duplicate flows.

### Select execution phase first
1. `RT-1`: dispatch core stabilization.
2. `RT-2`: dynamic locator switch to Redis-first.

Do not mix both phases in one PR unless explicitly requested.

### Primary goals
1. For `RT-1`:
   - Make manual `raise-range` path equivalent to timeout-driven expansion.
   - Harden hub authorization and rescuer identity validation.
   - Emit request-expired notifications in timeout flow.
2. For `RT-2`:
   - Introduce `Redis-first` candidate lookup with `PostGIS-fallback`.
   - Add locator abstraction and rollout safety controls.

### Non-goals
1. Do not redesign payment flows.
2. Do not rewrite mission business domain outside rescue-trigger scope.
3. Do not introduce breaking changes in `ApiResponse` envelope.

### Required code constraints
1. Keep status transition semantics for:
   - `SnakebiteIncident`
   - `RescueRequestSession`
   - `RescuerRequest`
2. Keep race-winning acceptance logic intact.
3. Keep timeout scheduler observable via monitoring endpoints.

### Required tasks
1. If implementing `RT-1`:
   - Refactor `RaiseSessionRangeAsync` to use session service orchestration for:
     - session creation
     - request broadcast
     - timeout scheduling
   - Add authorization policy to `RescuerHub` and claim-based rescuer identity checks.
   - Add timeout expiration notification fanout.
2. If implementing `RT-2`:
   - Create locator interface (for example `IRescuerLocator`).
   - Implement Redis strategy and PostGIS fallback strategy.
   - Add feature flag and telemetry for safe rollout.

### Testing requirements
1. `RT-1` test minimum:
   - manual raise-range flow
   - unauthorized/mismatched hub action
   - timeout-expired notifications
   - accept-race winner behavior
2. `RT-2` test minimum:
   - Redis hit and PostGIS fallback behavior
   - candidate ordering consistency
   - feature flag switch behavior

### Documentation requirements
After implementation, update:
1. `rescue-trigger.sourcecode.md`
2. `rescue-trigger.usageguide.md`
3. `rescue-trigger.introduction.md` if scope/constraints changed
4. `live-tracking.sourcecode.md` if tracking contract dependency changed
