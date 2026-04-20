---
doc_role: implementation
module: scheduled-booking-cancel
kind: flow
doc_type: roadmap
status: current
last_updated: 2026-04-20
owners: [backend-team]
verification_status: implemented-and-code-verified
---

# Scheduled Booking Cancel Roadmap

## Current Status Snapshot

- module status: `Implemented`
- current scheduled booking cancel endpoint: `Available`
- current scheduled booking cancel service: `Available`
- current scheduled booking refund path: `Available for expert-cancel`
- reusable refund infrastructure: `Available`
- docs status: `Baseline updated`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Current verified state:

- `IBookingService` supports create, cancel, list, and auto-complete
- `ConsultationScheduledController` supports create booking, list endpoints, and cancel endpoint
- scheduled payment supports both `WalletBalance` and `PayOs`
- wallet payment escrows immediately and marks booking `Confirmed`
- PayOs payment creates an intent first, then confirms and escrows later
- scheduled cancel handles both `PendingPayment` and `Confirmed`
- pending unpaid PayOs cancellation deletes the local pending payment transaction and best-effort cancels the provider payment link
- expert-cancel of a paid booking refunds the booking owner
- member-cancel of a paid booking does not refund
- scheduled auto-complete settles escrow to the expert at the end of the slot

## Delivered Outcome

1. member can cancel a future scheduled booking
2. expert can cancel a future scheduled booking
3. unpaid booking cancellation releases the slot with no refund
4. expert-cancel on a paid booking refunds the booking owner
5. member-cancel on a paid booking does not refund
6. booking, consultation, and slot states remain internally consistent
7. mobile has an active endpoint contract and response example

## Locked Functional Direction

- [x] Scheduled booking cancellation is a dedicated flow
- [x] Refund policy is actor-based
- [x] Expert-cancel on paid booking triggers refund
- [x] Member-cancel on paid booking does not trigger refund
- [x] Cancellation also updates linked `Consultation.Status = Cancelled`
- [x] Cancellation reason is stored in `ConsultationBooking.CancellationReason`
- [x] Paid member-cancel remains modeled as base `BookingStatus = Cancelled`
- [x] Cancellation reason is enum-backed and follows project enum-as-number persistence
- [x] Endpoint output renders enum names as strings through the API JSON enum converter

## Implementation Checklist

### Phase 1. Contract Design

- [x] Lock endpoint route and HTTP verb
- [x] Lock minimal request body shape
- [x] Lock minimal response payload shape
- [x] Lock exact cancellation window rule
- [x] Lock initial cancellation-reason enum vocabulary

### Phase 2. Booking Service

- [x] Add `CancelScheduledBookingAsync(...)` to `IBookingService`
- [x] Implement actor ownership validation in `BookingService`
- [x] Validate booking state and slot-start boundary
- [x] Release reserved slot back to `Available`
- [x] Update booking state to cancellation outcome
- [x] Update linked consultation state to `Cancelled`
- [x] Persist cancellation reason into existing booking field
- [x] Replace string reason field usage with nullable enum in domain and response code

### Phase 3. Payment Service

- [x] Add scheduled booking refund method to `IConsultationPaymentService`
- [x] Reuse escrow availability validation
- [x] Reuse internal wallet crediting and `ConsultationRefund` transaction creation
- [x] Prevent duplicate refund for the same booking
- [x] Keep no-refund path explicit for member-cancel
- [x] Add pending scheduled payment cleanup for unpaid cancellation

### Phase 4. API Layer

- [x] Add cancel endpoint in `ConsultationScheduledController`
- [x] Restrict auth to `User` and `Expert`
- [x] Return `ApiResponse<ConsultationBookingResponse>`
- [x] Preserve route style used by the consultation module

### Phase 5. Tests

- [x] Integration test: member can cancel own `PendingPayment` booking
- [x] Integration test: expert can cancel assigned `Confirmed` booking
- [x] Integration test: expert-cancel `Confirmed` booking refunds the member
- [x] Integration test: member-cancel `Confirmed` booking does not refund
- [x] Integration test: pending PayOs booking cancellation deletes local pending payment transaction
- [x] Focused regression tests for admin consultation history and payment flow still pass

### Phase 6. Docs Sync

- [x] Update introduction with implemented state
- [x] Update roadmap with implementation status
- [x] Update sourcecode diagrams with final method names
- [x] Update useguide with active contract

### Phase 7. Data Migration

- [x] Change `ConsultationBooking.CancellationReason` column from string storage to numeric enum storage
- [x] Drop backward-compatibility for legacy string values
- [x] Align DB storage with enum culture used elsewhere in the codebase

## Verification Strategy

Minimum verified flow:

1. create future scheduled booking
2. pay booking successfully
3. cancel as expert
4. verify booking is cancelled
5. verify slot is available again
6. verify member wallet is refunded
7. verify `ConsultationRefund` transaction exists once
8. repeat with member-cancel and verify no refund
9. repeat with pending `PayOs` and verify pending transaction cleanup

## Change Log

### 2026-04-18

- Created the planning set for scheduled booking cancellation

### 2026-04-20

- Implemented `POST /api/consultations/scheduled/{bookingId}/cancel`
- Implemented expert-cancel refund and member-cancel no-refund flow
- Implemented pending PayOs cleanup for cancelled unpaid bookings
- Finalized destructive migration from string `CancellationReason` to numeric enum storage
