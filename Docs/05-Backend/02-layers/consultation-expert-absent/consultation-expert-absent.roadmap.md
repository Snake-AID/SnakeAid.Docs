---
doc_role: planning
module: consultation-expert-absent
kind: layer
doc_type: roadmap
status: draft
last_updated: 2026-04-20
owners: [backend-team]
verification_status: current-state-code-verified-implementation-not-started
---

# Consultation Expert Absent Roadmap

## Current Status Snapshot

- module status: `Planning`
- current member absent-report API: `Not available`
- current consultation report field: `Not available`
- current admin report visibility: `Not available`
- current docs status: `Planning set created`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Code-verified reality:

- `ConsultationStatus` enum already contains `ExpertAbsent`, but no active write flow uses it.
- `Consultation` entity currently has no free-text absent report field.
- member consultation history endpoint is active but returns no absent report content.
- admin consultation history endpoints are active but return no absent report content.
- lifecycle auto-complete currently transitions elapsed consultations to `Completed`.

## Task Group Scope

Requested task group:

1. ConsultaionAbsent Task1: add `CustomerReport` field for member absent-expert reporting.
2. ConsultaionAbsent Task2: build API for member to submit report into `Member Report` input.
3. ConsultaionAbsent Task3: update admin consultation endpoint contract to include `CustomerReport`.

## Locked Implementation Direction (Current Draft)

- [x] Use a dedicated module folder under `Docs/05-Backend/02-layers/consultation-expert-absent`.
- [x] Keep docs English-first and self-resumable.
- [x] Keep planning explicit: clearly separate `current code-verified` vs `planned` contract.
- [ ] Finalize canonical naming (`CustomerReport` vs `MemberReport`) from open decision bucket.
- [ ] Finalize whether report submission also changes `Consultation.Status` to `ExpertAbsent` immediately.

## Implementation Checklist

### Phase 0 - Discovery and Planning Baseline

- [x] Verify current consultation domain/status usage in code
- [x] Verify current member consultation endpoint surface
- [x] Verify current admin consultation endpoint surface
- [x] Identify test suites affected by DTO/contract changes
- [x] Create planning docs set (`introduction/roadmap/hallucination/sourcecode/useguide`)

### Phase 1 - Task1 (`CustomerReport` storage)

- [ ] Add nullable `CustomerReport` field to `Consultation` domain model
- [ ] Add EF configuration constraint (max length) for the new field
- [ ] Create and apply migration for `SnakeAid.Consultations.CustomerReport`
- [ ] Decide and apply response exposure for member view (`MyConsultationResponse`)
- [ ] Ensure serialization naming aligns with API conventions

### Phase 2 - Task2 (Member absent-report API)

- [ ] Add request DTO for report submission (`MemberReport` input field name, pending final naming decision)
- [ ] Extend `IConsultationService` with report submission method
- [ ] Implement service validation:
  - [ ] caller must be consultation member
  - [ ] caller must be consultation owner side (member/caller), not expert
  - [ ] consultation must be in allowed status set
  - [ ] optional: enforce report time window relative to consultation start
- [ ] Persist report text to consultation report field
- [ ] Add controller endpoint for member report submission
- [ ] Return stable API envelope with updated consultation/report payload

### Phase 3 - Task3 (Admin response update)

- [ ] Add `CustomerReport` to `AdminConsultationResponse`
- [ ] Update Mapster mappings (`Consultation -> AdminConsultationResponse`)
- [ ] Ensure admin list endpoint includes report text
- [ ] Ensure admin detail endpoint includes report text
- [ ] Keep orphan scheduled/emergency fallback behavior unchanged

### Phase 4 - Test Coverage

- [ ] Unit test: controller route/auth metadata for new report endpoint
- [ ] Unit test: service rejects non-owner / unauthorized report attempts
- [ ] Integration test: member can submit absent report successfully
- [ ] Integration test: report appears in `GET /api/users/me/consultations`
- [ ] Integration test: report appears in both admin list and admin detail
- [ ] Regression test: existing price/status mapping behavior remains unchanged

### Phase 5 - Documentation Sync

- [x] Create planning docs for this module
- [ ] Update docs status to `in_progress` after implementation starts
- [ ] Update changelog and checkbox progress after each completed phase
- [ ] Mark module status as `implemented` only after tests pass

## Candidate Backend File Targets

- `SnakeAid.Core/Domains/Consultation.cs`
- `SnakeAid.Repository/Data/Configurations/ConsultationConfiguration.cs`
- `SnakeAid.Repository/Migrations/*`
- `SnakeAid.Core/Requests/Consultation/*` (new request DTO)
- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Api/Controllers/ConsultationsController.cs`
- `SnakeAid.Tests/Unit/*`
- `SnakeAid.Tests/Integration/AdminConsultationHistoryIntegrationTests.cs`
- `SnakeAid.Tests/Integration/ConsultationPricePreservationTests.cs`

## Validation Plan

Minimum acceptance flow:

1. member opens consultation and submits absent-expert report
2. backend persists report text on target consultation
3. member consultation list returns report text
4. admin list endpoint returns report text
5. admin detail endpoint returns same report text
6. unauthorized actors cannot submit report
7. existing consultation history fields still map correctly

## Resume Procedure

If resumed later:

1. read `consultation-expert-absent.introduction.md`
2. read `consultation-expert-absent.hallucination.md` and lock unresolved decisions
3. implement Phase 1 -> Phase 4 in order
4. after each completed phase, tick roadmap checklist and append changelog entry
5. update `consultation-expert-absent.useguide.md` from planned contract to active contract only after code/tests confirm

## Change Log

### 2026-04-20

- Initialized ConsultaionExpertAbsent planning module.
- Verified current backend gap: no member absent-report field/API, no admin report field.
- Created roadmap with phased implementation and resume-safe checklist.
