---
doc_role: implementation
module: scheduled-booking-cancel
kind: flow
doc_type: introduction
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: implemented-and-code-verified
---
# Scheduled Booking Cancel Introduction

## Goal

This module implements the cancellation flow for scheduled consultation bookings.

Active business rule:

- allow a scheduled booking to be cancelled before the booked slot starts
- if the cancellation is initiated by the `Expert`, the paid booking amount is refunded to the booking owner
- if the cancellation is initiated by the `Expert`, backend also queues a member-facing push notification in Vietnamese
- if the cancellation is initiated by the `Member`, the booking is cancelled without refund and the existing escrow is settled instead of left pending
- unpaid bookings are cancelled and the reserved slot is released

## Resume Summary

If this work is resumed later without prior chat history, the current code-verified state is:

1. Scheduled booking creation exists through `POST /api/consultations/scheduled`.
2. Scheduled booking payment exists through `POST /api/consultations/scheduled/{bookingId}/payments`.
3. Scheduled booking cancel exists through `POST /api/consultations/scheduled/{bookingId}/cancel`.
4. Paid expert-cancel refunds the member wallet through consultation escrow refund infrastructure.
5. Paid member-cancel does not refund and settles the confirmed consultation escrow.
6. Pending `PayOs` cancellation deletes the local pending payment transaction and best-effort cancels the provider link.
7. Attempting to cancel a "pending" payment that already has a confirmed external transaction now fails explicitly instead of being treated like a missing payment.
8. Expert-cancel now also queues a member-targeted push notification after successful commit.

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

### Booking cancellation

`BookingService.CancelScheduledBookingAsync(...)` currently:

- allows only the booking `User` or assigned `Expert`
- rejects elapsed/started bookings
- for `PendingPayment`:
  - cancels booking
  - releases slot to `Available`
  - calls pending payment cleanup for local `PayOs` intent when present
- for `Confirmed` cancelled by `Expert`:
  - cancels booking
  - releases slot to `Available`
  - refunds the member wallet
  - publishes a member push notification in Vietnamese after the cancellation commit succeeds
- for `PendingPayment` cancelled by `Expert`:
  - cancels booking
  - releases slot to `Available`
  - publishes the same member push notification in Vietnamese after the cancellation commit succeeds
- for `Confirmed` cancelled by `Member`:
  - cancels booking
  - releases slot to `Available`
  - does not refund
  - settles escrow instead of leaving confirmed payment funds stranded
- updates linked `Consultation.Status = Cancelled`

### Booking completion

`BookingService.AutoCompleteElapsedScheduledConsultationsAsync(...)` currently:

- finds elapsed `Confirmed` scheduled bookings
- emits `ConsultationCallEnded`
- completes booking and consultation
- marks slot `Booked`
- triggers `SettleConsultationEscrowAsync(...)`

## Cancellation Reason Direction

The cancellation-reason model in the active codebase is:

- domain field: `ConsultationBooking.CancellationReason`
- domain type: nullable enum
- persistence convention: numeric enum value
- API behavior: typed enum in response models, serialized to string in JSON by the global API enum converter

Current enum set:

- `CancelledByMember = 1`
- `CancelledByExpert = 2`
- `CancelledByAdmin = 3`
- `CancelledBySystem = 4`

This enum is intentionally:

- generic
- actor-centric
- not coupled to refund policy wording
- not coupled to slot timing details

## Migration Note

Current verified codebase shape:

- `ConsultationBooking.Status` is stored as numeric enum value
- `ConsultationBooking.CancellationReason` is stored as numeric enum value

Delivered migration behavior:

1. drop the old string `CancellationReason` column
2. recreate `CancellationReason` as nullable integer enum storage
3. do not preserve backward compatibility for legacy string values

## Delivered Artifacts

- `scheduled-booking-cancel.introduction.md`
- `scheduled-booking-cancel.roadmap.md`
- `scheduled-booking-cancel.hallucination.md`
- `scheduled-booking-cancel.sourcecode.md`
- `scheduled-booking-cancel.useguide.md`

## Implemented Notification Extension

Delivered behavior:

- when the expert cancels a scheduled booking
- backend should send a push notification to the member of that meeting

Code-verified infrastructure already available in the codebase:

- `NotificationQueueService.PublishAsync(...)`
- `NotificationConsumer`
- `FirebaseNotificationService`
- broker publishing through `MassTransit`
- broker failure checks that explicitly reference RabbitMQ transport exceptions

Language rule for this extension:

- push notification title and body must be Vietnamese only
- English push content is not acceptable for this user-facing flow

Implemented copy:

- title: `Lịch tư vấn đã bị chuyên gia hủy`
- body: `Chuyên gia đã hủy lịch tư vấn của bạn. Vui lòng kiểm tra lại lịch hẹn trong ứng dụng.`
