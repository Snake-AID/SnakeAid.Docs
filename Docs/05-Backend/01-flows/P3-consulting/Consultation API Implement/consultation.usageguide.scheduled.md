---
doc_role: baseline
module: consultation.scheduled
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-28
owners: [backend-team, mobile-team]
---

# Scheduled Consultation — Usage Guide

## Scope

Đặt lịch tư vấn: create booking → pay → video room → end → review. Bao gồm cả expert-side inbox.

## User Flow

### `POST /api/consultation-bookings`

Tạo booking từ available time slot.

**Request**:
```json
{
  "timeSlotId": "3fa85f64-...",
  "problemDescription": "Patient has progressive swelling after snakebite."
}
```

**Response**: `ApiResponse<ConsultationBookingResponse>`

```json
{
  "data": {
    "id": "7da8e6c6-...",
    "userId": "2a6d6f8f-...",
    "expertId": "3fa85f64-...",
    "expertName": "Dr. John Doe",
    "price": 150000,
    "bookedAt": "2026-03-08T09:25:00Z",
    "paymentDeadline": "2026-03-08T09:40:00Z",
    "status": "PendingPayment",
    "problemDescription": "Patient has progressive swelling after snakebite.",
    "timeSlotId": "3fa85f64-...",
    "slotStartTime": "2026-03-10T08:00:00Z",
    "slotEndTime": "2026-03-10T08:30:00Z",
    "consultationId": null,
    "roomId": null
  }
}
```

### `POST /api/consultation-bookings/{bookingId}/payments`

Thanh toán booking. Chi tiết payment contract xem `consultation.payment.md`.

**Request**: `{ "paymentMethod": "WalletBalance" }` hoặc `{ "paymentMethod": "PayOs" }`

Sau payment success: booking chuyển `Confirmed`, `consultationId` và `roomId` được gán.

### `GET /api/users/me/consultation-bookings`

Lịch sử booking của user hiện tại.

**Response**: `ApiResponse<IEnumerable<ConsultationBookingResponse>>` — cùng shape như create response, nhưng `status` có thể là `Confirmed` hoặc `Completed`.

### `POST /api/videocall/livekit-token/{consultationId}`

Lấy LiveKit token để vào phòng video. Chỉ Caller/Callee mới được cấp.

**Response**:
```json
{
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "wsUrl": "wss://livekit.example.com",
    "roomName": "consultation-bfa25be0..."
  }
}
```

### `POST /api/consultations/{consultationId}/end`

Kết thúc consultation. Không có body. Triggers escrow settlement to expert.

### `POST /api/consultations/{consultationId}/reviews`

**Request**:
```json
{
  "rating": 5,
  "comments": "Very helpful consultation."
}
```

**Response**: `ApiResponse<UserFeedbackResponse>`

```json
{
  "data": {
    "id": "5e801b64-...",
    "raterId": "2a6d6f8f-...",
    "targetUserId": "3fa85f64-...",
    "referenceId": "bfa25be0-...",
    "type": "Consultation",
    "rating": 5,
    "comments": "Very helpful consultation.",
    "createdAt": "2026-03-08T10:10:00Z",
    "raterName": "Alice Smith",
    "targetUserName": "Dr. John Doe",
    "updatedAverageRating": 4.9,
    "updatedRatingCount": 121
  }
}
```

## Expert Flow

### `GET /api/experts/me/consultation-bookings`

Expert xem scheduled bookings đã chốt. Trả `Confirmed` + `Completed`.

Response cùng shape `ConsultationBookingResponse` nhưng thêm `userName` để expert nhận diện member.

Hiện tại là REST pull, chưa có SignalR inbox cho scheduled consultation.

## Money Flow

1. Booking created → `PendingPayment`
2. Payment success → escrow → `Confirmed`
3. Consultation complete (explicit end hoặc slot hết giờ) → settle to expert
4. Auto-complete: background service sweep mỗi 30s

## Error Notes

- `POST /api/consultation-bookings`: `409` nếu slot đã bị book bởi request khác (optimistic concurrency)
- `POST .../payments`: `409` nếu booking đã paid hoặc không còn `PendingPayment`
- `POST /api/videocall/livekit-token/{id}`: `403` nếu không phải participant
