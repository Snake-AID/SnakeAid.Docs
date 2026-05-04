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

Decision status:

- no final integration contract is locked yet
- three approaches are under review
- frontend/mobile impact depends on the selected approach

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

## 5. Candidate History Contract Approaches

No approach in this section is implemented yet.

### Approach 1: Split The Contract And Force Mobile To Build Two History Screens

The history contract explicitly separates real consultation rows from request-level instant request rows.

Potential row kinds:

- `Consultation`
- `EmergencyRequest`

Example request-level row:

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

Frontend/mobile impact:

- mobile must render by row kind
- mobile must build two screens or sections:
  - consultation history
  - instant request history
- request-level rows must not expose consultation-scoped actions

### Approach 2: Keep The Old Contract And Force `ConsultationPingRequest` Rows Into Consultation History

The backend fetches rejected `ConsultationPingRequest` records and maps them into the existing history response shape without creating `Consultation` rows.

Frontend/mobile impact:

- mobile receives rows that look like consultation history rows
- backend must define how fields such as `consultationId`, `roomId`, `startTime`, `endTime`, and `status` behave for request-only rows
- if no discriminator is added, mobile must infer row type from special values, which is fragile

Open contract questions:

- should `consultationId` be `Guid.Empty`, null, reused from `emergencyRequestId`, or omitted?
- should `status` expose `Cancelled` or `DeclinedByExpert`?
- how should mobile identify that the row is not a real consultation?

### Approach 3: Keep The Old Contract By Creating A Fake `Consultation`

The backend creates a Fake cancelled emergency `Consultation` when the expert rejects the instant request.

Frontend/mobile impact:

- mobile can keep receiving a non-null `consultationId`
- the rejected request appears as a cancelled consultation row
- mobile may need fewer response-model changes

Backend/domain impact:

- database contains a Fake `Consultation` that did not represent a real session
- `RoomId`, `StartTime`, `EndTime`, and `Status` must be synthesized
- room, chat, review, payment, cleanup, reporting, and admin flows need guards so they do not treat the Fake `Consultation` as a real session

## 6. Field Notes

- `ConsultationPingRequest.Status = DeclinedByExpert` is the current exact backend state for expert-rejected instant/emergency requests.
- `ConsultationPingRequest.ConsultationId = null` is the current reason these rows do not fit the existing consultation-only history contract.
- Approach 1 requires explicit row typing in the API contract and mobile two-screen work.
- Approach 2 keeps database semantics cleaner than Approach 3 but risks ambiguous response fields.
- Approach 3 keeps the response shape closest to today but introduces Fake `Consultation` data.

## 7. Verified Endpoint List

Current verified endpoints:

- `POST /api/consultations/instant`
- `POST /api/consultations/instant/{requestId}/accept`
- `POST /api/consultations/instant/{requestId}/reject`
- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

## 8. Changelog

### 2026-05-04

- Documented three candidate frontend/mobile integration approaches for expert-rejected instant/emergency history with stronger wording.
- Marked all candidate history contracts as planned and not implemented.
- Removed the previous single-path framing from this guide.
