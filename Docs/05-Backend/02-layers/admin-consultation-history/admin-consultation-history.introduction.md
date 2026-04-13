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
- Current sort order is `StartTime desc`
- Current pagination is calculated after merging the list in memory
- Scheduled price comes from `ConsultationBooking.Price`
- Emergency price comes from `Transaction`:
  - admin history: prefer `TransactionType = ConsultationPayment`, `ReferenceId = ConsultationPingRequest.Id`
  - admin history fallback: `TransactionType = ExpertPayout`, `ReferenceId = Consultation.Id`
- Admin history includes edge-case handling for scheduled consultations that have a `Consultation` record but no `ConsultationBooking`

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
   - query scheduled consultations
   - query emergency consultations
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
- admin get consultation detail
- admin update consultation status
- CSV / Excel export
- free-text search
- analytics / dashboard aggregates

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

## Delivered Artifacts

- `admin-consultation-history.roadmap.md`
- `admin-consultation-history.sourcecode.md`
- `admin-consultation-history.useguide.md`
- implementation code
- test coverage for service and controller surface
