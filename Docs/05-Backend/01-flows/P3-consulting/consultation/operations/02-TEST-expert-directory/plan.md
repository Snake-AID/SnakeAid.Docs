---
doc_role: operation
operation_id: 02-TEST-expert-directory
type: FEAT
status: draft
created_at: 2026-03-05
affects:
  - Tests/Consultation/ExpertControllerIntegrationTests.cs
  - Tests/Consultation/ExpertServiceUnitTests.cs
  - Tests/Shared/TestWebApplicationFactory.cs
---

# Plan: Runtime Test Coverage for Consultation Operation 1

## 1. As-Is

Operation `01-INIT-expert-directory` is implemented and build passes, but there is no focused automated test suite to verify runtime behavior for critical expert-directory flows.

## 2. Gap Analysis

- No integration tests for consultation expert endpoints (`settings`, `bulk time slots`, `experts list`, `profile`, `reviews`, `time slots`).
- No unit tests for `ExpertService.CreateBulkTimeSlotsAsync` covering deduplication and UTC validation.
- No concurrent runtime test to verify conflict behavior when duplicate slot creation happens at the same time.

## 3. To-Be Design

Implement targeted test coverage for Operation 1:

- **Integration tests**:
  - `POST /api/v1/experts/me/time-slots/bulk` with valid UTC payload returns success and persists slots.
  - `POST /api/v1/experts/me/time-slots/bulk` with non-UTC `weekStartDate` returns `422`.
  - Concurrent duplicate insert path returns `409` for at least one racing request.
  - `GET /api/v1/experts/{expertId}/reviews` returns consultation feedback only.
- **Unit tests**:
  - In-request overlap deduplication in `CreateBulkTimeSlotsAsync`.
  - Existing-slot overlap skip behavior.
  - UTC normalization rule enforcement.

## 4. Impacted Components

- **Tests**: consultation-focused integration and unit test classes.
- **Test Infrastructure**: shared test factory / seeded fixtures for accounts, expert profile, and feedback data.

## 5. Risks & Constraints

- Concurrency tests can be flaky if test DB setup is not isolated per test.
- Integration tests must seed deterministic data and always use UTC values to avoid timezone drift.
- Runtime test scope should remain focused on Operation 1 to avoid coupling with planned Operations 03/04/05.

## 6. Validation Plan

- Run full test suite locally and in CI.
- Verify newly added tests fail against intentionally broken behavior and pass on correct implementation.
- Keep test outputs deterministic with fixed seed data and explicit cleanup between cases.


