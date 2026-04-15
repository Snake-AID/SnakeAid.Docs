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
- current timeout payload standardization: `Partial`
- current Flutter endcall contract: `Not formalized in backend docs`

## Target Outcome

After implementation:

1. consultation timeout for both scheduled and emergency consultations will broadcast one clear termination event to both participants, and Flutter will use that push to perform `endcall`
2. `POST /api/consultations/{consultationId}/end` will broadcast a termination event before the room is closed, and Flutter will use that push to perform `endcall`
3. Flutter will be able to listen on `ConsultationHub` and perform:
   - end call
   - leave LiveKit room
   - transition the user to the correct post-call UI state

## Recommended Contract Decision

Recommended decision:

- use one shared server-to-client event name:
  - `ConsultationCallTerminated`
- distinguish the cause through payload values:
  - `reason = "timeout"` for auto-complete
  - `reason = "participant_ended"` for manual end
- also send:
  - `endedByUserId`
  - `endedByRole`
  - `shouldLeaveCall`

Reasoning:

- Flutter does not need to branch on multiple event names
- timeout flow and manual-end flow can use the same parser because both pushes drive the same endcall action
- UI wording can remain a client concern instead of being hard-coded in the backend

## Implementation Checklist

### Phase 1. Lock Contract

- [ ] Finalize the server-to-client event name
- [ ] Finalize enum/string values for `reason`
- [ ] Finalize the payload fields required for Flutter endcall
- [ ] Decide whether to keep `RoomExpiring` for backward compatibility
- [ ] Finalize the operation order for timeout and manual-end flows

### Phase 2. Shared Realtime Abstraction

- [ ] Create `IConsultationRealtimeNotifier`
- [ ] Create an implementation that emits to group `consultation:{consultationId}`
- [ ] Move payload creation out of `BookingService`
- [ ] Add consistent logging for timeout and manual-end signals

### Phase 3. Timeout Flow

- [ ] Refactor `AutoCompleteElapsedScheduledConsultationsAsync(...)`
- [ ] Refactor `AutoCompleteElapsedEmergencyConsultationsAsync(...)`
- [ ] Broadcast the termination event to both participants
- [ ] Preserve `DeleteRoomAsync(...)` in the timeout path
- [ ] Preserve best-effort behavior when SignalR send fails

### Phase 4. Manual End Flow

- [ ] Refactor `ConsultationService.EndConsultationAsync(...)`
- [ ] Broadcast the termination event to the consultation group
- [ ] Delete the LiveKit room after broadcast
- [ ] Keep the flow idempotent when the consultation is already `Completed`
- [ ] Preserve existing business rules for booking, slot, and settlement

### Phase 5. Tests

- [ ] Unit test that timeout flow emits the new event with the correct payload
- [ ] Unit test that timeout flow preserves `signal -> delete room -> commit`
- [ ] Unit test that manual-end flow emits the event before room shutdown
- [ ] Unit test that manual-end flow still completes DB state when SignalR send fails
- [ ] Unit test that manual-end flow still closes the room when event send fails
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
- [ ] `SnakeAid.Core/Responses/Consultation/ConsultationCallTerminationSignal.cs`
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
   - event is sent to the consultation group
   - room is closed
   - consultation status is correct
   - booking and slot statuses are correct for scheduled consultations

## Open Questions

1. Should the backend emit the same termination event to an admin participant if an admin joined the room through the video-token endpoint?
2. Does Flutter need explicit values such as `member_ended` and `expert_ended`, or is `participant_ended` plus `endedByRole` sufficient when both pushes are used only to trigger `endcall`?
3. Should the backend emit both `RoomExpiring` and `ConsultationCallTerminated` during a rollout period to avoid breaking older clients?

## Change Log

### 2026-04-15

- Initialized the roadmap for consultation end-call SignalR flow
- Captured the code-verified current state of timeout and manual-end flows
- Proposed a shared termination event for Flutter
