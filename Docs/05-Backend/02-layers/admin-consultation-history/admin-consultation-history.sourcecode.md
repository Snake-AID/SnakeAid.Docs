---
doc_role: planning
module: admin-consultation-history
kind: layer
doc_type: sourcecode
status: implemented
last_updated: 2026-04-13
owners: [backend-team]
verification_status: code-verified
---

# Admin Consultation History Sourcecode

## 1. Relevant Classes

- `ConsultationsController`
- `ExpertController`
- `IConsultationService`
- `ConsultationService`
- `Consultation`
- `ConsultationBooking`
- `ConsultationPingRequest`
- `Transaction`
- `MyConsultationsQueryRequest`
- `MyConsultationResponse`
- `ExpertConsultationResponse`
- `AdminConsultationsController`
- `AdminConsultationsQueryRequest`
- `AdminConsultationResponse`
- `AdminConsultationHistoryIntegrationTests`
- `AdminConsultationsControllerTests`

## 2. Implemented New Classes

- `AdminConsultationsController`
- `AdminConsultationsQueryRequest`
- `AdminConsultationResponse`

## 3. Target Class Diagram

```mermaid
classDiagram
    class AdminConsultationsController {
        +GetAllConsultations(query)
    }

    class IConsultationService {
        +GetMyConsultationsAsync(userId, query)
        +GetExpertConsultationsAsync(expertId, query)
        +GetAllConsultationsForAdminAsync(query)
    }

    class ConsultationService {
        -IUnitOfWork _unitOfWork
        -ILogger _logger
        +GetAllConsultationsForAdminAsync(query)
    }

    class AdminConsultationsQueryRequest {
        +int PageNumber
        +int PageSize
        +string? Status
        +string? Type
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
        +Guid? EmergencyRequestId
        +DateTime? SlotStartTime
        +DateTime? SlotEndTime
    }

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
        +string? ProblemDescription
        +Guid? ConsultationId
        +Guid TimeSlotId
    }

    class ConsultationPingRequest {
        +Guid Id
        +Guid RescuerId
        +Guid ExpertId
        +ConsultationPingStatus Status
        +Guid? ConsultationId
    }

    class Transaction {
        +Guid ReferenceId
        +TransactionType TransactionType
        +decimal Amount
    }

    AdminConsultationsController --> IConsultationService
    ConsultationService ..|> IConsultationService
    ConsultationService --> ConsultationBooking
    ConsultationService --> ConsultationPingRequest
    ConsultationService --> Consultation
    ConsultationService --> Transaction
    AdminConsultationsController --> AdminConsultationsQueryRequest
    ConsultationService --> AdminConsultationsQueryRequest
    ConsultationService --> AdminConsultationResponse
```

## 4. Admin List Sequence Diagram

```mermaid
sequenceDiagram
    participant AdminApp as Admin App
    participant Controller as AdminConsultationsController
    participant Service as ConsultationService
    participant BookingRepo as ConsultationBooking Repo
    participant PingRepo as ConsultationPingRequest Repo
    participant TxRepo as Transaction Repo

    AdminApp->>Controller: GET /api/admin/consultations?status=Completed&type=Emergency&pageNumber=1&pageSize=10
    Controller->>Controller: Authorize Admin role
    Controller->>Service: GetAllConsultationsForAdminAsync(query)

    alt include Scheduled
        Service->>BookingRepo: Get scheduled bookings with Consultation + User + Expert + TimeSlot
        BookingRepo-->>Service: scheduled rows
        Service->>Service: map to AdminConsultationResponse
    end

    alt include Emergency
        Service->>PingRepo: Get accepted emergency requests with Consultation + Rescuer + Expert
        PingRepo-->>Service: emergency rows
        Service->>TxRepo: Get matching transactions for emergency pricing
        TxRepo-->>Service: transaction rows
        Service->>Service: map to AdminConsultationResponse
    end

    alt emergency consultation fallback
        Service->>PingRepo: verify no linked ping request exists
        Service->>Service: map from Consultation + Caller + Callee
    end

    Service->>Service: merge scheduled + emergency
    Service->>Service: sort by StartTime desc
    Service->>Service: paginate and build PagingResponse
    Service-->>Controller: PagingResponse<AdminConsultationResponse>
    Controller-->>AdminApp: ApiResponse<PagingResponse<AdminConsultationResponse>>
```

## 5. Mapping Rules

### Scheduled Consultation

Sources:
- `ConsultationBooking`
- `Consultation`
- `Account User`
- `Account Expert`
- `ExpertTimeSlot`

Primary mapping:
- `ConsultationId` <- `ConsultationBooking.ConsultationId`
- `Type` <- `"Scheduled"`
- `Status` <- `Consultation.Status.ToString()`
- `UserId` <- `ConsultationBooking.UserId`
- `UserName` <- `ConsultationBooking.User.FullName`
- `ExpertId` <- `ConsultationBooking.ExpertId`
- `ExpertName` <- `ConsultationBooking.Expert.FullName`
- `Price` <- `ConsultationBooking.Price`
- `ProblemDescription` <- `ConsultationBooking.ProblemDescription`
- `BookingId` <- `ConsultationBooking.Id`
- `SlotStartTime` <- `TimeSlot.StartTime`
- `SlotEndTime` <- `TimeSlot.EndTime`

### Emergency Consultation

Sources:
- `ConsultationPingRequest`
- `Consultation`
- `Account Rescuer`
- `Account Expert`
- `Transaction`

Primary mapping:
- `ConsultationId` <- `Consultation.Id`
- `Type` <- `"Emergency"`
- `Status` <- `Consultation.Status.ToString()`
- `UserId` <- `ConsultationPingRequest.RescuerId`
- `UserName` <- `ConsultationPingRequest.Rescuer.FullName`
- `ExpertId` <- `ConsultationPingRequest.ExpertId`
- `ExpertName` <- `ConsultationPingRequest.Expert.FullName`
- `EmergencyRequestId` <- `ConsultationPingRequest.Id`
- `Price` <- first matching transaction by rule:
  - prefer `TransactionType = ConsultationPayment`, `ReferenceId = ConsultationPingRequest.Id`
  - fallback `TransactionType = ExpertPayout`, `ReferenceId = Consultation.Id`

### Orphan Scheduled Consultation

Sources:
- `Consultation`
- `Account Caller`
- `Account Callee`

Primary mapping:
- included only when `Consultation.Type = Scheduled`
- included only when there is no matching `ConsultationBooking`
- `Price` <- `null`
- `BookingId` <- `null`

### Orphan Emergency Consultation

Sources:
- `Consultation`
- `Account Caller`
- `Account Callee`
- `Transaction`

Primary mapping:
- included only when `Consultation.Type = Emergency`
- included only when there is no matching `ConsultationPingRequest`
- `EmergencyRequestId` <- `null`
- `Price` <- latest `ExpertPayout` by `Consultation.Id` when available, otherwise `null`

## 6. Implementation Notes

1. The new controller does not need to be added into `ConsultationsController`
   - existing admin modules in the repo prefer separate controllers such as `AdminWithdrawalsController`
2. Admin response should not be forced into an actor-specific shape
   - that would lose half of the context
3. Status filter should parse from the domain enum `ConsultationStatus`
4. Type filter should parse from `ConsultationType`
5. Pagination should continue to use `PaginationRequest`
6. Pagination helper and status parsing are shared across admin/user/expert history methods in `ConsultationService`

## 7. Code-Verified Tests

- `AdminConsultationHistoryIntegrationTests`
  - scheduled item maps `BookingId`, slot times, and problem description correctly
  - emergency item maps `EmergencyRequestId` correctly
  - emergency price prefers `ConsultationPayment` and falls back to `ExpertPayout`
  - orphan emergency consultation is still returned through `Consultation` fallback
  - no duplicate item is returned when a consultation already exists in the main path
  - sort uses `StartTime desc`
  - `PageNumber`, `PageSize`, `TotalItems`, `TotalPages` are correct
- `AdminConsultationsControllerTests`
  - route is `api/admin/consultations`
  - controller requires role `Admin`
  - action exposes `HttpGet`
  - action returns `ApiResponse<PagingResponse<AdminConsultationResponse>>`
