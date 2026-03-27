---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-18
owners: [backend-team]
---

# Consultation Module Source Code

## Mục tiêu tài liệu

Tài liệu này mô tả hiện trạng source code của **toàn bộ consultation flow**.

Phân vai với tài liệu operation-specific:
- `consultation.sourcecode.md`
  - mô tả toàn module consultation ở mức tổng quan hiện tại
  - bao phủ các operation đã và đang shape flow consultation
- `operations/*/sourcecode.md`
  - mô tả source code trong phạm vi một operation cụ thể
  - ví dụ `Operation 06` chỉ nói phần gap-closure của Operations 01, 03, 04

## Consultation Flow Scope

Toàn flow consultation hiện được hình thành từ các operation chính:
- `Operation 01 (expert-directory)`: expert directory, expert profile, weekly time slots, expert settings, test coverage
- `Operation 02 (scheduled-consultation)`: scheduled consultation booking, waiting room, video room, end consultation, review
- `Operation 03 (emergency-consultation)`: expert presence, emergency consultation request, emergency request room, accept/reject realtime flow, directory filter/sort
- `Operation 04 (in-room-features)`: consultation chat và in-room signaling, đã hoàn tất
- `Operation 05 (payment-and-stabilization)`: payment orchestration (WalletBalance + PayOS), dual pricing, profile stats, escrow lifecycle, background automation

## High-Level Architecture

### Main REST Controllers

- `ExpertController`
  - expert settings
  - bulk time slots
  - expert directory/profile/reviews/time slots
- `ConsultationBookingsController`
  - create scheduled booking
  - get my bookings
  - get expert scheduled bookings
- `ConsultationsController`
  - create emergency request
  - accept/reject emergency request
  - end consultation
  - create consultation review
- `ConsultationPaymentsController`
  - scheduled booking payment
  - emergency request payment
- `VideoCallController`
  - generate LiveKit token for consultation room

### Main Services

- `ExpertService`
  - expert directory, profile mapping, settings update, slot generation
- `BookingService`
  - create scheduled booking, booking history, slot-end auto-complete path
- `ConsultationService`
  - end consultation, review handling, settlement trigger on explicit completion
- `EmergencyConsultationService`
  - create emergency request, accept/reject emergency request
- `ConsultationPaymentService`
  - pre-payment, escrow movement, refund, expert settlement
- `SignalRExpertEmergencyNotificationService`
  - expert connection map, directed push, request-room push

### Realtime Layer

- `ExpertHub` (`/hubs/expert`)
  - member presence subscription
  - expert presence registration
  - emergency request room subscription
- `ConsultationHub` (`/hubs/consultation`)
  - consultation room chat messaging
  - image attachment support
  - UI state signaling (volatile)
  - consultation participant authorization

### Background Automation

- `ConsultationLifecycleBackgroundService`
  - expire timed-out emergency requests
  - auto-complete scheduled consultations whose slot has ended
  - in multi-replica deployment, a PostgreSQL session advisory lock is used so only one API replica runs a lifecycle sweep at a time

## Core Domain Model

### Consultation

`Consultation`
- core consultation session record
- stores caller/callee, room id, start/end time, status, type
- used by both scheduled and emergency consultation

### ConsultationBooking

`ConsultationBooking`
- scheduled consultation booking record
- links user, expert, selected slot, price, booking status, consultation id
- current booking flow starts at `PendingPayment`
- current booking response used by mobile now also includes `UserName` so expert app can identify the member on scheduled consultation cards

### ConsultationPingRequest

`ConsultationPingRequest`
- emergency consultation request record
- current emergency state machine:
  - `PendingPayment`
  - `PendingExpertResponse`
  - `AcceptedByExpert`
  - `DeclinedByExpert`
  - `RescuerCancelled`
  - `Expired`
- request no longer goes directly to expert at create-time; payment happens first

### ExpertProfile

`ExpertProfile`
- stores expert metadata for directory/profile screens
- current pricing-related fields:
  - `ConsultationFee` (backward compatibility)
  - `EmergencyConsultationFee`
- profile response now also exposes:
  - `ScheduledConsultationFee`
  - `EmergencyConsultationFee`
  - `TotalConsultations`
  - `AverageResponseTimeMinutes`
  - `SuccessRate`

### ExpertTimeSlot

`ExpertTimeSlot`
- discrete 30-minute slot for scheduled booking
- protected by optimistic concurrency + unique DB constraint semantics in booking/slot generation flow

### Wallet / Transaction

Consultation payment currently reuses wallet infrastructure already present in the codebase:
- `Wallet`
- `Transaction`

Consultation-specific money movement currently uses:
- `TransactionType.ConsultationPayment`
- `TransactionType.ConsultationRefund`
- `TransactionType.ExpertPayout`
- plus internal mirror records with existing wallet transaction types

## Public API Surface

### Expert Directory & Availability

Implemented in `ExpertController` + `ExpertService`.

- `PUT /api/experts/me/settings`
- `POST /api/experts/me/time-slots/bulk`
- `GET /api/experts`
- `GET /api/experts/{expertId}`
- `GET /api/experts/{expertId}/reviews`
- `GET /api/experts/{expertId}/time-slots`

### Scheduled Consultation

Implemented in `ConsultationBookingsController`, `ConsultationsController`, `BookingService`, `ConsultationService`, `ConsultationPaymentService`.

- `POST /api/consultation-bookings`
- `GET /api/users/me/consultation-bookings`
- `GET /api/experts/me/consultation-bookings`
- `POST /api/consultation-bookings/{bookingId}/payments`
- `POST /api/consultations/{consultationId}/end`
- `POST /api/consultations/{consultationId}/reviews`
- `POST /api/videocall/livekit-token/{consultationId}`

### Snake Species Search

Implemented in `SnakesController` + `SnakeSpeciesService`.

- `GET /api/v1/snakes/search?q={query}`
  - search across scientific name, common name, and alternative names
  - returns venom and antivenom information for expert reference

### Emergency Consultation

Implemented in `ConsultationsController`, `EmergencyConsultationService`, `ConsultationPaymentService`, `ExpertHub`, `SignalRExpertEmergencyNotificationService`.

- `POST /api/consultations/emergency-requests`
- `POST /api/consultations/emergency-requests/{requestId}/payments`
- `POST /api/consultations/emergency-requests/{requestId}/accept`
- `POST /api/consultations/emergency-requests/{requestId}/reject`

## Current End-to-End Flow in Code

### 1. Expert Directory / Profile Flow

- mobile loads expert directory through `GET /api/experts`
- user presence-aware UI is enhanced by `ExpertHub`:
  - `JoinAsMember`
  - `OnlineExpertsSnapshot`
  - `ExpertPresenceChanged`
- profile screen reads:
  - expert profile
  - review list
  - available time slots

### 2. Scheduled Consultation Flow

- user creates booking via `POST /api/consultation-bookings`
- booking starts at `PendingPayment`
- user pays via `POST /api/consultation-bookings/{bookingId}/payments`
- payment success moves money into system escrow and booking becomes `Confirmed`
- expert loads own scheduled consultations through `GET /api/experts/me/consultation-bookings`
  - current endpoint returns `Confirmed` and `Completed` bookings
  - response includes `consultationId`, `roomId`, slot window, and `UserName`
  - current discovery model is REST pull; there is still no scheduled-consultation SignalR inbox
- at consultation time, participant gets room token via `POST /api/videocall/livekit-token/{consultationId}`
- consultation ends either:
  - explicitly via `POST /api/consultations/{consultationId}/end`
  - or implicitly by background auto-completion after slot end
- when consultation completes, escrow settles to expert
- user can submit review via `POST /api/consultations/{consultationId}/reviews`

### 3. Emergency Consultation Flow

- user creates request via `POST /api/consultations/emergency-requests`
- request starts at `PendingPayment`
- user joins request room via `JoinEmergencyRequestRoom(requestId)`
- user pays via `POST /api/consultations/emergency-requests/{requestId}/payments`
- payment success:
  - moves money into system escrow
  - sets request to `PendingExpertResponse`
  - sets `ExpiresAt`
  - pushes `EmergencyConsultationRequest` to selected expert
- expert is connected through `JoinAsExpert`
- expert accepts or rejects via REST API
- if accepted:
  - consultation is created
  - request becomes `AcceptedByExpert`
  - requester receives `EmergencyRequestStatusChanged`
- if rejected:
  - request becomes `DeclinedByExpert`
  - escrow refunds to member
- if expert does not respond before TTL:
  - background worker marks request `Expired`
  - escrow refunds to member
- after accepted consultation completes, escrow settles to expert

## SignalR Contract in Current Code

### ExpertHub Responsibilities

`ExpertHub` currently owns 3 concerns:

1. Expert presence registration
- `JoinAsExpert`
- binds connection to expert id
- updates `ExpertProfile.IsOnline`
- broadcasts `ExpertPresenceChanged`

2. Member presence subscription
- `JoinAsMember`
- joins common member group
- returns `OnlineExpertsSnapshot`

3. Emergency request room subscription
- `JoinEmergencyRequestRoom(requestId)`
- owner-only group join for request-specific status updates

### ConsultationHub Responsibilities

`ConsultationHub` provides real-time communication within consultation rooms:

1. Consultation room access control
- `OnConnectedAsync`: Validates user is Caller or Callee, joins consultation group
- Throws `HubException` for unauthorized access

2. Text messaging with attachments
- `ReceiveMessage(string content, string? attachmentUrl)`: Saves to DB, broadcasts to group
- Rate limited to 10 messages/minute per user
- Supports image attachments via Cloudinary URLs

3. Volatile UI signaling
- `Signal(string eventType, string payload)`: Broadcasts UI state without persistence
- Used for real-time interface updates (typing indicators, etc.)

### Current Events

- `OnlineExpertsSnapshot`
- `ExpertPresenceChanged`
- `EmergencyConsultationRequest`
- `EmergencyRequestStatusChanged`
- `MessageReceived` (consultation chat messages)
- `SignalReceived` (volatile UI state signals)

## Current Money Lifecycle

### Scheduled Consultation

- `PendingPayment`
- payment success -> `Escrowed`
- consultation complete -> `SettledToExpert`

### Emergency Consultation

- `PendingPayment`
- payment success -> escrow + `PendingExpertResponse`
- `DeclinedByExpert` -> refund to member wallet
- `Expired` -> refund to member wallet
- `AcceptedByExpert` -> escrow remains locked
- consultation complete -> settle to expert

### Important Business Rule Reflected in Code

- payment is required before consultation proceeds
- emergency request is only pushed to expert after payment success
- payout to expert does not happen on accept; it happens on consultation completion

## Cross-Cutting Concerns

### Concurrency and Slot Integrity

- slot creation and slot booking are protected against duplication/race conditions
- emergency accept reserves overlapping slots in the 30-minute emergency window
- consultation lifecycle jobs are coordinated across API replicas through a shared PostgreSQL advisory lock
- payment/refund/settlement critical sections now also take transaction-scoped advisory locks keyed by booking id, request id, or consultation id to reduce double-execution races in multi-replica deployments

### Multi-Replica Lifecycle Coordination

The consultation module now assumes that multiple API replicas may run at the same time against one shared write database.

Before the current hot patch, every replica would start its own `ConsultationLifecycleBackgroundService`, which created race windows for:
- `ExpireEmergencyRequestsAsync`
- `AutoCompleteElapsedScheduledConsultationsAsync`
- escrow refund flows
- escrow settlement flows

Current mitigation in code:
- one global PostgreSQL session advisory lock for the lifecycle worker loop
- one transaction-scoped advisory lock per critical aggregate during:
  - scheduled booking payment
  - emergency request payment
  - emergency refund
  - consultation settlement

This is a pragmatic safeguard for the current topology:
- many API replicas
- one shared PostgreSQL writer

It should not be mistaken for a final scheduler architecture. If consultation lifecycle throughput or operational complexity grows, the preferred evolution is a dedicated worker or centralized scheduler.

### Time Standard

- weekly slot generation requires UTC `weekStartDate`
- consultation slot times are persisted and compared in UTC

### Presence Strategy

- realtime online/offline decisions are SignalR-first
- `ExpertProfile.IsOnline` is maintained as eventual consistency state
- presence self-healing is guarded by hardcoded switch `EnablePresenceSelfHealing = false`

### Response / Error Contract

- success responses use `ApiResponse<T>` through `ApiResponseBuilder`
- error handling uses typed exceptions and centralized middleware

## Hardcoded / Temporary Implementation Points

The full consultation flow currently still contains some hardcoded or temporary-by-design implementation decisions:

- emergency request TTL is hardcoded to `2 minutes`
- consultation lifecycle background polling interval is hardcoded to `30 seconds`
- system escrow wallet id is hardcoded in current payment implementation
- consultation payment currently supports `WalletBalance` and `PayOs`
- `IsVerified` remains deferred for MVP
- current distributed coordination for lifecycle jobs depends on PostgreSQL advisory locks and therefore assumes consultation write paths run on the shared writer database, not on read replicas

## Current Limits of the Whole Flow

These limits apply to the current consultation module overall, not just Operation 06:

- consultation `PayOS` path now exists as an additional payment option beside wallet
- consultation payment status query endpoint is not implemented yet
- consultation completion/payment summary contract is still incomplete for full mobile UI
- expert no-show / dispute handling is not implemented in MVP

## Verification Snapshot

Current module-level reality after the latest implementation state:

- backend solution builds successfully
- targeted consultation test suite passes
- Operation 04 in-room features fully implemented (chat, signaling, snake search)
- Operation 05 payment and stabilization details documented in:
  - `operations/05-payment-and-stabilization/plan.md`

## Relationship with Operation-Specific SourceCode Docs

This file should stay at **module scope**.

Rule to maintain docs consistency:
- update `consultation.sourcecode.md` when the overall consultation architecture or current module flow changes
- update `operations/*/plan.md` when an individual operation introduces or refines implementation in its own scope

Example:
- if Operation 04 adds chat hub + consultation message APIs:
  - update `consultation.sourcecode.md` to reflect the whole module now includes chat
  - update `operations/04-in-room-features/plan.md` to explain exactly how that operation implemented chat

