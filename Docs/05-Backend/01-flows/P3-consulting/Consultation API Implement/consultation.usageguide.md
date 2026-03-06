---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-06
owners: [backend-team]
---

# Consultation Usage Guide

## Endpoints

### Expert Directory & Availability

- **Update Settings**:     `PUT  /api/v1/experts/me/settings`
- **Setup Weekly Hours**:  `POST /api/v1/experts/me/time-slots/bulk`
- **List Experts**:        `GET  /api/v1/experts`
- **Expert Profile**:      `GET  /api/v1/experts/{expertId}`
- **Expert Reviews**:      `GET  /api/v1/experts/{expertId}/reviews`
- **Available Time Slots**:`GET  /api/v1/experts/{expertId}/time-slots`

### Scheduled Consultation

- **Create Booking**: `POST /api/v1/consultation-bookings`
- **My Bookings**: `GET /api/v1/consultation-bookings/my-bookings`
- **End Consultation**: `POST /api/v1/consultations/{consultationId}/end`
- **Create Consultation Review**: `POST /api/v1/consultations/{consultationId}/reviews`
- **Generate LiveKit Token**: `POST /api/videocall/livekit-token/{consultationId}`

### Emergency Consultation

- **Create Emergency Request**: `POST /api/v1/consultations/emergency`
- **Accept Emergency Request**: `POST /api/v1/consultations/emergency-requests/{requestId}/accept`
- **Reject Emergency Request**: `POST /api/v1/consultations/emergency-requests/{requestId}/reject`

## Request/Response Examples

## Response Envelope (Code Culture)

All successful API responses are wrapped in `ApiResponse<T>`:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {}
}
```

All error responses are produced by `ApiExceptionHandlerMiddleware` from typed exceptions (`ValidationException`, `NotFoundException`, `ConflictException`, etc.).

### Update Settings (ExpertSettingsRequest)

_Endpoint: `PUT /api/v1/experts/me/settings`_

```json
{
  "biography": "Senior Herpetologist with 15 years experience.",
  "consultationFee": 500000.0
}
```

### Setup Weekly Hours (BulkTimeSlotRequest)

_Endpoint: `POST /api/v1/experts/me/time-slots/bulk`_

`weekStartDate` must be UTC (ISO-8601 with `Z` suffix), for example `2026-03-09T00:00:00Z`.

```json
{
  "weekStartDate": "2023-11-20T00:00:00Z",
  "days": [
    {
      "dayOfWeek": 1,
      "timeBlocks": [
        { "startTime": "08:00:00", "endTime": "12:00:00" },
        { "startTime": "13:00:00", "endTime": "17:00:00" }
      ]
    }
  ]
}
```

### Get Expert Profile (ExpertProfileResponse)

_Endpoint: `GET /api/v1/experts/{expertId}`_

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "accountId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "Dr. John Doe",
    "avatarUrl": "https://example.com/avatars/john.jpg",
    "biography": "Senior Herpetologist with 15 years experience.",
    "consultationFee": 500000.0,
    "rating": 4.8,
    "ratingCount": 120,
    "isVerified": true,
    "isOnline": true,
    "specializations": ["Venomous Snakes", "First Aid"]
  }
}
```

### Get Available Time Slots (ExpertTimeSlotResponse)

_Endpoint: `GET /api/v1/experts/{expertId}/time-slots`_

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "startTime": "2023-11-20T08:00:00Z",
      "endTime": "2023-11-20T08:30:00Z"
    },
    {
      "id": "4b9c1d2e-3f5g-6h7i-8j9k-0l1m2n3o4p5q",
      "startTime": "2023-11-20T08:30:00Z",
      "endTime": "2023-11-20T09:00:00Z"
    }
  ]
}
```

### List Experts (PagingResponse<ExpertProfileResponse>)

_Endpoint: `GET /api/v1/experts`_

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "items": [
      {
        "accountId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "name": "Dr. John Doe",
        "avatarUrl": "https://example.com/avatars/john.jpg",
        "biography": "Senior Herpetologist...",
        "consultationFee": 500000.0,
        "rating": 4.8,
        "ratingCount": 120,
        "isVerified": true,
        "isOnline": true,
        "specializations": ["Venomous Snakes"]
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

### Expert Reviews (PagingResponse<UserFeedbackResponse>)

_Endpoint: `GET /api/v1/experts/{expertId}/reviews`_

Returns only `FeedbackType = Consultation` reviews for the target expert.

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "items": [
      {
        "id": "5e801b64-c717-4562-b3fc-2c963f66afa6",
        "raterId": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
        "rating": 5,
        "comments": "Very helpful and professional.",
        "createdAt": "2023-11-21T10:00:00Z",
        "raterName": "Alice Smith"
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

## Query Parameter Rules

- Pagination: `pageNumber >= 1`
- Pagination: `1 <= pageSize <= 100`
- `GET /api/v1/experts` supports optional:
  - `specialization` (substring filter)
  - `isOnline` (`true` / `false`)
  - `sortBy` (`isOnline` | `rating` | `consultationFee`)
  - `sortOrder` (`asc` | `desc`)

## Scheduled Consultation Examples

### Create Booking

_Endpoint: `POST /api/v1/consultation-bookings`_

```json
{
  "timeSlotId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "problemDescription": "Patient has progressive swelling after snakebite. Need consultation."
}
```

### Create Consultation Review

_Endpoint: `POST /api/v1/consultations/{consultationId}/reviews`_

```json
{
  "rating": 5,
  "comments": "Very helpful consultation."
}
```

## Emergency Consultation Examples

### Create Emergency Request

_Endpoint: `POST /api/v1/consultations/emergency`_

```json
{
  "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

### Accept Emergency Request

_Endpoint: `POST /api/v1/consultations/emergency-requests/{requestId}/accept`_

No body.

### Reject Emergency Request

_Endpoint: `POST /api/v1/consultations/emergency-requests/{requestId}/reject`_

No body.

## Error Notes

- Error payloads are standardized by middleware (`ApiExceptionHandlerMiddleware`) with `ApiResponse<object>` shape.
- `POST /api/v1/experts/me/time-slots/bulk` returns `422` when `weekStartDate` is not UTC (`...Z`).
- `POST /api/v1/experts/me/time-slots/bulk` returns `409` when a concurrent request inserts the same slot first.
- `POST /api/v1/consultation-bookings` returns `409` when another request reserves the same slot first.
- `POST /api/videocall/livekit-token/{consultationId}` returns `403` if caller is not a consultation participant.
- `POST /api/v1/consultations/emergency` returns `409` when selected expert is offline or an active request already exists.
- `POST /api/v1/consultations/emergency-requests/{requestId}/accept|reject` returns `403` when caller is not targeted expert.
- `POST /api/v1/consultations/emergency-requests/{requestId}/accept|reject` returns `409` when request is no longer pending.
