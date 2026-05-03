---
doc_role: planning
module: consultation-expert-absent
kind: engineering
doc_type: sourcecode
status: implemented-with-follow-up-planned
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-verified
---

# Consultation Expert Absent Sourcecode Notes

## Overview

This file captures:

- the current code-verified structure
- the recommended target structure
- the sequence of the planned member absent-report flow
- the follow-up implementation target for end-flow protection

## Current Code-Verified Structure

Relevant code-verified facts:

- `Consultation` owns:
  - `Id`
  - `CallerId`
  - `CalleeId`
  - `RoomId`
  - `StartTime`
  - `EndTime`
  - `Status`
  - `Type`
- `ConsultationBooking` owns scheduled-only booking details
- `MyConsultationResponse` is the current member history DTO
- `AdminConsultationResponse` is the current admin history/detail DTO
- `ConsultationService` owns member history and admin history logic
- `ConsultationsController` owns member consultation endpoints
- `AdminConsultationsController` owns admin consultation endpoints

## Current Class Diagram

```mermaid
classDiagram
    class Consultation {
        +Guid Id
        +Guid CallerId
        +Guid CalleeId
        +string RoomId
        +DateTime StartTime
        +DateTime? EndTime
        +ConsultationStatus Status
        +ConsultationType Type
    }

    class ConsultationBooking {
        +Guid Id
        +Guid UserId
        +Guid ExpertId
        +decimal Price
        +DateTime BookedAt
        +string? ProblemDescription
        +DateTime? PaymentDeadline
        +BookingStatus Status
        +Guid? ConsultationId
        +Guid TimeSlotId
    }

    class MyConsultationResponse {
        +Guid ConsultationId
        +string Type
        +string Status
        +Guid ExpertId
        +string? ExpertName
        +string? RoomId
        +DateTime? StartTime
        +DateTime? EndTime
        +decimal? Price
        +string? ProblemDescription
        +Guid? BookingId
        +DateTime? SlotStartTime
        +DateTime? SlotEndTime
        +Guid? EmergencyRequestId
    }

    class AdminConsultationResponse {
        +Guid ConsultationId
        +string Type
        +string Status
        +Guid UserId
        +string? UserName
        +Guid ExpertId
        +string? ExpertName
        +string? ProblemDescription
        +Guid? BookingId
        +Guid? EmergencyRequestId
    }

    ConsultationBooking --> Consultation : optional ConsultationId
```

## Recommended Target Class Diagram

Recommended direction:

- add `CustomerReport` to `Consultation`
- add `CustomerReportSubmittedAt` in v1
- project that field into member and admin DTOs
- add one request DTO for the member command endpoint
- add one admin command endpoint to mark the case as handled

```mermaid
classDiagram
    class Consultation {
        +Guid Id
        +Guid CallerId
        +Guid CalleeId
        +string RoomId
        +DateTime StartTime
        +DateTime? EndTime
        +ConsultationStatus Status
        +ConsultationType Type
        +string? CustomerReport
        +DateTime? CustomerReportSubmittedAt
    }

    class ReportExpertAbsentRequest {
        +string CustomerReport
    }

    class MyConsultationResponse {
        +Guid ConsultationId
        +string Type
        +string Status
        +string? CustomerReport
        +DateTime? CustomerReportSubmittedAt
    }

    class AdminConsultationResponse {
        +Guid ConsultationId
        +string Type
        +string Status
        +string? CustomerReport
        +DateTime? CustomerReportSubmittedAt
    }

    class IConsultationService {
        +ReportExpertAbsentAsync(Guid consultationId, Guid memberId, ReportExpertAbsentRequest request)
        +ConfirmExpertAbsentHandledAsync(Guid consultationId)
        +GetMyConsultationsAsync(Guid userId, MyConsultationsQueryRequest query)
        +GetAllConsultationsForAdminAsync(AdminConsultationsQueryRequest query)
        +GetConsultationByIdForAdminAsync(Guid consultationId)
    }

    class ConsultationsController {
        +POST /api/consultations/{consultationId}/expert-absent-report
        +GET /api/users/me/consultations
    }

    class AdminConsultationsController {
        +POST /api/admin/consultations/{consultationId}/expert-absent/confirm-handled
        +GET /api/admin/consultations
        +GET /api/admin/consultations/{consultationId}
    }

    ConsultationsController --> IConsultationService
    AdminConsultationsController --> IConsultationService
    IConsultationService --> Consultation
    IConsultationService --> MyConsultationResponse
    IConsultationService --> AdminConsultationResponse
```

## Planned Sequence Diagram

This sequence is implemented.

```mermaid
sequenceDiagram
    actor Member
    participant API as ConsultationsController
    participant Service as ConsultationService
    participant Repo as UnitOfWork/Repositories
    participant DB as Database

    Member->>API: POST /api/consultations/{consultationId}/expert-absent-report
    API->>Service: ReportExpertAbsentAsync(consultationId, memberId, request)
    Service->>Repo: Load Consultation by Id
    Repo->>DB: SELECT Consultation
    DB-->>Repo: Consultation
    Repo-->>Service: Consultation

    Service->>Service: Validate ownership and eligibility
    Service->>Service: Validate time >= StartTime
    Service->>Service: Reject if CustomerReport already exists
    Service->>Service: Set CustomerReport
    Service->>Service: Set CustomerReportSubmittedAt if used
    Service->>Service: Set Status = ExpertAbsent
    Service->>Repo: Update entity and save
    Repo->>DB: UPDATE Consultations
    DB-->>Repo: Saved
    Repo-->>Service: Success
    Service-->>API: Updated consultation object
    API-->>Member: ApiResponse(updated consultation)
```

## Admin Read Sequence After Implementation

```mermaid
sequenceDiagram
    actor Admin
    participant API as AdminConsultationsController
    participant Service as ConsultationService
    participant Repo as UnitOfWork/Repositories
    participant Mapper as AdminConsultationMapper

    Admin->>API: GET /api/admin/consultations/{consultationId}
    API->>Service: GetConsultationByIdForAdminAsync(consultationId)
    Service->>Repo: Load Consultation + linked data
    Repo-->>Service: Consultation + Booking/Ping data
    Service->>Mapper: Build AdminConsultationResponse
    Mapper-->>Service: DTO including CustomerReport and CustomerReportSubmittedAt
    Service-->>API: AdminConsultationResponse
    API-->>Admin: ApiResponse<AdminConsultationResponse>
```

## Admin Resolution Sequence

```mermaid
sequenceDiagram
    actor Admin
    participant API as AdminConsultationsController
    participant Service as ConsultationService
    participant Repo as UnitOfWork/Repositories
    participant DB as Database

    Admin->>API: POST /api/admin/consultations/{consultationId}/expert-absent/confirm-handled
    API->>Service: ConfirmExpertAbsentHandledAsync(consultationId)
    Service->>Repo: Load Consultation by Id
    Repo->>DB: SELECT Consultation
    DB-->>Repo: Consultation
    Repo-->>Service: Consultation
    Service->>Service: Validate Status == ExpertAbsent
    Service->>Service: Set Status = ExpertAbsentHandled
    Service->>Repo: Update entity and save
    Repo->>DB: UPDATE Consultations
    DB-->>Repo: Saved
    Service-->>API: Updated AdminConsultationResponse
    API-->>Admin: ApiResponse(updated consultation)
```

## Follow-up Sequence: Mobile Ends Call After Expert-Absent Report

This sequence is implemented and code-verified.

```mermaid
sequenceDiagram
    actor Member
    participant API as ConsultationsController
    participant Service as ConsultationService
    participant SignalR as ConsultationHub
    participant LiveKit as LiveKitService
    participant Repo as UnitOfWork/Repositories
    participant DB as Database

    Member->>API: POST /api/consultations/{consultationId}/end
    API->>Service: EndConsultationAsync(consultationId, memberId)
    Service->>Repo: Load Consultation
    Repo-->>Service: Consultation(Status = ExpertAbsent)
    Service->>Service: Validate actor is participant
    Service->>Service: Detect ExpertAbsent or ExpertAbsentHandled
    Service->>SignalR: Send ConsultationCallEnded signal if available
    Service->>LiveKit: Delete room if available
    Service->>Service: Preserve Status = ExpertAbsent
    Service->>Service: Set EndTime if null
    Service->>Repo: Update Consultation only
    Repo->>DB: UPDATE Consultations
    DB-->>Repo: Saved
    API-->>Member: Success
```

Implementation notes:

- do not set `Consultation.Status = Completed` when current status is `ExpertAbsent` or `ExpertAbsentHandled`
- do set `EndTime` for expert-absent calls when the call is ended
- do not set `ConsultationBooking.Status = Completed` from this expert-absent end-call path
- do not call `SettleConsultationEscrowAsync(...)` from this expert-absent end-call path
- keep room cleanup behavior separate from business completion

### Important Extension Point For Future Side-effect Work

The implemented guard in `ConsultationService.EndConsultationAsync(...)` is the intentional extension point for future expert-absent side effects.

Current shape:

```csharp
if (consultation.Status is ConsultationStatus.ExpertAbsent or ConsultationStatus.ExpertAbsentHandled)
{
    consultation.EndTime ??= DateTime.UtcNow;
    consultationRepo.Update(consultation);
    await _unitOfWork.CommitAsync();
    return;
}
```

When the open side-effect research is closed, update this branch directly.

Do:

- add the approved expert-absent payment or booking-state handling inside this branch or delegate from this branch to a clearly named helper
- keep the normal `Completed` path below this branch reserved for successful consultation completion
- keep tests that prove `ExpertAbsent` and `ExpertAbsentHandled` are not overwritten by `Completed`
- add new tests for any approved refund, settlement, booking-status, or admin-resolution side effect

Do not:

- remove the guard and rely on later code to undo `Completed`
- let execution fall through into the normal `Completed` flow for expert-absent cases
- add an outer workaround in the controller to compensate for service behavior
- create a second end-call path that duplicates room cleanup
- settle escrow or mutate booking state from `/end` unless that exact behavior has been approved by the follow-up research

Reason:

- this branch separates runtime call cleanup from business completion
- future side effects must preserve that boundary instead of patching over it later

## Follow-up Sequence: Scheduled Auto-complete Denylist

This sequence is planned for the follow-up implementation.

```mermaid
sequenceDiagram
    participant Job as ConsultationLifecycleBackgroundService
    participant Booking as BookingService
    participant Repo as UnitOfWork/Repositories

    Job->>Booking: AutoCompleteElapsedScheduledConsultationsAsync()
    Booking->>Repo: Query elapsed confirmed bookings
    Repo-->>Booking: Bookings where Consultation.Status not in denylist
    Booking->>Booking: Denylist includes Completed, ExpertAbsent, ExpertAbsentHandled
    Booking->>Booking: Complete only remaining consultations
```

Implementation notes:

- keep the existing denylist style
- extend the current denylist from `Completed` to include `ExpertAbsent` and `ExpertAbsentHandled`
- do not auto-complete expert-absent cases
- do not settle expert-absent escrow from scheduled auto-complete

## Notes For Resume

If implementation resumes later, inspect these code areas first:

- `SnakeAid.Core/Domains/Consultation.cs`
- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Api/Controllers/ConsultationsController.cs`
- `SnakeAid.Service/Implements/BookingService.cs`
- `SnakeAid.Service/Implements/ConsultationLifecycleBackgroundService.cs`

## Metadata Recommendation

For v1, the preferred model is:

- `CustomerReport`
- `CustomerReportSubmittedAt`

Avoid putting admin-resolution fields into `Consultation` until there is an actual admin resolution workflow.

Current implemented admin resolution is intentionally minimal:

- status-only resolution through `ExpertAbsentHandled`
- no extra admin note field
- no handled-by / handled-at metadata yet

## Follow-up Open Research

These topics are intentionally not part of the end-flow protection patch:

- whether `ConfirmExpertAbsentHandledAsync(...)` should also refund the member, settle the expert, or accept an admin-selected payment outcome
- whether `ConsultationBooking.Status` should remain `Confirmed` for expert-absent cases or move to a future dedicated terminal status
- whether a dedicated payment-dispute state is needed for scheduled consultation escrow

Track these questions in `consultation-expert-absent.hallucination.md` before implementing payment or booking-status changes.
