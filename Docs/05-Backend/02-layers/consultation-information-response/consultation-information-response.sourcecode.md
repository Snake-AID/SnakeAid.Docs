---
doc_role: implementation
module: consultation-information-response
kind: flow
doc_type: sourcecode
status: active
last_updated: 2026-04-23
owners: [backend-team]
verification_status: code-verified
---

# Consultation Information Response Sourcecode

## 1. Relevant Classes

- `ExpertController`
- `IConsultationService`
- `ConsultationService`
- `ExpertConsultationResponse`
- `ConsultationBooking`
- `ConsultationPingRequest`
- `Transaction`
- `ConsultationPaymentService`

## 2. Code-Verified Current Surface

### HTTP

Current affected route:

- `GET /api/experts/me/consultations`

### DTO

Current response class:

- `ExpertConsultationResponse`

Current money fields:

- `GrossPrice`
- `NetPrice`

### Persistence Inputs Used By The Current Mapper

Current scheduled sources:

- `ConsultationBooking.Price`
- `ExpertPayout.Amount` by `Consultation.Id` when payout exists

Current emergency sources:

- latest gross transaction where:
  - `TransactionType = ConsultationPayment`
  - `ReferenceId = ConsultationPingRequest.Id`
- latest net transaction where:
  - `TransactionType = ExpertPayout`
  - `ReferenceId = Consultation.Id`

Related settlement behavior:

- consultation settlement already produces:
  - `PlatformFee`
  - `ExpertPayout`

## 3. Current Mapping Flow

### 3.1 Scheduled Expert History

Current scheduled branch in `ConsultationService.GetExpertConsultationsAsync(...)`:

1. query `ConsultationBooking` by `ExpertId`
2. include `User`, `TimeSlot`, `Consultation`
3. map each item to `ExpertConsultationResponse`
4. assign `GrossPrice = ConsultationBooking.Price`
5. assign `NetPrice = ExpertPayout.Amount` by `Consultation.Id` when payout exists, otherwise `null`

### 3.2 Emergency Expert History

Current emergency branch in `ConsultationService.GetExpertConsultationsAsync(...)`:

1. query accepted `ConsultationPingRequest` by `ExpertId`
2. include `Rescuer`, `Consultation`
3. batch-load gross `Transaction` rows where:
   - `TransactionType = ConsultationPayment`
   - `ReferenceId in ConsultationPingRequest.Id`
4. batch-load net `Transaction` rows where:
   - `TransactionType = ExpertPayout`
   - `ReferenceId in ConsultationIds`
5. assign `GrossPrice = ConsultationPayment.Amount` when found, otherwise `null`
6. assign `NetPrice = ExpertPayout.Amount` when found, otherwise `null`

## 4. Implemented Target Shape

Locked response direction:

- remove legacy `price`
- add:
  - `grossPrice`
  - `netPrice`

Locked money-source direction:

- `grossPrice` = gross consultation amount before platform fee
- `netPrice` = persisted `ExpertPayout.Amount`
- if persisted payout is absent, `netPrice = null`

## 5. Class Diagram

```mermaid
classDiagram
    class ExpertController {
        +GetMyConsultations(MyConsultationsQueryRequest query)
    }

    class IConsultationService {
        +GetExpertConsultationsAsync(Guid expertId, MyConsultationsQueryRequest query)
    }

    class ConsultationService {
        +GetExpertConsultationsAsync(Guid expertId, MyConsultationsQueryRequest query)
    }

    class ExpertConsultationResponse {
        +Guid ConsultationId
        +string Type
        +string Status
        +decimal? GrossPrice
        +decimal? NetPrice
        +Guid? BookingId
        +Guid? EmergencyRequestId
    }

    class ConsultationBooking {
        +Guid Id
        +Guid? ConsultationId
        +decimal Price
    }

    class ConsultationPingRequest {
        +Guid Id
        +Guid? ConsultationId
        +Guid ExpertId
        +Guid RescuerId
    }

    class Transaction {
        +Guid Id
        +Guid ReferenceId
        +decimal Amount
        +TransactionType TransactionType
    }

    class ConsultationPaymentService {
        +SettleConsultationEscrowAsync(Guid consultationId)
    }

    ExpertController --> IConsultationService
    ConsultationService ..|> IConsultationService
    ConsultationService --> ConsultationBooking
    ConsultationService --> ConsultationPingRequest
    ConsultationService --> Transaction
    ConsultationPaymentService --> Transaction
```

## 6. Sequence Diagram

### 6.1 Current Scheduled History Mapping

```mermaid
sequenceDiagram
    participant App as Expert App
    participant API as ExpertController
    participant Service as ConsultationService
    participant DB as Database

    App->>API: GET /api/experts/me/consultations?type=Scheduled
    API->>Service: GetExpertConsultationsAsync(expertId, query)
    Service->>DB: load ConsultationBooking + Consultation + User + TimeSlot
    Service->>Service: map Price = ConsultationBooking.Price
    Service-->>API: PagingResponse<ExpertConsultationResponse>
    API-->>App: ApiResponse(...)
```

### 6.2 Current Emergency History Mapping

```mermaid
sequenceDiagram
    participant App as Expert App
    participant API as ExpertController
    participant Service as ConsultationService
    participant DB as Database

    App->>API: GET /api/experts/me/consultations?type=Emergency
    API->>Service: GetExpertConsultationsAsync(expertId, query)
    Service->>DB: load accepted ConsultationPingRequest + Consultation + Rescuer
    Service->>DB: load ExpertPayout transactions by Consultation.Id
    Service->>Service: map Price = ExpertPayout.Amount when found
    Service-->>API: PagingResponse<ExpertConsultationResponse>
    API-->>App: ApiResponse(...)
```

### 6.3 Implemented Contract Direction

```mermaid
sequenceDiagram
    participant App as Expert App
    participant API as ExpertController
    participant Service as ConsultationService
    participant DB as Database

    App->>API: GET /api/experts/me/consultations
    API->>Service: GetExpertConsultationsAsync(expertId, query)
    Service->>Service: resolve gross source
    Service->>Service: resolve persisted payout source
    Service->>Service: populate grossPrice and netPrice
    Service-->>API: PagingResponse<ExpertConsultationResponse>
    API-->>App: ApiResponse(...)
```

## 7. Test Focus

- scheduled history returns explicit gross source from booking
- emergency history returns explicit net source from payout
- missing payout behavior returns `netPrice = null`
- client no longer needs hardcoded fee subtraction
- docs/examples stay aligned with final implemented DTO
