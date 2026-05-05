---
doc_role: usageguide
module: consultation-instant-booking-cancel
kind: flow
doc_type: useguide
status: implemented
last_updated: 2026-05-06
api_version: v1
owners: [backend-team]
verification_status: verified
---

# Consultation Instant Booking Cancel Useguide

## Overview

This guide is for frontend/mobile integration.

This module is implemented and verified.

Current member/expert consultation history is a typed union contract:

- `kind = consultation`: a real `Consultation` row
- `kind = instant`: a terminal instant request row (`ConsultationPingRequest`) without linked `Consultation`

Current request-level statuses included in history:

- `DeclinedByExpert`
- `Expired`

Out of scope for this module:

- Admin consultation history behavior
- New request-level query parameter filtering such as `requestStatus`

## Authentication & Authorization

All endpoints in this guide require JWT Bearer authentication.

Role rules:

- `User`: create/pay instant request, read own consultation history
- `Expert`: accept/reject assigned instant request, read own consultation history

## Expert/Member Business + Expert/Member APIs

### Business Flow Summary

1. User creates an emergency request.
2. User pays emergency request escrow.
3. Expert accepts or rejects while request is pending.
4. Background lifecycle worker can expire pending requests.
5. Consultation history returns mixed rows by `kind`.

### POST /api/consultations/instant

Create emergency consultation request.

Auth:

- `User`

Request body:

```json
{
  "expertId": "22222222-2222-2222-2222-222222222222"
}
```

Request constraints:

- `expertId` is required.
- `expertId` must belong to an active expert account.
- Duplicate active request for same user-expert pair is rejected.

Success response (200):

```json
{
  "data": {
    "requestId": "11111111-1111-1111-1111-111111111111",
    "requesterId": "55555555-5555-5555-5555-555555555555",
    "expertId": "22222222-2222-2222-2222-222222222222",
    "status": "PendingPayment",
    "requestedAt": "2026-05-05T10:00:00Z",
    "expiresAt": "2026-05-05T10:02:00Z",
    "respondedAt": null,
    "consultationId": null,
    "roomId": null
  }
}
```

Common errors:

- `404`: selected expert not found
- `409`: active emergency request already exists for this expert

Side effects:

- Insert `ConsultationPingRequest` with `Status = PendingPayment`
- Send emergency request created notification

Idempotency/retry note:

- Not idempotent. Retrying same payload can fail with `409` if first request succeeded.

### POST /api/consultations/instant/{requestId}/payments

Pay emergency request escrow.

Auth:

- `User`

Request body:

```json
{
  "returnUrl": "https://app.example/success",
  "cancelUrl": "https://app.example/cancel"
}
```

Success response (200):

```json
{
  "data": {
    "transactionId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
    "bookingId": null,
    "requestId": "11111111-1111-1111-1111-111111111111",
    "status": "Pending",
    "paymentUrl": "https://pay.example/checkout/123"
  }
}
```

Common errors:

- `404`: request not found
- `403`: request does not belong to caller
- `409`: request no longer payable state

Side effects:

- Moves request from `PendingPayment` to `PendingExpertResponse` on successful payment creation/confirmation flow

Idempotency/retry note:

- Retry must be handled carefully; gateway/order state can already exist.

### POST /api/consultations/instant/{requestId}/accept

Expert accepts emergency request.

Auth:

- `Expert`

Request body:

- none

Success response (200):

```json
{
  "data": {
    "requestId": "11111111-1111-1111-1111-111111111111",
    "requesterId": "55555555-5555-5555-5555-555555555555",
    "expertId": "22222222-2222-2222-2222-222222222222",
    "status": "AcceptedByExpert",
    "requestedAt": "2026-05-05T10:00:00Z",
    "expiresAt": "2026-05-05T10:02:00Z",
    "respondedAt": "2026-05-05T10:01:00Z",
    "consultationId": "33333333-3333-3333-3333-333333333333",
    "roomId": "consultation-33333333-3333-3333-3333-333333333333"
  }
}
```

Common errors:

- `404`: request not found
- `403`: request is not assigned to this expert
- `409`: request not in pending-expert-response state or expired

Side effects:

- Create `Consultation`
- Set request status `AcceptedByExpert`
- Set `ConsultationId`

Idempotency/retry note:

- Not idempotent.

### POST /api/consultations/instant/{requestId}/reject

Expert rejects emergency request.

Auth:

- `Expert`

Request body:

- none

Success response (200):

```json
{
  "data": {
    "requestId": "44444444-4444-4444-4444-444444444444",
    "requesterId": "55555555-5555-5555-5555-555555555555",
    "expertId": "22222222-2222-2222-2222-222222222222",
    "status": "DeclinedByExpert",
    "requestedAt": "2026-05-05T11:00:00Z",
    "expiresAt": "2026-05-05T11:02:00Z",
    "respondedAt": "2026-05-05T11:01:00Z",
    "consultationId": null,
    "roomId": null
  }
}
```

Common errors:

- `404`: request not found
- `403`: request is not assigned to this expert
- `409`: request not in pending-expert-response state or expired

Side effects:

- Set request status `DeclinedByExpert`
- Set `RespondedAt`
- Trigger emergency escrow refund flow
- No `Consultation` is created

Idempotency/retry note:

- Not idempotent.

### GET /api/users/me/consultations

Get member consultation history.

Auth:

- `User`

Query params:

- `pageNumber`, `pageSize`
- `type`: `Scheduled` or `Emergency`
- `status`: `ConsultationStatus` enum string

Validation rules:

- Invalid `type` or `status` enum values are rejected by backend parsing.

Filter behavior:

- `status` filters consultation rows only.
- `kind = instant` rows are returned only when `status` is omitted.

Success response (200):

```json
{
  "data": {
    "items": [
      {
        "kind": "consultation",
        "consultationId": "33333333-3333-3333-3333-333333333333",
        "type": "Emergency",
        "status": "Completed",
        "expertId": "22222222-2222-2222-2222-222222222222",
        "expertName": "Expert A",
        "expertAvatarUrl": null,
        "roomId": "consultation-33333333-3333-3333-3333-333333333333",
        "startTime": "2026-05-05T10:01:00Z",
        "endTime": "2026-05-05T10:25:00Z",
        "price": 5000,
        "emergencyRequestId": "11111111-1111-1111-1111-111111111111"
      },
      {
        "kind": "instant",
        "instantRequestId": "44444444-4444-4444-4444-444444444444",
        "type": "Emergency",
        "requestStatus": "DeclinedByExpert",
        "requestedAt": "2026-05-05T11:00:00Z",
        "respondedAt": "2026-05-05T11:01:00Z",
        "expertId": "22222222-2222-2222-2222-222222222222",
        "expertName": "Expert A",
        "expertAvatarUrl": null
      }
    ],
    "meta": {
      "currentPage": 1,
      "pageSize": 10,
      "totalItems": 2,
      "totalPages": 1
    }
  }
}
```

Important client notes:

- Branch UI strictly by `kind`.
- For `kind = instant`, do not call consultation-scoped actions:
  - consultation detail
  - join room
  - message history
  - expert absent report
  - review APIs
- Timeline sorting is newest first using `respondedAt ?? requestedAt` for instant rows.

### GET /api/experts/me/consultations

Get expert consultation history.

Auth:

- `Expert`

Query params and filter rules:

- Same behavior as member history endpoint.

Success response (200):

```json
{
  "data": {
    "items": [
      {
        "kind": "consultation",
        "consultationId": "33333333-3333-3333-3333-333333333333",
        "type": "Emergency",
        "status": "Completed",
        "userId": "55555555-5555-5555-5555-555555555555",
        "userName": "Member A",
        "userAvatarUrl": null,
        "roomId": "consultation-33333333-3333-3333-3333-333333333333",
        "startTime": "2026-05-05T10:01:00Z",
        "endTime": "2026-05-05T10:25:00Z",
        "grossPrice": 5000,
        "netPrice": 4000,
        "emergencyRequestId": "11111111-1111-1111-1111-111111111111"
      },
      {
        "kind": "instant",
        "instantRequestId": "66666666-6666-6666-6666-666666666666",
        "type": "Emergency",
        "requestStatus": "Expired",
        "requestedAt": "2026-05-05T12:00:00Z",
        "respondedAt": "2026-05-05T12:02:00Z",
        "userId": "55555555-5555-5555-5555-555555555555",
        "userName": "Member A",
        "userAvatarUrl": null
      }
    ],
    "meta": {
      "currentPage": 1,
      "pageSize": 10,
      "totalItems": 2,
      "totalPages": 1
    }
  }
}
```

Important client notes:

- Branch UI by `kind`.
- Do not expose consultation-scoped actions for `kind = instant`.
- `RescuerCancelled` exists in enum but is not currently documented as active runtime behavior for this history flow.

## Admin Business + Admin APIs

### Compatibility Check Scope

Checked endpoints:

- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`

Compatibility result with member/expert union format (`kind = consultation | instant`):

- `Not compatible`

### Why Not Compatible

1. Both endpoints return `AdminConsultationResponse` (single consultation-centric DTO), not union rows.
2. No discriminator field such as `kind` exists in admin response.
3. Admin list endpoint maps emergency rows from requests only when request is accepted and linked to `Consultation`.
4. Admin detail endpoint is keyed by `consultationId`, so request-only terminal events without `Consultation` cannot be addressed directly.

### GET /api/admin/consultations

List admin consultation history (scheduled + emergency consultation rows).

Auth:

- `Admin`

Query params:

- `pageNumber`, `pageSize`
- `type`: `Scheduled` or `Emergency`
- `status`: `ConsultationStatus` enum string

Success response shape (200):

```json
{
  "data": {
    "items": [
      {
        "consultationId": "33333333-3333-3333-3333-333333333333",
        "type": "Emergency",
        "status": "Completed",
        "userId": "55555555-5555-5555-5555-555555555555",
        "userName": "Member A",
        "expertId": "22222222-2222-2222-2222-222222222222",
        "expertName": "Expert A",
        "roomId": "consultation-33333333-3333-3333-3333-333333333333",
        "startTime": "2026-05-05T10:01:00Z",
        "endTime": "2026-05-05T10:25:00Z",
        "price": 5000,
        "emergencyRequestId": "11111111-1111-1111-1111-111111111111",
        "emergencyRequestStatus": "AcceptedByExpert",
        "requestedAt": "2026-05-05T10:00:00Z",
        "respondedAt": "2026-05-05T10:01:00Z",
        "expiresAt": "2026-05-05T10:02:00Z"
      }
    ],
    "meta": {
      "currentPage": 1,
      "pageSize": 10,
      "totalItems": 1,
      "totalPages": 1
    }
  }
}
```

Important note:

- `DeclinedByExpert` and `Expired` request-only rows (without `Consultation`) are not returned as independent timeline items.

### GET /api/admin/consultations/{consultationId}

Get admin detail by consultation id.

Auth:

- `Admin`

Path params:

- `consultationId` (guid)

Success response shape (200):

```json
{
  "data": {
    "consultationId": "33333333-3333-3333-3333-333333333333",
    "type": "Emergency",
    "status": "Completed",
    "userId": "55555555-5555-5555-5555-555555555555",
    "expertId": "22222222-2222-2222-2222-222222222222",
    "roomId": "consultation-33333333-3333-3333-3333-333333333333",
    "startTime": "2026-05-05T10:01:00Z",
    "endTime": "2026-05-05T10:25:00Z",
    "price": 5000,
    "emergencyRequestId": "11111111-1111-1111-1111-111111111111",
    "emergencyRequestStatus": "AcceptedByExpert",
    "requestedAt": "2026-05-05T10:00:00Z",
    "respondedAt": "2026-05-05T10:01:00Z",
    "expiresAt": "2026-05-05T10:02:00Z"
  }
}
```

Common errors:

- `404`: consultation not found

Black-box note:

- Endpoint requires a real consultation id, so request-only terminal event ids cannot be queried here.

## Shared Data Models

### History DTO Union

Member endpoint returns:

- `PagingResponse<MyConsultationHistoryUnionResponse>`

Expert endpoint returns:

- `PagingResponse<ExpertConsultationHistoryUnionResponse>`

Concrete DTOs:

- `MyConsultationHistoryResponse` (`kind = consultation`)
- `MyInstantConsultationRequestHistoryResponse` (`kind = instant`)
- `ExpertConsultationHistoryResponse` (`kind = consultation`)
- `ExpertInstantConsultationRequestHistoryResponse` (`kind = instant`)

### kind = consultation fields

Consultation-scoped fields can include:

- `consultationId`
- `status`
- `roomId`
- `startTime`, `endTime`
- price fields
- `emergencyRequestId` (for accepted emergency consultation rows)

### kind = instant fields

Request-scoped fields:

- `instantRequestId`
- `requestStatus`
- `requestedAt`
- `respondedAt`
- actor fields (`expert*` for member history, `user*` for expert history)

`kind = instant` does not expose:

- `consultationId`
- `roomId`
- `startTime`
- `endTime`
- `rescueMissionId`
- `expiresAt`
- `price`
- `grossPrice`
- `netPrice`

## Verified Endpoint List

Implemented endpoints in this module scope:

- `POST /api/consultations/instant`
- `POST /api/consultations/instant/{requestId}/payments`
- `POST /api/consultations/instant/{requestId}/accept`
- `POST /api/consultations/instant/{requestId}/reject`
- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

## Changelog

### 2026-05-06

- Added admin compatibility assessment for:
  - `GET /api/admin/consultations`
  - `GET /api/admin/consultations/{consultationId}`
- Confirmed admin flow is consultation-centric and not compatible with member/expert union format (`kind = consultation | instant`).
- Added concrete admin endpoint contracts and examples for integration clarity.
- Verified related tests passed:
  - `dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "AdminConsultationsControllerTests|AdminConsultationHistoryIntegrationTests"`

### 2026-05-05

- Synced useguide to implemented runtime behavior.
- Removed stale wording that suggested implementation was paused.
- Confirmed union history contract is active in production code path.
- Documented exact status filter behavior: consultation rows only when `status` is provided.
- Added request/response/error examples for create, pay, accept, reject, and history APIs.
- Re-verified by tests:
  - `rtk dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter ConsultationInstantHistoryIntegrationTests`
  - `rtk dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "ConsultationPriceBugConditionTests|ConsultationPricePreservationTests|ExpertConsultationPriceResponseTests|ConsultationExpertAbsentIntegrationTests|ConsultationPropertyTests"`

### 2026-05-04

- Created baseline useguide for instant booking cancel history problem and candidate directions.
