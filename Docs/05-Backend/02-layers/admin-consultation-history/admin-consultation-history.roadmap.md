---
doc_role: planning
module: admin-consultation-history
kind: flow
doc_type: roadmap
status: completed
last_updated: 2026-04-13
owners: [backend-team]
verification_status: implementation-verified
---

# Admin Consultation History Roadmap

## Current Status Snapshot

- Planning status: `Completed`
- Code status: admin consultation list endpoint `is implemented`
- Docs status: implementation docs are synchronized
- Main objective: `GET /api/admin/consultations` delivered

## Target Outcome

Admin can call:

`GET /api/admin/consultations?pageNumber=1&pageSize=10&type=Emergency&status=Completed`

and receive:
- `ApiResponse<PagingResponse<AdminConsultationResponse>>`
- unified data from scheduled + emergency consultations
- both user and expert information

## Implementation Checklist

### Phase 1 - Contract And Shapes

- [x] Create `AdminConsultationsQueryRequest`
- [x] Create `AdminConsultationResponse`
- [x] Finalize enum / string conventions for `Type` and `Status`
- [x] Finalize route `GET /api/admin/consultations`

### Phase 2 - Service Surface

- [x] Add a new method to `IConsultationService`
- [x] Add implementation to `ConsultationService`
- [x] Query scheduled consultations for admin
- [x] Query emergency consultations for admin
- [x] Merge, sort, paginate

### Phase 3 - API Layer

- [x] Create `AdminConsultationsController`
- [x] Add `[Authorize(Roles = "Admin")]`
- [x] Return `ApiResponseBuilder.BuildSuccessResponse(...)`
- [x] Add response metadata / `ProducesResponseType`

### Phase 4 - Tests

- [x] Route/auth attribute test for admin consultation endpoint
- [x] Service test for scheduled mapping
- [x] Service test for emergency mapping
- [x] Service test for `Status` filter
- [x] Service test for `Type` filter
- [x] Service test for sort order
- [x] Service test for pagination metadata
- [x] Controller response envelope test

### Phase 5 - Documentation Sync

- [x] Update `useguide` from `planned` to `active` after the endpoint is implemented
- [x] Fill the `Verified Endpoint List`
- [x] Update changelog with final route and response contract

## Proposed File Targets

### Backend

- [x] `SnakeAid.Core/Requests/Consultation/AdminConsultationsQueryRequest.cs`
- [x] `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- [x] `SnakeAid.Service/Interfaces/IConsultationService.cs`
- [x] `SnakeAid.Service/Implements/ConsultationService.cs`
- [x] `SnakeAid.Api/Controllers/AdminConsultationsController.cs`
- [x] `SnakeAid.Tests/Integration/AdminConsultationHistoryIntegrationTests.cs`
- [x] `SnakeAid.Tests/Unit/AdminConsultationsControllerTests.cs`

### Docs

- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/admin-consultation-history/admin-consultation-history.introduction.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/admin-consultation-history/admin-consultation-history.roadmap.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/admin-consultation-history/admin-consultation-history.sourcecode.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/admin-consultation-history/admin-consultation-history.useguide.md`

## Verification Summary

- `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "FullyQualifiedName~AdminConsultationHistoryIntegrationTests|FullyQualifiedName~AdminConsultationsControllerTests" -p:TreatWarningsAsErrors=false -p:NuGetAudit=false` passed
- `dotnet build SnakeAid.Service/SnakeAid.Service.csproj -p:TreatWarningsAsErrors=false -p:NuGetAudit=false` passed

## Decisions Already Made

- Do not reuse `MyConsultationResponse`
- Do not reuse `ExpertConsultationResponse`
- Prefer route pattern consistent with existing admin modules:
  - `api/admin/consultations`
- Keep the existing response envelope pattern:
  - `ApiResponse<PagingResponse<T>>`
- Default sort:
  - `StartTime desc`

## Decisions Closed During Implementation

- [x] `userId` / `expertId` filters are not included in v1
- [x] `ProblemDescription` remains scheduled-only
- [x] raw source statuses such as `BookingStatus` / `ConsultationPingStatus` are not exposed
- [x] DB-level optimization is deferred; v1 keeps in-memory merge + paginate
- [x] emergency price uses `ConsultationPayment` first and `ExpertPayout` as fallback
- [x] orphan scheduled consultations are included with `Price = null`

## Validation Notes

Implemented verification:

- endpoint is restricted to role `Admin`
- pagination metadata is correct
- status filter accepts actual domain enum values
- `Price` is mapped correctly for both scheduled and emergency records
- no regression is introduced for existing user/expert consultation history APIs

## Change Log

### 2026-04-13

- Initialized roadmap for the `admin get all consultations in the system` use case
- Proposed route `GET /api/admin/consultations`
- Chose a dedicated admin DTO instead of reusing actor-specific responses
- Marked roadmap as completed after implementation and test verification
