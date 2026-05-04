---
doc_role: planning
module: consultation-instant-booking-cancel
kind: flow
doc_type: useguide
status: planned
last_updated: 2026-05-04
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

Locked planned behavior:

- include expert-rejected instant/emergency requests in existing history endpoints as request-level rows
- do not fabricate a consultation id for rows that do not have a `Consultation`
- expose request-only rows with `recordKind = "EmergencyRequest"`
- expose exact request state through `requestStatus`

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

### `POST /api/consultations/instant/{requestId}/accept`

Accepts an instant/emergency request.

Auth:

- role `Expert`

Current side effect:

- creates a `Consultation`
- sets `ConsultationPingRequest.Status = AcceptedByExpert`
- sets `ConsultationPingRequest.ConsultationId`

### `POST /api/consultations/instant/{requestId}/reject`

Rejects an instant/emergency request.

Auth:

- role `Expert`

Current side effect:

- sets `ConsultationPingRequest.Status = DeclinedByExpert`
- does not create a `Consultation`
- returns a response where `consultationId` may be null

## 4. Current Verified History APIs

### `GET /api/users/me/consultations`

Current behavior:

- returns scheduled consultation rows
- returns accepted instant/emergency consultation rows
- does not return expert-rejected instant/emergency request rows

### `GET /api/experts/me/consultations`

Current behavior:

- returns scheduled consultation rows
- returns accepted instant/emergency consultation rows
- does not return expert-rejected instant/emergency request rows

## 5. Planned History Contract For Option 2B

This contract is not implemented yet.

Request-only rows should be distinguishable from real consultation rows.

### System Behavior After Option 2B Is Implemented

History endpoints return a single timeline with two row kinds.

`recordKind = "Consultation"`:

- represents a real `Consultation` row
- has a non-null `consultationId`
- may have a `roomId`
- can be used for supported consultation actions such as detail, message history, room join when still active, review when eligible, and other consultation-scoped flows
- applies to scheduled consultations and accepted emergency consultations

`recordKind = "EmergencyRequest"`:

- represents a rejected instant/emergency request, not a consultation session
- has `consultationId = null`
- has `emergencyRequestId`
- has `roomId = null`
- uses `status = "Cancelled"` for the existing history status grouping
- uses `requestStatus = "DeclinedByExpert"` for the exact backend request state
- cannot be used for consultation-scoped actions

Sorting behavior:

- real consultation rows sort by their existing consultation start time
- request-only rows sort by `respondedAt` when available
- request-only rows fall back to `requestedAt` when `respondedAt` is not available

Filtering behavior:

- no `status` filter returns both consultation rows and rejected request-only rows
- `status=Cancelled` includes request-only rows with `requestStatus = "DeclinedByExpert"`
- other consultation status filters apply only to real consultation rows unless future requirements add more request lifecycle mappings

Planned user history row example:

```json
{
  "recordKind": "EmergencyRequest",
  "consultationId": null,
  "emergencyRequestId": "11111111-1111-1111-1111-111111111111",
  "type": "Emergency",
  "status": "Cancelled",
  "requestStatus": "DeclinedByExpert",
  "expertId": "22222222-2222-2222-2222-222222222222",
  "expertName": "Dr. Snake Aid",
  "expertAvatarUrl": null,
  "roomId": null,
  "startTime": "2026-05-04T03:10:00Z",
  "endTime": "2026-05-04T03:10:00Z",
  "price": 150000
}
```

Planned expert history row example:

```json
{
  "recordKind": "EmergencyRequest",
  "consultationId": null,
  "emergencyRequestId": "11111111-1111-1111-1111-111111111111",
  "type": "Emergency",
  "status": "Cancelled",
  "requestStatus": "DeclinedByExpert",
  "userId": "33333333-3333-3333-3333-333333333333",
  "userName": "Member User",
  "userAvatarUrl": null,
  "roomId": null,
  "startTime": "2026-05-04T03:10:00Z",
  "endTime": "2026-05-04T03:10:00Z",
  "grossPrice": 150000,
  "netPrice": null
}
```

## 6. Field Notes

- `recordKind = "EmergencyRequest"` means this row is a request-level event, not a completed consultation session.
- `consultationId = null` means no `Consultation` was created.
- `roomId = null` means no call room was created.
- `requestStatus = "DeclinedByExpert"` is the exact backend request status.
- `status = "Cancelled"` is the locked unified display status for expert-rejected instant/emergency request rows.
- `emergencyRequestId` is the stable id for request-only rows.
- `startTime` and `endTime` on request-only rows are timeline timestamps, not consultation session timestamps.
- frontend/mobile must branch by `recordKind` before showing actions.

## 7. Verified Endpoint List

Current verified endpoints:

- `POST /api/consultations/instant`
- `POST /api/consultations/instant/{requestId}/accept`
- `POST /api/consultations/instant/{requestId}/reject`
- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

## 8. Changelog

### 2026-05-04

- Created planned frontend/mobile contract for showing expert-rejected instant/emergency requests in existing member/expert history endpoints.
- Marked request-only history rows as planned and not implemented.
- Locked Option 2B behavior: mixed history rows, nullable `consultationId`, `recordKind`, `requestStatus`, and `status=Cancelled` mapping for `DeclinedByExpert`.
