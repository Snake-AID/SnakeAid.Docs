---
doc_role: baseline
module: consultation
kind: flow
doc_type: sourcecode
status: active
last_updated: 2026-03-28
owners: [backend-team]
---

# Consultation Module — Source Code Overview

## Muc tieu

Mo ta hien trang kien truc source code cua consultation module. Khong lap lai business rules (xem consultation.introduction.md) hay API payloads (xem consultation.usageguide.*.md).

## Architecture

### Controllers

- ExpertController: expert settings, bulk time slots, directory/profile/reviews/time-slots
- ConsultationBookingsController: create booking, my bookings, expert scheduled inbox
- ConsultationsController: emergency request, accept/reject, end consultation, review
- ConsultationPaymentsController: scheduled + emergency payment
- VideoCallController: LiveKit token generation

### Services

- ExpertService: directory, profile mapping, settings, slot generation
- BookingService: create booking, booking history, slot-end auto-complete
- ConsultationService: end consultation, review, settlement trigger
- EmergencyConsultationService: create/accept/reject emergency request
- ConsultationPaymentService: payment orchestration, escrow, refund, settlement
- SignalRExpertEmergencyNotificationService: expert connection map, directed push

### Hubs

- ExpertHub (/hubs/expert): presence, emergency routing, request room
- ConsultationHub (/hubs/consultation): chat, image attachments, UI signaling

### Background

- ConsultationLifecycleBackgroundService: expire emergency requests, auto-complete scheduled consultations (30s polling, PostgreSQL advisory lock for multi-replica)

## Domain Model

| Entity | Role |
|--------|------|
| Consultation | Core session record (caller/callee, room, status, type) |
| ConsultationBooking | Scheduled booking (user, expert, slot, price, status) |
| ConsultationPingRequest | Emergency request (state machine: PendingPayment, PendingExpertResponse, Accepted/Declined/Expired) |
| ExpertProfile | Expert metadata (fees, stats, IsOnline) |
| ExpertTimeSlot | 30-min discrete slot (optimistic concurrency + unique constraint) |
| ChatMessage | In-room chat message (content, attachmentUrl, senderId) |
| Wallet / Transaction | Reused wallet infrastructure for consultation money movement |

### Transaction Types

- ConsultationPayment: member to system escrow
- ConsultationRefund: system escrow to member
- ExpertPayout: system escrow to expert

## Cross-Cutting Concerns

### Concurrency and Slot Integrity

- Slot booking: optimistic concurrency via Version field
- Slot generation: unique composite index (ExpertId, StartTime, EndTime)
- Emergency accept: reserve overlapping slots (Slot Paradox)

### Multi-Replica Safety

- Lifecycle worker: global PostgreSQL session advisory lock
- Payment/refund/settlement: transaction-scoped advisory lock per aggregate
- Topology: many API replicas, one shared PostgreSQL writer

### Presence

- SignalR-first: ExpertProfile.IsOnline as eventual consistency
- EnablePresenceSelfHealing hardcoded false

### Response Contract

- Success: ApiResponse<T> via ApiResponseBuilder
- Errors: typed exceptions + centralized middleware

## Hardcoded Values

- Emergency TTL: 2 minutes
- Lifecycle polling: 30 seconds
- System escrow wallet id: hardcoded
- Payment methods: WalletBalance, PayOs
- IsVerified: deferred from MVP

## Current Limits

- Payment status query endpoint: not implemented
- Completion/payment summary: incomplete for mobile
- Expert no-show / dispute: not in MVP
- Lifecycle coordination: PostgreSQL advisory locks (not a final scheduler)

## Relationship with Operation Docs

- Update this file when overall consultation architecture changes
- Update operations/*/plan.md when an individual operation refines its own scope
