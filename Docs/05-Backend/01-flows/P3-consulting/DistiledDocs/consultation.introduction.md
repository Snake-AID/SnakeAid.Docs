---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-04-17
owners: [backend-team]
---

# Consultation Module Introduction

## Purpose

This folder is the source-of-truth reference for the implemented consultation module in `SnakeAid.Backend`.
It is intentionally limited to behavior that is verified in code.

## Supported Flows

- Expert directory and profile lookup
- Expert availability setup and slot generation
- Scheduled consultation booking
- Emergency consultation request
- Consultation payment by `WalletBalance` or `PayOs`
- LiveKit token generation for consultation rooms
- In-room chat and lightweight UI signaling over SignalR
- Consultation completion and post-consultation review
- Member, expert, and admin consultation history APIs

## Current Business Shape

- A scheduled booking creates both a `ConsultationBooking` and a `Consultation` immediately.
- Scheduled bookings start in `BookingStatus.PendingPayment`.
- The scheduled consultation record starts in `ConsultationStatus.Scheduled`.
- An emergency request starts in `ConsultationPingStatus.PendingPayment`.
- Paying an emergency request moves it to `PendingExpertResponse` and pushes a SignalR notification to the selected online expert.
- Accepting an emergency request creates a `Consultation` with `Type = Emergency` and `Status = Ongoing`.
- Rejecting or expiring a paid emergency request triggers refund logic through consultation payment service.
- Ending a consultation or background auto-completion triggers settlement logic for consultation escrow.

## Hard Rules Verified In Code

- Scheduled booking requires an available future `ExpertTimeSlot`.
- Emergency payment is blocked if the selected expert is offline in SignalR presence memory.
- Emergency request TTL is 2 minutes.
- Scheduled booking payment deadline is 15 minutes from booking creation.
- Consultation room name is `consultation-{consultationId}`.
- Consultation hub access is limited to consultation participants.
- Chat messages in `ConsultationHub` are rate-limited to 10 messages per minute per user.
- Expert acceptance of an emergency consultation reserves overlapping available slots for the next 30 minutes.

## Realtime Surface

- `ExpertHub` at `/hubs/expert` handles presence and emergency request routing.
- `ConsultationHub` at `/hubs/consultation` handles in-room chat and signaling.
- The implemented room-end event is `ConsultationCallEnded`.
- Timeout cleanup emits reason `timeout`.
- Manual end emits reason `participant_ended`.
- There is no implemented `RoomExpiring` SignalR event in the current codebase.

## Read This Folder In Order

1. `consultation.usageguide.md`
2. `consultation.sourcecode.md`
3. This introduction
