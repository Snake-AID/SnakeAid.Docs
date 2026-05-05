---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: sourcecode
status: implemented
last_updated: 2026-05-06
owners: [backend-team]
verification_status: verified
---

# Consultation Instant Booking Cancel Sourcecode

## 1. Relevant Classes

Active backend surface:

- `ConsultationInstantController`
- `EmergencyConsultationService`
- `ConsultationPaymentService`
- `ConsultationService`
- `ConsultationPingRequest`
- `Consultation`
- `MyConsultationResponse` for expert absent response, unchanged by history union
- `ExpertConsultationResponse` for existing expert consultation row contracts, not used as request-level DTO

Implemented response surface:

- `Responses/Consultation/History/MyConsultationHistoryUnionResponse.cs`
- `Responses/Consultation/History/MyConsultationHistoryResponse.cs`
- `Responses/Consultation/History/MyInstantConsultationRequestHistoryResponse.cs`
- `Responses/Consultation/History/ExpertConsultationHistoryUnionResponse.cs`
- `Responses/Consultation/History/ExpertConsultationHistoryResponse.cs`
- `Responses/Consultation/History/ExpertInstantConsultationRequestHistoryResponse.cs`
- no `object` / `dynamic` public response contracts
- no `JsonIgnore(WhenWritingNull)` for shaping history DTOs

## 2. Current HTTP Surface

Current verified endpoints:

- `POST /api/consultations/instant`
- `POST /api/consultations/instant/{requestId}/accept`
- `POST /api/consultations/instant/{requestId}/reject`
- `POST /api/consultations/instant/{requestId}/payments`
- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`
- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`

## 3. Observed Current Code

### Admin Endpoint Contract Surface (Observed)

- `GET /api/admin/consultations`
    - controller response type: `ApiResponse<PagingResponse<AdminConsultationResponse>>`
    - service: `GetAllConsultationsForAdminAsync(AdminConsultationsQueryRequest)`
- `GET /api/admin/consultations/{consultationId}`
    - controller response type: `ApiResponse<AdminConsultationResponse>`
    - service: `GetConsultationByIdForAdminAsync(Guid consultationId)`

Observed emergency inclusion rules in admin list:

- include only `ConsultationPingRequest` where:
    - `ConsultationId.HasValue`
    - `Status == AcceptedByExpert`
- terminal request-only rows (`DeclinedByExpert`, `Expired` without linked consultation) are not materialized as independent rows.

Observed admin detail behavior:

- requires `consultationId` to exist in `Consultation` table.
- emergency request fields are mapped as optional fields into consultation-centric `AdminConsultationResponse`.

### Instant Request Creation

```mermaid
sequenceDiagram
    participant User as Member App
    participant Api as ConsultationInstantController
    participant Service as EmergencyConsultationService
    participant DB as Database

    User->>Api: POST /api/consultations/instant
    Api->>Service: CreateEmergencyRequestAsync(userId, request)
    Service->>DB: insert ConsultationPingRequest
    Service->>DB: Status = PendingPayment
    Service-->>Api: EmergencyConsultationRequestResponse
```

### Expert Accept Flow

```mermaid
sequenceDiagram
    participant Expert as Expert App
    participant Api as ConsultationInstantController
    participant Service as EmergencyConsultationService
    participant DB as Database

    Expert->>Api: POST /api/consultations/instant/{requestId}/accept
    Api->>Service: AcceptEmergencyRequestAsync(requestId, expertId)
    Service->>DB: insert Consultation
    Service->>DB: set ping Status = AcceptedByExpert
    Service->>DB: set ping ConsultationId
    Service-->>Api: response with consultationId and roomId
```

### Expert Reject Flow

```mermaid
sequenceDiagram
    participant Expert as Expert App
    participant Api as ConsultationInstantController
    participant Service as EmergencyConsultationService
    participant Payment as ConsultationPaymentService
    participant DB as Database

    Expert->>Api: POST /api/consultations/instant/{requestId}/reject
    Api->>Service: RejectEmergencyRequestAsync(requestId, expertId)
    Service->>DB: set ping Status = DeclinedByExpert
    Service->>DB: set ping RespondedAt
    Service->>Payment: RefundEmergencyEscrowAsync(requestId, reason)
    Service-->>Api: response with consultationId = null
```

### Expiration Flow

```mermaid
sequenceDiagram
    participant Worker as ConsultationLifecycleBackgroundService
    participant Payment as ConsultationPaymentService
    participant DB as Database

    Worker->>Payment: ExpireEmergencyRequestsAsync()
    Payment->>DB: find PendingPayment/PendingExpertResponse pings past expiry
    Payment->>DB: set ping Status = Expired
    Payment->>DB: set ping RespondedAt
    Payment->>Payment: refund escrow when needed
```

### Previous History Query Behavior

Current member history emergency branch:

- filters by `p.RescuerId == userId`
- requires `p.ConsultationId.HasValue`
- requires `p.Status == AcceptedByExpert`
- maps from linked `Consultation`

Current expert history emergency branch:

- filters by `p.ExpertId == expertId`
- requires `p.ConsultationId.HasValue`
- requires `p.Status == AcceptedByExpert`
- maps from linked `Consultation`

Previous result:

- accepted instant/emergency requests appear in history
- expert-rejected instant/emergency requests did not appear in history
- expired instant/emergency requests did not appear in history

## 4. Implemented Code

The implementation uses a typed union dedicated to the History usecase.

### Implemented Union Mapping

```mermaid
flowchart TD
    A[History query] --> B{Source row}
    B -->|Consultation exists| C[kind = consultation]
    B -->|Ping terminal without Consultation| D[kind = instant]
    C --> E[Expose consultationId, roomId, startTime, endTime, consultation status]
    D --> F[Expose instantRequestId, requestStatus, requestedAt, respondedAt, flat actor fields]
```

### Admin Compatibility Assessment Against Union Format

Member/expert endpoints implement typed union timeline.

Admin endpoints currently do not:

- no `kind` discriminator in admin DTO
- no union base/derived DTO hierarchy in admin responses
- no request-only terminal row emission as timeline items
- detail endpoint cannot represent request-only rows because it is keyed by `Consultation.Id`

Conclusion:

- `GET /api/admin/consultations` and `GET /api/admin/consultations/{consultationId}` are not compatible with the current member/expert union timeline contract.

### `kind = consultation`

Rows with real `Consultation` records remain consultation rows.

Includes:

- scheduled consultations
- accepted instant/emergency requests linked by `ConsultationPingRequest.ConsultationId`

### `kind = instant`

Rows without `Consultation` are request-level rows.

Currently covered statuses:

- `ConsultationPingStatus.DeclinedByExpert`
- `ConsultationPingStatus.Expired`

Not currently covered:

- `PendingPayment`: active request state
- `PendingExpertResponse`: active request state
- `AcceptedByExpert`: has linked `Consultation`
- `RescuerCancelled`: enum value exists, but no production flow currently sets it

### Implemented Member DTO Shape For `kind = instant`

```csharp
new MyInstantConsultationRequestHistoryResponse
{
    InstantRequestId = ping.Id,
    Type = "Emergency",
    RequestStatus = ping.Status.ToString(),
    RequestedAt = ping.RequestedAt,
    RespondedAt = ping.RespondedAt,
    ExpertId = ping.ExpertId,
    ExpertName = ping.Expert?.FullName,
    ExpertAvatarUrl = ping.Expert?.AvatarUrl
}
```

### Implemented Expert DTO Shape For `kind = instant`

```csharp
new ExpertInstantConsultationRequestHistoryResponse
{
    InstantRequestId = ping.Id,
    Type = "Emergency",
    RequestStatus = ping.Status.ToString(),
    RequestedAt = ping.RequestedAt,
    RespondedAt = ping.RespondedAt,
    UserId = ping.RescuerId,
    UserName = ping.Rescuer?.FullName,
    UserAvatarUrl = ping.Rescuer?.AvatarUrl
}
```

### Fields Not Exposed By `kind = instant`

Do not expose these fields for request-level rows:

- `consultationId`
- `roomId`
- `startTime`
- `endTime`
- `rescueMissionId`
- `expiresAt`
- `price`
- `grossPrice`
- `netPrice`

### Implemented Service Contracts

```csharp
Task<PagingResponse<MyConsultationHistoryUnionResponse>> GetMyConsultationsAsync(...)
Task<PagingResponse<ExpertConsultationHistoryUnionResponse>> GetExpertConsultationsAsync(...)
Task<MyConsultationResponse> ReportExpertAbsentAsync(...)
```

### Implemented Filter And Sort Semantics

- `query.status` is parsed as `ConsultationStatus`; invalid enum input raises `ArgumentException`.
- `query.status` filters only consultation-backed rows.
- Instant request-level rows are included only when `query.status` is omitted.
- Instant request-level rows are fetched only when:
    - `ConsultationId == null`
    - `Status in { DeclinedByExpert, Expired }`
- Final list is sorted descending by `HistorySortTime`:
    - `kind = consultation` uses `StartTime`
    - `kind = instant` uses `RespondedAt ?? RequestedAt`

## 5. Code Decision Status

No open product decision remains for member/expert history.

The H-003 DTO boundary has been implemented:

- history has typed union DTOs
- expert absent remains `Task<MyConsultationResponse>`
- existing `MyConsultationResponse` and `ExpertConsultationResponse` remain free of `kind` and instant request fields

H-002 is closed with conservative behavior:

- `status` filters `kind = consultation` rows only.
- `kind = instant` rows are returned only when `status` is omitted.
- Future request-level filtering should use a separate contract such as `requestStatus`.

## 6. Test Focus

- member history includes `DeclinedByExpert` instant request as `kind = instant`
- expert history includes `DeclinedByExpert` instant request as `kind = instant`
- member history includes `Expired` instant request as `kind = instant`
- expert history includes `Expired` instant request as `kind = instant`
- accepted instant request behavior remains unchanged as `kind = consultation`
- selected contract behavior is explicit and tested
- paging and sorting handle `respondedAt` for `kind = instant`
- status filtering follows H-002
- `kind = instant` DTOs do not define consultation-scoped fields
- `MyConsultationResponse` remains unchanged for expert absent
- serialization test proves derived history DTO fields appear in JSON
- admin endpoint tests verify consultation-centric contract and mapping behavior

Verification commands:

```bash
dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter ConsultationInstantHistoryIntegrationTests
dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "ConsultationPriceBugConditionTests|ConsultationPricePreservationTests|ExpertConsultationPriceResponseTests|ConsultationExpertAbsentIntegrationTests|ConsultationPropertyTests"
dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "AdminConsultationsControllerTests|AdminConsultationHistoryIntegrationTests"
```
