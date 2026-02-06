# Rescue Trigger Prompt

Use this prompt when implementing the next rescue-trigger refinement.

## Prompt

You are improving SnakeAid rescue-trigger flow. Keep existing API contracts stable unless explicitly documented.

### Primary goals
1. Make manual `raise-range` path equivalent to timeout-driven expansion.
2. Harden hub authorization and rescuer identity validation.
3. Persist rescuer live location updates for dispatch matching.
4. Emit request-expired notifications in timeout flow.

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
1. Refactor `RaiseSessionRangeAsync` to use session service orchestration for:
   - session creation
   - request broadcast
   - timeout scheduling
2. Add authorization policy to `RescuerHub` and claim-based rescuer identity checks.
3. Implement persistence in `UpdateLocation` path.
4. Add timeout expiration notification fanout.

### Testing requirements
1. Add/extend tests for manual raise-range flow.
2. Add tests for unauthorized/mismatched rescuer hub actions.
3. Add tests for timeout-expired notifications.
4. Add regression test for accept-race winner behavior.

### Documentation requirements
After implementation, update:
1. `rescue-trigger.sourcecode.md`
2. `rescue-trigger.usageguide.md`
3. `live-tracking.sourcecode.md` (if tracking contract changes)

