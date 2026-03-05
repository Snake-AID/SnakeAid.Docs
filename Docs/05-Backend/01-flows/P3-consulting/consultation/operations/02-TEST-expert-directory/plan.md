---
doc_role: operation
operation_id: 02-TEST-expert-directory
type: TEST
status: done
created_at: 2026-03-05
affects:
  - SnakeAid.Tests/Integration/ExpertControllerIntegrationTests.cs
  - SnakeAid.Tests/Unit/ExpertServiceTests.cs
  - SnakeAid.Tests/SnakeAid.Tests.csproj
  - SnakeAid.Backend.sln
---

# Plan: Runtime Test Coverage for Consultation Operation 1

## 1. As-Is

Operation `01-INIT-expert-directory` is implemented and build passes, but there is no focused automated test suite to verify runtime behavior for critical expert-directory flows.

## 2. Gap Analysis

- No focused tests for operation-1 critical path (`bulk time slots`, `reviews`, `time slots`).
- No unit tests for `ExpertService.CreateBulkTimeSlotsAsync` covering deduplication and UTC validation.
- No baseline guard to ensure `ExpertTimeSlot` unique composite index is present in EF model metadata.

## 3. To-Be Design

Implement targeted test coverage for Operation 1:

- **Integration tests**:
  - Controller-level `CreateBulkTimeSlots` with valid UTC payload returns success and persists slots.
  - Controller-level `CreateBulkTimeSlots` with non-UTC `weekStartDate` throws `ValidationException`.
  - Controller-level `GetExpertReviews` returns consultation feedback only.
  - Controller-level `GetExpertTimeSlots` returns future + available slots only.
- **Unit tests**:
  - In-request overlap deduplication in `CreateBulkTimeSlotsAsync`.
  - Existing-slot overlap skip behavior.
  - UTC normalization rule enforcement.
  - `ExpertTimeSlot` unique composite index (`ExpertId`, `StartTime`, `EndTime`) presence in model.

## 4. Impacted Components

- **Tests**: `SnakeAid.Tests/Integration/ExpertControllerIntegrationTests.cs`, `SnakeAid.Tests/Unit/ExpertServiceTests.cs`.
- **Test Infrastructure**: in-memory `SnakeAidDbContext` per test, explicit seed for account/profile/feedback.

## 5. Risks & Constraints

- Controller integration scope does not validate full HTTP middleware behavior (e.g., centralized exception-to-422 mapping).
- Integration tests must seed deterministic data and always use UTC values to avoid timezone drift.
- Runtime test scope should remain focused on Operation 1 to avoid coupling with planned Operations 03/04/05.

## 6. Validation Plan

- Run full test suite locally and in CI.
- Verify newly added tests fail against intentionally broken behavior and pass on correct implementation.
- Keep test outputs deterministic with fixed seed data and explicit cleanup between cases.


