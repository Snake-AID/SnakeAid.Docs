---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: roadmap
status: in_progress
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-implemented-and-runtime-verification-pending
---

# Consultation EndCall SignalR Roadmap

## Current Status Snapshot

- module status: `In Progress`
- current backend event name: `ConsultationCallEnded`
- current Flutter event model: `ConsultationCallEndedEvent`
- target backend and Flutter event name: `ConsultationCallEnded`
- migration mode: `Full migration without backward compatibility`
- end-to-end expert auto-leave: `Not trusted`

## Current Truth To Resume From

This roadmap is written so work can resume from zero context.

Current verified state:

- backend timeout flows emit `ConsultationCallEnded`
- backend manual-end flow emits `ConsultationCallEnded`
- backend manual-end flow attempts LiveKit room deletion
- Flutter interprets `ConsultationCallEnded` as a forced termination trigger
- member and expert active-call flows use the same video consultation screen

Reported but not fully closed:

- a runtime edge case exists where manual-end-triggered termination signaling may still fail to auto-remove expert from the room

## Target Outcome

After this work is complete:

1. timeout flow emits `ConsultationCallEnded`
2. manual-end flow emits `ConsultationCallEnded`
3. backend no longer emits `RoomExpiring`
4. Flutter no longer listens for `RoomExpiring`
5. Flutter member client auto-leaves active call on `ConsultationCallEnded`
6. Flutter expert client auto-leaves active call on `ConsultationCallEnded`
7. reason values are normalized to business terms

## Locked Decisions

- [x] Do full migration from `RoomExpiring`
- [x] Use canonical event name `ConsultationCallEnded`
- [x] Do not keep backward compatibility
- [x] Rename both backend and Flutter internals to match the new contract
- [x] Normalize timeout reason naming to `timeout`

## Implementation Checklist

### Phase 1. Backend Naming Migration

- [x] Rename SignalR event from `RoomExpiring` to `ConsultationCallEnded`
- [x] Rename event constants in backend realtime classes
- [x] Normalize active backend contract around `ConsultationCallEnded`
- [x] Normalize timeout reason from `slot_elapsed` to `timeout`
- [x] Preserve manual-end reason as `participant_ended`

### Phase 2. Flutter Naming Migration

- [x] Rename `ConsultationRoomExpiringEvent` to `ConsultationCallEndedEvent`
- [x] Rename `roomExpiringStream` to `consultationCallEndedStream`
- [x] Rename `_handleRoomExpiringEvent(...)` to `_handleConsultationCallEndedEvent(...)`
- [x] Replace event parsing from `RoomExpiring` to `ConsultationCallEnded`
- [x] Replace `SignalReceived` event type mapping from `roomexpiring` to the new event name where still used

### Phase 3. Backend Tests

- [x] Update backend tests to assert `ConsultationCallEnded`
- [x] Update payload assertions for `reason = "timeout"` where applicable
- [ ] Keep manual-end integration coverage
- [ ] Verify room deletion behavior still passes after rename

### Phase 4. Mobile Verification

- [ ] Verify member active-call screen receives `ConsultationCallEnded`
- [ ] Verify expert active-call screen receives `ConsultationCallEnded`
- [ ] Verify both modes disconnect the room and navigate away
- [ ] Verify duplicate delivery does not break due to the in-screen guard

### Phase 5. Edge Case Resolution

- [ ] Reproduce the reported expert-not-auto-leaving issue after full rename
- [ ] Determine whether failure is in:
  - backend event delivery
  - consultation group membership
  - expert active-call subscription state
  - room disconnect timing
  - navigation state
- [ ] Fix the actual failing layer

### Phase 6. Documentation Sync

- [x] Update `useguide` to remove `RoomExpiring` as active contract
- [x] Update `sourcecode` diagrams to the renamed event
- [ ] Record final verified runtime behavior for both member and expert

## Candidate File Targets

### Backend

- [ ] `SnakeAid.Service/Hubs/ConsultationRealtimeEvents.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationService.cs`
- [ ] `SnakeAid.Service/Implements/BookingService.cs`
- [ ] related hub/notifier abstractions if present
- [ ] `SnakeAid.Tests/Integration/ScheduledConsultationIntegrationTests.cs`

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

Minimum verification before calling the migration complete:

1. scheduled timeout:
   - `ConsultationCallEnded` sent
   - room closes
   - consultation state completes
2. emergency timeout:
   - `ConsultationCallEnded` sent
   - room closes
   - consultation state completes
3. manual end:
   - `ConsultationCallEnded` sent
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
6. no active runtime dependency remains on `RoomExpiring`

## Open Questions

1. Should backend expose the renamed event through a dedicated notifier abstraction now or later?
2. Should `SignalReceived` remain as an indirection path on Flutter, or should the hub send the direct event only?
3. After the full rename, does the expert-not-leaving issue still reproduce?

## Change Log

### 2026-04-15

- Replaced the previous `RoomExpiring` contract with `ConsultationCallEnded` in backend and Flutter code
- Kept backward compatibility disabled
- Normalized timeout reason to `timeout`
- Preserved the reported expert-not-auto-leaving runtime issue as a post-migration verification target
