---
doc_role: usageguide
module: consultation
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-04-17
owners: [backend-team, mobile-team]
---

# Consultation Usage Guide

## Overview

This document is the canonical integration contract for the implemented consultation module.
It only describes behavior that is currently verified in `SnakeAid.Backend`.

All successful REST responses use `ApiResponse<T>`:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {}
}
```

Consultation currently has these route groups:

- `api/experts/*`: expert directory, availability, expert history
- `api/consultations/scheduled/*`: scheduled booking flow
- `api/consultations/instant/*`: emergency flow
- `api/consultations/{consultationId}/*`: actions on an existing consultation
- `api/admin/consultations/*`: admin consultation monitoring

## Authentication & Authorization

### JWT requirements

- Protected REST endpoints use `Bearer JWT`.
- Protected SignalR hubs also require authenticated connection.

### Role matrix

| Surface | Auth | Role |
|---|---|---|
| `GET /api/experts` | optional | public |
| `GET /api/experts/{id}` | optional | public |
| `GET /api/experts/{id}/reviews` | optional | public |
| `GET /api/experts/{id}/time-slots` | optional | public |
| `PUT /api/experts/me/settings` | required | `Expert` |
| `POST /api/experts/me/time-slots/bulk` | required | `Expert` |
| `GET /api/experts/me/consultations` | required | `Expert` |
| `POST /api/consultations/scheduled` | required | `User` |
| `GET /api/users/me/consultations/scheduled` | required | `User` |
| `GET /api/experts/me/consultations/scheduled` | required | `Expert` |
| `POST /api/consultations/instant` | required | `User` |
| `POST /api/consultations/instant/{requestId}/accept` | required | `Expert` |
| `POST /api/consultations/instant/{requestId}/reject` | required | `Expert` |
| `POST /api/consultations/scheduled/{bookingId}/payments` | required | `User` |
| `POST /api/consultations/instant/{requestId}/payments` | required | `User` |
| `POST /api/consultations/payments/confirm` | required | `User` |
| `GET /api/users/me/consultations` | required | `User` |
| `POST /api/consultations/{consultationId}/video-token` | required | participant or `Admin` |
| `POST /api/consultations/{consultationId}/end` | required | authenticated participant path enforced in service |
| `POST /api/consultations/{consultationId}/reviews` | required | `User` |
| `GET /api/consultations/{consultationId}/reviews` | required | authenticated participant |
| `GET /api/admin/consultations` | required | `Admin` |
| `GET /api/admin/consultations/{consultationId}` | required | `Admin` |
| `/hubs/expert` | required | `Expert` or `User` depending on hub method |
| `/hubs/consultation` | required | consultation participant |

## Expert/Member Business + Expert/Member APIs

### Expert directory and availability

#### `GET /api/experts`

List experts for browsing.

Query parameters:

| Field | Type | Notes |
|---|---|---|
| `specialization` | string | optional |
| `isOnline` | bool | optional |
| `sortBy` | string | `isOnline`, `rating`, or `consultationFee` |
| `sortOrder` | string | `asc` or `desc` |
| `pageNumber` | int | inherited from `PaginationRequest` |
| `pageSize` | int | inherited from `PaginationRequest` |

Response item shape is `ExpertProfileResponse`:

| Field |
|---|
| `accountId` |
| `name` |
| `avatarUrl` |
| `biography` |
| `scheduledConsultationFee` |
| `emergencyConsultationFee` |
| `rating` |
| `ratingCount` |
| `isVerified` |
| `isOnline` |
| `totalConsultations` |
| `averageResponseTimeMinutes` |
| `successRate` |
| `specializations` |

#### `GET /api/experts/{id}`

Returns one `ExpertProfileResponse`.

#### `GET /api/experts/{id}/reviews`

Returns paged `UserFeedbackResponse` records for the expert.

#### `GET /api/experts/{id}/time-slots`

Returns `ExpertTimeSlotResponse[]`.

`ExpertTimeSlotResponse` fields:

- `id`
- `expertId`
- `startTime`
- `endTime`
- `status`

#### `PUT /api/experts/me/settings`

Request body:

```json
{
  "biography": "10 years handling venomous snake incidents.",
  "scheduledConsultationFee": 150000,
  "emergencyConsultationFee": 250000
}
```

Request rules:

- `biography` is required, max 2000 chars
- `scheduledConsultationFee` range: `0` to `999999.99`
- `emergencyConsultationFee` range: `0` to `999999.99`

#### `POST /api/experts/me/time-slots/bulk`

Request body:

```json
{
  "weekStartDate": "2026-04-20T00:00:00Z",
  "days": [
    {
      "dayOfWeek": "Monday",
      "timeBlocks": [
        {
          "startTime": "08:00:00",
          "endTime": "12:00:00"
        }
      ]
    }
  ]
}
```

Request rules:

- `weekStartDate` is required
- at least one `day`
- each `day` must contain at least one `timeBlock`
- `endTime` must be later than `startTime`

### Scheduled consultation

#### Business behavior

- Booking requires an available future slot.
- Booking creation immediately creates both a `ConsultationBooking` and a `Consultation`.
- New booking status is `PendingPayment`.
- New consultation type is `Scheduled`.
- New consultation status is `Scheduled`.
- Slot status is set to `Reserved` during booking creation.
- Scheduled payment success moves booking status to `Confirmed`.

#### `POST /api/consultations/scheduled`

Request body:

```json
{
  "timeSlotId": "11111111-1111-1111-1111-111111111111",
  "problemDescription": "Snake bite swelling after 20 minutes."
}
```

Request rules:

- `timeSlotId` is required
- `problemDescription` max length is 2000

Response shape is `ConsultationBookingResponse`:

| Field |
|---|
| `id` |
| `userId` |
| `userName` |
| `expertId` |
| `expertName` |
| `price` |
| `bookedAt` |
| `paymentDeadline` |
| `status` |
| `problemDescription` |
| `timeSlotId` |
| `slotStartTime` |
| `slotEndTime` |
| `consultationId` |
| `roomId` |

Important error cases:

- slot not found
- slot already started
- slot no longer available
- concurrency conflict when another request books the same slot first

#### `GET /api/users/me/consultations/scheduled`

Returns the caller's scheduled booking list as `ConsultationBookingResponse[]`.

Important note:
This endpoint is booking-oriented, not consultation-history-oriented. It can include scheduled rows that are still waiting for payment.

#### `GET /api/experts/me/consultations/scheduled`

Returns the expert's scheduled booking list as `ConsultationBookingResponse[]`.

Current filter behavior in code:

- only `Confirmed` and `Completed` bookings are returned

### Scheduled and emergency payment

#### Payment methods

- `WalletBalance`
- `PayOs`

#### `POST /api/consultations/scheduled/{bookingId}/payments`

#### `POST /api/consultations/instant/{requestId}/payments`

Request body for both:

```json
{
  "paymentMethod": "WalletBalance"
}
```

or

```json
{
  "paymentMethod": "PayOs"
}
```

Response shape is `ConsultationPaymentResponse`:

| Field |
|---|
| `referenceId` |
| `referenceType` |
| `transactionId` |
| `amount` |
| `currency` |
| `paymentMethod` |
| `status` |
| `userWalletBalanceAfter` |
| `paidAtUtc` |
| `provider` |
| `checkoutUrl` |
| `orderCode` |
| `paymentLinkId` |
| `externalTransactionId` |

Verified behavior:

- scheduled wallet payment moves booking to `Confirmed`
- emergency wallet payment moves request to `PendingExpertResponse`
- emergency payment is rejected if the expert is offline
- PayOS responses return `status = "Pending"` until confirmed
- wallet success returns `status = "Escrowed"`

PayOS example response:

```json
{
  "referenceId": "11111111-1111-1111-1111-111111111111",
  "referenceType": "ScheduledBooking",
  "transactionId": "22222222-2222-2222-2222-222222222222",
  "amount": 150000,
  "currency": "VND",
  "paymentMethod": "PayOs",
  "status": "Pending",
  "userWalletBalanceAfter": null,
  "paidAtUtc": null,
  "provider": "PayOS",
  "checkoutUrl": "https://pay.payos.vn/web/example",
  "orderCode": 240321000123,
  "paymentLinkId": "plink_example",
  "externalTransactionId": null
}
```

#### `POST /api/consultations/payments/confirm`

Request body:

```json
{
  "transactionId": "22222222-2222-2222-2222-222222222222"
}
```

Use this as a manual fallback when the client still sees a pending PayOS consultation payment.

### Emergency consultation

#### Business behavior

- Creating an emergency request does not create a `Consultation` yet.
- New request status is `PendingPayment`.
- After successful payment, request status becomes `PendingExpertResponse`.
- Expert accept creates the consultation and marks request `AcceptedByExpert`.
- Expert reject marks request `DeclinedByExpert`.
- Expired paid requests are refunded by payment service.

#### `POST /api/consultations/instant`

Request body:

```json
{
  "expertId": "33333333-3333-3333-3333-333333333333"
}
```

Response shape is `EmergencyConsultationRequestResponse`:

| Field |
|---|
| `requestId` |
| `requesterId` |
| `expertId` |
| `status` |
| `requestedAt` |
| `expiresAt` |
| `respondedAt` |
| `consultationId` |
| `roomId` |

Current create response behavior:

- `status = PendingPayment`
- `consultationId = null`
- `roomId = null`

#### `POST /api/consultations/instant/{requestId}/accept`

Expert-only.

Current accept behavior:

- request must belong to the expert
- request must still be `PendingExpertResponse`
- consultation is created with `Type = Emergency`
- consultation status is `Ongoing`
- overlapping available slots for the next 30 minutes are reserved

#### `POST /api/consultations/instant/{requestId}/reject`

Expert-only.

Current reject behavior:

- request must belong to the expert
- request must still be `PendingExpertResponse`
- payment service attempts refund after reject

### Unified consultation history

#### `GET /api/users/me/consultations`

Member consultation history endpoint.

Query parameters:

| Field | Type | Notes |
|---|---|---|
| `status` | string | any `ConsultationStatus` enum name |
| `type` | string | `Scheduled` or `Emergency` |
| `pageNumber` | int | inherited from `PaginationRequest` |
| `pageSize` | int | inherited from `PaginationRequest` |

Important implementation note:

- `status` is not limited to `Ongoing` and `Completed`
- the code parses any valid `ConsultationStatus` enum name
- when `status` is omitted, scheduled rows can include `Scheduled` consultations

Response shape is `PagingResponse<MyConsultationResponse>`.

`MyConsultationResponse` fields:

| Field |
|---|
| `consultationId` |
| `type` |
| `status` |
| `expertId` |
| `expertName` |
| `roomId` |
| `startTime` |
| `endTime` |
| `price` |
| `problemDescription` |
| `bookingId` |
| `slotStartTime` |
| `slotEndTime` |
| `emergencyRequestId` |

Example:

```json
{
  "items": [
    {
      "consultationId": "44444444-4444-4444-4444-444444444444",
      "type": "Scheduled",
      "status": "Scheduled",
      "expertId": "33333333-3333-3333-3333-333333333333",
      "expertName": "Dr. Nguyen",
      "roomId": "consultation-44444444-4444-4444-4444-444444444444",
      "startTime": "2026-04-20T09:00:00Z",
      "endTime": null,
      "price": 150000,
      "problemDescription": "Snake bite swelling after 20 minutes.",
      "bookingId": "55555555-5555-5555-5555-555555555555",
      "slotStartTime": "2026-04-20T09:00:00Z",
      "slotEndTime": "2026-04-20T09:30:00Z",
      "emergencyRequestId": null
    }
  ],
  "meta": {
    "currentPage": 1,
    "pageSize": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

#### `GET /api/experts/me/consultations`

Expert consultation history endpoint.

Query parameters are the same as member history.

Response shape is `PagingResponse<ExpertConsultationResponse>`.

`ExpertConsultationResponse` fields:

| Field |
|---|
| `consultationId` |
| `type` |
| `status` |
| `userId` |
| `userName` |
| `roomId` |
| `startTime` |
| `endTime` |
| `price` |
| `bookingId` |
| `slotStartTime` |
| `slotEndTime` |
| `emergencyRequestId` |

### In-room features and post-consultation APIs

#### `POST /api/consultations/{consultationId}/video-token`

Returns `VideoTokenResponse`:

```json
{
  "token": "livekit_jwt",
  "wsUrl": "wss://your-livekit-host",
  "roomName": "consultation-44444444-4444-4444-4444-444444444444"
}
```

Verified access rules:

- consultation must exist
- authenticated user must be caller, callee, or admin
- `roomId` must be initialized

#### `POST /api/consultations/{consultationId}/end`

Ends a consultation through consultation service.

Verified behavior:

- service checks the current user as actor
- best-effort SignalR event `ConsultationCallEnded` is sent to `consultation:{consultationId}`
- manual end reason is `participant_ended`
- escrow settlement is triggered

#### `POST /api/consultations/{consultationId}/reviews`

Request body:

```json
{
  "rating": 5,
  "comments": "Very helpful consultation."
}
```

Rules:

- `rating` is required
- `rating` range is 1 to 5
- `comments` max length is 2000
- endpoint requires `User` role

#### `GET /api/consultations/{consultationId}/reviews`

Returns one `UserFeedbackResponse` or `null`.

Current no-review response:

```json
{
  "status_code": 200,
  "message": "No review found for this consultation.",
  "is_success": true,
  "data": null
}
```

#### `POST /api/media/upload-image`

Multipart form-data endpoint used by consultation chat.

Form fields:

| Field | Type | Notes |
|---|---|---|
| `file` | file | required |
| `domain` | string | optional, default `uploads` |

Validation:

- max size 5 MB
- allowed extensions: `.jpg`, `.jpeg`, `.png`, `.gif`

Response shape is `CloudinaryUploadResult`:

| Field |
|---|
| `secureUrl` |
| `publicId` |
| `resourceType` |
| `format` |
| `bytes` |
| `width` |
| `height` |
| `folder` |
| `tags` |

#### `GET /api/snake-species/search?q={query}`

Returns `SearchSnakeSpeciesResponse[]` for in-call reference search.

### SignalR hubs used by consultation

#### `ExpertHub` at `/hubs/expert`

Hub methods:

- `JoinAsExpert()`
- `JoinAsMember()`
- `JoinEmergencyRequestRoom(requestId)`

Server events verified in code:

- `JoinedAsExpert`
- `OnlineExpertsSnapshot`
- `JoinedEmergencyRequestRoom`
- `EmergencyConsultationRequest`
- `EmergencyRequestStatusChanged`

Important rules:

- `JoinAsExpert` requires `Expert`
- `JoinAsMember` requires `User`
- `JoinEmergencyRequestRoom` requires the current user to own that emergency request

#### `ConsultationHub` at `/hubs/consultation?consultationId={guid}`

Hub methods:

- `ReceiveMessage(content, attachmentUrl?)`
- `Signal(eventType, payload)`

Server events verified in code:

- `ReceiveMessage`
- `Signal`
- `ConsultationCallEnded`

Important rules:

- connection query string must contain valid `consultationId`
- caller must be a consultation participant
- server auto-adds the connection to group `consultation:{consultationId}` during connect
- there is no explicit consultation join method
- message requires either non-empty `content` or `attachmentUrl`
- rate limit is 10 messages per minute per user

Broadcast payload for `ReceiveMessage`:

```json
{
  "id": "66666666-6666-6666-6666-666666666666",
  "content": "Please look at this photo.",
  "attachmentUrl": "https://res.cloudinary.com/example/image/upload/sample.jpg",
  "senderId": "77777777-7777-7777-7777-777777777777",
  "sentAt": "2026-04-17T12:00:00Z"
}
```

Broadcast payload for `Signal`:

```json
{
  "eventType": "ToggleMic",
  "payload": "{\"enabled\":false}",
  "senderId": "77777777-7777-7777-7777-777777777777",
  "timestamp": "2026-04-17T12:00:00Z"
}
```

Forced termination payload for `ConsultationCallEnded`:

```json
{
  "consultationId": "44444444-4444-4444-4444-444444444444",
  "reason": "timeout"
}
```

Current verified reason values:

- `timeout`
- `participant_ended`

## Admin Business + Admin APIs

### `GET /api/admin/consultations`

Admin list endpoint across scheduled and emergency consultations.

Query parameters:

| Field | Type | Notes |
|---|---|---|
| `status` | string | any `ConsultationStatus` enum name |
| `type` | string | `Scheduled` or `Emergency` |
| `pageNumber` | int | inherited from `PaginationRequest` |
| `pageSize` | int | inherited from `PaginationRequest` |

Response type:

- `PagingResponse<AdminConsultationResponse>`

Current implementation behavior:

- scheduled and emergency consultations are loaded separately
- results are merged in memory
- final sort order is `StartTime desc`
- pagination is applied after merge
- orphan scheduled consultations can still be returned when the linked booking row is missing
- orphan emergency consultations can still be returned when the linked ping request row is missing

### `GET /api/admin/consultations/{consultationId}`

Admin detail endpoint for a single consultation.

Current implementation behavior:

- lookup starts from `Consultation`
- scheduled detail is enriched from `ConsultationBooking`
- emergency detail is enriched from `ConsultationPingRequest`
- related booking or ping fields can be `null` when the side record is missing

`AdminConsultationResponse` fields:

| Field |
|---|
| `consultationId` |
| `type` |
| `status` |
| `userId` |
| `userName` |
| `expertId` |
| `expertName` |
| `roomId` |
| `startTime` |
| `endTime` |
| `price` |
| `problemDescription` |
| `bookingId` |
| `bookingStatus` |
| `bookedAt` |
| `paymentDeadline` |
| `cancelledAt` |
| `cancellationReason` |
| `emergencyRequestId` |
| `emergencyRequestStatus` |
| `requestedAt` |
| `respondedAt` |
| `expiresAt` |
| `slotStartTime` |
| `slotEndTime` |

Important admin field rules:

- scheduled price comes from `ConsultationBooking.Price`
- emergency price prefers latest `TransactionType = ConsultationPayment` by `EmergencyRequestId`
- emergency price falls back to latest `TransactionType = ExpertPayout` by `ConsultationId`
- `problemDescription` is expected only for scheduled consultations
- booking metadata fields are expected only for scheduled consultations
- emergency request metadata fields are expected only for emergency consultations

## Shared Data Models

### Enum values currently used by consultation APIs

`ConsultationStatus`

- `Scheduled`
- `Ongoing`
- `Completed`
- `Cancelled`
- `UserAbsent`
- `ExpertAbsent`
- `AllAbsent`

`ConsultationType`

- `Emergency`
- `Scheduled`

`BookingStatus`

- `PendingPayment`
- `Confirmed`
- `Cancelled`
- `Refunded`
- `Expired`
- `Completed`

`ConsultationPingStatus`

- `PendingPayment`
- `PendingExpertResponse`
- `AcceptedByExpert`
- `DeclinedByExpert`
- `RescuerCancelled`
- `Expired`

### Important interpretation notes

- `GET /api/users/me/consultations/scheduled` is a booking list.
- `GET /api/users/me/consultations` is a consultation list.
- Emergency history price is resolved from consultation payment transactions.
- Scheduled history price comes from `ConsultationBooking.Price`.
- The current implementation emits `ConsultationCallEnded`, not `RoomExpiring`.
- Timeout cleanup emits `ConsultationCallEnded` with `reason = "timeout"`.
- Manual end emits `ConsultationCallEnded` with `reason = "participant_ended"`.

## Verified Endpoint List

| Method | Endpoint |
|---|---|
| `GET` | `/api/experts` |
| `GET` | `/api/experts/{id}` |
| `GET` | `/api/experts/{id}/reviews` |
| `GET` | `/api/experts/{id}/time-slots` |
| `PUT` | `/api/experts/me/settings` |
| `POST` | `/api/experts/me/time-slots/bulk` |
| `GET` | `/api/experts/me/consultations` |
| `POST` | `/api/consultations/scheduled` |
| `GET` | `/api/users/me/consultations/scheduled` |
| `GET` | `/api/experts/me/consultations/scheduled` |
| `POST` | `/api/consultations/scheduled/{bookingId}/payments` |
| `POST` | `/api/consultations/instant` |
| `POST` | `/api/consultations/instant/{requestId}/payments` |
| `POST` | `/api/consultations/instant/{requestId}/accept` |
| `POST` | `/api/consultations/instant/{requestId}/reject` |
| `POST` | `/api/consultations/payments/confirm` |
| `GET` | `/api/users/me/consultations` |
| `POST` | `/api/consultations/{consultationId}/video-token` |
| `POST` | `/api/consultations/{consultationId}/end` |
| `POST` | `/api/consultations/{consultationId}/reviews` |
| `GET` | `/api/consultations/{consultationId}/reviews` |
| `POST` | `/api/media/upload-image` |
| `GET` | `/api/snake-species/search` |
| `GET` | `/api/admin/consultations` |
| `GET` | `/api/admin/consultations/{consultationId}` |
| `SignalR` | `/hubs/expert` |
| `SignalR` | `/hubs/consultation` |

## Changelog

- `2026-04-17`: Rebuilt this folder as a code-verified reference set. Removed roadmap, quicknote, operation, and split-guide documents that described planning history or outdated implementation.
- `2026-04-17`: Merged verified implementation details from `consultation-endcall-signalr` and `admin-consultation-history` into the distilled source-of-truth set.
