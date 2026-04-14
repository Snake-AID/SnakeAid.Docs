---
doc_role: planning
module: admin-consultation-history
kind: layer
doc_type: introduction
status: implemented
last_updated: 2026-04-13
owners: [backend-team]
verification_status: code-verified
---

# Admin Consultation History Introduction

## Goal

This document defines the implemented backend scope for the `admin get all consultations in the system` use case.

Business goal:
- Admin can view consultation history across the whole system through a single endpoint
- Admin can filter by `status` and `type`
- Admin can see enough operational data:
  - user information
  - expert information
  - consultation type
  - consultation status
  - start / end timestamps
  - consultation price when available
  - related booking or emergency request

## Current Codebase Status

The codebase already has 2 related history API groups:

- User history:
  - `GET /api/users/me/consultations`
  - `ConsultationsController.GetMyConsultations(...)`
  - `IConsultationService.GetMyConsultationsAsync(...)`
- Expert history:
  - `GET /api/experts/me/consultations`
  - `ExpertController.GetMyConsultations(...)`
  - `IConsultationService.GetExpertConsultationsAsync(...)`

The codebase now has:
- `GET /api/admin/consultations`
- `AdminConsultationsController`
- `GetAllConsultationsForAdminAsync(...)` in `IConsultationService`
- dedicated request DTO / response DTO for admin consultation history
- service integration tests and controller attribute/response tests for admin consultation history

## Code-Verified Data Sources

Current consultation history is assembled from multiple sources:

1. `Consultation`
   - central record for room, type, status, start/end time
2. `ConsultationBooking`
   - source for scheduled consultation data
   - contains `Price`, `ProblemDescription`, `TimeSlotId`, `UserId`, `ExpertId`
3. `ConsultationPingRequest`
   - source for emergency consultation data
   - links the consultation created after expert acceptance
4. `Transaction`
   - used to derive `Price` for emergency consultation records

## Important Current Behavior

The implemented admin history service keeps these characteristics:

- Scheduled and Emergency consultations are queried separately, then merged into one list
- Admin history is `booking/ping-first`:
  - Scheduled uses `ConsultationBooking` as the main source
  - Emergency uses `ConsultationPingRequest` as the main source
- Current sort order is `StartTime desc`
- Current pagination is calculated after merging the list in memory
- Scheduled price comes from `ConsultationBooking.Price`
- Emergency price comes from `Transaction`:
  - admin history: prefer `TransactionType = ConsultationPayment`, `ReferenceId = ConsultationPingRequest.Id`
  - admin history fallback: `TransactionType = ExpertPayout`, `ReferenceId = Consultation.Id`
- Admin history includes edge-case handling for scheduled consultations that have a `Consultation` record but no `ConsultationBooking`
- Admin history includes edge-case handling for emergency consultations that have a `Consultation` record but no `ConsultationPingRequest`

## Implemented Design

Implemented v1 direction:

1. Add a dedicated admin endpoint
   - proposed route: `GET /api/admin/consultations`
   - role: `Admin`
2. Add a dedicated query DTO
   - `AdminConsultationsQueryRequest : PaginationRequest`
   - minimum fields:
     - `Status`
     - `Type`
3. Add a dedicated admin response DTO
   - do not reuse `MyConsultationResponse` or `ExpertConsultationResponse`
   - reason:
     - admin needs both `User` and `Expert`
     - admin needs one normalized shape for scheduled and emergency records
4. Extend `IConsultationService`
   - add `GetAllConsultationsForAdminAsync(...)`
5. Implement service logic using the existing pattern
   - query scheduled consultations from `ConsultationBooking`
   - query emergency consultations from `ConsultationPingRequest`
   - add `Consultation` fallback for scheduled edge cases
   - add `Consultation` fallback for emergency edge cases
   - map to `AdminConsultationResponse`
   - merge, sort, paginate
6. Add tests
   - controller route/auth attributes + response envelope
   - service mapping / filter / pagination / sort

## Why A Separate Admin DTO

`MyConsultationResponse` and `ExpertConsultationResponse` are both actor-specific:

- user view only returns `ExpertId`, `ExpertName`
- expert view only returns `UserId`, `UserName`

Admin view needs:
- `UserId`, `UserName`
- `ExpertId`, `ExpertName`
- `Type`
- `Status`
- `RoomId`
- `StartTime`, `EndTime`
- `Price`
- `ProblemDescription` for scheduled consultation when available
- `BookingId` for scheduled consultation
- `EmergencyRequestId` for emergency consultation
- `SlotStartTime`, `SlotEndTime` for scheduled consultation

## Scope Boundary

In scope:
- admin list consultations across the whole system
- pagination
- filters by `status` and `type`
- API contract for admin app / mobile developers to read

Out of scope:
- admin get consultation detail in the current implemented scope
- admin update consultation status
- CSV / Excel export
- free-text search
- analytics / dashboard aggregates

## Next Planned Scope

The next additive scope for this module is:

- `GET /api/admin/consultations/{consultationId}`

Goal:
- Admin can open one consultation from the list screen and retrieve a richer detail payload for that specific consultation

Why this is a separate step:
- the current list endpoint is optimized for merged history browsing
- a detail endpoint should be `consultation-first`, not `list-first`
- detail view can safely expose more source-specific metadata than the list payload

## Planned Direction For Get One Consultation

Recommended route:
- `GET /api/admin/consultations/{consultationId}`

Recommended service surface:
- add `GetConsultationByIdForAdminAsync(Guid consultationId)`

Recommended response shape:
- do not reuse `AdminConsultationResponse` directly as the final detail shape
- prefer a dedicated detail DTO such as `AdminConsultationDetailResponse`

Reason:
- the current list DTO is intentionally compact
- a detail endpoint can include source-specific fields that should not bloat the list response

Recommended detail-only fields to consider:
- `BookingStatus`
- `BookedAt`
- `PaymentDeadline`
- `CancelledAt`
- `CancellationReason`
- `EmergencyRequestStatus`
- `RequestedAt`
- `RespondedAt`
- `ExpiresAt`

## Key Design Insight

The list endpoint is currently `booking/ping-first` and only falls back to `Consultation` for orphan cases.

The detail endpoint should invert that:

1. load `Consultation` by `consultationId` first
2. inspect `Consultation.Type`
3. enrich from the matching `ConsultationBooking` or `ConsultationPingRequest`
4. derive price using the same source rules already documented for the list endpoint

This keeps the detail implementation direct and avoids scanning the entire merged history pipeline just to return one record.

## Remaining Risks

1. In-memory merge + paginate
   - acceptable for v1
   - should be monitored when consultation volume grows
2. Emergency price source differs from scheduled price source
   - must be documented clearly to avoid incorrect mapping
3. Actual status enum is broader than the comment in `MyConsultationsQueryRequest`
   - current domain enum includes:
     - `Scheduled`
     - `Ongoing`
     - `Completed`
     - `Cancelled`
     - `UserAbsent`
     - `ExpertAbsent`
     - `AllAbsent`
4. Route convention
   - implemented consistently with the existing `api/admin/...` pattern used in the repo
5. Admin god-eye behavior is pragmatic, not consultation-first
   - main path still follows business sources first
   - edge cases are backfilled from `Consultation`
6. Detail contract creep
   - adding too many source-specific fields to the existing list DTO would make the list endpoint heavier and less stable
   - the detail endpoint should isolate that expansion in a separate response type

## Delivered Artifacts

- `admin-consultation-history.roadmap.md`
- `admin-consultation-history.sourcecode.md`
- `admin-consultation-history.useguide.md`
- implementation code
- test coverage for service and controller surface
- booking/ping-first admin history with consultation fallback for edge cases

## Planned Follow-Up Artifact

- implementation plan for `GET /api/admin/consultations/{consultationId}`
