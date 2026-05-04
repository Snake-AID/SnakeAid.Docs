---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: sourcecode
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-investigated
---

# Consultation Instant Booking Cancel Sourcecode

## 1. Relevant Classes

Active backend surface:

- `ConsultationInstantController`
- `EmergencyConsultationService`
- `ConsultationService`
- `ConsultationPingRequest`
- `Consultation`
- `MyConsultationResponse`
- `ExpertConsultationResponse`
- `ConsultationPaymentService`

## 2. Current HTTP Surface

Current verified endpoints:

- `POST /api/consultations/instant`
- `POST /api/consultations/instant/{requestId}/accept`
- `POST /api/consultations/instant/{requestId}/reject`
- `POST /api/consultations/instant/{requestId}/payments`
- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

## 3. Current Instant Request Flow

```mermaid
sequenceDiagram
    participant User as Member App
    participant Api as ConsultationInstantController
    participant Service as EmergencyConsultationService
    participant DB as Database

    User->>Api: POST /api/consultations/instant
    Api->>Service: CreateEmergencyRequestAsync(userId, request)
    Service->>DB: insert ConsultationPingRequest
    Service-->>Api: EmergencyConsultationRequestResponse
```

## 4. Current Expert Accept Flow

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

## 5. Current Expert Reject Flow

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
    Service->>Payment: RefundEmergencyEscrowAsync(requestId, reason)
    Service-->>Api: response with consultationId = null
```

## 6. Current History Query Behavior

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

Result:

- accepted instant/emergency requests appear in history
- expert-rejected instant/emergency requests do not appear in history

## 7. Desired/Planned Code For Locked Option 2B

Option 2B is chosen.

- update history response DTOs to allow `ConsultationId` to be null
- add `RecordKind`
- add `RequestStatus`
- include rejected pings in emergency history queries
- map rejected pings as request-only rows
- keep accepted pings mapped from linked `Consultation`

### Planned Runtime Behavior

Member history:

- query accepted emergency pings exactly as today for real consultation rows
- additionally query `DeclinedByExpert` pings for the current member where `ConsultationId == null`
- map declined pings to `MyConsultationResponse` request-only rows
- merge scheduled, accepted emergency, and declined emergency request rows before sorting/pagination

Expert history:

- query accepted emergency pings exactly as today for real consultation rows
- additionally query `DeclinedByExpert` pings for the current expert where `ConsultationId == null`
- map declined pings to `ExpertConsultationResponse` request-only rows
- merge scheduled, accepted emergency, and declined emergency request rows before sorting/pagination

Accepted emergency preservation:

- accepted emergency rows stay `RecordKind = "Consultation"`
- accepted emergency rows keep non-null `ConsultationId`
- accepted emergency rows keep current `RoomId`, `StartTime`, `EndTime`, and pricing behavior

Rejected emergency mapping:

- rejected emergency rows become `RecordKind = "EmergencyRequest"`
- rejected emergency rows set `ConsultationId = null`
- rejected emergency rows set `EmergencyRequestId = ping.Id`
- rejected emergency rows set `Status = "Cancelled"`
- rejected emergency rows set `RequestStatus = "DeclinedByExpert"`
- rejected emergency rows set `RoomId = null`
- rejected emergency rows use `RespondedAt ?? RequestedAt` as timeline time

Planned request-only mapping:

```csharp
new MyConsultationResponse
{
    RecordKind = "EmergencyRequest",
    ConsultationId = null,
    EmergencyRequestId = ping.Id,
    Type = "Emergency",
    Status = "Cancelled",
    RequestStatus = ping.Status.ToString(),
    ExpertId = ping.ExpertId,
    ExpertName = ping.Expert?.FullName,
    RoomId = null,
    StartTime = ping.RespondedAt ?? ping.RequestedAt,
    EndTime = ping.RespondedAt,
    Price = ResolveLookupAmount(paymentLookup, ping.Id)
}
```

## 8. Test Focus

- rejected instant request appears in member history
- rejected instant request appears in expert history
- rejected row has `consultationId = null`
- rejected row has `emergencyRequestId`
- rejected row has `roomId = null`
- accepted instant request behavior remains unchanged
- paging and sorting handle request-only rows
- status filtering follows the chosen contract
