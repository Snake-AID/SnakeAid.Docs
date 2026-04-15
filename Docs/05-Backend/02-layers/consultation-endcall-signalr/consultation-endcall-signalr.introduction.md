---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: introduction
status: proposed
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-and-target-design
---

# Consultation EndCall SignalR Introduction

## Goal

This module defines the implementation plan for evolving the existing `RoomExpiring` consultation SignalR contract so it becomes the single forced-endcall trigger for Flutter.

- when a consultation reaches its time limit, the backend should emit `RoomExpiring` to both `member` and `expert`; Flutter will receive that push and perform `endcall`, then the backend will close the LiveKit room
- when a participant calls `POST /api/consultations/{consultationId}/end`, the backend should also emit `RoomExpiring`; Flutter should receive that push and perform `endcall`, then the backend should close the LiveKit room

Backend goals:

- provide one stable event contract for Flutter to consume
- avoid splitting call-termination logic across `BookingService`, `ConsultationService`, and the hub
- preserve a clear operation order: `broadcast -> Flutter endcall/leave room -> backend room shutdown`

## Current Codebase Status

The current codebase already contains the required building blocks:

- `ConsultationHub` is mapped at `"/hubs/consultation"`
- the consultation SignalR group format is `consultation:{consultationId}`
- the LiveKit room name format is `consultation-{consultationId}`
- `POST /api/consultations/{consultationId}/end` already exists and calls `ConsultationService.EndConsultationAsync(...)`
- `POST /api/consultations/{consultationId}/video-token` already exists so participants can join the LiveKit room
- `ConsultationLifecycleBackgroundService` runs a sweep every `30s`
- both scheduled-timeout and emergency-timeout flows in `BookingService` already send `RoomExpiring` before `DeleteRoomAsync(...)`

## Code-Verified Current Behavior

### Timeout Flow

The current timeout methods are:

- `BookingService.AutoCompleteElapsedScheduledConsultationsAsync(...)`
- `BookingService.AutoCompleteElapsedEmergencyConsultationsAsync(...)`

Both currently do the following:

1. `SendAsync("RoomExpiring", { consultationId, reason = "slot_elapsed" })` to the consultation group
2. `DeleteRoomAsync("consultation-{consultationId}")`
3. update DB state to `Completed`
4. `CommitAsync()`
5. settle escrow

The current behavior is unit-test verified:

- the SignalR send happens before `CommitAsync`
- `DeleteRoomAsync` happens before `CommitAsync`
- the current `RoomExpiring` payload only includes `ConsultationId` and `Reason`

### Manual End Flow

`ConsultationService.EndConsultationAsync(...)` currently:

- verifies the actor is either `Caller` or `Callee`
- sets `Consultation.Status = Completed`
- sets `Consultation.EndTime = UtcNow`
- if a booking exists, sets `BookingStatus = Completed`
- if a reserved slot exists, changes `TimeSlotStatus = Booked`
- calls `CommitAsync()`
- calls `SettleConsultationEscrowAsync(...)`

This flow currently does not:

- emit a SignalR event announcing call termination
- close the LiveKit room
- align its realtime contract with the timeout flow

## Problem Statement

Current gaps:

- the timeout flow already has a usable SignalR event, but its payload is still too thin for the long-term Flutter endcall trigger contract
- the manual-end flow needs to reuse `RoomExpiring` instead of introducing a second event name
- consultation termination broadcasting is implemented directly inside `BookingService` and is not reusable from `ConsultationService`
- there is an observed integration edge case: when `POST /api/consultations/{consultationId}/end` triggers `RoomExpiring`, the expert does not automatically leave the room in practice

## Proposed Direction

Recommended implementation direction:

1. Create a shared abstraction, for example:
   - `IConsultationRealtimeNotifier`
   - `ConsultationRealtimeNotifier`
2. Keep `RoomExpiring` as the single server-to-client forced-endcall trigger, because Flutter already treats it as a termination signal today
3. Upgrade the `RoomExpiring` payload only where it adds real value:
   - `consultationId`
   - `reason`
4. Both the timeout path and the manual-end path should call the same notifier because both pushes are intended to make Flutter perform `endcall`
5. The timeout path should preserve the order `broadcast -> delete room -> update status`
6. The manual-end path should send SignalR before the room is closed, then delete the room and commit state

## Observed Integration Finding

Observed finding from current integration behavior:

- when a user calls `POST /api/consultations/{consultationId}/end`, SignalR may emit `RoomExpiring`, but the expert still does not automatically leave the room

Current interpretation:

- Flutter already treats `RoomExpiring` as a hard termination trigger in the live consultation screen
- therefore the main risk is not event naming anymore
- the main risk is end-to-end delivery and handling consistency for the manual-end path, especially on the expert side

## Assumptions To Lock During Implementation

To avoid ambiguity, this module uses the following assumptions:

- `RoomExpiring` remains the only forced-endcall event for consultation termination
- Flutter should not need a second event name for the same endcall action
- the current manual-end expert-not-leaving issue is treated as an implementation/integration bug, not as evidence that a new event name is required

## Scope Boundary

In scope:

- timeout signals for scheduled and emergency consultations
- manual-end signal for the existing `POST /api/consultations/{consultationId}/end`
- LiveKit room shutdown after broadcast
- test coverage for ordering, payload, and resiliency
- backend and mobile-facing documentation

Out of scope:

- changing settlement or pricing business rules
- changing `ConsultationHub` authentication
- changing Flutter UI implementation
- adding a new endpoint if the existing one is sufficient

## Delivered Artifacts

- `consultation-endcall-signalr.introduction.md`
- `consultation-endcall-signalr.roadmap.md`
- `consultation-endcall-signalr.sourcecode.md`
- `consultation-endcall-signalr.useguide.md`
