---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-08
owners: [backend-team]
---

# Consultation Usage Guide

## Mục tiêu tài liệu

Tài liệu này là integration reference cho mobile.
- `Consultation Screen API.md` trả lời: màn hình nào gọi gì, đi tiếp ra sao.
- Tài liệu này trả lời: endpoint/ws đó nhận payload gì, trả payload gì.

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

Mobile nên bám đúng enum/status string của backend:
- Booking: `PendingPayment`, `Confirmed`, `Completed`
- Emergency Request: `PendingPayment`, `PendingExpertResponse`, `AcceptedByExpert`, `DeclinedByExpert`, `Expired`

Không dùng tên rút gọn như `Accepted`, `Rejected`, `Pending` trong state handling. Nếu UI cần label ngắn, map riêng ở presentation layer.

## Public API Summary

### Expert Directory & Availability

- **Update Settings**: `PUT /api/experts/me/settings`
- **Setup Weekly Hours**: `POST /api/experts/me/time-slots/bulk`
- **List Experts**: `GET /api/experts`
- **Get Expert Profile**: `GET /api/experts/{expertId}`
- **Get Expert Reviews**: `GET /api/experts/{expertId}/reviews`
- **Get Available Time Slots**: `GET /api/experts/{expertId}/time-slots`

### Scheduled Consultation

- **Create Booking**: `POST /api/consultation-bookings`
- **Get My Bookings**: `GET /api/users/me/consultation-bookings`
- **Get Expert Scheduled Bookings**: `GET /api/experts/me/consultation-bookings`
- **Pay Scheduled Booking**: `POST /api/consultation-bookings/{bookingId}/payments`
- **End Consultation**: `POST /api/consultations/{consultationId}/end`
- **Create Consultation Review**: `POST /api/consultations/{consultationId}/reviews`
- **Generate LiveKit Token**: `POST /api/videocall/livekit-token/{consultationId}`

### Emergency Consultation

- **Create Emergency Request**: `POST /api/consultations/emergency-requests`
- **Pay Emergency Request**: `POST /api/consultations/emergency-requests/{requestId}/payments`
- **Accept Emergency Request**: `POST /api/consultations/emergency-requests/{requestId}/accept`
- **Reject Emergency Request**: `POST /api/consultations/emergency-requests/{requestId}/reject`

### SignalR Presence and Emergency Realtime

- **Hub Endpoint**: `/hubs/expert`
- **User Methods**:
  - `JoinAsMember`
  - `JoinEmergencyRequestRoom(requestId)`
- **Expert Methods**:
  - `JoinAsExpert`
- **Server Events**:
  - `OnlineExpertsSnapshot`
  - `ExpertPresenceChanged`
  - `EmergencyConsultationRequest`
  - `EmergencyRequestStatusChanged`

## User Journey Endpoint Contracts

### 1. Screen: Danh sách chuyên gia

#### Endpoint: List Experts

`GET /api/experts`

**Query params**
- `pageNumber >= 1`
- `pageSize` in `1..100`
- `specialization` optional
- `isOnline` optional
- `sortBy = isOnline | rating | consultationFee`
- `sortOrder = asc | desc`

**Response DTO**
- `ApiResponse<PagingResponse<ExpertProfileResponse>>`

**Response sample**
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

#### WS: Presence when opening directory

**Connect**: `/hubs/expert`

**Invoke**
- `JoinAsMember`

**Event: OnlineExpertsSnapshot**
```json
{
  "onlineExpertIds": [
    "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "17f1929e-72f3-4684-b727-242ef5b7c748"
  ],
  "serverTimeUtc": "2026-03-08T09:20:00Z"
}
```

**Event: ExpertPresenceChanged**
```json
{
  "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "isOnline": true,
  "changedAtUtc": "2026-03-08T09:21:30Z"
}
```

### 2. Screen: Thông tin profile chuyên gia

#### Endpoint: Get Expert Profile

`GET /api/experts/{expertId}`

**Response DTO**
- `ApiResponse<ExpertProfileResponse>`

**Response sample**
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
    "specializations": ["Venomous Snakes", "First Aid"]
  }
}
```

#### Endpoint: Get Expert Reviews

`GET /api/experts/{expertId}/reviews?pageNumber=1&pageSize=10`

**Response DTO**
- `ApiResponse<PagingResponse<UserFeedbackResponse>>`

**Response sample**
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
        "targetUserId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "referenceId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
        "type": "Consultation",
        "rating": 5,
        "comments": "Very helpful and professional.",
        "createdAt": "2026-03-07T10:00:00Z",
        "updatedAt": "2026-03-07T10:00:00Z",
        "raterName": "Alice Smith",
        "targetUserName": "Dr. John Doe",
        "updatedAverageRating": 4.8,
        "updatedRatingCount": 120
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

#### Endpoint: Get Available Time Slots

`GET /api/experts/{expertId}/time-slots`

**Response DTO**
- `ApiResponse<IEnumerable<ExpertTimeSlotResponse>>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "expertId": "2fa85f64-5717-4562-b3fc-2c963f66afa6",
      "startTime": "2026-03-10T08:00:00Z",
      "endTime": "2026-03-10T08:30:00Z",
      "status": "Available"
    }
  ]
}
```

### 3. Usecase: Đặt lịch tư vấn

#### Step endpoint: Create Booking

`POST /api/consultation-bookings`

**Request DTO**
- `CreateConsultationBookingRequest`

**Request sample**
```json
{
  "timeSlotId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "problemDescription": "Patient has progressive swelling after snakebite. Need consultation."
}
```

**Response DTO**
- `ApiResponse<ConsultationBookingResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "id": "7da8e6c6-6e18-4d25-8fb0-2e62784d49a0",
    "userId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
    "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "expertName": "Dr. John Doe",
    "price": 150000,
    "bookedAt": "2026-03-08T09:25:00Z",
    "paymentDeadline": "2026-03-08T09:40:00Z",
    "status": "PendingPayment",
    "problemDescription": "Patient has progressive swelling after snakebite. Need consultation.",
    "timeSlotId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "slotStartTime": "2026-03-10T08:00:00Z",
    "slotEndTime": "2026-03-10T08:30:00Z",
    "consultationId": null,
    "roomId": null
  }
}
```

#### Step endpoint: Pay Scheduled Booking

`POST /api/consultation-bookings/{bookingId}/payments`

**Request DTO**
- `ProcessConsultationPaymentRequest`

**Request sample**
```json
{
  "paymentMethod": "WalletBalance"
}
```

**Response DTO**
- `ApiResponse<ConsultationPaymentResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "7da8e6c6-6e18-4d25-8fb0-2e62784d49a0",
    "referenceType": "ScheduledBooking",
    "transactionId": "f31d8f2c-1f6e-42a7-8c45-71cb1e0aa3b2",
    "amount": 150000,
    "currency": "VND",
    "paymentMethod": "WalletBalance",
    "status": "Escrowed",
    "userWalletBalanceAfter": 850000,
    "systemWalletBalanceAfter": 150000,
    "paidAtUtc": "2026-03-08T09:30:00Z"
  }
}
```

#### Step endpoint: Get My Bookings

`GET /api/users/me/consultation-bookings`

**Response DTO**
- `ApiResponse<IEnumerable<ConsultationBookingResponse>>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": "7da8e6c6-6e18-4d25-8fb0-2e62784d49a0",
      "userId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
      "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "expertName": "Dr. John Doe",
      "price": 150000,
      "bookedAt": "2026-03-08T09:25:00Z",
      "paymentDeadline": "2026-03-08T09:40:00Z",
      "status": "Confirmed",
      "problemDescription": "Patient has progressive swelling after snakebite. Need consultation.",
      "timeSlotId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "slotStartTime": "2026-03-10T08:00:00Z",
      "slotEndTime": "2026-03-10T08:30:00Z",
      "consultationId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
      "roomId": "consultation-bfa25be010ae4b9cb9232180703eeb7e"
    }
  ]
}
```

#### Step endpoint: Get Expert Scheduled Bookings

`GET /api/experts/me/consultation-bookings`

**Response DTO**
- `ApiResponse<IEnumerable<ConsultationBookingResponse>>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": "7da8e6c6-6e18-4d25-8fb0-2e62784d49a0",
      "userId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
      "userName": "Alice Smith",
      "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "expertName": "Dr. John Doe",
      "price": 150000,
      "bookedAt": "2026-03-08T09:25:00Z",
      "paymentDeadline": "2026-03-08T09:40:00Z",
      "status": "Confirmed",
      "problemDescription": "Patient has progressive swelling after snakebite. Need consultation.",
      "timeSlotId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "slotStartTime": "2026-03-10T08:00:00Z",
      "slotEndTime": "2026-03-10T08:30:00Z",
      "consultationId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
      "roomId": "consultation-bfa25be010ae4b9cb9232180703eeb7e"
    }
  ]
}
```

### 4. Usecase: Tư vấn ngay

#### Step endpoint: Create Emergency Request

`POST /api/consultations/emergency-requests`

**Request DTO**
- `CreateEmergencyConsultationRequest`

**Request sample**
```json
{
  "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Response DTO**
- `ApiResponse<EmergencyConsultationRequestResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "requestId": "d90d01f7-d9ff-45a2-88bf-a5eb8ce2f9be",
    "requesterId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
    "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "status": "PendingPayment",
    "requestedAt": "2026-03-08T09:31:00Z",
    "expiresAt": null,
    "respondedAt": null,
    "consultationId": null,
    "roomId": null
  }
}
```

#### WS step: Join request room after create success

**Invoke**
- `JoinEmergencyRequestRoom(requestId)`

**Why**
- để user nhận `EmergencyRequestStatusChanged` cho đúng request vừa tạo

#### Step endpoint: Pay Emergency Request

`POST /api/consultations/emergency-requests/{requestId}/payments`

**Request DTO**
- `ProcessConsultationPaymentRequest`

**Request sample**
```json
{
  "paymentMethod": "WalletBalance"
}
```

**Response DTO**
- `ApiResponse<ConsultationPaymentResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "d90d01f7-d9ff-45a2-88bf-a5eb8ce2f9be",
    "referenceType": "EmergencyRequest",
    "transactionId": "b14ad8fd-5fef-412f-bb16-3d7984a04b81",
    "amount": 500000,
    "currency": "VND",
    "paymentMethod": "WalletBalance",
    "status": "Escrowed",
    "userWalletBalanceAfter": 350000,
    "systemWalletBalanceAfter": 650000,
    "paidAtUtc": "2026-03-08T09:31:10Z"
  }
}
```

**Business effect after success**
- request `PendingPayment -> PendingExpertResponse`
- backend mới push `EmergencyConsultationRequest` sang expert

#### WS event: EmergencyRequestStatusChanged

**Payload sample**
```json
{
  "requestId": "d90d01f7-d9ff-45a2-88bf-a5eb8ce2f9be",
  "requesterId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
  "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "status": "AcceptedByExpert",
  "requestedAt": "2026-03-08T09:31:10Z",
  "expiresAt": "2026-03-08T09:33:10Z",
  "respondedAt": "2026-03-08T09:31:30Z",
  "consultationId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
  "roomId": "consultation-bfa25be010ae4b9cb9232180703eeb7e"
}
```

**Status values mobile phải handle**
- `PendingPayment`
- `PendingExpertResponse`
- `AcceptedByExpert`
- `DeclinedByExpert`
- `Expired`

### 5. Screen: Sảnh phòng chờ / Trong phòng tư vấn

#### Endpoint: Generate LiveKit Token

`POST /api/videocall/livekit-token/{consultationId}`

**Response DTO**
- `ApiResponse<VideoTokenResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Video token generated successfully",
  "is_success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "wsUrl": "wss://livekit.example.com",
    "roomName": "consultation-bfa25be010ae4b9cb9232180703eeb7e"
  }
}
```

#### Endpoint: End Consultation

`POST /api/consultations/{consultationId}/end`

**Request body**
- không có body

**Response DTO**
- `ApiResponse<object>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Consultation ended successfully",
  "is_success": true,
  "data": null
}
```

#### Endpoint: Create Review

`POST /api/consultations/{consultationId}/reviews`

**Request DTO**
- `CreateConsultationReviewRequest`

**Request sample**
```json
{
  "rating": 5,
  "comments": "Very helpful consultation."
}
```

**Response DTO**
- `ApiResponse<UserFeedbackResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "id": "5e801b64-c717-4562-b3fc-2c963f66afa6",
    "raterId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
    "targetUserId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "referenceId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
    "type": "Consultation",
    "rating": 5,
    "comments": "Very helpful consultation.",
    "createdAt": "2026-03-08T10:10:00Z",
    "updatedAt": "2026-03-08T10:10:00Z",
    "raterName": "Alice Smith",
    "targetUserName": "Dr. John Doe",
    "updatedAverageRating": 4.9,
    "updatedRatingCount": 121
  }
}
```

## Expert Journey Endpoint Contracts

### 6. Screen: Thiết đặt

#### Endpoint: Update Settings

`PUT /api/experts/me/settings`

**Request DTO**
- `ExpertSettingsRequest`

**Request sample**
```json
{
  "biography": "Senior Herpetologist with 15 years experience.",
  "consultationFee": 150000,
  "scheduledConsultationFee": 150000,
  "emergencyConsultationFee": 500000
}
```

**Response DTO**
- `ApiResponse<object>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Settings updated successfully",
  "is_success": true,
  "data": "Settings updated successfully"
}
```

#### Endpoint: Setup Weekly Hours

`POST /api/experts/me/time-slots/bulk`

**Request DTO**
- `BulkTimeSlotRequest`

**Request sample**
```json
{
  "weekStartDate": "2026-03-09T00:00:00Z",
  "days": [
    {
      "dayOfWeek": 1,
      "timeBlocks": [
        {
          "startTime": "08:00:00",
          "endTime": "12:00:00"
        },
        {
          "startTime": "13:00:00",
          "endTime": "17:00:00"
        }
      ]
    }
  ]
}
```

**Response DTO**
- `ApiResponse<object>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Time slots generated successfully",
  "is_success": true,
  "data": "Time slots generated successfully"
}
```

### 7. Supporting Screen: Lịch tư vấn đã chốt của expert

#### Endpoint: Get Expert Scheduled Bookings

`GET /api/experts/me/consultation-bookings`

**Use**
- load các scheduled booking đã chốt cho expert
- lấy `consultationId` và `roomId` để expert vào đúng phòng khi tới giờ
- đây là REST pull endpoint, chưa có SignalR inbox riêng cho scheduled consultation

### 8. Screen: Các tư vấn khẩn cấp

#### WS: Join as expert

**Connect**: `/hubs/expert`

**Invoke**
- `JoinAsExpert`

#### WS Event: EmergencyConsultationRequest

```json
{
  "requestId": "d90d01f7-d9ff-45a2-88bf-a5eb8ce2f9be",
  "requesterId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
  "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "requestedAt": "2026-03-08T09:31:10Z",
  "expiresAt": "2026-03-08T09:33:10Z"
}
```

#### Endpoint: Accept Emergency Request

`POST /api/consultations/emergency-requests/{requestId}/accept`

**Request body**
- không có body

**Response DTO**
- `ApiResponse<EmergencyConsultationRequestResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "requestId": "d90d01f7-d9ff-45a2-88bf-a5eb8ce2f9be",
    "requesterId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
    "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "status": "AcceptedByExpert",
    "requestedAt": "2026-03-08T09:31:10Z",
    "expiresAt": "2026-03-08T09:33:10Z",
    "respondedAt": "2026-03-08T09:31:30Z",
    "consultationId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
    "roomId": "consultation-bfa25be010ae4b9cb9232180703eeb7e"
  }
}
```

#### Endpoint: Reject Emergency Request

`POST /api/consultations/emergency-requests/{requestId}/reject`

**Request body**
- không có body

**Response DTO**
- `ApiResponse<EmergencyConsultationRequestResponse>`

**Response sample**
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "requestId": "d90d01f7-d9ff-45a2-88bf-a5eb8ce2f9be",
    "requesterId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
    "expertId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "status": "DeclinedByExpert",
    "requestedAt": "2026-03-08T09:31:10Z",
    "expiresAt": "2026-03-08T09:33:10Z",
    "respondedAt": "2026-03-08T09:31:35Z",
    "consultationId": null,
    "roomId": null
  }
}
```

## Error Notes

- `POST /api/experts/me/time-slots/bulk`
  - `422` when `weekStartDate` is not UTC
  - `409` when concurrent slot creation collides
- `POST /api/consultation-bookings`
  - `409` when another request reserves the same slot first
- `POST /api/consultation-bookings/{bookingId}/payments`
  - `409` when booking already paid or not in `PendingPayment`
  - `409` when wallet balance is insufficient
- `POST /api/consultations/emergency-requests/{requestId}/payments`
  - `409` when request already paid or not in `PendingPayment`
  - `409` when expert offline at payment time
  - `409` when wallet balance is insufficient
- `POST /api/videocall/livekit-token/{consultationId}`
  - `403` if caller is not a consultation participant
- `POST /api/consultations/emergency-requests/{requestId}/accept|reject`
  - `403` when caller is not targeted expert
  - `409` when request no longer pending

## Current MVP Limits

- Consultation payment method hiện chỉ có `WalletBalance`.
- `PayOS` chưa cắm cho consultation.
- `IsVerified` chưa có semantics nghiệp vụ hoàn chỉnh.
- Chat consultation vẫn thuộc Operation 5.
- Completion/payment summary screen chưa có contract chuyên biệt đầy đủ cho mobile.



