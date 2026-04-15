---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: introduction
status: proposed
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-and-target-migration-plan
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

1. The current backend and Flutter code still use `RoomExpiring`.
2. The current business behavior already matches a hard termination flow rather than a warning flow.
3. Manual end is already implemented in backend and currently emits `RoomExpiring`.
4. The new task is not behavior expansion first. The new task is coordinated naming migration.
5. The migration must be applied across backend and Flutter in one compatible release because no backward path will be kept.
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

- timeout flows emit `RoomExpiring`
- manual end emits `RoomExpiring`
- manual-end reason value is currently `participant_ended`
- timeout reason value is currently `slot_elapsed`

### Mobile

The current mobile workspace contains:

- `ConsultationChatSignalRService`
- typed event model `ConsultationRoomExpiringEvent`
- active-call subscription to `roomExpiringStream`
- handler `_handleRoomExpiringEvent(...)`
- shared active-call screen for both member and expert mode

Code-verified Flutter behavior:

- Flutter listens for direct `RoomExpiring`
- Flutter also maps `SignalReceived` with `roomexpiring` to the same handling path
- the active video consultation screen handles the event as a forced termination flow

This means the business behavior is already correct, but the naming is not.

## Naming Problem Statement

`RoomExpiring` is no longer an accurate business name.

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

This means `slot_elapsed` should also be normalized during migration.

## Migration Strategy

This is a coordinated full migration.

Rules:

- backend and Flutter must be updated in the same implementation wave
- no dual-event emission
- no compatibility alias for `RoomExpiring`
- no mixed contract where backend emits one name and Flutter listens for another

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
