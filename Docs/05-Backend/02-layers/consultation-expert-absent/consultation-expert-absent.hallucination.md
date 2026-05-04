---
doc_role: research
module: consultation-expert-absent
kind: decision-log
doc_type: hallucination
status: active
last_updated: 2026-05-04
owners: [backend-team]
verification_status: planning-only
---

# Consultation Expert Absent Hallucination Log

This file records assumptions, open questions, and decisions that must not be silently treated as implemented backend behavior.

## Closed Decisions For Follow-up Implementation

### HD1: Preserve expert-absent status during end-call

Decision:

- `EndConsultationAsync` must not overwrite `ExpertAbsent` or `ExpertAbsentHandled` with `Completed`.

Reason:

- mobile may report expert absence and then call the normal end-consultation endpoint to close the room
- ending the call is a runtime cleanup action, not proof that consultation business completed successfully

Implementation impact:

- keep SignalR and LiveKit cleanup
- set `EndTime` when ending an expert-absent call
- skip completion side effects for expert-absent calls

### HD2: Mobile ends after report; backend preserves business status

Decision:

- mobile can call normal end-consultation after reporting expert absence
- backend must preserve `ExpertAbsent` / `ExpertAbsentHandled`

Reason:

- mobile flow can stay simple
- backend owns business-state protection

### HD3: Scheduled auto-complete remains denylist-based

Decision:

- keep the current denylist style in scheduled auto-complete
- extend the denylist from `Completed` to include `ExpertAbsent` and `ExpertAbsentHandled`

Reason:

- this is a narrow patch that matches current implementation style
- it prevents absent cases from being completed by the lifecycle worker

### HD4: Admin approval refunds the member

Decision:

- `ConfirmExpertAbsentHandledAsync(...)` must perform the member refund in the same transaction/flow as the admin approval.
- The member is not refunded when the report is submitted.
- The expert is not settled for approved expert-absent cases.

Reason:

- report-time refund can be abused
- admin approval is the verification point for the member refund
- expert absence should not be paid as a successfully completed consultation

Implementation impact:

- implemented: `ConfirmExpertAbsentHandledAsync(...)` refunds the scheduled consultation escrow to the member
- keep report submission payment-neutral
- creates no expert payout/settlement for this path

### HD5: Approved expert-absent cases become handled and refunded

Decision:

- after admin approval, set `Consultation.Status = ExpertAbsentHandled`
- after refund completes, set `ConsultationBooking.Status = Refunded`

Reason:

- `ExpertAbsentHandled` represents the consultation-level admin resolution
- `ConsultationBooking.BookingStatus.Refunded` is the chosen booking terminal state for approved expert-absent cases

Implementation impact:

- implemented: do not leave the booking in `Confirmed` after refund
- implemented: do not convert approved expert-absent bookings to `Completed`

### HD6: Expert-absent refund is idempotent

Decision:

- if the booking is already `Refunded` or the consultation is already `ExpertAbsentHandled`, the admin approval flow must not create a duplicate refund
- return the current state for repeat approval attempts

Reason:

- admin retries or double-clicks must not credit the member twice
- the API should be safe to retry after a transient client/network issue

Implementation impact:

- check existing consultation and booking state before creating refund transactions
- return the current admin consultation response when the case is already handled/refunded

## Closed Research Questions

### HR1: Refund or settlement policy for expert-absent cases

Question:

- when admin handles an `ExpertAbsent` case, should the system refund the member, settle the expert, or let admin choose?

Status:

- Closed by HD4 on 2026-05-04.

Current verified behavior:

- `ReportExpertAbsentAsync(...)` does not refund
- `ConfirmExpertAbsentHandledAsync(...)` only changes status to `ExpertAbsentHandled`
- scheduled booking cancel docs define refund for expert-cancel and settlement for member-cancel, but do not define expert-absent payment resolution

Resolved policy:

- do not refund at report submission
- refund the member during admin approval
- do not settle the expert

Still not part of the decision:

- whether admin should provide reason or evidence notes

### HR2: Final booking status for expert-absent cases

Question:

- should `ConsultationBooking.Status` remain `Confirmed`, move to an existing status, or gain a dedicated expert-absent/disputed status?

Status:

- Closed by HD5 on 2026-05-04.

Current verified behavior:

- paid scheduled booking becomes `Confirmed`
- normal end or elapsed slot can move booking to `Completed`
- cancel flow can move booking to `Cancelled`
- `BookingStatus.Refunded` exists but current scheduled refund flow records refund transactions and does not use it as a booking state in the observed code

Resolved policy:

- do not set booking to `Completed` from expert-absent end-call or auto-complete
- set `ConsultationBooking.Status = Refunded` after admin approval refund succeeds

### HR3: Escrow dispute state

Question:

- should scheduled consultation escrow have a visible dispute state when consultation is `ExpertAbsent`?

Status:

- Closed by HD4 and HD6 on 2026-05-04.

Current verified behavior:

- scheduled payment moves money into escrow
- normal completion settles escrow to expert/platform split
- expert-cancel refunds member
- member-cancel settles escrow without refund
- expert-absent flow does not define escrow resolution

Resolved policy:

- do not settle escrow from end-call or auto-complete when consultation is `ExpertAbsent` / `ExpertAbsentHandled`
- keep escrow unresolved after member report until admin approval
- implemented: reverse/refund escrow to the member during admin approval
- implemented: prevent duplicate refund on repeated approval attempts

## Remaining Open Questions

### HR4: Admin approval note/report input

Question:

- should admin approval accept an admin-authored note/report field?

Current verified behavior:

- `POST /api/admin/consultations/{consultationId}/expert-absent/confirm-handled` has no request body
- `AdminConsultationsController.ConfirmExpertAbsentHandled(...)` only receives `consultationId`
- `ConfirmExpertAbsentHandledAsync(...)` only changes the consultation status today
- no admin note, handled-by, or handled-at field exists in the current endpoint contract

Decision needed before implementation only if the approval flow must capture admin notes.

## Change Log

### 2026-05-04

- Added closed decisions for expert-absent end-call protection
- Added open research items for refund, booking status, and escrow dispute handling
- Closed refund, booking terminal-status, and escrow-resolution research with admin-approval refund policy
- Added open research for optional admin approval note/report input
- Marked admin approval refund, booking `Refunded`, and repeat-approval idempotency as implemented
