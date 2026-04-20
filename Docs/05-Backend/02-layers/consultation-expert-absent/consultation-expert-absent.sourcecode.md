---
doc_role: planning
module: consultation-expert-absent
kind: engineering
doc_type: sourcecode
status: proposed
last_updated: 2026-04-21
owners: [backend-team]
verification_status: mixed
---

# Consultation Expert Absent Sourcecode Notes

## Overview

This file captures:

- the current code-verified structure
- the recommended target structure
- the sequence of the planned member absent-report flow

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
- optionally add `CustomerReportSubmittedAt` in v1
- project that field into member and admin DTOs
- add one request DTO for the member command endpoint

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
    }

    class AdminConsultationResponse {
        +Guid ConsultationId
        +string Type
        +string Status
        +string? CustomerReport
    }

    class IConsultationService {
        +ReportExpertAbsentAsync(Guid consultationId, Guid memberId, ReportExpertAbsentRequest request)
        +GetMyConsultationsAsync(Guid userId, MyConsultationsQueryRequest query)
        +GetAllConsultationsForAdminAsync(AdminConsultationsQueryRequest query)
        +GetConsultationByIdForAdminAsync(Guid consultationId)
    }

    class ConsultationsController {
        +POST /api/consultations/{consultationId}/expert-absent-report
        +GET /api/users/me/consultations
    }

    ConsultationsController --> IConsultationService
    IConsultationService --> Consultation
    IConsultationService --> MyConsultationResponse
    IConsultationService --> AdminConsultationResponse
```

## Planned Sequence Diagram

This sequence is proposed, not yet implemented.

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
    Mapper-->>Service: DTO including CustomerReport
    Service-->>API: AdminConsultationResponse
    API-->>Admin: ApiResponse<AdminConsultationResponse>
```

## Notes For Resume

If implementation resumes later, inspect these code areas first:

- `SnakeAid.Core/Domains/Consultation.cs`
- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Api/Controllers/ConsultationsController.cs`

## Metadata Recommendation

For v1, the preferred model is:

- `CustomerReport`
- `CustomerReportSubmittedAt`

Avoid putting admin-resolution fields into `Consultation` until there is an actual admin resolution workflow.
