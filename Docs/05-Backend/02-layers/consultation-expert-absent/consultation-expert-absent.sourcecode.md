---
doc_role: planning
module: consultation-expert-absent
kind: layer
doc_type: sourcecode
status: draft
last_updated: 2026-04-20
owners: [backend-team]
verification_status: current-state-code-verified-target-design-drafted
---

# Consultation Expert Absent Sourcecode

## 1. Relevant Classes and Files

### Current Backend Classes

- `Consultation`
- `ConsultationStatus`
- `ConsultationsController`
- `AdminConsultationsController`
- `IConsultationService`
- `ConsultationService`
- `MyConsultationResponse`
- `AdminConsultationResponse`
- `AdminConsultationMapper`
- `BookingService`
- `ConsultationLifecycleBackgroundService`
- `VideoCallController`

### Planned Additions

- report request DTO for member submission
- report response projection updates for member/admin contracts
- service method to persist absent report

## 2. Code-Verified Current Surface

### Current HTTP Endpoints (Relevant)

Member-facing:

- `GET /api/users/me/consultations`
- `POST /api/consultations/{consultationId}/video-token`
- `POST /api/consultations/{consultationId}/end`

Admin-facing:

- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`

Current gap:

- no endpoint to submit absent-expert report
- no report field in current member/admin consultation response DTOs

### Current Lifecycle Behavior

- elapsed scheduled consultation -> `Completed`
- elapsed emergency consultation -> `Completed`
- no active path currently sets `ExpertAbsent`

## 3. As-Is Class Diagram

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

    class ConsultationStatus {
        <<enumeration>>
        Scheduled
        Ongoing
        Completed
        Cancelled
        UserAbsent
        ExpertAbsent
        AllAbsent
    }

    class ConsultationsController {
        +EndConsultation(consultationId)
        +GetMyConsultations(query)
        +CreateReview(consultationId, request)
        +GetReview(consultationId)
    }

    class AdminConsultationsController {
        +GetAllConsultations(query)
        +GetConsultationById(consultationId)
    }

    class IConsultationService {
        +EndConsultationAsync(consultationId, actorId)
        +GetMyConsultationsAsync(userId, query)
        +GetAllConsultationsForAdminAsync(query)
        +GetConsultationByIdForAdminAsync(consultationId)
    }

    class ConsultationService {
        +GetMyConsultationsAsync(userId, query)
        +GetAllConsultationsForAdminAsync(query)
        +GetConsultationByIdForAdminAsync(consultationId)
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
        +string? RoomId
        +DateTime? StartTime
        +DateTime? EndTime
        +decimal? Price
        +string? ProblemDescription
        +Guid? BookingId
        +string? BookingStatus
        +DateTime? BookedAt
        +DateTime? PaymentDeadline
        +DateTime? CancelledAt
        +CancellationReason? CancellationReason
        +Guid? EmergencyRequestId
        +string? EmergencyRequestStatus
        +DateTime? RequestedAt
        +DateTime? RespondedAt
        +DateTime? ExpiresAt
        +DateTime? SlotStartTime
        +DateTime? SlotEndTime
    }

    ConsultationsController --> IConsultationService
    AdminConsultationsController --> IConsultationService
    ConsultationService ..|> IConsultationService
    ConsultationService --> Consultation
    ConsultationService --> MyConsultationResponse
    ConsultationService --> AdminConsultationResponse
```

## 4. Planned To-Be Class Diagram

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
    }

    class ReportExpertAbsentRequest {
        +string MemberReport
    }

    class ReportExpertAbsentResponse {
        +Guid ConsultationId
        +string? CustomerReport
        +DateTime UpdatedAt
    }

    class ConsultationsController {
        +ReportExpertAbsent(consultationId, request)
        +GetMyConsultations(query)
        +EndConsultation(consultationId)
    }

    class IConsultationService {
        +ReportExpertAbsentAsync(consultationId, memberId, request)
        +GetMyConsultationsAsync(userId, query)
        +GetAllConsultationsForAdminAsync(query)
        +GetConsultationByIdForAdminAsync(consultationId)
    }

    class ConsultationService {
        +ReportExpertAbsentAsync(consultationId, memberId, request)
        +GetMyConsultationsAsync(userId, query)
        +GetAllConsultationsForAdminAsync(query)
        +GetConsultationByIdForAdminAsync(consultationId)
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

    class AdminConsultationMapper {
        +Map Consultation to AdminConsultationResponse
    }

    ConsultationsController --> IConsultationService
    ConsultationService ..|> IConsultationService
    ConsultationService --> Consultation
    ConsultationService --> ReportExpertAbsentResponse
    ConsultationService --> MyConsultationResponse
    ConsultationService --> AdminConsultationResponse
    AdminConsultationMapper --> AdminConsultationResponse
```

## 5. Sequence Diagrams

### 5.1 Planned Member Report Flow (Task2)

```mermaid
sequenceDiagram
    participant MemberApp as Member App
    participant API as ConsultationsController
    participant Service as ConsultationService
    participant ConsultationRepo as Consultation Repository
    participant DB as Database

    MemberApp->>API: POST /api/consultations/{consultationId}/expert-absence-report
    API->>Service: ReportExpertAbsentAsync(consultationId, memberId, request)
    Service->>ConsultationRepo: load consultation by id
    ConsultationRepo-->>Service: consultation

    Service->>Service: validate ownership + status + time window
    alt validation fails
        Service-->>API: throw validation/forbidden exception
        API-->>MemberApp: error response
    else valid
        Service->>ConsultationRepo: set CustomerReport from request.MemberReport
        Service->>DB: CommitAsync
        Service-->>API: ReportExpertAbsentResponse
        API-->>MemberApp: 200 success envelope
    end
```

### 5.2 Planned Admin Visibility Flow (Task3)

```mermaid
sequenceDiagram
    participant AdminApp as Admin App
    participant API as AdminConsultationsController
    participant Service as ConsultationService
    participant Repos as Consultation + Booking/Ping Repositories

    AdminApp->>API: GET /api/admin/consultations
    API->>Service: GetAllConsultationsForAdminAsync(query)
    Service->>Repos: load consultations and related records
    Repos-->>Service: merged rows
    Service->>Service: map AdminConsultationResponse including CustomerReport
    Service-->>API: paging response
    API-->>AdminApp: items with customerReport field
```

### 5.3 Current Gap Flow (As-Is)

```mermaid
sequenceDiagram
    participant MemberApp as Member App
    participant API as Existing Consultation APIs

    MemberApp->>API: join consultation room and wait for expert
    MemberApp->>API: need to report expert absence
    Note over MemberApp,API: No dedicated report endpoint exists currently
```

## 6. Task-to-Code Impact Map

### Task1 - Add `CustomerReport`

- `Consultation` domain: add nullable report field
- EF configuration: set column constraint
- migration: add new column in `SnakeAid.Consultations`
- member/admin response contracts: include report output

### Task2 - Build member report API

- request DTO
- new controller action in `ConsultationsController`
- new service method in `IConsultationService` and `ConsultationService`
- validation and persistence logic

### Task3 - Update admin endpoints

- `AdminConsultationResponse` adds `CustomerReport`
- `AdminConsultationMapper` updates mapping
- list/detail methods include and preserve report value

## 7. Test Focus

- report submit success for valid member
- report submit forbidden for non-owner/non-participant
- report submit rejected for invalid status/time window
- admin list/detail include report value
- member consultation list includes report value
- existing mapping fields and price behavior unchanged
