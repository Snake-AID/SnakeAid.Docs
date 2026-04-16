---
doc_role: baseline
module: consultation
kind: flow
doc_type: index
status: active
last_updated: 2026-03-30
owners: [backend-team, mobile-team]
---

# Consultation Usage Guide — Index

## Mục tiêu

File này là điểm vào cho mobile integration. Mỗi hạng mục có usageguide riêng với payload chi tiết.

## Response Envelope

Tất cả REST success response đều dùng `ApiResponse<T>`:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {}
}
```

## Canonical Status Names

- Booking: `PendingPayment`, `Confirmed`, `Completed`
- Emergency Request: `PendingPayment`, `PendingExpertResponse`, `AcceptedByExpert`, `DeclinedByExpert`, `Expired`

## API Summary & Usageguide Links

### Expert Directory & Availability → `consultation.usageguide.expert-directory.md`

| Endpoint | Method |
|----------|--------|
| `PUT /api/experts/me/settings` | Update expert settings |
| `POST /api/experts/me/time-slots/bulk` | Setup weekly hours |
| `GET /api/experts` | List experts (filter/sort/paginate) |
| `GET /api/experts/{expertId}` | Expert profile |
| `GET /api/experts/{expertId}/reviews` | Expert reviews |
| `GET /api/experts/{expertId}/time-slots` | Available time slots |
| `/hubs/expert` — `JoinAsMember` | Subscribe expert presence |

### Scheduled Consultation → `consultation.usageguide.scheduled.md`

| Endpoint | Method |
|----------|--------|
| `POST /api/consultations/scheduled` | Create booking |
| `GET /api/users/me/consultations/scheduled` | My bookings |
| `GET /api/experts/me/consultations/scheduled` | Expert scheduled inbox |
| `POST /api/consultations/scheduled/{bookingId}/payments` | Pay booking |
| `POST /api/consultations/{consultationId}/video-token` | Get video token |
| `POST /api/consultations/{consultationId}/end` | End consultation |
| `POST /api/consultations/{consultationId}/reviews` | Submit review |

### Emergency Consultation → `consultation.usageguide.emergency.md`

| Endpoint | Method |
|----------|--------|
| `POST /api/consultations/instant` | Create request |
| `POST /api/consultations/instant/{requestId}/payments` | Pay request |
| `POST /api/consultations/instant/{requestId}/accept` | Expert accept |
| `POST /api/consultations/instant/{requestId}/reject` | Expert reject |
| `/hubs/expert` — `JoinAsExpert` | Expert presence |
| `/hubs/expert` — `JoinEmergencyRequestRoom` | Request status room |

### In-Room Features → `consultation.usageguide.in-room.md`

| Endpoint | Method |
|----------|--------|
| `/hubs/consultation` — `ReceiveMessage` | Chat message |
| `/hubs/consultation` — `Signal` | UI signaling |
| `/hubs/consultation` — `RoomExpiring` event | Room expiry notification |
| `GET /api/snake-species/search?q={query}` | Snake species search |
| `POST /api/media/upload-image` | Chat media upload |

### Expert History & Room Expiry → `consultation.usageguide.expert-history.md`

| Endpoint | Method |
|----------|--------|
| `GET /api/experts/me/consultations` | Expert consultation history (scheduled + emergency) |
| `/hubs/consultation` — `RoomExpiring` event | Auto room expiry signal |

### Payment → `consultation.payment.md`

| Endpoint | Method |
|----------|--------|
| `POST /api/consultations/scheduled/{bookingId}/payments` | Scheduled payment (Wallet/PayOS) |
| `POST /api/consultations/instant/{requestId}/payments` | Emergency payment (Wallet/PayOS) |
| `POST /api/consultations/payments/confirm` | Manual PayOS confirm |

## Current MVP Limits

- `PayOS` đã tích hợp cho consultation
- `IsVerified` chưa có semantics nghiệp vụ hoàn chỉnh
- Completion/payment summary screen chưa có contract đầy đủ cho mobile

## New Endpoints (Roadmap Tasks) → `consultation.usageguide.roadmap-tasks.md`

| Endpoint | Method |
|----------|--------|
| `POST /api/wallet/topup` | Wallet top-up via PayOS |
| `GET /api/users/me/consultations` | Unified consultation history (scheduled + emergency) |
| `GET /api/consultations/{id}/reviews` | Get consultation review |
| `GET /api/experts/me/consultations` | Expert consultation history (scheduled + emergency) |
