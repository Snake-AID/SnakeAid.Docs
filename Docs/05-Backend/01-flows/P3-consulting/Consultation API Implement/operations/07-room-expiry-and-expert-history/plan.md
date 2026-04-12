---
doc_role: operation
operation_id: 07-FEAT-room-expiry-and-expert-history
type: FEAT
status: done
created_at: 2026-03-30
implemented_at: 2026-03-30
affects:
  - Service/Implements/BookingService.cs
  - Service/Implements/ConsultationService.cs
  - Service/Implements/ConsultationLifecycleBackgroundService.cs
  - Service/Interfaces/IBookingService.cs
  - Service/Interfaces/IConsultationService.cs
  - Service/Hubs/ConsultationHub.cs
  - Api/Controllers/ExpertController.cs
  - Core/Responses/Consultation/ExpertConsultationResponse.cs
  - Tests/Unit/ConsultationPropertyTests.cs
  - Tests/Unit/RoomCleanupTests.cs
---

# Operation 07: LiveKit Room Expiry & Expert Consultation History

## Mục tiêu

1. Tự động kick user và xóa phòng LiveKit khi consultation hết giờ (scheduled: slot hết giờ, emergency: hết 30 phút)
2. Endpoint lịch sử tư vấn cho Expert (cả scheduled và emergency)

## Scope đã implement

### Room Cleanup (tích hợp vào auto-complete)

Mở rộng `AutoCompleteElapsedScheduledConsultationsAsync` trong `BookingService`:
- Inject `IHubContext<ConsultationHub>` và `ILiveKitService`
- TRƯỚC KHI cập nhật trạng thái: gửi `RoomExpiring` signal qua SignalR → gọi `DeleteRoomAsync`
- Mỗi bước có try-catch riêng (signal fail → vẫn delete room, delete fail → vẫn update status)
- Structured logging: `ConsultationId`, `RoomId`, `StartTime`, `ExpiryAction`
- Mỗi booking wrap trong try-catch riêng (lỗi 1 consultation không chặn các consultation khác)

### Emergency Auto-Complete (method MỚI)

`AutoCompleteElapsedEmergencyConsultationsAsync` trong `BookingService`:
- Query `Consultation` có `Status == Ongoing`, `Type == Emergency`, `StartTime + 30min <= UtcNow`
- Cùng flow: signal → delete room → update status Completed + EndTime = UtcNow → commit → settlement
- Tích hợp vào `ConsultationLifecycleBackgroundService` sau scheduled auto-complete

### Thứ tự sweep cycle (sau thay đổi)

1. `ExpireEmergencyRequestsAsync` (TTL 2 phút, refund)
2. `AutoCompleteElapsedScheduledConsultationsAsync` (slot hết giờ + room cleanup)
3. `AutoCompleteElapsedEmergencyConsultationsAsync` (emergency hết 30 phút + room cleanup)

### Expert Consultation History

- `GET /api/experts/me/consultations` — endpoint mới trong `ExpertController`
- `ExpertConsultationResponse` DTO: hiển thị thông tin user (caller) thay vì expert
- `GetExpertConsultationsAsync` trong `ConsultationService`: query cả scheduled (qua `ConsultationBooking`) và emergency (qua `ConsultationPingRequest`)
- Phân trang, lọc theo status/type, sort by startTime descending
- Edge case: scheduled consultation thiếu booking → price = null, log warning

### SignalR RoomExpiring Signal

- Event name: `RoomExpiring`
- Group: `consultation:{consultationId}`
- Payload: `{ "ConsultationId": guid, "Reason": "slot_elapsed" }`
- Best-effort: nếu gửi thất bại → log warning, tiếp tục xóa phòng

### Domain Rules

- Room deletion format: `consultation-{consultationId}`
- Scheduled EndTime = SlotEndTime, Emergency EndTime = DateTime.UtcNow
- Settlement triggered sau mỗi auto-complete thành công
- Advisory lock vẫn dùng pattern hiện tại (global session lock)

## Test Coverage

- 7 property-based tests (FsCheck, 100 iterations mỗi test): detection filters, history completeness/filtering/pagination/sorting/response shape
- 14 unit tests: room cleanup ordering, signal payload, room name format, edge cases (SignalR/LiveKit/settlement failure, consultation isolation)
