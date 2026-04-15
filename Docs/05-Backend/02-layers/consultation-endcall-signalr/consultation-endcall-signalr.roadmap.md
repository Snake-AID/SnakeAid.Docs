---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: roadmap
status: proposed
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-and-target-design
---

# Consultation EndCall SignalR Roadmap

## Current Status Snapshot

- module status: `Proposed`
- current timeout signal: `Implemented`
- current manual-end signal: `Missing`
- current strategic direction: `Upgrade RoomExpiring`
- current timeout payload standardization: `Partial`
- current Flutter endcall contract: `Not formalized in backend docs`

## Target Outcome

After implementation:

1. consultation timeout for both scheduled and emergency consultations will broadcast `RoomExpiring` to both participants, and Flutter will use that push to perform `endcall`
2. `POST /api/consultations/{consultationId}/end` will also broadcast `RoomExpiring` before the room is closed, and Flutter will use that push to perform `endcall`
3. Flutter will be able to listen on `ConsultationHub` and perform:
   - end call
   - leave LiveKit room
   - transition the user to the correct post-call UI state

## Recommended Contract Decision

Recommended decision:

- keep the existing server-to-client event name:
  - `RoomExpiring`
- distinguish the cause through payload values:
  - `reason = "timeout"` for auto-complete
  - `reason = "participant_ended"` for manual end

Reasoning:

- Flutter already has a working `RoomExpiring` handling path
- timeout flow and manual-end flow can use the same parser because both pushes drive the same endcall action
- a second event name would duplicate semantics without solving the observed expert-side manual-end issue

## Implementation Checklist

### Phase 1. Lock Contract

- [ ] Finalize the server-to-client event name
- [ ] Finalize enum/string values for `reason`
- [ ] Finalize the payload fields required for Flutter endcall
- [x] Keep `RoomExpiring` as the single event name
- [ ] Finalize the operation order for timeout and manual-end flows

### Phase 2. Shared Realtime Abstraction

- [ ] Create `IConsultationRealtimeNotifier`
- [ ] Create an implementation that emits to group `consultation:{consultationId}`
- [ ] Move payload creation out of `BookingService`
- [ ] Add consistent logging for timeout and manual-end signals

### Phase 3. Timeout Flow

- [ ] Refactor `AutoCompleteElapsedScheduledConsultationsAsync(...)`
- [ ] Refactor `AutoCompleteElapsedEmergencyConsultationsAsync(...)`
- [ ] Broadcast upgraded `RoomExpiring` to both participants
- [ ] Preserve `DeleteRoomAsync(...)` in the timeout path
- [ ] Preserve best-effort behavior when SignalR send fails

### Phase 4. Manual End Flow

- [ ] Refactor `ConsultationService.EndConsultationAsync(...)`
- [ ] Broadcast `RoomExpiring` to the consultation group
- [ ] Delete the LiveKit room after broadcast
- [ ] Keep the flow idempotent when the consultation is already `Completed`
- [ ] Preserve existing business rules for booking, slot, and settlement

### Phase 5. Tests

- [ ] Unit test that timeout flow emits the new event with the correct payload
- [ ] Unit test that timeout flow preserves `signal -> delete room -> commit`
- [ ] Unit test that manual-end flow emits the event before room shutdown
- [ ] Unit test that manual-end flow still completes DB state when SignalR send fails
- [ ] Unit test that manual-end flow still closes the room when event send fails
- [ ] End-to-end verify that the expert client automatically leaves the room after manual-end-triggered `RoomExpiring`
- [ ] Integration test `POST /api/consultations/{id}/end` for scheduled consultation
- [ ] Integration test `POST /api/consultations/{id}/end` for emergency consultation if the flow is allowed

### Phase 6. Documentation Sync

- [ ] Update `useguide` with the active contract after the code is merged
- [ ] Update `sourcecode` with the final class and sequence diagrams
- [ ] Update the module changelog
- [ ] Sync the final event contract with the Flutter team

## Candidate File Targets

### Backend

- [ ] `SnakeAid.Service/Interfaces/IConsultationRealtimeNotifier.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationRealtimeNotifier.cs`
- [ ] `SnakeAid.Service/Implements/BookingService.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationService.cs`
- [ ] `SnakeAid.Api/Program.cs`

### Tests

- [ ] `SnakeAid.Tests/Unit/RoomCleanupTests.cs`
- [ ] `SnakeAid.Tests/Integration/ScheduledConsultationIntegrationTests.cs`
- [ ] add a new manual-end SignalR test file if needed

### Docs

- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.introduction.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.roadmap.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.sourcecode.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.useguide.md`

## Verification Strategy

Minimum verification before closing the task:

1. scheduled consultation timeout:
   - event is sent to the correct group
   - payload is correct
   - room is closed
   - consultation, booking, and slot statuses are correct
2. emergency consultation timeout:
   - event is sent to the correct group
   - payload is correct
   - room is closed
   - consultation status is correct
3. manual end:
   - only a valid actor can end the consultation
   - `RoomExpiring` is sent to the consultation group
   - room is closed
   - consultation status is correct
   - booking and slot statuses are correct for scheduled consultations
   - expert client actually leaves the room after the event

## Open Questions

1. Does the backend manual-end path actually deliver `RoomExpiring` to every active participant connection in the consultation group?
2. Why does the expert client fail to auto-leave after the manual-end-triggered `RoomExpiring` in the observed edge case?
3. Is the issue caused by backend delivery, expert-screen subscription state, or room-disconnect timing?

## Change Log

### 2026-04-15

- Initialized the roadmap for consultation end-call SignalR flow
- Captured the code-verified current state of timeout and manual-end flows
- Chose to upgrade `RoomExpiring` instead of introducing a second event name
- Added the observed manual-end edge case where expert does not automatically leave the room
