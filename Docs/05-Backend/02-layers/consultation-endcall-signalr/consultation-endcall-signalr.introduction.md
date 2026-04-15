---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: introduction
status: in_progress
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-implemented-and-runtime-verification-pending
---

# Consultation EndCall SignalR Introduction

## Goal

This module now defines a full naming migration for consultation termination signaling across backend and Flutter.

The chosen direction is:

- stop using `RoomExpiring` as the business event name
- rename the canonical SignalR event to `ConsultationCallEnded`
- rename related backend and Flutter models, streams, handlers, and constants to match that business meaning
- perform a coordinated full migration across backend and Flutter
- do not keep a backward-compatibility alias or dual-event rollout

Target user-visible behavior remains unchanged:

- when timeout ends a consultation, backend emits `ConsultationCallEnded`
- when `POST /api/consultations/{consultationId}/end` ends a consultation, backend also emits `ConsultationCallEnded`
- Flutter receives the push, disconnects the active call, and navigates to the completion flow

## Resume Summary

If this work must be resumed later without any memory of prior discussion, the current situation is:

1. The backend and Flutter code in the current workspace now use `ConsultationCallEnded` as the active termination event name.
2. The current business behavior remains a hard termination flow rather than a warning flow.
3. Timeout reason has been normalized to `timeout`.
4. Manual-end reason remains `participant_ended`.
5. This was implemented as a coordinated full migration with no backward path.
6. A reported runtime edge case still exists:
   - after manual-end-triggered termination signaling, expert may still fail to auto-leave the room

## Code-Verified Current Workspace Status

### Backend

The current backend workspace contains:

- `ConsultationHub` mapped at `"/hubs/consultation"`
- consultation SignalR group format: `consultation:{consultationId}`
- LiveKit room name format: `consultation-{consultationId}`
- `POST /api/consultations/{consultationId}/end`
- `POST /api/consultations/{consultationId}/video-token`
- `ConsultationLifecycleBackgroundService`

Code-verified current naming:

- timeout flows emit `ConsultationCallEnded`
- manual end emits `ConsultationCallEnded`
- manual-end reason value is `participant_ended`
- timeout reason value is `timeout`

### Mobile

The current mobile workspace contains:

- `ConsultationChatSignalRService`
- typed event model `ConsultationCallEndedEvent`
- active-call subscription to `consultationCallEndedStream`
- handler `_handleConsultationCallEndedEvent(...)`
- shared active-call screen for both member and expert mode

Code-verified Flutter behavior:

- Flutter listens for direct `ConsultationCallEnded`
- Flutter also maps `SignalReceived` with `roomexpiring` to the same handling path
- the active video consultation screen handles the event as a forced termination flow

This means the business behavior and naming are now aligned in code.

## Naming Problem Statement

`RoomExpiring` was no longer an accurate business name.

Why it is incorrect:

- it sounds like a pre-expiry warning
- it is room-centric instead of consultation-centric
- it hides the fact that Flutter immediately leaves the call
- it does not read cleanly in backend APIs, DTOs, streams, and handler names

The business meaning is:

- the consultation call has ended
- the client must immediately endcall and leave the LiveKit room

So the canonical name should reflect end-state, not warning-state.

## Chosen Canonical Naming

The current chosen direction is final unless explicitly changed later:

- canonical SignalR event name: `ConsultationCallEnded`
- canonical backend DTO/model name: `ConsultationCallEndedEvent`
- canonical Flutter model name: `ConsultationCallEndedEvent`
- canonical Flutter stream name: `consultationCallEndedStream`
- canonical Flutter handler name: `_handleConsultationCallEndedEvent(...)`
- canonical reason type: `ConsultationCallEndReason`

Canonical reason values:

- `timeout`
- `participant_ended`

This normalization has been implemented in the current workspace.

## Migration Strategy

This has been implemented as a coordinated full migration.

Rules:

- backend and Flutter were updated in the same implementation wave
- there is no dual-event emission
- there is no compatibility alias for `RoomExpiring`
- there is no mixed contract where backend emits one name and Flutter listens for another

Required outcome:

- backend emits only `ConsultationCallEnded`
- Flutter listens only for `ConsultationCallEnded`
- internal class names and handler names follow the same business naming

## Reported Runtime Finding

The reported runtime finding must still be preserved:

- when manual-end-triggered termination signaling happens, expert may still fail to automatically leave the room

Status of this finding:

- `reported runtime behavior`
- `not yet re-verified after the upcoming naming migration`

Why this matters:

- renaming should not be treated as purely cosmetic
- the end-to-end behavior still requires member and expert verification after the migration

## Scope Boundary

In scope:

- SignalR event rename from `RoomExpiring` to `ConsultationCallEnded`
- backend DTO and notifier naming alignment
- Flutter model/stream/handler naming alignment
- reason value cleanup from `slot_elapsed` to `timeout`
- docs that preserve current state and target migration state separately
- end-to-end verification after rename

Out of scope:

- keeping backward compatibility
- dual publishing two event names
- changing consultation business rules unrelated to termination signaling
- changing Flutter route structure unless verification proves it is necessary

## Delivered Artifacts

- `consultation-endcall-signalr.introduction.md`
- `consultation-endcall-signalr.roadmap.md`
- `consultation-endcall-signalr.sourcecode.md`
- `consultation-endcall-signalr.useguide.md`
