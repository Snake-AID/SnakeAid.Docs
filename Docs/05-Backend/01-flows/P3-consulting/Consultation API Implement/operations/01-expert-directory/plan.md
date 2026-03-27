---
doc_role: operation
operation_id: 01-expert-directory
type: INIT
status: done
created_at: 2026-03-05
merged_from: [00-Analysis, 01-INIT-expert-directory, 02-TEST-expert-directory]
affects:
  - Api/Controllers/ExpertController.cs
  - Service/Implements/ExpertService.cs
  - Service/Interfaces/IExpertService.cs
  - Core/Requests/Expert/ExpertSettingsRequest.cs
  - Core/Requests/Expert/BulkTimeSlotRequest.cs
  - Core/Responses/Expert/ExpertProfileResponse.cs
  - Core/Responses/Expert/ExpertTimeSlotResponse.cs
  - Repository/Data/Configurations/ExpertTimeSlotConfiguration.cs
  - Tests/Integration/ExpertControllerIntegrationTests.cs
  - Tests/Unit/ExpertServiceTests.cs
---

# Operation 01: Expert Directory & Availability

## Mục tiêu

Xây dựng nền tảng expert directory: expert settings, weekly time slot generation, public directory/profile/reviews/time-slots APIs, và test coverage cho các critical paths.

## Scope đã implement

### REST Endpoints

- `PUT /api/experts/me/settings` — cập nhật profile (bio, fee)
- `POST /api/experts/me/time-slots/bulk` — tạo discrete 30-min slots từ time blocks theo tuần
- `GET /api/experts` — paginated expert list
- `GET /api/experts/{expertId}` — expert profile chi tiết
- `GET /api/experts/{expertId}/reviews` — consultation reviews
- `GET /api/experts/{expertId}/time-slots` — available future slots

### Domain Rules

- Bulk slot generation xử lý dedup trong cùng request và skip slots đã tồn tại trong DB
- `weekStartDate` bắt buộc UTC, reject nếu không phải UTC
- `ExpertTimeSlot` có unique composite index (`ExpertId`, `StartTime`, `EndTime`)
- Expert directory trả paginated list, chỉ hiện verified experts

### Test Coverage (từ Operation 02 gốc)

- Integration: bulk time slot creation (UTC valid + non-UTC reject), reviews filter `Consultation` only, time slots filter future + available
- Unit: in-request overlap dedup, existing-slot skip, UTC enforcement, composite index assertion

## Giới hạn tại thời điểm Operation 01

Các gap sau đã được đóng bởi operations sau:

- Profile stats (`TotalConsultations`, `AverageResponseTimeMinutes`, `SuccessRate`) → đóng bởi Op 05-payment-and-stabilization
- Dual pricing (`ScheduledConsultationFee`, `EmergencyConsultationFee`) → đóng bởi Op 05-payment-and-stabilization
- Directory filter/sort (`specialization`, `isOnline`, `sortBy`, `sortOrder`) → đóng bởi Op 03-emergency-consultation
- `IsVerified` → deferred khỏi MVP

## Tham chiếu

- Analysis gốc: `analysis/decision-log.md`, `analysis/Endpoints-design.md` (giữ nguyên trong subfolder analysis)
