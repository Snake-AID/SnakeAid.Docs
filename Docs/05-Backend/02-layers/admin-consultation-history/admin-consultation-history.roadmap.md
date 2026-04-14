---
doc_role: planning
module: admin-consultation-history
kind: flow
doc_type: roadmap
status: implemented
last_updated: 2026-04-15
owners: [backend-team]
verification_status: code-verified
---

# Admin Consultation History Roadmap

## Current Status Snapshot

- module status: `Implemented`
- code status:
  - `GET /api/admin/consultations` is implemented
  - `GET /api/admin/consultations/{consultationId}` is implemented
- contract status:
  - both endpoints use `AdminConsultationResponse`
- verification status:
  - controller tests and integration tests are present

## Target Outcome

Admin can:

1. Browse consultations with:
   - `GET /api/admin/consultations`
2. Open one consultation with:
   - `GET /api/admin/consultations/{consultationId}`

Both endpoints return the same normalized consultation shape:

- list:
  - `ApiResponse<PagingResponse<AdminConsultationResponse>>`
- single item:
  - `ApiResponse<AdminConsultationResponse>`

## Implementation Checklist

### Contract And Shapes

- [x] Create `AdminConsultationsQueryRequest`
- [x] Create `AdminConsultationResponse`
- [x] Finalize enum / string conventions for `Type` and `Status`
- [x] Standardize list and single-item payloads on one DTO
- [x] Register admin consultation mapping in `SnakeAid.Core/Mappings`

### Service Surface

- [x] Add `GetAllConsultationsForAdminAsync(...)`
- [x] Add `GetConsultationByIdForAdminAsync(...)`
- [x] Query scheduled consultations for admin
- [x] Query emergency consultations for admin
- [x] Merge, sort, paginate for list
- [x] Use consultation-first lookup for single item

### API Layer

- [x] Create `AdminConsultationsController`
- [x] Add `[Authorize(Roles = "Admin")]`
- [x] Implement `GET /api/admin/consultations`
- [x] Implement `GET /api/admin/consultations/{consultationId}`
- [x] Return `ApiResponseBuilder.BuildSuccessResponse(...)`

### Tests

- [x] Route/auth attribute test for admin consultation controller
- [x] Controller response envelope test for list
- [x] Controller response envelope test for single item
- [x] Service test for scheduled list mapping
- [x] Service test for emergency list mapping
- [x] Service test for list sort order
- [x] Service test for list pagination metadata
- [x] Service test for scheduled single-item mapping
- [x] Service test for emergency single-item mapping
- [x] Service test for orphan consultation behavior
- [x] Service test for not-found behavior

### Documentation Sync

- [x] Align `useguide` with active routes
- [x] Align `sourcecode` with the unified DTO contract
- [x] Align `introduction` with implemented scope
- [x] Remove outdated `planned extension` framing

## File Targets

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

- `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "AdminConsultationHistoryIntegrationTests|AdminConsultationsControllerTests"` passed

## Closed Decisions

- [x] Use dedicated admin DTO instead of reusing actor-specific consultation DTOs
- [x] Keep route pattern consistent with `api/admin/...`
- [x] Keep default sort as `StartTime desc`
- [x] Keep in-memory merge + paginate for v1
- [x] Use one shared admin response shape for both list and single item
- [x] Keep orphan consultation fallback instead of hiding missing-linked-record cases

## Remaining Risks

1. In-memory merge + paginate may need optimization later
2. Emergency price still depends on transaction availability
3. Scheduled and emergency consultations still come from different business-side sources even though the API contract is unified

## Change Log

### 2026-04-13

- Initialized roadmap for admin consultation history
- Implemented list endpoint contract and test coverage

### 2026-04-14

- Implemented single-item endpoint
- Unified list and single-item contracts on `AdminConsultationResponse`
- Consolidated documentation away from `main + extension` framing

### 2026-04-15

- Moved admin consultation Mapster registration into `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- Updated docs to reflect unified response shape in both list and single-item examples
