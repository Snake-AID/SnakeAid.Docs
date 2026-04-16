---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-04-12
owners: [backend-team]
---

# Consultation Module Introduction

## Domain Context

The Consultation module enables members to find a suitable snake expert, choose a consultation mode, complete payment, and receive advice through a live consultation session. The module supports two business journeys:

- `Đặt lịch tư vấn`: member selects a future slot from expert availability.
- `Tư vấn ngay`: member selects a currently online expert and submits an immediate consultation request.

This module is not just a video-call function. It is a full business flow covering expert discovery, pricing, availability, payment, consultation lifecycle, completion, post-consultation feedback, and the LiveKit-based room join needed for remote expert interaction.

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
7. When a valid consultation session is ready, the client requests a LiveKit access token and joins room `consultation-{consultationId}`.
8. When the consultation completes, the system finalizes business state and allows feedback/review.

## Business Rules / Invariants

1. **Expert Availability**: Experts define their availability dynamically week-by-week. These configurations are ingested and converted into bookable `ExpertTimeSlot` records.
2. **Dual Consultation Modes**: The system distinguishes between scheduled consultation and immediate consultation, even if both ultimately end in a live consultation session.
3. **Pre-Payment Rule**: Both scheduled consultation and emergency consultation require payment before the business flow proceeds.
4. **Escrow Rule**: After successful payment, consultation money enters a ledger-driven escrow state. Escrow is inferred from consultation transactions rather than from a system-wallet balance side effect, and the expert is not paid immediately.
5. **Emergency Request Lifecycle**:
   - Member selects one expert explicitly.
   - Emergency request is created only after payment success.
   - Expert may `Accept` or `Reject`.
   - `Reject` or `Expired` must refund escrow immediately back to the member `SApay` balance.
6. **Slot Paradox Guard**: If an expert accepts an emergency consultation in the middle of the normal 30-minute slot grid, overlapping regular `ExpertTimeSlot`s are reserved in the background to avoid double-booking.
7. **Settlement Rule**: Escrow is released only when the consultation is considered completed. The release is represented by `PlatformFee + ExpertPayout` instead of a single gross payout to the expert. Completion can be triggered by:
   - natural end of the booked slot/window
   - explicit finish/end API
8. **Realtime Separation**:
   - `ExpertHub` handles expert presence, emergency routing, and emergency request-room status events.
   - `ConsultationHub` handles in-room realtime features for an active consultation.
9. **Emergency Request Room**: After creating an emergency request, the requester joins a request-specific SignalR group (`consultation:emergency:request:{requestId}`) to receive terminal status updates in real time.
10. **Video Call Provider Rule**: Consultation media transport uses LiveKit Cloud. Backend responsibility is token generation, room lifecycle integration, and webhook handling, not WebRTC media processing internals.
11. **MVP Rule for Verification**: `IsVerified` is not a completed MVP business capability yet and should not be treated as a finalized trust signal until a later operation formalizes it.
12. **Multi-Replica Lifecycle Rule**: Background lifecycle work for consultation is a distributed systems concern, not just an in-process timer. If multiple API replicas run against the same write database, only one replica may execute lifecycle sweep work at a time and financial side effects must be protected against duplicate execution.

## Scope

- Expert directory search, profile viewing, and availability discovery.
- Expert pricing and settings.
- Scheduled consultation booking lifecycle.
- Emergency consultation request lifecycle.
- Payment-before-consultation flow and escrow business behavior.
- Consultation room entry, completion, and post-consultation review.
- Realtime presence and emergency request-room updates.
- In-room realtime communication for active consultation sessions.
- LiveKit token generation, room naming convention, and webhook-driven room lifecycle integration.

## Out of Scope

- WebRTC transport internals beyond the backend integration boundary with LiveKit Cloud.
- Generic wallet/payment infrastructure outside consultation-specific orchestration.
- Finalized verification policy for expert trust badges in MVP.

## Relationship Between Operations

- `Operation 01 (livekit-video-foundation)`: establish LiveKit Cloud integration, token generation, webhook handling, and the initial video-call testing surface that the consultation flow builds on.
- `Operation 02 (expert-directory)`: expert directory, availability, settings, profile, test coverage.
- `Operation 03 (scheduled-consultation)`: scheduled consultation booking, payment, room join, completion, review.
- `Operation 04 (emergency-consultation)`: emergency consultation, ExpertHub presence, directory filter/sort, Slot Paradox.
- `Operation 05 (in-room-features)`: ConsultationHub chat, image attachments, UI signaling, snake species search.
- `Operation 06 (payment-and-stabilization)`: payment orchestration (WalletBalance + PayOS), dual pricing, profile stats, escrow lifecycle, background automation, multi-replica safety.
- `Operation 07 (room-expiry-and-expert-history)`: room cleanup, room expiry signaling, and unified expert consultation history.

## Operational Note

The current consultation implementation includes a multi-replica safety patch:

- `ConsultationLifecycleBackgroundService` uses a shared PostgreSQL advisory lock so not every API replica runs the lifecycle sweep at the same time.
- payout, refund, and settlement paths also use aggregate-scoped transaction locks to reduce duplicate financial side effects.

This note matters because consultation is not only a UX flow. It also includes timed business transitions, room cleanup, and escrow movement, which become unsafe if every horizontally scaled API instance executes the same background work independently.
