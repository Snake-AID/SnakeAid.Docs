---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-07
owners: [backend-team]
---

# Consultation Module Introduction

## Domain Context

The Consultation module enables members to find a suitable snake expert, choose a consultation mode, complete payment, and receive advice through a live consultation session. The module supports two business journeys:

- `Đặt lịch tư vấn`: member selects a future slot from expert availability.
- `Tư vấn ngay`: member selects a currently online expert and submits an immediate consultation request.

This module is not just a video-call function. It is a full business flow covering expert discovery, pricing, availability, payment, consultation lifecycle, completion, and post-consultation feedback.

## Core User Journey

1. Member enters the consultation area and sees expert options.
2. Member selects an expert from the directory.
3. Member reviews profile details, price, reviews, and availability.
4. Member chooses either:
   - `Tư vấn ngay`
   - `Đặt lịch tư vấn`
5. Member completes payment before the consultation flow is allowed to proceed.
6. The consultation either:
   - starts immediately after expert acceptance for emergency consultation
   - starts at the booked time for scheduled consultation
7. When the consultation completes, the system finalizes business state and allows feedback/review.

## Business Rules / Invariants

1. **Expert Availability**: Experts define their availability dynamically week-by-week. These configurations are ingested and converted into bookable `ExpertTimeSlot` records.
2. **Dual Consultation Modes**: The system distinguishes between scheduled consultation and immediate consultation, even if both ultimately end in a live consultation session.
3. **Pre-Payment Rule**: Both scheduled consultation and emergency consultation require payment before the business flow proceeds.
4. **Escrow Rule**: After successful payment, funds move into the system wallet escrow layer (`SApay` as business wallet concept) rather than transferring immediately to the expert.
5. **Emergency Request Lifecycle**:
   - Member selects one expert explicitly.
   - Emergency request is created only after payment success.
   - Expert may `Accept` or `Reject`.
   - `Reject` or `Expired` must refund escrow immediately back to the member `SApay` balance.
6. **Slot Paradox Guard**: If an expert accepts an emergency consultation in the middle of the normal 30-minute slot grid, overlapping regular `ExpertTimeSlot`s are reserved in the background to avoid double-booking.
7. **Settlement Rule**: Escrow is released to the expert only when the consultation is considered completed. Completion can be triggered by:
   - natural end of the booked slot/window
   - explicit finish/end API
8. **Realtime Separation**:
   - `ExpertHub` handles expert presence, emergency routing, and emergency request-room status events.
   - `ConsultationHub` handles in-room realtime features for an active consultation.
9. **Emergency Request Room**: After creating an emergency request, the requester joins a request-specific SignalR group (`consultation:emergency:request:{requestId}`) to receive terminal status updates in real time.
10. **MVP Rule for Verification**: `IsVerified` is not a completed MVP business capability yet and should not be treated as a finalized trust signal until a later operation formalizes it.

## Scope

- Expert directory search, profile viewing, and availability discovery.
- Expert pricing and settings.
- Scheduled consultation booking lifecycle.
- Emergency consultation request lifecycle.
- Payment-before-consultation flow and escrow business behavior.
- Consultation room entry, completion, and post-consultation review.
- Realtime presence and emergency request-room updates.
- In-room realtime communication for active consultation sessions.

## Out of Scope

- WebRTC transport internals (delegated to LiveKit Cloud).
- Generic wallet/payment infrastructure outside consultation-specific orchestration.
- Finalized verification policy for expert trust badges in MVP.

## Relationship Between Operations

- `Operation 01`: expert directory and availability foundation.
- `Operation 03`: scheduled consultation foundation.
- `Operation 04`: emergency consultation and presence foundation.
- `Operation 05`: in-room features.
- `Operation 06`: stabilization pass to close remaining mobile-readiness gaps inside Operations 01, 03, and 04.
