---
doc_role: planning
module: scheduled-booking-cancel
kind: flow
doc_type: roadmap
status: proposed
last_updated: 2026-04-18
owners: [backend-team]
verification_status: current-code-reviewed-target-not-implemented
---

# Scheduled Booking Cancel Roadmap

## Current Status Snapshot

- module status: `Proposed`
- current scheduled booking cancel endpoint: `Missing`
- current scheduled booking cancel service: `Missing`
- current scheduled booking refund path: `Missing for scheduled bookings`
- reusable refund infrastructure: `Available`
- docs status: `Planning set created`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Current verified state:

- `IBookingService` supports create/list/auto-complete only
- `ConsultationScheduledController` supports create booking and list endpoints only
- scheduled payment supports both `WalletBalance` and `PayOs`
- wallet payment escrows immediately and marks booking `Confirmed`
- PayOs payment creates an intent first, then confirms and escrows later
- scheduled auto-complete settles escrow to the expert at the end of the slot
- emergency rejection already refunds escrow through `RefundEmergencyEscrowAsync(...)`

Missing today:

- cancel API for scheduled bookings
- cancel orchestration for `PendingPayment` and `Confirmed`
- scheduled-booking-specific refund helper
- tests covering expert-cancel refund and member-cancel no-refund

## Target Outcome

After implementation:

1. member can cancel a future scheduled booking
2. expert can cancel a future scheduled booking
3. unpaid booking cancellation releases the slot with no refund
4. expert-cancel on a paid booking refunds the booking owner
5. member-cancel on a paid booking does not refund
6. booking/consultation/slot states remain internally consistent
7. mobile has a clean endpoint contract and response example

## Locked Functional Direction

- [x] Scheduled booking cancellation is a new dedicated flow
- [x] Refund policy is actor-based
- [x] Expert-cancel on paid booking triggers refund
- [x] Member-cancel on paid booking does not trigger refund
- [x] Docs must clearly separate current verified behavior from target planned behavior

## Implementation Checklist

### Phase 1. Contract Design

- [ ] Lock endpoint route and HTTP verb
- [ ] Lock request body shape
- [ ] Lock response payload shape
- [ ] Lock final status mapping for cancelled paid bookings
- [ ] Lock exact cancellation window rule

### Phase 2. Booking Service

- [ ] Add `CancelScheduledBookingAsync(...)` to `IBookingService`
- [ ] Implement actor ownership validation in `BookingService`
- [ ] Validate booking state and slot-start boundary
- [ ] Release reserved slot back to `Available`
- [ ] Update booking state to cancellation outcome
- [ ] Update linked consultation state if needed

### Phase 3. Payment Service

- [ ] Add scheduled booking refund method to `IConsultationPaymentService`
- [ ] Reuse escrow availability validation
- [ ] Reuse internal wallet crediting and `ConsultationRefund` transaction creation
- [ ] Prevent duplicate refund for the same booking
- [ ] Keep no-refund path explicit for member-cancel

### Phase 4. API Layer

- [ ] Add cancel endpoint in `ConsultationScheduledController`
- [ ] Restrict auth to `User` and `Expert`
- [ ] Return consistent `ApiResponse<ConsultationBookingResponse>` or chosen final DTO
- [ ] Preserve route style used by the consultation module

### Phase 5. Tests

- [ ] Unit test: member can cancel own `PendingPayment` booking
- [ ] Unit test: expert can cancel assigned `PendingPayment` booking
- [ ] Unit test: expert-cancel `Confirmed` booking refunds the member
- [ ] Unit test: member-cancel `Confirmed` booking does not refund
- [ ] Unit test: duplicate refund is blocked
- [ ] Unit test: past-start or completed bookings cannot be cancelled
- [ ] Integration test: API route + auth for member
- [ ] Integration test: API route + auth for expert
- [ ] Integration test: wallet balances and refund transaction after expert cancel

### Phase 6. Docs Sync

- [ ] Update introduction after design decisions are finalized
- [ ] Update roadmap with implementation status
- [ ] Update sourcecode diagrams with final method names
- [ ] Update useguide with active contract once endpoint is implemented

## Candidate File Targets

### Backend

- [ ] `SnakeAid.Api/Controllers/ConsultationScheduledController.cs`
- [ ] `SnakeAid.Service/Interfaces/IBookingService.cs`
- [ ] `SnakeAid.Service/Implements/BookingService.cs`
- [ ] `SnakeAid.Service/Interfaces/IConsultationPaymentService.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationPaymentService.cs`
- [ ] `SnakeAid.Core/Requests/Consultation/...` for cancel request DTO if needed
- [ ] `SnakeAid.Core/Responses/Consultation/...` only if a new response shape is needed

### Tests

- [ ] `SnakeAid.Tests/Unit/...Booking...Tests.cs`
- [ ] `SnakeAid.Tests/Integration/ConsultationBookingsControllerIntegrationTests.cs`
- [ ] `SnakeAid.Tests/Integration/ConsultationPaymentIntegrationTests.cs`

### Docs

- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/scheduled-booking-cancel/scheduled-booking-cancel.introduction.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/scheduled-booking-cancel/scheduled-booking-cancel.roadmap.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/scheduled-booking-cancel/scheduled-booking-cancel.sourcecode.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/scheduled-booking-cancel/scheduled-booking-cancel.useguide.md`

## Suggested Endpoint Shape

Recommended current proposal:

- route: `POST /api/consultations/scheduled/{bookingId}/cancel`
- auth roles: `User,Expert`
- request body:
  - optional `reason`
- success payload:
  - updated booking object including new `status`
  - explicit refund summary fields if the team wants mobile to display the outcome directly

Why this proposal:

- the codebase already uses action-style routes like `.../{id}/cancel`
- `POST` avoids forcing a `DELETE` endpoint to carry a request body
- the operation is business-stateful, not a pure resource deletion

## Verification Strategy

Minimum verification before marking complete:

1. create future scheduled booking
2. pay booking successfully
3. cancel as expert
4. verify booking is cancelled
5. verify slot is available again
6. verify member wallet is refunded
7. verify `ConsultationRefund` transaction exists once
8. repeat with member-cancel and verify no refund

## Open Questions

1. Should cancelled paid bookings end at `Cancelled` or `Refunded`?
2. Should the API expose `cancelledByRole` for mobile clarity?
3. Should cancellation emit a realtime event to the other participant now or later?
4. Should `Member` cancellation after payment deduct a platform fee, or strictly no refund and no extra ledger entry?

## Change Log

### 2026-04-18

- Created a fresh planning set for scheduled booking cancellation
- Recorded current code-verified booking/payment/refund state
- Locked the actor-based refund direction for implementation planning
