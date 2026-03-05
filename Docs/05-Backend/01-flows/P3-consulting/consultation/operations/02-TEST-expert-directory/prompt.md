---
doc_role: operation
operation_id: 02-TEST-expert-directory
generated_from: plan.md
status: done
created_at: 2026-03-05
---

# Prompt: Implement Runtime Test Coverage for Consultation Operation 1 (Controller + Service)

## Requirements

Create automated tests that validate runtime behavior for the completed expert-directory baseline.

Specific tasks:

1. Add integration tests for:
   - Controller action `CreateBulkTimeSlots` success with UTC payload.
   - Controller action `CreateBulkTimeSlots` failure with non-UTC payload (`ValidationException`).
   - Controller action `GetExpertReviews` returning only `FeedbackType.Consultation`.
   - Controller action `GetExpertTimeSlots` returning future + available slots only.
2. Add unit tests for `ExpertService.CreateBulkTimeSlotsAsync`:
   - Deduplicate overlapping blocks within the same request payload.
   - Skip slots already existing in DB range.
   - Reject non-UTC `WeekStartDate`.
   - Assert unique composite index exists for `ExpertTimeSlot`.
3. Ensure test seed data includes:
   - Expert account/profile.
   - At least one consultation feedback and one non-consultation feedback for filtering validation.
4. Ensure test execution is deterministic (isolated in-memory DB state and explicit setup).

## Constraints

- Do not change consultation business behavior in this operation; only add runtime verification.
- Keep tests scoped to Operation 1 controller/service logic.
- Use UTC timestamps in all test payloads and assertions.


