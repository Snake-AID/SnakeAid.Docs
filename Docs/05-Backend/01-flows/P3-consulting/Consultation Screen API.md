---
doc_role: handoff
module: consultation
kind: api-mapping
status: active
last_updated: 2026-03-05
owners: [backend-team, mobile-team]
---

# Consultation API Mapping and Mobile Handoff

## Backend Baseline (Operation 01)

### Endpoints Ready

- `PUT /api/v1/experts/me/settings`
- `POST /api/v1/experts/me/time-slots/bulk`
- `GET /api/v1/experts`
- `GET /api/v1/experts/{expertId}`
- `GET /api/v1/experts/{expertId}/reviews`
- `GET /api/v1/experts/{expertId}/time-slots`

### Planned (Not Ready in Operation 01)

- Operation 03
- Operation 04
- Operation 05

## API Mapping by Screen

### Mapping theo màn hình (User Series)

#### Screen: Danh sách chuyên gia

- Data source chính: `GET /api/v1/experts`
- Dùng cho:
  - Họ tên
  - Chuyên ngành
  - Trạng thái xác minh
  - Đánh giá và số lượng đánh giá
  - Đơn giá tư vấn
  - Trạng thái online/offline

#### Screen: Thông tin profile chuyên gia

- Data source chính:
  - `GET /api/v1/experts/{expertId}`
  - `GET /api/v1/experts/{expertId}/reviews`
  - `GET /api/v1/experts/{expertId}/time-slots`
- Dùng cho:
  - Thông tin cá nhân chuyên gia
  - Danh sách review (chỉ review consultation)
  - Lịch trống khả dụng để đặt lịch

#### Screen: Chọn loại tư vấn

- Data source:
  - `GET /api/v1/experts/{expertId}` để hiển thị thông tin chuyên gia
  - `GET /api/v1/experts/{expertId}/time-slots` cho nhánh đặt lịch
- Dùng cho:
  - Hiển thị thông tin expert trước khi chọn loại tư vấn
  - Hiển thị trạng thái slot để user quyết định đặt lịch
- Ghi chú:
  - API tạo tư vấn ngay/chốt booking chưa nằm trong Operation 01

### Usecase: Đặt lịch tư vấn

- Data source:
  - `GET /api/v1/experts/{expertId}/time-slots`
- Dùng cho:
  - Render danh sách ngày/slot khả dụng
  - Cho phép user chọn khung giờ trước bước thanh toán

### Mapping theo màn hình (Expert Series)

#### Screen: Thiết đặt

- Data source/API:
  - `PUT /api/v1/experts/me/settings`
  - `POST /api/v1/experts/me/time-slots/bulk`
- Dùng cho:
  - Cập nhật hồ sơ tư vấn, phí tư vấn
  - Thiết lập lịch làm việc theo tuần để mở slot cho user đặt

## Mobile Handoff Checklist (Operation 01)

### 1. Screen: Danh sách chuyên gia

- API: `GET /api/v1/experts`
- Query:
  - `pageNumber` (>= 1)
  - `pageSize` (1..100)
- Request DTO: `PaginationRequest` (query params)
- Response DTO: `PagingResponse<ExpertProfileResponse>`
- Error cần handle:
  - `400` cho query không hợp lệ
- Trạng thái: Ready

### 2. Screen: Thông tin profile chuyên gia

- API: `GET /api/v1/experts/{expertId}`
- Request DTO: Không có body
- Response DTO: `ExpertProfileResponse`
- Error cần handle:
  - `404` khi expert không tồn tại
- Trạng thái: Ready

### 3. Screen: Review chuyên gia (tab/section trong profile)

- API: `GET /api/v1/experts/{expertId}/reviews`
- Query:
  - `pageNumber` (>= 1)
  - `pageSize` (1..100)
- Request DTO: `PaginationRequest` (query params)
- Response DTO: `PagingResponse<UserFeedbackResponse>`
- Ghi chú nghiệp vụ:
  - Chỉ trả `FeedbackType = Consultation`
- Error cần handle:
  - `400` cho query không hợp lệ
  - `404` khi expert không tồn tại
- Trạng thái: Ready

### 4. Screen: Chọn thời gian đặt lịch

- API: `GET /api/v1/experts/{expertId}/time-slots`
- Request DTO: Không có body
- Response DTO: `IEnumerable<ExpertTimeSlotResponse>`
- Ghi chú nghiệp vụ:
  - Slot trả về là slot khả dụng để đặt
- Error cần handle:
  - `404` khi expert không tồn tại
- Trạng thái: Ready

### 5. Screen: Thiết đặt chuyên gia (cập nhật hồ sơ)

- API: `PUT /api/v1/experts/me/settings`
- Request DTO: `ExpertSettingsRequest`
- Response: `200 OK` (payload theo implementation backend hiện tại)
- Error cần handle:
  - `401/403` khi không phải role Expert hoặc chưa đăng nhập
  - `400/422` khi payload không hợp lệ
- Trạng thái: Ready

### 6. Screen: Thiết đặt chuyên gia (set lịch khả dụng tuần)

- API: `POST /api/v1/experts/me/time-slots/bulk`
- Request DTO: `BulkTimeSlotRequest`
- Response: `200 OK` khi tạo slot thành công
- Ghi chú nghiệp vụ:
  - `weekStartDate` bắt buộc UTC (`...Z`)
  - Hệ thống chia slot 30 phút
- Error cần handle:
  - `422` khi `weekStartDate` không phải UTC
  - `409` khi race-condition tạo trùng slot
  - `401/403` khi không phải Expert
- Trạng thái: Ready

### 7. Các màn hình chưa sẵn sàng API (Operation 01)

- Consulting Homepage (danh sách buổi tư vấn của tôi)
- Chọn loại tư vấn: nhánh tạo tư vấn ngay / tạo booking
- Tài liệu tư vấn
- Xác nhận thanh toán
- Sảnh phòng chờ
- Trong phòng tư vấn
- Chat
- Hoàn thành / gửi đánh giá
- Các tư vấn khẩn cấp (expert inbox)
- Trạng thái: Not Ready (đợi Operation 03/04/05)
