---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: roadmap
status: proposed
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-implemented-and-reported-runtime-behavior
---

# Consultation EndCall SignalR Roadmap

## Current Status Snapshot

- module status: `In Progress`
- chosen event name: `RoomExpiring`
- timeout backend signal: `Implemented`
- manual-end backend signal in current workspace: `Implemented`
- Flutter `RoomExpiring` handling: `Implemented`
- end-to-end manual-end expert auto-leave: `Not trusted`

## Current Truth To Resume From

This roadmap is written so work can resume from zero context.

Current verified state:

- backend timeout flows already emit `RoomExpiring`
- backend manual-end service path emits `RoomExpiring` in the current checkout
- backend manual-end service path attempts LiveKit room deletion in the current checkout
- Flutter already interprets `RoomExpiring` as a forced termination trigger
- member and expert active-call flows use the same video consultation screen

Reported but not fully code-verified-from-this-checkout:

- a runtime edge case exists where manual end may trigger `RoomExpiring`, but expert still does not automatically leave the room

## Target Outcome

After this work is complete:

1. timeout flow emits `RoomExpiring` consistently
2. manual-end flow also emits `RoomExpiring`
3. Flutter member client auto-leaves active call on `RoomExpiring`
4. Flutter expert client auto-leaves active call on `RoomExpiring`
5. the event contract remains single-name and single-path

## Locked Decisions

- [x] Keep `RoomExpiring` as the single consultation termination event
- [x] Do not introduce `ConsultationCallTerminated`
- [x] Treat Flutter’s existing `RoomExpiring` handling as the base contract
- [x] Prioritize backend manual-end emission before deeper expert-side diagnosis

## Implementation Checklist

### Phase 1. Backend Manual-End Alignment

- [x] Add SignalR emission to `ConsultationService.EndConsultationAsync(...)`
- [x] Use event name `RoomExpiring`
- [x] Reuse consultation group `consultation:{consultationId}`
- [x] Choose and document the manual-end `reason` value: `participant_ended`
- [ ] Decide exact ordering:
  - signal before room delete
  - signal before or after DB commit

### Phase 2. Backend Room Shutdown

- [x] Add LiveKit room shutdown to manual-end flow
- [ ] Keep timeout/manual-end ordering consistent enough for debugging
- [ ] Confirm idempotency when consultation is already `Completed`

### Phase 3. Backend Tests

- [x] Integration test manual-end emits `RoomExpiring`
- [x] Integration test manual-end attempts room deletion
- [ ] Unit test manual-end preserves chosen operation order
- [ ] Unit test manual-end still completes business state when SignalR send fails
- [ ] Unit test manual-end still behaves safely when room deletion fails

### Phase 4. Mobile Verification

- [ ] Verify member active-call screen receives manual-end-triggered `RoomExpiring`
- [ ] Verify expert active-call screen receives manual-end-triggered `RoomExpiring`
- [ ] Verify both modes disconnect the room and navigate away
- [ ] Verify duplicate delivery does not break due to `_isHandlingRoomExpiry`

### Phase 5. Edge Case Resolution

- [ ] Reproduce the reported expert-not-auto-leaving issue after backend manual-end emission exists in code
- [ ] Determine whether failure is in:
  - backend event delivery
  - consultation group membership
  - expert active-call subscription state
  - room disconnect timing
  - navigation state
- [ ] Fix the actual failing layer

### Phase 6. Documentation Sync

- [x] Update `useguide` after backend manual-end signal becomes active
- [ ] Update `sourcecode` diagrams after final order is implemented
- [ ] Record final verified runtime behavior for both member and expert

## Candidate File Targets

### Backend

- [ ] `SnakeAid.Service/Interfaces/IConsultationRealtimeNotifier.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationRealtimeNotifier.cs`
- [x] `SnakeAid.Service/Implements/ConsultationService.cs`
- [ ] `SnakeAid.Service/Implements/BookingService.cs`
- [ ] `SnakeAid.Tests/Unit/RoomCleanupTests.cs`
- [x] `SnakeAid.Tests/Integration/ScheduledConsultationIntegrationTests.cs`

### Mobile

- [ ] `lib/core/services/consultation_chat_signalr_service.dart`
- [ ] `lib/features/consultation/screens/members/video_consultation_screen.dart`
- [ ] related expert entry flow files only if verification shows a real expert-specific gap

### Docs

- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.introduction.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.roadmap.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.sourcecode.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-endcall-signalr/consultation-endcall-signalr.useguide.md`

## Verification Strategy

Minimum verification before calling the work complete:

1. scheduled timeout:
   - `RoomExpiring` sent
   - room closes
   - consultation state completes
2. emergency timeout:
   - `RoomExpiring` sent
   - room closes
   - consultation state completes
3. manual end:
   - `RoomExpiring` sent
   - room closes
   - consultation state completes
4. Flutter member active call:
   - receives event
   - disconnects room
   - navigates away
5. Flutter expert active call:
   - receives event
   - disconnects room
   - navigates away

## Open Questions

1. Should manual end signal before commit or after commit?
2. Should timeout and manual-end room shutdown ordering be normalized further?
3. After the backend patch, does the expert-not-leaving issue still reproduce?

## Change Log

### 2026-04-15

- Rewrote roadmap to preserve current truth for resume-from-scratch work
- Locked the direction to upgrade `RoomExpiring`
- Recorded that manual-end backend emission and room deletion are now implemented in the current workspace
- Preserved the reported expert-not-auto-leaving runtime issue as a verification target
