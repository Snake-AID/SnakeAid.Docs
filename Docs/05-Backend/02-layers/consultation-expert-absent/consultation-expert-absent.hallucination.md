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

## Open Research Questions

### HR1: Refund or settlement policy for expert-absent cases

Question:

- when admin handles an `ExpertAbsent` case, should the system refund the member, settle the expert, or let admin choose?

Current verified behavior:

- `ReportExpertAbsentAsync(...)` does not refund
- `ConfirmExpertAbsentHandledAsync(...)` only changes status to `ExpertAbsentHandled`
- scheduled booking cancel docs define refund for expert-cancel and settlement for member-cancel, but do not define expert-absent payment resolution

Do not implement:

- automatic refund from report submission
- automatic settlement from end-call
- payment changes inside `ConfirmExpertAbsentHandledAsync(...)`

Research needed:

- expected admin workflow
- financial policy for no-show expert
- audit fields needed for payment resolution
- whether admin should provide reason or evidence notes

### HR2: Final booking status for expert-absent cases

Question:

- should `ConsultationBooking.Status` remain `Confirmed`, move to an existing status, or gain a dedicated expert-absent/disputed status?

Current verified behavior:

- paid scheduled booking becomes `Confirmed`
- normal end or elapsed slot can move booking to `Completed`
- cancel flow can move booking to `Cancelled`
- `BookingStatus.Refunded` exists but current scheduled refund flow records refund transactions and does not use it as a booking state in the observed code

Current follow-up decision:

- do not set booking to `Completed` from expert-absent end-call or auto-complete
- leave booking status unchanged during this narrow patch

Research needed:

- whether admin history and mobile history need a booking-level terminal state for expert-absent cases
- whether existing `Cancelled` or `Refunded` would misrepresent the absent-report flow
- whether a new status such as `Disputed` or `ExpertAbsent` is needed

### HR3: Escrow dispute state

Question:

- should scheduled consultation escrow have a visible dispute state when consultation is `ExpertAbsent`?

Current verified behavior:

- scheduled payment moves money into escrow
- normal completion settles escrow to expert/platform split
- expert-cancel refunds member
- member-cancel settles escrow without refund
- expert-absent flow does not define escrow resolution

Current follow-up decision:

- do not settle escrow from end-call or auto-complete when consultation is `ExpertAbsent` / `ExpertAbsentHandled`
- keep escrow unresolved until a dedicated payment-resolution design exists

Research needed:

- payment status shown to member/admin
- admin operations needed to resolve escrow
- idempotency and duplicate-resolution rules

## Change Log

### 2026-05-04

- Added closed decisions for expert-absent end-call protection
- Added open research items for refund, booking status, and escrow dispute handling
