---
doc_role: planning
module: scheduled-booking-cancel
kind: flow
doc_type: introduction
status: proposed
last_updated: 2026-04-20
owners: [backend-team]
verification_status: current-code-reviewed-target-not-implemented
---
# Scheduled Booking Cancel Introduction

## Goal

This module plans a new cancellation flow for scheduled consultation bookings.

Target business rule:

- allow a scheduled booking to be cancelled before the booked slot starts
- if the cancellation is initiated by the `Expert`, the paid booking amount must be refunded to the booking owner
- if the cancellation is initiated by the `Member`, the booking is cancelled without refund
- unpaid bookings should simply be cancelled and the reserved slot should be released

## Resume Summary

If this work is resumed later without prior chat history, the current code-verified state is:

1. Scheduled booking creation already exists through `POST /api/consultations/scheduled`.
2. Scheduled booking payment already exists through `POST /api/consultations/scheduled/{bookingId}/payments`.
3. The current payment flow moves money into consultation escrow and marks the booking `Confirmed`.
4. Booking auto-complete currently settles escrow to the expert after the slot ends.
5. A generic consultation escrow refund path already exists for emergency consultation rejection.
6. There is currently no scheduled booking cancel endpoint, no scheduled booking cancel service method, and no scheduled booking cancel refund flow.

## Code-Verified Current State

### Booking creation

`BookingService.CreateScheduledBookingAsync(...)` currently:

- validates the selected slot
- creates `Consultation` with status `Scheduled`
- creates `ConsultationBooking` with status `PendingPayment`
- marks `ExpertTimeSlot.Status = Reserved`

### Booking payment

`ConsultationPaymentService.PayScheduledBookingAsync(...)` currently supports:

- `WalletBalance` via `PayScheduledBookingWithWalletAsync(...)`
- `PayOs` via `CreateScheduledBookingPayOsIntentAsync(...)`

Current code-verified payment behavior:

- validates booking ownership
- requires `BookingStatus.PendingPayment`
- wallet payment:
  - creates `TransactionType.ConsultationPayment`
  - moves booking money into escrow immediately
  - sets `BookingStatus.Confirmed`
- PayOs payment:
  - creates a pending payment intent first
  - escrow and `BookingStatus.Confirmed` happen only after PayOs confirmation

### Booking completion

`BookingService.AutoCompleteElapsedScheduledConsultationsAsync(...)` currently:

- finds elapsed `Confirmed` scheduled bookings
- emits `ConsultationCallEnded`
- completes booking and consultation
- marks slot `Booked`
- triggers `SettleConsultationEscrowAsync(...)`

### Existing refund reference implementation

`EmergencyConsultationService.RejectEmergencyRequestAsync(...)` currently calls:

- `IConsultationPaymentService.RefundEmergencyEscrowAsync(...)`

That path proves the codebase already has:

- escrow refund validation
- wallet credit restoration
- `TransactionType.ConsultationRefund`

## Main Gap

The current gap is not payment infrastructure.

The real gap is the lack of a scheduled-booking-specific cancellation orchestration that combines:

- actor validation
- cancellable-state validation
- slot release
- consultation/booking state transition
- conditional escrow refund
- predictable API contract for mobile

## Proposed Implementation Direction

The cleanest implementation direction is:

1. add a scheduled booking cancel endpoint under `ConsultationScheduledController`
2. add a cancel method to `IBookingService` and `BookingService`
3. add a scheduled booking refund method to `IConsultationPaymentService` and `ConsultationPaymentService`
4. reuse the existing consultation escrow/refund model instead of inventing a second money path
5. update tests and docs in the same workstream

## Proposed Business Rules

The intended rule set for implementation is:

- only the booking `User` or the assigned `Expert` can cancel
- cancellation is allowed only while the slot has not started
- `PendingPayment` booking:
  - cancel booking
  - release slot back to `Available`
  - no refund because no money entered escrow
- `Confirmed` booking cancelled by `Expert`:
  - cancel booking
  - release slot back to `Available`
  - refund the full booking amount to the member wallet
- `Confirmed` booking cancelled by `Member`:
  - cancel booking
  - release slot back to `Available`
  - do not refund
- `Completed`, `Cancelled`, `Refunded`, and elapsed/started bookings are not cancellable

## Locked Decisions

The following decisions are already locked for this planning set:

1. Cancellation should explicitly update the linked `Consultation.Status` to `Cancelled`.
2. The implementation will write cancellation input into the existing `ConsultationBooking.CancellationReason` field.
3. For this wave, paid member-cancel remains represented by base `BookingStatus = Cancelled`; business context is carried by `CancellationReason`, and no extra abstraction is introduced.
4. `ConsultationBooking.CancellationReason` will be upgraded to an enum-backed field for type safety.
5. The enum must stay actor-centric and generic enough for future reuse, while endpoint output should continue rendering string values so the outward API contract stays stable.
6. The persistence model should follow the existing project convention of enum-as-number, so the current string column requires migration to numeric storage.

## Deferred Topics

The following topics are intentionally deferred and are not treated as locked contract in this doc set:

1. Whether refund descriptions need a newly standardized finance-audit taxonomy in this wave.

## Locked Cancellation Reason Direction

The cancellation-reason model for this module is now:

- domain field: `ConsultationBooking.CancellationReason`
- domain type: nullable enum
- persistence convention: numeric enum value
- API output convention: enum value rendered as string

Recommended initial enum set:

- `CancelledByMember = 1`
- `CancelledByExpert = 2`
- `CancelledByAdmin = 3`
- `CancelledBySystem = 4`

This set is intentionally:

- generic
- actor-centric
- not coupled to refund policy wording
- not coupled to slot timing details

That means refund/no-refund remains a flow outcome, not part of the enum itself.

## Migration Note

Current verified codebase shape:

- `ConsultationBooking.Status` is stored as numeric enum value
- `ConsultationBooking.CancellationReason` is still stored as string

Implementation therefore requires:

1. changing `ConsultationBooking.CancellationReason` from `string?` to nullable enum
2. adding EF migration to convert the column from string to numeric enum storage
3. backfilling existing values if any non-null legacy text already exists
4. updating mapping/response code to render `CancellationReason?.ToString()` where the field is exposed outward

## Delivered Artifacts

- `scheduled-booking-cancel.introduction.md`
- `scheduled-booking-cancel.roadmap.md`
- `scheduled-booking-cancel.sourcecode.md`
- `scheduled-booking-cancel.useguide.md`
