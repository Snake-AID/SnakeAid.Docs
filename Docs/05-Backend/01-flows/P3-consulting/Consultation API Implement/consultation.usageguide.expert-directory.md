---
doc_role: baseline
module: consultation.expert-directory
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-28
owners: [backend-team, mobile-team]
---

# Expert Directory & Availability — Usage Guide

## Scope

Expert list, profile, reviews, time slots, settings, weekly hours setup, và real-time presence.

## Endpoints

### `GET /api/experts`

Paginated expert directory với filter/sort.

**Query params**: `pageNumber`, `pageSize` (1..100), `specialization?`, `isOnline?`, `sortBy?` (isOnline|rating|consultationFee), `sortOrder?` (asc|desc)

**Response**: `ApiResponse<PagingResponse<ExpertProfileResponse>>`

```json
{
  "data": {
    "items": [{
      "accountId": "3fa85f64-...",
      "name": "Dr. John Doe",
      "avatarUrl": "https://example.com/avatars/john.jpg",
      "biography": "Senior Herpetologist with 15 years experience.",
      "consultationFee": 150000,
      "scheduledConsultationFee": 150000,
      "emergencyConsultationFee": 500000,
      "rating": 4.8,
      "ratingCount": 120,
      "isVerified": false,
      "isOnline": true,
      "totalConsultations": 500,
      "averageResponseTimeMinutes": 4.2,
      "successRate": 98.0,
      "specializations": ["Venomous Snakes"]
    }],
    "meta": { "currentPage": 1, "pageSize": 10, "totalItems": 1, "totalPages": 1 }
  }
}
```

### `GET /api/experts/{expertId}`

**Response**: `ApiResponse<ExpertProfileResponse>` — cùng shape như item trong list.

### `GET /api/experts/{expertId}/reviews?pageNumber=1&pageSize=10`

**Response**: `ApiResponse<PagingResponse<UserFeedbackResponse>>`

```json
{
  "data": {
    "items": [{
      "id": "5e801b64-...",
      "raterId": "1a2b3c4d-...",
      "targetUserId": "3fa85f64-...",
      "referenceId": "bfa25be0-...",
      "type": "Consultation",
      "rating": 5,
      "comments": "Very helpful and professional.",
      "createdAt": "2026-03-07T10:00:00Z",
      "raterName": "Alice Smith",
      "targetUserName": "Dr. John Doe",
      "updatedAverageRating": 4.8,
      "updatedRatingCount": 120
    }],
    "meta": { "currentPage": 1, "pageSize": 10, "totalItems": 1, "totalPages": 1 }
  }
}
```

### `GET /api/experts/{expertId}/time-slots`

Trả slots `Available` và `StartTime > Now`.

**Response**: `ApiResponse<IEnumerable<ExpertTimeSlotResponse>>`

```json
{
  "data": [{
    "id": "3fa85f64-...",
    "expertId": "2fa85f64-...",
    "startTime": "2026-03-10T08:00:00Z",
    "endTime": "2026-03-10T08:30:00Z",
    "status": "Available"
  }]
}
```

### `PUT /api/experts/me/settings`

Expert cập nhật profile. Requires `Expert` role.

**Request**:
```json
{
  "biography": "Senior Herpetologist with 15 years experience.",
  "consultationFee": 150000,
  "scheduledConsultationFee": 150000,
  "emergencyConsultationFee": 500000
}
```

**Response**: `ApiResponse<object>` — `"Settings updated successfully"`

### `POST /api/experts/me/time-slots/bulk`

Expert tạo weekly time slots. Requires `Expert` role. `weekStartDate` bắt buộc UTC.

**Request**:
```json
{
  "weekStartDate": "2026-03-09T00:00:00Z",
  "days": [{
    "dayOfWeek": 1,
    "timeBlocks": [
      { "startTime": "08:00:00", "endTime": "12:00:00" },
      { "startTime": "13:00:00", "endTime": "17:00:00" }
    ]
  }]
}
```

**Response**: `ApiResponse<object>` — `"Time slots generated successfully"`

## SignalR Presence

### Hub: `/hubs/expert`

**User app** gọi `JoinAsMember` khi mở directory → nhận:

- `OnlineExpertsSnapshot`: `{ "onlineExpertIds": ["uuid1", "uuid2"], "serverTimeUtc": "..." }`
- `ExpertPresenceChanged`: `{ "expertId": "uuid", "isOnline": true, "changedAtUtc": "..." }`

**Expert app** gọi `JoinAsExpert` → set `IsOnline = true`, broadcast `ExpertPresenceChanged`. Auto-offline on disconnect.

## Error Notes

- `POST /api/experts/me/time-slots/bulk`: `422` nếu `weekStartDate` không UTC, `409` nếu concurrent slot creation
- `GET /api/experts`: pagination stable với deterministic secondary ordering
