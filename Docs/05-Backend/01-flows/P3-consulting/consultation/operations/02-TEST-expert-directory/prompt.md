---
doc_role: operation
operation_id: 02-TEST-expert-directory
generated_from: plan.md
status: draft
created_at: 2026-03-05
---

# Prompt: Implement Runtime Test Coverage for Consultation Operation 1

## Requirements

Create automated tests that validate runtime behavior for the completed expert-directory baseline.

Specific tasks:

1. Add integration tests for:
   - `POST /api/v1/experts/me/time-slots/bulk` success with UTC payload.
   - `POST /api/v1/experts/me/time-slots/bulk` failure with non-UTC payload (`422`).
   - Parallel duplicate-slot creation flow producing conflict (`409`) for at least one request.
   - `GET /api/v1/experts/{expertId}/reviews` returning only `FeedbackType.Consultation`.
2. Add unit tests for `ExpertService.CreateBulkTimeSlotsAsync`:
   - Deduplicate overlapping blocks within the same request payload.
   - Skip slots already existing in DB range.
   - Reject non-UTC `WeekStartDate`.
3. Ensure test seed data includes:
   - Expert account/profile.
   - At least one consultation feedback and one non-consultation feedback for filtering validation.
4. Ensure test execution is deterministic (isolated DB state and explicit setup/teardown).

## Constraints

- Do not change consultation business behavior in this operation; only add runtime verification.
- Keep tests scoped to Operation 1 endpoints/service logic.
- Use UTC timestamps in all test payloads and assertions.


