---
doc_role: planning
module: consultation-expert-absent
kind: flow
doc_type: roadmap
status: proposed
last_updated: 2026-04-21
owners: [backend-team]
verification_status: mixed
---

# Consultation Expert Absent Roadmap

## Current Status Snapshot

- module status: `Planned`
- code status:
  - no absent-report command endpoint exists yet
  - no consultation report field exists yet
  - admin consultation responses do not yet expose a report field
- docs status:
  - this doc set is initialized for resume-safe implementation tracking

## Target Outcome

After implementation:

1. member can report an expert as absent for a consultation
2. backend persists the member-authored report
3. member-facing consultation payload exposes the report field if required
4. admin consultation endpoints expose the report field
5. the feature is traceable in tests and docs

## Task Breakdown

### Task 1

`ConsultaionAbsent Task1 - Expert absent: add field Customer Report for member report`

Planning interpretation:

- add one nullable report field to the member-facing consultation response
- make sure the field is populated from the canonical persistence source
- do not mark the field as active in `useguide` until code is implemented

Likely code targets:

- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`

### Task 2

`ConsultaionAbsent Task2 - build API for member to write into Member Report`

Planning interpretation:

- add a member-only endpoint under `api/consultations`
- validate ownership
- validate consultation type and state
- prevent duplicate or invalid reports depending on final business rule
- update consultation state if required by the chosen rule

Likely code targets:

- `SnakeAid.Core/Requests/Consultation/*`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Api/Controllers/ConsultationsController.cs`

### Task 3

`ConsultaionAbsent Task3 - update admin endpoint to include Customer Report`

Planning interpretation:

- extend `AdminConsultationResponse`
- extend admin mapping logic
- verify both list and detail endpoints expose the same field

Likely code targets:

- `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Tests/Integration/AdminConsultationHistoryIntegrationTests.cs`

## Recommended Implementation Order

1. Add persistence fields + migration
2. Add request DTO and service method
3. Add member endpoint
4. Extend member response DTO and mapping
5. Extend admin response DTO and mapping
6. Add tests
7. Update docs with verified contracts

## Implementation Checklist

### Requirement Lock

- [x] Use `Customer Report` as the baseline field name in docs
- [x] Store the report field on `Consultation`
- [x] Set `Consultation.Status = ExpertAbsent` on successful report
- [x] Allow reporting any time after `StartTime`
- [x] Reject repeated report submissions
- [x] Return updated consultation object from the command endpoint
- [x] Include `CustomerReportSubmittedAt` in v1
- [x] Exclude additional metadata from v1 baseline

### Persistence

- [ ] Add nullable `CustomerReport` field to `Consultation`
- [ ] Add nullable `CustomerReportSubmittedAt` field to `Consultation`
- [ ] Configure max length and column mapping if needed
- [ ] Create EF migration
- [ ] Verify migration naming and backward compatibility

### Service Surface

- [ ] Add a member absent-report command to `IConsultationService`
- [ ] Load consultation with ownership validation
- [ ] Validate actor is the caller/member of that consultation
- [ ] Validate current time is after `StartTime`
- [ ] Reject duplicate reporting when `CustomerReport` already exists
- [ ] Persist `CustomerReport`
- [ ] Persist `CustomerReportSubmittedAt` if included
- [ ] Persist status change to `ExpertAbsent`

### API Layer

- [ ] Add request DTO
- [ ] Add member endpoint in `ConsultationsController`
- [ ] Add auth requirement `User`
- [ ] Return `ApiResponseBuilder.BuildSuccessResponse(...)` with updated consultation object

### Read Models

- [ ] Extend `MyConsultationResponse` with `CustomerReport`
- [ ] Populate member report field in `GetMyConsultationsAsync(...)`
- [ ] Extend `AdminConsultationResponse` with `CustomerReport`
- [ ] Populate admin report field in list and detail flows

### Tests

- [ ] Controller auth/envelope tests for new member endpoint
- [ ] Service test for successful report submission
- [ ] Service test for unauthorized actor
- [ ] Service test for invalid consultation state
- [ ] Service test for duplicate report behavior
- [ ] Admin integration test for list payload including report field
- [ ] Admin integration test for detail payload including report field
- [ ] Member integration test for history payload including report field

### Docs

- [ ] Update `useguide` only after the endpoint and fields are code-verified
- [ ] Update `sourcecode` diagrams after implementation is stable
- [ ] Record all decisions in `hallucination` as closed or resolved

## Recommended File Targets

### Backend

- [ ] `SnakeAid.Core/Domains/Consultation.cs`
- [ ] `SnakeAid.Repository/Data/Configurations/ConsultationConfiguration.cs`
- [ ] `SnakeAid.Repository/Migrations/*`
- [ ] `SnakeAid.Core/Requests/Consultation/ReportExpertAbsentRequest.cs` or equivalent
- [ ] `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- [ ] `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- [ ] `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- [ ] `SnakeAid.Service/Interfaces/IConsultationService.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationService.cs`
- [ ] `SnakeAid.Api/Controllers/ConsultationsController.cs`
- [ ] `SnakeAid.Tests/Integration/AdminConsultationHistoryIntegrationTests.cs`
- [ ] `SnakeAid.Tests/Integration/*Consultation*.cs`
- [ ] `SnakeAid.Tests/Unit/*Consultation*.cs`

### Docs

- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-expert-absent/consultation-expert-absent.introduction.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-expert-absent/consultation-expert-absent.roadmap.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-expert-absent/consultation-expert-absent.hallucination.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-expert-absent/consultation-expert-absent.sourcecode.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-expert-absent/consultation-expert-absent.useguide.md`

## Risks

1. Allowing reporting any time after `StartTime` is broad and may require later safeguards if very old consultations should be blocked.
2. The current consultation history implementation merges scheduled and emergency paths separately, so the new field must be populated consistently in both list/detail branches.

## Resume Notes

If implementation resumes later, start with these facts:

- `Consultation.Status` already has `ExpertAbsent`
- no report field exists yet on `Consultation` or `ConsultationBooking`
- member history uses `MyConsultationResponse`
- admin history uses `AdminConsultationResponse`
- admin list/detail logic lives in `ConsultationService`
- member history logic also lives in `ConsultationService`
- confirmed baseline field name is `Customer Report`
- confirmed persistence target is `Consultation`
- confirmed command should return updated consultation object

## Change Log

### 2026-04-21

- Initialized resumable planning docs for `ConsultaionExpertAbsent`
- Verified the current backend does not yet implement absent-report persistence or API surface
- Proposed `Consultation` as the canonical storage location for the report field
- Recorded confirmed decisions OD1 to OD6
- Closed OD7 with v1 metadata = `CustomerReport` + `CustomerReportSubmittedAt`
