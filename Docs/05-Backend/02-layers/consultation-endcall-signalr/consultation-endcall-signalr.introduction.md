---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: introduction
status: proposed
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-and-reported-runtime-behavior
---

# Consultation EndCall SignalR Introduction

## Goal

This module defines the implementation direction for consultation end-call signaling.

The chosen direction is:

- keep `RoomExpiring` as the single consultation termination SignalR event
- upgrade `RoomExpiring` so it is used by both timeout flow and manual-end flow
- do not introduce a second event such as `ConsultationCallTerminated`

Target user-visible behavior:

- when timeout ends a consultation, backend emits `RoomExpiring`
- when `POST /api/consultations/{consultationId}/end` ends a consultation, backend also emits `RoomExpiring`
- Flutter receives the push, disconnects the active call, and navigates to the completion flow

## Resume Summary

If this work must be resumed later without any memory of prior discussion, the current situation is:

1. Flutter already treats `RoomExpiring` as a forced call-termination event.
2. Timeout cleanup in backend already emits `RoomExpiring`.
3. In the current backend workspace, manual end does **not** emit `RoomExpiring`.
4. A reported runtime edge case exists:
   - when a user calls `{consultationId}/end`, SignalR may emit `RoomExpiring`, but the expert still does not automatically leave the room
5. Because the checked-out backend code does not yet emit `RoomExpiring` for manual end, the first implementation priority is to add that backend path and then re-verify the reported expert-side issue.

## Code-Verified Workspace Status

### Backend

The current backend workspace contains:

- `ConsultationHub` mapped at `"/hubs/consultation"`
- consultation SignalR group format: `consultation:{consultationId}`
- LiveKit room name format: `consultation-{consultationId}`
- `POST /api/consultations/{consultationId}/end`
- `POST /api/consultations/{consultationId}/video-token`
- `ConsultationLifecycleBackgroundService`

Code-verified timeout behavior:

- `BookingService.AutoCompleteElapsedScheduledConsultationsAsync(...)` emits `RoomExpiring`
- `BookingService.AutoCompleteElapsedEmergencyConsultationsAsync(...)` emits `RoomExpiring`
- both timeout flows then delete the LiveKit room and complete business state

Code-verified manual-end behavior in the current workspace:

- `ConsultationService.EndConsultationAsync(...)` completes consultation state and settlement
- it does **not** emit `RoomExpiring`
- it does **not** close the LiveKit room

### Mobile

The current mobile workspace contains:

- `ConsultationChatSignalRService`
- typed event model `ConsultationRoomExpiringEvent`
- active-call subscription to `roomExpiringStream`
- shared active-call screen for both member and expert mode

Code-verified Flutter behavior:

- Flutter listens for direct `RoomExpiring`
- Flutter also maps `SignalReceived` with `roomexpiring` to the same handling path
- the active video consultation screen handles `RoomExpiring` by:
  - showing a snackbar
  - disconnecting the active room
  - calling backend `endConsultation(...)`
  - navigating to completion UI

This means Flutter already behaves as if `RoomExpiring` were the final consultation termination trigger.

## Reported Runtime Finding

There is one important reported runtime finding that must be preserved for future resume:

- when a user calls `{consultationId}/end`, SignalR may emit `RoomExpiring`, but the expert does not automatically leave the room

Status of this finding:

- `reported runtime behavior`
- `not fully reproducible from the current backend workspace alone`

Why this matters:

- the current backend checkout does not yet emit `RoomExpiring` in the manual-end service path
- therefore the runtime issue may belong to a deployed version, another branch, or an uncommitted local state
- the issue must still be tracked because it defines the next validation target after backend manual-end emission is implemented

## Problem Statement

The actual problem is now narrow and concrete:

1. Backend timeout flow already uses `RoomExpiring`.
2. Flutter already treats `RoomExpiring` as a hard end-call signal.
3. Manual-end backend path is not yet aligned with timeout flow in the checked-out code.
4. Even when manual-end-triggered `RoomExpiring` is reported to happen in runtime, expert auto-leave is not yet trusted end-to-end.

So the remaining work is not naming. The remaining work is implementation and verification.

## Chosen Direction

The current chosen direction is final unless explicitly changed later:

- keep event name: `RoomExpiring`
- reuse that event for:
  - timeout flow
  - manual-end flow
- keep Flutter on one parsing path
- investigate expert-not-leaving only after backend manual-end emission is code-complete

## Scope Boundary

In scope:

- manual-end `RoomExpiring` emission
- timeout/manual-end contract alignment
- room shutdown ordering
- expert/member active-call verification
- documentation that preserves workspace truth and runtime findings separately

Out of scope:

- renaming the event
- introducing `ConsultationCallTerminated`
- changing Flutter route structure unless verification proves it is necessary
- changing consultation business rules unrelated to termination signaling

## Delivered Artifacts

- `consultation-endcall-signalr.introduction.md`
- `consultation-endcall-signalr.roadmap.md`
- `consultation-endcall-signalr.sourcecode.md`
- `consultation-endcall-signalr.useguide.md`
