---
doc_role: planning
module: admin-consultation-history
kind: layer
doc_type: sourcecode
status: implemented
last_updated: 2026-04-14
last_updated: 2026-04-15
owners: [backend-team]
verification_status: code-verified
---

# Admin Consultation History Sourcecode

## 1. Relevant Classes

- `AdminConsultationsController`
- `IConsultationService`
- `ConsultationService`
- `AdminConsultationsQueryRequest`
- `AdminConsultationResponse`
- `AdminConsultationMapper`
- `Consultation`
- `ConsultationBooking`
- `ConsultationPingRequest`
- `ExpertTimeSlot`
- `Transaction`
- `AdminConsultationHistoryIntegrationTests`
- `AdminConsultationsControllerTests`

## 2. Implemented Backend Surface

### Controller

- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`

### Service

- `GetAllConsultationsForAdminAsync(AdminConsultationsQueryRequest query)`
- `GetConsultationByIdForAdminAsync(Guid consultationId)`

### Mapping

- `AdminConsultationMapper`
- `MapsterConfig.RegisterMappings()`

### Shared Response Contract

Both endpoints use:

- `AdminConsultationResponse`

The difference is only payload cardinality:

- list endpoint:
  - `ApiResponse<PagingResponse<AdminConsultationResponse>>`
- single-item endpoint:
  - `ApiResponse<AdminConsultationResponse>`

## 3. Class Diagram

```mermaid
classDiagram
    class AdminConsultationsController {
        +GetAllConsultations(query)
        +GetConsultationById(consultationId)
    }

    class IConsultationService {
        +GetAllConsultationsForAdminAsync(query)
        +GetConsultationByIdForAdminAsync(consultationId)
    }

    class ConsultationService {
        -IUnitOfWork _unitOfWork
        -ILogger _logger
        +GetAllConsultationsForAdminAsync(query)
        +GetConsultationByIdForAdminAsync(consultationId)
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
        +string? BookingStatus
        +DateTime? BookedAt
        +DateTime? PaymentDeadline
        +DateTime? CancelledAt
        +string? CancellationReason
        +Guid? EmergencyRequestId
        +string? EmergencyRequestStatus
        +DateTime? RequestedAt
        +DateTime? RespondedAt
        +DateTime? ExpiresAt
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
        +DateTime BookedAt
        +DateTime? PaymentDeadline
        +BookingStatus Status
        +DateTime? CancelledAt
        +string? CancellationReason
        +string? ProblemDescription
        +Guid? ConsultationId
        +Guid TimeSlotId
    }

    class ConsultationPingRequest {
        +Guid Id
        +Guid RescuerId
        +Guid ExpertId
        +ConsultationPingStatus Status
        +DateTime RequestedAt
        +DateTime? RespondedAt
        +DateTime? ExpiresAt
        +Guid? ConsultationId
    }

    class ExpertTimeSlot {
        +Guid Id
        +DateTime StartTime
        +DateTime EndTime
    }

    class Transaction {
        +Guid ReferenceId
        +TransactionType TransactionType
        +decimal Amount
        +DateTime CreatedAt
    }

    AdminConsultationsController --> IConsultationService
    ConsultationService ..|> IConsultationService
    ConsultationService --> Consultation
    ConsultationService --> ConsultationBooking
    ConsultationService --> ConsultationPingRequest
    ConsultationService --> ExpertTimeSlot
    ConsultationService --> Transaction
    ConsultationService --> AdminConsultationMapper
    AdminConsultationsController --> AdminConsultationsQueryRequest
    ConsultationService --> AdminConsultationResponse
```

## 4. Retrieval Flows

### 4.1 List Flow

```mermaid
sequenceDiagram
    participant AdminApp as Admin App
    participant Controller as AdminConsultationsController
    participant Service as ConsultationService
    participant BookingRepo as ConsultationBooking Repo
    participant PingRepo as ConsultationPingRequest Repo
    participant ConsultationRepo as Consultation Repo
    participant TxRepo as Transaction Repo

    AdminApp->>Controller: GET /api/admin/consultations
    Controller->>Service: GetAllConsultationsForAdminAsync(query)

    alt include Scheduled
        Service->>BookingRepo: Load bookings + user + expert + slot + consultation
        BookingRepo-->>Service: scheduled rows
        Service->>Service: adapt consultation + booking via Mapster
        Service->>ConsultationRepo: Load orphan scheduled consultations
        ConsultationRepo-->>Service: orphan scheduled rows
        Service->>Service: backfill scheduled fallback rows
    end

    alt include Emergency
        Service->>PingRepo: Load ping requests + rescuer + expert + consultation
        PingRepo-->>Service: emergency rows
        Service->>TxRepo: Load consultation payments + expert payouts
        TxRepo-->>Service: transaction rows
        Service->>Service: adapt consultation + request via Mapster
        Service->>ConsultationRepo: Load orphan emergency consultations
        ConsultationRepo-->>Service: orphan emergency rows
        Service->>Service: backfill emergency fallback rows
    end

    Service->>Service: merge + sort + paginate
    Service-->>Controller: PagingResponse<AdminConsultationResponse>
    Controller-->>AdminApp: ApiResponse<PagingResponse<AdminConsultationResponse>>
```

### 4.2 Single-Item Flow

```mermaid
sequenceDiagram
    participant AdminApp as Admin App
    participant Controller as AdminConsultationsController
    participant Service as ConsultationService
    participant ConsultationRepo as Consultation Repo
    participant BookingRepo as ConsultationBooking Repo
    participant PingRepo as ConsultationPingRequest Repo
    participant TxRepo as Transaction Repo

    AdminApp->>Controller: GET /api/admin/consultations/{consultationId}
    Controller->>Service: GetConsultationByIdForAdminAsync(consultationId)
    Service->>ConsultationRepo: Load consultation + caller + callee
    ConsultationRepo-->>Service: consultation or null

    alt consultation not found
        Service-->>Controller: throw NotFoundException("Consultation not found.")
    else scheduled consultation
        Service->>BookingRepo: Load booking + user + expert + slot
        BookingRepo-->>Service: booking or null
        Service->>Service: adapt scheduled response via Mapster
    else emergency consultation
        Service->>PingRepo: Load ping request + rescuer + expert
        PingRepo-->>Service: request or null
        Service->>TxRepo: Resolve emergency price
        TxRepo-->>Service: matching transactions
        Service->>Service: adapt emergency response via Mapster
    end

    Service-->>Controller: AdminConsultationResponse
    Controller-->>AdminApp: ApiResponse<AdminConsultationResponse>
```

## 5. Mapping Rules

### 5.1 Shared Base Fields

All admin responses normalize these fields from `Consultation` plus related records:

- `ConsultationId`
- `Type`
- `Status`
- `UserId`
- `UserName`
- `ExpertId`
- `ExpertName`
- `RoomId`
- `StartTime`
- `EndTime`
- `Price`

### 5.2 Scheduled Consultation

Sources:

- `Consultation`
- `ConsultationBooking`
- `Account User`
- `Account Expert`
- `ExpertTimeSlot`

Primary mapping:

- `ProblemDescription` <- `ConsultationBooking.ProblemDescription`
- `BookingId` <- `ConsultationBooking.Id`
- `BookingStatus` <- `ConsultationBooking.Status.ToString()`
- `BookedAt` <- `ConsultationBooking.BookedAt`
- `PaymentDeadline` <- `ConsultationBooking.PaymentDeadline`
- `CancelledAt` <- `ConsultationBooking.CancelledAt`
- `CancellationReason` <- `ConsultationBooking.CancellationReason`
- `SlotStartTime` <- `ExpertTimeSlot.StartTime`
- `SlotEndTime` <- `ExpertTimeSlot.EndTime`
- `Price` <- `ConsultationBooking.Price`

Fallback behavior:

- if `ConsultationBooking` is missing, the consultation is still returned
- booking-related fields become `null`

### 5.3 Emergency Consultation

Sources:

- `Consultation`
- `ConsultationPingRequest`
- `Account Rescuer`
- `Account Expert`
- `Transaction`

Primary mapping:

- `EmergencyRequestId` <- `ConsultationPingRequest.Id`
- `EmergencyRequestStatus` <- `ConsultationPingRequest.Status.ToString()`
- `RequestedAt` <- `ConsultationPingRequest.RequestedAt`
- `RespondedAt` <- `ConsultationPingRequest.RespondedAt`
- `ExpiresAt` <- `ConsultationPingRequest.ExpiresAt`

Emergency price resolution:

1. prefer latest `TransactionType = ConsultationPayment` by `ConsultationPingRequest.Id`
2. fallback latest `TransactionType = ExpertPayout` by `Consultation.Id`
3. otherwise `Price = null`

Fallback behavior:

- if `ConsultationPingRequest` is missing, the consultation is still returned
- emergency-request-related fields become `null`

## 6. Implementation Notes

1. The controller is separate from `ConsultationsController`
   - consistent with the existing `api/admin/...` module style
2. The admin contract is intentionally actor-neutral
   - unlike `MyConsultationResponse` and `ExpertConsultationResponse`
3. List and single-item endpoints now share one DTO
   - frontend can reuse one model for list rows and detail views
4. List and single-item retrieval strategies are different
   - list is merged-source-first
   - single item is consultation-first
5. The API contract stays normalized even though source systems differ
6. Base and enrichment field mapping are registered in `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
   - `ConsultationService` still owns fallback logic and emergency price resolution
   - Mapster is used to reduce property-assignment boilerplate, not to replace business branching

## 7. Code-Verified Tests

- `AdminConsultationHistoryIntegrationTests`
  - merged list is sorted by `StartTime desc`
  - scheduled list mapping includes booking and slot fields
  - emergency list mapping includes request fields and price fallback
  - orphan emergency consultations are still returned
  - pagination metadata is correct
  - invalid type throws validation error
  - scheduled single-item mapping includes booking detail fields
  - emergency single-item mapping includes request detail fields
  - missing consultation throws `NotFoundException`

- `AdminConsultationsControllerTests`
  - route is `api/admin/consultations`
  - controller requires role `Admin`
  - list action exposes `HttpGet`
  - list action returns `ApiResponse<PagingResponse<AdminConsultationResponse>>`
  - detail action exposes `HttpGet("{consultationId:guid}")`
  - detail action returns `ApiResponse<AdminConsultationResponse>`
    class AdminConsultationMapper {
        +Register(config)
    }
