---
doc_role: planning
module: consultation-expert-absent
kind: layer
doc_type: introduction
status: implemented-with-follow-up-planned
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-verified
---

# Consultation Expert Absent Introduction

## Goal

This document defines the implementation target for the `ConsultaionExpertAbsent` workstream.

Business goal:

- when a scheduled consultation reaches video-call start time
- the member joins the room
- the expert does not appear
- the member must have a backend-supported way to report the absent expert
- admin must be able to see that report and handle the case
- admin must be able to confirm that the absent-expert case has been handled

## Scope Covered By This Module

The requested scope is currently split into three tasks:

1. `ConsultaionAbsent Task1`
   - expose a member-originated report field in the member-facing consultation surface
2. `ConsultaionAbsent Task2`
   - build an API that lets the member submit the absent-expert report
3. `ConsultaionAbsent Task3`
   - extend the admin consultation endpoint so admin can see the report

## Current Codebase Status

The current codebase already has these relevant consultation surfaces:

- member consultation history:
  - `GET /api/users/me/consultations`
- expert consultation history:
  - `GET /api/experts/me/consultations`
- admin consultation history:
  - `GET /api/admin/consultations`
  - `GET /api/admin/consultations/{consultationId}`

Code-verified observations from the backend:

- `Consultation.Status` already contains `ExpertAbsent`
- `Consultation` now contains:
  - `CustomerReport`
  - `CustomerReportSubmittedAt`
- `ConsultationBooking` currently does `not` contain any absent-report field
- `MyConsultationResponse` now exposes:
  - `CustomerReport`
  - `CustomerReportSubmittedAt`
- `AdminConsultationResponse` now exposes:
  - `CustomerReport`
  - `CustomerReportSubmittedAt`
- `IConsultationService` now exposes a member report command
- `IConsultationService` now exposes an admin handled-confirmation command
- `ConsultationsController` now provides a member report endpoint
- `AdminConsultationsController` now provides an admin handled-confirmation endpoint

## Recommended Implementation Direction

Recommended direction for this module:

- keep the absent-report data on `Consultation`
- expose that data outward through response DTOs
- add one member-only command endpoint under `api/consultations`
- keep admin visibility inside the existing admin consultation endpoints
- add one admin-only command endpoint under `api/admin/consultations`

Why this direction fits the current code:

- admin detail already starts from `Consultation`
- both member and admin views already project from consultation-centric flows
- `Consultation.Status` already owns the `ExpertAbsent` state
- storing the report on `Consultation` avoids duplicating the same business fact across `ConsultationBooking` and admin projections

## Confirmed Baseline Decisions

- canonical field name in baseline docs:
  - `Customer Report`
- persistence location:
  - `Consultation`
- report action result:
  - persist `Customer Report`
  - set `Consultation.Status = ExpertAbsent`
- eligibility window:
  - any time after `StartTime`
- duplicate behavior:
  - reject repeated submissions
- command response style:
  - return updated consultation object

## Remaining Design Question

The only meaningful design area still open is audit metadata.

That is now closed for v1.

Confirmed v1 metadata:

- `CustomerReport`
- `CustomerReportSubmittedAt`

Still out of current scope:

- `CustomerReportSubmittedBy`
- admin resolution metadata fields

## Implemented Backend Shape

Implemented shape:

- persistence:
  - add `CustomerReport` on `Consultation`
  - add `CustomerReportSubmittedAt` in v1
- member command:
  - add one request DTO for reporting absent expert
  - add one service method to validate and persist the report
  - add one controller endpoint for the member
  - return the updated consultation object
- member read model:
  - extend `MyConsultationResponse` with `Customer Report`
- admin read model:
  - extend `AdminConsultationResponse` with `Customer Report`
- tests:
  - integration tests for member report flow
  - integration tests for admin response mapping
  - controller tests for new endpoint authorization and envelope shape

Implemented endpoint:

- `POST /api/consultations/{consultationId}/expert-absent-report`

## Suggested Scope Boundary

In scope:

- member can submit absent-expert report for a consultation
- report is stored and retrievable by admin
- report submission changes consultation status to `ExpertAbsent`
- member-facing consultation payload can show the report field if required by mobile
- admin consultation list/detail includes the report field
- admin can move `Consultation.Status` from `ExpertAbsent` to `ExpertAbsentHandled`
- follow-up implementation must protect `ExpertAbsent` and `ExpertAbsentHandled` from being overwritten by consultation end flows

Out of scope for this module unless explicitly requested later:

- expert penalty workflow
- notification workflow
- auto-detecting absence from video-room presence
- emergency consultation absent-report flow
- admin approval note/report capture unless explicitly added to the follow-up contract

## Follow-up Implementation Decision: End-Flow Protection

The current follow-up implementation target is narrow:

- after a member submits an expert-absent report, mobile may call the normal end-consultation endpoint to close the call
- backend must not treat that end-call action as a successful consultation completion
- `EndConsultationAsync` must preserve `Consultation.Status = ExpertAbsent` or `ExpertAbsentHandled`
- `EndConsultationAsync` should still set `EndTime` when ending an expert-absent call
- `EndConsultationAsync` may still perform runtime cleanup such as SignalR end-call notification and LiveKit room deletion
- scheduled auto-complete must use a denylist that excludes at least `Completed`, `ExpertAbsent`, and `ExpertAbsentHandled` from completion
- auto-complete must not convert an expert-absent case into `Completed`

Refund and booking terminal-state policy is now decided for the next follow-up implementation:

- report submission does not refund the member
- admin approval through `ConfirmExpertAbsentHandledAsync(...)` refunds the member in the same transaction/flow
- approved expert-absent cases set `Consultation.Status = ExpertAbsentHandled`
- approved expert-absent refunds set `ConsultationBooking.Status = Refunded`
- escrow is refunded/reversed to the member and not settled to the expert
- repeat approval returns the current state without creating a duplicate refund when already handled/refunded

Current code verification: the admin handled-confirmation endpoint has no request body, so admin-authored note/report input remains a separate decision.

## File Areas Likely To Change

### Backend

- `SnakeAid.Core/Domains/Consultation.cs`
- `SnakeAid.Repository/Data/Configurations/ConsultationConfiguration.cs`
- `SnakeAid.Core/Requests/Consultation/*`
- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Api/Controllers/ConsultationsController.cs`
- `SnakeAid.Tests/Integration/*Consultation*.cs`
- `SnakeAid.Tests/Unit/*Consultation*.cs`

### Docs

- `consultation-expert-absent.introduction.md`
- `consultation-expert-absent.roadmap.md`
- `consultation-expert-absent.hallucination.md`
- `consultation-expert-absent.sourcecode.md`
- `consultation-expert-absent.useguide.md`
