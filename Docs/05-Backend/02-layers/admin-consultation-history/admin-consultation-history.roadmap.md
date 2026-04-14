---
doc_role: planning
module: admin-consultation-history
kind: flow
doc_type: roadmap
status: active
last_updated: 2026-04-13
owners: [backend-team]
verification_status: implemented-list-plus-planned-detail
---

# Admin Consultation History Roadmap

## Current Status Snapshot

- Planning status:
  - list endpoint: `Completed`
  - detail endpoint: `Planned`
- Code status:
  - `GET /api/admin/consultations` is implemented
  - `GET /api/admin/consultations/{consultationId}` does not exist yet
- Docs status: implementation docs are synchronized for list, and planning has been added for detail
- Main objective:
  - delivered: `GET /api/admin/consultations`
  - next: `GET /api/admin/consultations/{consultationId}`

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

## Next Milestone - Get One Consultation

Target endpoint:

`GET /api/admin/consultations/{consultationId}`

Target outcome:
- admin can open one consultation from the list screen
- backend returns one consultation detail payload
- payload remains normalized but can contain richer source metadata than the list item

### Phase 6 - Contract And Shape For Detail

- [ ] Decide whether to create `AdminConsultationDetailResponse`
- [ ] Keep `AdminConsultationResponse` as the list DTO
- [ ] Finalize route `GET /api/admin/consultations/{consultationId}`
- [ ] Finalize 404 behavior for missing consultation

### Phase 7 - Service Surface For Detail

- [ ] Add `GetConsultationByIdForAdminAsync(Guid consultationId)` to `IConsultationService`
- [ ] Implement a consultation-first lookup in `ConsultationService`
- [ ] Load related booking when `Type = Scheduled`
- [ ] Load related ping request when `Type = Emergency`
- [ ] Reuse emergency pricing resolution order already implemented for the list endpoint

### Phase 8 - API Layer For Detail

- [ ] Add `GetConsultationById(Guid consultationId)` to `AdminConsultationsController`
- [ ] Add `[HttpGet("{consultationId:guid}")]`
- [ ] Add `ProducesResponseType` for `200` and `404`
- [ ] Keep role restriction as `Admin`

### Phase 9 - Tests For Detail

- [ ] Controller attribute/route test for detail endpoint
- [ ] Controller response envelope test for detail endpoint
- [ ] Service test for scheduled detail mapping
- [ ] Service test for emergency detail mapping
- [ ] Service test for not-found behavior
- [ ] Service test for orphan scheduled consultation detail
- [ ] Service test for orphan emergency consultation detail

### Phase 10 - Documentation Sync For Detail

- [ ] Update `useguide` only after the detail endpoint is implemented
- [ ] Add detail endpoint to `Verified Endpoint List`
- [ ] Add request/response example for detail endpoint
- [ ] Update changelog with final detail contract

## Decisions Already Made

- Do not reuse `MyConsultationResponse`
- Do not reuse `ExpertConsultationResponse`
- Prefer route pattern consistent with existing admin modules:
  - `api/admin/consultations`
- Keep the existing response envelope pattern:
  - `ApiResponse<PagingResponse<T>>`
- Default sort:
  - `StartTime desc`
- Keep the active `useguide` limited to code-verified endpoints only

## Decisions Closed During Implementation

- [x] `userId` / `expertId` filters are not included in v1
- [x] `ProblemDescription` remains scheduled-only
- [x] raw source statuses such as `BookingStatus` / `ConsultationPingStatus` are not exposed
- [x] DB-level optimization is deferred; v1 keeps in-memory merge + paginate
- [x] emergency price uses `ConsultationPayment` first and `ExpertPayout` as fallback
- [x] orphan scheduled consultations are included with `Price = null`

## Proposed Decisions For Detail Endpoint

- [ ] Prefer `GET /api/admin/consultations/{consultationId}` over a query-parameter detail route
- [ ] Prefer a dedicated `AdminConsultationDetailResponse`
- [ ] Keep detail lookup `consultation-first`
- [ ] Keep not-found handling consistent with existing consultation service behavior

## Validation Notes

Implemented verification:

- endpoint is restricted to role `Admin`
- pagination metadata is correct
- status filter accepts actual domain enum values
- `Price` is mapped correctly for both scheduled and emergency records
- no regression is introduced for existing user/expert consultation history APIs

Planned verification for detail:

- detail endpoint returns one record by `consultationId`
- scheduled detail includes booking metadata when present
- emergency detail includes ping metadata when present
- orphan consultations are still retrievable through `Consultation`
- missing `consultationId` returns `404`

## Change Log

### 2026-04-13

- Initialized roadmap for the `admin get all consultations in the system` use case
- Proposed route `GET /api/admin/consultations`
- Chose a dedicated admin DTO instead of reusing actor-specific responses
- Marked roadmap as completed after implementation and test verification

### 2026-04-14

- Added next-phase planning for `GET /api/admin/consultations/{consultationId}`
- Chose detail planning direction as consultation-first lookup
- Kept active `useguide` unchanged until detail endpoint is implemented
