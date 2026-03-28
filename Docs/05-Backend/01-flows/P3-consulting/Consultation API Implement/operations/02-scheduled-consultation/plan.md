---
doc_role: operation
operation_id: 02-scheduled-consultation
type: FEAT
status: done
created_at: 2026-03-05
merged_from: [03-FEAT-scheduled-consultation]
affects:
  - Api/Controllers/ConsultationBookingsController.cs
  - Api/Controllers/ConsultationsController.cs
  - Api/Controllers/VideoCallController.cs
  - Service/Implements/BookingService.cs
  - Service/Implements/ConsultationService.cs
  - Core/Domains/ConsultationBooking.cs
  - Core/Domains/Consultation.cs
  - Tests/Integration/ScheduledConsultationIntegrationTests.cs
  - Tests/Unit/BookingServiceConcurrencyTests.cs
---

# Operation 02: Scheduled Consultation Flow

## Mục tiêu

Implement luồng đặt lịch tư vấn: user chọn slot → tạo booking → thanh toán → vào phòng → kết thúc → review.

## Scope đã implement

### REST Endpoints

- `POST /api/consultations/scheduled` — tạo booking, optimistic concurrency trên `ExpertTimeSlot.Version`
- `GET /api/users/me/consultations/scheduled` — lịch sử booking của user
- `GET /api/experts/me/consultations/scheduled` — expert xem scheduled bookings (Confirmed + Completed)
- `POST /api/consultations/scheduled/{bookingId}/payments` — thanh toán booking
- `POST /api/consultations/{consultationId}/video-token` — lấy LiveKit token cho phòng tư vấn
- `POST /api/consultations/{consultationId}/end` — kết thúc consultation
- `POST /api/consultations/{consultationId}/reviews` — đánh giá

### Domain Rules

- Booking bắt đầu ở `PendingPayment`, chuyển `Confirmed` sau payment success
- Slot booking dùng optimistic concurrency (`Version` field), trả 409 nếu conflict
- Schema có `ProblemDescription` cho user notes
- LiveKit token chỉ cấp cho participant hợp lệ (Caller/Callee)
- Consultation kết thúc bằng explicit end API hoặc auto-complete khi slot hết giờ
- Settlement escrow → expert khi consultation complete, idempotent

### Money Flow

1. Booking created → `PendingPayment`
2. Payment success → escrow vào system wallet → `Confirmed`
3. Consultation complete → settle escrow → expert wallet
4. Auto-complete: background service sweep mỗi 30s cho slots đã hết giờ

### Expert Inbox

- `GET /api/experts/me/consultations/scheduled` trả `Confirmed` + `Completed` bookings
- Response có `consultationId`, `roomId`, `UserName`, slot timing
- Hiện tại là REST pull, chưa có SignalR inbox cho scheduled consultation
