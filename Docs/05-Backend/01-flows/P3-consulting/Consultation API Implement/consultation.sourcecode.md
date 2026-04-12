---
doc_role: baseline
module: consultation
kind: flow
doc_type: sourcecode
status: active
last_updated: 2026-04-12
owners: [backend-team]
---

# Consultation Module - Source Code Overview

## Objective

Describe the current source-code shape of the consultation module. This file does not repeat business rules from `consultation.introduction.md` or client payload details from `consultation.usageguide.*.md`.

## Architecture

### Controllers

- `ExpertController`: expert settings, bulk time slots, directory/profile/reviews/time-slots
- `ConsultationBookingsController`: create booking, my bookings, expert scheduled inbox (route: `api/consultations/scheduled`)
- `ConsultationsController`: instant request, accept/reject, end consultation, review, member consultation history (route: `api/consultations`)
- `ConsultationPaymentsController`: scheduled + instant payment (route: `api/consultations/payments`)
- `VideoCallController`: LiveKit token generation for consultation rooms, demo token endpoint for dev testing, and LiveKit webhook ingestion (route family: `api/consultations/{id}/video-token` + `api/videocall/*`)

### Services

- `ExpertService`: directory, profile mapping, settings, slot generation
- `BookingService`: create booking, booking history, slot-end auto-complete, elapsed-room cleanup hooks
- `ConsultationService`: end consultation, review, settlement trigger, expert consultation history
- `EmergencyConsultationService`: create/accept/reject emergency request
- `ConsultationPaymentService`: payment orchestration, escrow, refund, settlement
- `LiveKitService`: token generation, room operations, webhook signature validation
- `SignalRExpertEmergencyNotificationService`: expert connection map, directed push

### Hubs

- `ExpertHub` (`/hubs/expert`): presence, emergency routing, request room
- `ConsultationHub` (`/hubs/consultation`): chat, image attachments, UI signaling

### Background

- `ConsultationLifecycleBackgroundService`: expire emergency requests, auto-complete scheduled consultations, auto-complete elapsed emergency consultations, and coordinate room cleanup (30s polling, PostgreSQL advisory lock for multi-replica safety)

## Domain Model

| Entity | Role |
|--------|------|
| `Consultation` | Core session record (caller/callee, room, status, type) |
| `ConsultationBooking` | Scheduled booking (user, expert, slot, price, status) |
| `ConsultationPingRequest` | Emergency request (state machine: `PendingPayment`, `PendingExpertResponse`, `Accepted`/`Declined`/`Expired`) |
| `ExpertProfile` | Expert metadata (fees, stats, `IsOnline`) |
| `ExpertTimeSlot` | 30-min discrete slot (optimistic concurrency + unique constraint) |
| `ChatMessage` | In-room chat message (content, attachmentUrl, senderId) |
| `Wallet` / `Transaction` | Reused wallet infrastructure for consultation money movement |

### Transaction Types

- `ConsultationPayment`: member payment that establishes consultation escrow in the ledger
- `ConsultationRefund`: system escrow to member
- `ExpertPayout`: expert net payout when consultation settlement completes
- `PlatformFee`: platform-owned settlement amount retained from consultation escrow

## Cross-Cutting Concerns

### Video Call Foundation

- Provider: LiveKit Cloud
- Room naming: `consultation-{consultationId}`
- Join flow: client calls `POST /api/consultations/{consultationId}/video-token`, receives `{ token, wsUrl, roomName }`, then connects with the LiveKit client SDK
- Additional endpoints under `VideoCallController`:
  - `POST /api/videocall/livekit-token/demo/{roomname}` for development-only token generation
  - `POST /api/videocall/livekit-webhook` for LiveKit event callbacks
- Role-based publish grants: expert can publish `screen_share` and `screen_share_audio`; non-expert participants are limited to camera/microphone sources
- Webhook events tracked in docs and code path: `room_started`, `participant_joined`, `participant_left`, `room_finished`

### Concurrency and Slot Integrity

- Slot booking: optimistic concurrency via `Version` field
- Slot generation: unique composite index (`ExpertId`, `StartTime`, `EndTime`)
- Emergency accept: reserve overlapping slots (Slot Paradox)

### Multi-Replica Safety

- Lifecycle worker: global PostgreSQL session advisory lock
- Payment/refund/settlement: transaction-scoped advisory lock per aggregate
- Topology: many API replicas, one shared PostgreSQL writer

### Presence

- SignalR-first: `ExpertProfile.IsOnline` as eventual consistency
- `EnablePresenceSelfHealing` hardcoded `false`

### Response Contract

- Success: `ApiResponse<T>` via `ApiResponseBuilder`
- Errors: typed exceptions + centralized middleware

### Consultation Money Semantics

- consultation escrow is ledger-driven; it is no longer represented by `system.wallet` balance changes
- escrow hold is inferred from `ConsultationPayment`
- escrow release/refund is inferred from `ExpertPayout`, `ConsultationRefund`, and `PlatformFee`
- `ConsultationPaymentResponse.SystemWalletBalanceAfter` is no longer part of the public consultation contract

## Hardcoded Values

- Emergency TTL: 2 minutes
- Lifecycle polling: 30 seconds
- LiveKit token TTL: 10 minutes
- LiveKit room empty timeout: 600 seconds
- Payment methods: `WalletBalance`, `PayOs`
- Consultation platform fee default: `20%` when no explicit system setting is configured
- `IsVerified`: deferred from MVP

## Current Limits

- Payment status query endpoint: not implemented
- Completion/payment summary: incomplete for mobile
- Expert no-show / dispute: not in MVP
- Lifecycle coordination: PostgreSQL advisory locks (not a final scheduler)

## Relationship with Operation Docs

- `operations/01-INIT-livekit-video-foundation/` captures the pre-consultation LiveKit research and backend integration setup that the rest of the flow depends on
- update this file when overall consultation architecture changes
- update `operations/*/plan.md` when an individual operation refines its own scope
