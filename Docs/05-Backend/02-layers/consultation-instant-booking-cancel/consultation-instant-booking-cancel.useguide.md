---
doc_role: planning
module: consultation-instant-booking-cancel
kind: flow
doc_type: useguide
status: planned
last_updated: 2026-05-05
api_version: v1
owners: [backend-team]
verification_status: planned-not-implemented
---

# Consultation Instant Booking Cancel Useguide

## 1. Overview

This guide is for frontend/mobile integration planning.

Current verified backend behavior:

- accepted instant/emergency requests can appear in member/expert consultation history
- expert-rejected instant/emergency requests do not currently appear in member/expert consultation history
- expired instant/emergency requests do not currently appear in member/expert consultation history

Selected planned contract:

- history response items use `kind = consultation | instant`
- `kind = consultation` represents a real `Consultation`
- `kind = instant` represents a terminal instant/emergency request without `Consultation`
- `kind = instant` is a separate DTO with flat fields
- `kind = instant` currently covers `DeclinedByExpert` and `Expired`

Implementation status:

- planned contract is locked
- backend implementation is not done
- status filter behavior for `kind = instant` is still open

## 2. Authentication & Authorization

JWT Bearer token is required.

Relevant roles:

- `User` reads own consultation history
- `Expert` reads own consultation history
- `Expert` can reject an assigned instant/emergency request while it is pending expert response

## 3. Current Verified Instant APIs

### `POST /api/consultations/instant`

Creates an instant/emergency consultation request.

Auth:

- role `User`

Side effect:

- creates `ConsultationPingRequest`
- starts before a `Consultation` exists

### `POST /api/consultations/instant/{requestId}/accept`

Accepts an instant/emergency request.

Auth:

- role `Expert`

Current side effect:

- creates a `Consultation`
- sets `ConsultationPingRequest.Status = AcceptedByExpert`
- sets `ConsultationPingRequest.ConsultationId`

History effect:

- accepted request remains `kind = consultation` in planned history contract

### `POST /api/consultations/instant/{requestId}/reject`

Rejects an instant/emergency request.

Auth:

- role `Expert`

Current side effect:

- sets `ConsultationPingRequest.Status = DeclinedByExpert`
- sets `ConsultationPingRequest.RespondedAt`
- does not create a `Consultation`
- returns a response where `consultationId` may be null

Planned history effect:

- rejected request appears as `kind = instant`
- rejected request does not expose `consultationId` or `roomId`

## 4. Member Business + Member APIs

### `GET /api/users/me/consultations`

Returns the member's consultation history.

Auth:

- role `User`

Current verified behavior:

- returns scheduled consultation rows
- returns accepted instant/emergency consultation rows
- does not return expert-rejected instant/emergency request rows
- does not return expired instant/emergency request rows

Planned behavior:

- returns `kind = consultation` rows for real consultations
- returns `kind = instant` rows for member-owned instant/emergency requests with:
  - `requestStatus = DeclinedByExpert`
  - `requestStatus = Expired`

Query params:

- existing paging params continue to apply
- existing `type` behavior should continue to include emergency rows when emergency history is requested
- `status` filter behavior for `kind = instant` is not finalized

Success response shape:

```json
{
  "items": [
    {
      "kind": "consultation",
      "consultationId": "33333333-3333-3333-3333-333333333333",
      "type": "Emergency",
      "status": "Completed",
      "expertId": "22222222-2222-2222-2222-222222222222",
      "expertName": "Khiêm Expert",
      "expertAvatarUrl": null,
      "roomId": "consultation-33333333-3333-3333-3333-333333333333",
      "startTime": "2026-05-04T03:10:00Z",
      "endTime": "2026-05-04T03:40:00Z",
      "price": 5000,
      "emergencyRequestId": "11111111-1111-1111-1111-111111111111"
    },
    {
      "kind": "instant",
      "instantRequestId": "44444444-4444-4444-4444-444444444444",
      "type": "Emergency",
      "requestStatus": "DeclinedByExpert",
      "requestedAt": "2026-05-04T04:00:00Z",
      "respondedAt": "2026-05-04T04:01:00Z",
      "expertId": "22222222-2222-2222-2222-222222222222",
      "expertName": "Khiêm Expert",
      "expertAvatarUrl": null
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 2
}
```

Important client rules:

- branch UI by `kind`
- do not call consultation detail, join room, review, report absent, or message history using a `kind = instant` row
- use `respondedAt` as the display/sort time for `kind = instant`
- do not infer row type from missing `consultationId`, `roomId`, or `status`

## 5. Expert Business + Expert APIs

### `GET /api/experts/me/consultations`

Returns the expert's consultation history.

Auth:

- role `Expert`

Current verified behavior:

- returns scheduled consultation rows
- returns accepted instant/emergency consultation rows
- does not return expert-rejected instant/emergency request rows
- does not return expired instant/emergency request rows

Planned behavior:

- returns `kind = consultation` rows for real consultations
- returns `kind = instant` rows for assigned instant/emergency requests with:
  - `requestStatus = DeclinedByExpert`
  - `requestStatus = Expired`

Query params:

- existing paging params continue to apply
- existing `type` behavior should continue to include emergency rows when emergency history is requested
- `status` filter behavior for `kind = instant` is not finalized

Success response shape:

```json
{
  "items": [
    {
      "kind": "consultation",
      "consultationId": "33333333-3333-3333-3333-333333333333",
      "type": "Emergency",
      "status": "Completed",
      "userId": "55555555-5555-5555-5555-555555555555",
      "userName": "Khiêm User",
      "userAvatarUrl": null,
      "roomId": "consultation-33333333-3333-3333-3333-333333333333",
      "startTime": "2026-05-04T03:10:00Z",
      "endTime": "2026-05-04T03:40:00Z",
      "grossPrice": 5000,
      "netPrice": 4000,
      "emergencyRequestId": "11111111-1111-1111-1111-111111111111"
    },
    {
      "kind": "instant",
      "instantRequestId": "44444444-4444-4444-4444-444444444444",
      "type": "Emergency",
      "requestStatus": "Expired",
      "requestedAt": "2026-05-04T04:00:00Z",
      "respondedAt": "2026-05-04T04:02:00Z",
      "userId": "55555555-5555-5555-5555-555555555555",
      "userName": "Khiêm User",
      "userAvatarUrl": null
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 2
}
```

Important client rules:

- branch UI by `kind`
- do not expose message, room, payout detail, absent report, or consultation detail actions for `kind = instant`
- use `respondedAt` as the display/sort time for `kind = instant`

## 6. Shared Data Models

### `kind = consultation`

Represents a real consultation/session row.

Important fields:

- `kind`: `consultation`
- `consultationId`: non-null id of `Consultation`
- `type`: `Scheduled` or `Emergency`
- `status`: `ConsultationStatus`
- `roomId`: room id for consultation-scoped actions
- `startTime`: session start time
- `endTime`: session end time when available

### `kind = instant`

Represents a terminal request-level row without a `Consultation`.

Member shape:

```json
{
  "kind": "instant",
  "instantRequestId": "44444444-4444-4444-4444-444444444444",
  "type": "Emergency",
  "requestStatus": "DeclinedByExpert",
  "requestedAt": "2026-05-04T04:00:00Z",
  "respondedAt": "2026-05-04T04:01:00Z",
  "expertId": "22222222-2222-2222-2222-222222222222",
  "expertName": "Khiêm Expert",
  "expertAvatarUrl": null
}
```

Expert shape:

```json
{
  "kind": "instant",
  "instantRequestId": "44444444-4444-4444-4444-444444444444",
  "type": "Emergency",
  "requestStatus": "Expired",
  "requestedAt": "2026-05-04T04:00:00Z",
  "respondedAt": "2026-05-04T04:02:00Z",
  "userId": "55555555-5555-5555-5555-555555555555",
  "userName": "Khiêm User",
  "userAvatarUrl": null
}
```

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

Current planned `requestStatus` values:

- `DeclinedByExpert`
- `Expired`

`RescuerCancelled` is not documented as active/current behavior because no production flow currently sets it.

## 7. Verified Endpoint List

Current verified endpoints:

- `POST /api/consultations/instant`
- `POST /api/consultations/instant/{requestId}/accept`
- `POST /api/consultations/instant/{requestId}/reject`
- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

Planned response changes:

- `GET /api/users/me/consultations` will return union rows after implementation
- `GET /api/experts/me/consultations` will return union rows after implementation

## 8. Changelog

### 2026-05-05

- Locked the frontend/mobile history direction to a union response contract.
- Added planned `kind = instant` member and expert DTO examples.
- Documented `DeclinedByExpert` and `Expired` as planned request-level history rows.
- Documented fields omitted from `kind = instant`.
- Kept status filter behavior open.

### 2026-05-04

- Documented three candidate frontend/mobile integration approaches for expert-rejected instant/emergency history with stronger wording.
- Marked all candidate history contracts as planned and not implemented.
- Removed the previous single-path framing from this guide.
