---
doc_role: handoff
module: consultation
kind: api-mapping
status: active
last_updated: 2026-03-06
owners: [backend-team, mobile-team]
---

# Consultation API Mapping and Mobile Handoff

## Backend Ready (Operation 01 + 03 + 04)

### Operation 01 (Expert Directory & Availability)

Operation 01 mở luồng cấu hình hồ sơ chuyên gia, thiết lập slot và tra cứu chuyên gia/slot):

- `PUT /api/v1/experts/me/settings`
- `POST /api/v1/experts/me/time-slots/bulk`
- `GET /api/v1/experts`
- `GET /api/v1/experts/{expertId}`
- `GET /api/v1/experts/{expertId}/reviews`
- `GET /api/v1/experts/{expertId}/time-slots`
- `POST /api/v1/consultation-bookings`
- `GET /api/v1/consultation-bookings/my-bookings`
- `POST /api/v1/consultations/{consultationId}/end`
- `POST /api/v1/consultations/{consultationId}/reviews`
- `POST /api/videocall/livekit-token/{consultationId}`

### Operation 03 (Scheduled Consultation)

Operation 03 mở luồng đặt lịch tư vấn, kết thúc buổi tư vấn và gửi đánh giá:

- `POST /api/v1/consultation-bookings`
- `GET /api/v1/consultation-bookings/my-bookings`
- `POST /api/v1/consultations/{consultationId}/end`
- `POST /api/v1/consultations/{consultationId}/reviews`
- `POST /api/videocall/livekit-token/{consultationId}`

### Operation 04 (Emergency Consultation + Presence)

- `POST /api/v1/consultations/emergency`
- `POST /api/v1/consultations/emergency-requests/{requestId}/accept`
- `POST /api/v1/consultations/emergency-requests/{requestId}/reject`
- SignalR hub endpoint: `/hubs/expert`
  - Expert client method: `JoinAsExpert`
  - Member client method: `JoinAsMember`
  - Presence snapshot event: `OnlineExpertsSnapshot`
  - Presence delta event: `ExpertPresenceChanged`
  - Server push event: `EmergencyConsultationRequest`

### SignalR Contract (Operation 04)

- Auth:
  - Hub dùng bearer token như API.
  - `JoinAsExpert`: chỉ role `Expert`.
  - `JoinAsMember`: chỉ role `User`.
- Member (User app):
  - Vào màn hình danh sách chuyên gia: connect hub, gọi `JoinAsMember`.
  - Nhận snapshot `OnlineExpertsSnapshot` một lần sau khi join.
  - Nhận delta `ExpertPresenceChanged` khi expert online/offline thay đổi.
  - Sau khi tạo emergency request: gọi `JoinEmergencyRequestRoom(requestId)` để subscribe trạng thái request.
  - Nhận event `EmergencyRequestStatusChanged` khi expert accept/reject request đó.
  - Khi reconnect phải gọi lại `JoinAsMember`.
- Expert app:
  - Vào trạng thái sẵn sàng nhận ca: connect hub, gọi `JoinAsExpert`.
  - Nhận event `EmergencyConsultationRequest` cho request được user chọn đúng expert đó.
  - Khi reconnect phải gọi lại `JoinAsExpert`.
- Payload shape:
  - `OnlineExpertsSnapshot`: `{ onlineExpertIds: Guid[], serverTimeUtc: DateTime }`
  - `ExpertPresenceChanged`: `{ expertId: Guid, isOnline: bool, changedAtUtc: DateTime }`
  - `EmergencyConsultationRequest`: `{ requestId: Guid, requesterId: Guid, expertId: Guid, requestedAt: DateTime, expiresAt: DateTime }`
  - `EmergencyRequestStatusChanged`: `EmergencyConsultationRequestResponse` payload (`requestId`, `status`, `respondedAt`, `consultationId`, `roomId`, ...)

### Planned (Mobile Not Covered in Operation 01-04)

- Operation 05

## Coverage Legend (Mobile Handoff)

- `Mobile Covered`: Backend đã có endpoint + dữ liệu đủ để mobile dựng đúng component trong wireframe.
- `Mobile Partial`: Backend đã có endpoint nhưng response/behavior chưa đủ cho toàn bộ component wireframe.
- `Mobile Not Covered`: Backend chưa có endpoint cho component/use-case đó, nên mobile chưa thể build end-to-end.
- Lưu ý: `Mobile Partial`/`Mobile Not Covered` trong tài liệu này là khoảng trống backend cho nhu cầu mobile handoff.
- Lưu ý API contract: success response dùng envelope `ApiResponse<T>` (`status_code`, `message`, `is_success`, `data`).
- Lưu ý presence semantics: realtime online/offline ưu tiên SignalR events (`OnlineExpertsSnapshot`, `ExpertPresenceChanged`); `ExpertProfile.IsOnline` trong DB là trạng thái eventual consistency.

## Operation 01 Coverage Audit (Wireframe)

### Operation 01 Scope Note

- Chú thích chủ ngữ: phần audit này trả lời câu hỏi "Operation 01 cover cho mobile tới mức nào", không đánh giá chất lượng UI implementation phía mobile.

### Mobile Covered (Operation 01 đủ để build màn hình)

- `Screen: Danh sách chuyên gia`
  - `GET /api/v1/experts` đã trả các field chính cho card list.
- `Screen: Thiết đặt` (Expert Series)
  - `PUT /api/v1/experts/me/settings` và `POST /api/v1/experts/me/time-slots/bulk` đã đủ cho cập nhật hồ sơ + set lịch.

### Mobile Partial (Operation 01 có API nhưng chưa phủ hết component wireframe)

- `Screen: Thông tin profile chuyên gia`
  - Có: thông tin cơ bản, phí tư vấn, rating, review list, time slots.
  - Thiếu field theo wireframe: thống kê `Ca tư vấn`, `Thời gian phản hồi`, `Tỉ lệ thành công`.
  - `IsVerified` hiện tại đang trả cố định `true` từ service, chưa phản ánh trạng thái xác minh thực.
  - Handoff note: Mobile có thể render profile bản cơ bản, nhưng các block thống kê + verified badge chuẩn nghiệp vụ cần backend bổ sung.
- `Screen: Chọn loại tư vấn`
  - Có dữ liệu cho nhánh đặt lịch (expert info + available slots).
  - Nhánh tư vấn ngay đã có API ở Operation 04 (`POST /api/v1/consultations/emergency`).
  - Handoff note: luồng tư vấn ngay đã mở, riêng bước `Xác nhận thanh toán` vẫn cần Operation 05.
- `Usecase: Đặt lịch tư vấn`
  - Operation 01 chỉ có phần đọc slot (`GET /experts/{id}/time-slots`).
  - API chốt booking nằm ở Operation 03 (`POST /consultation-bookings`).
  - Handoff note: Nếu scope chỉ Operation 01 thì flow đặt lịch chưa end-to-end.

### Mobile Not Covered (đúng theo scope Operation 01)

- `Consulting Homepage`, `Sảnh phòng chờ`, `Trong phòng`, `Hoàn thành`, `Chat`, `Thanh toán`.

## API Mapping by Screen

### Mapping theo màn hình (User Series)

#### Screen: Consulting Homepage

- Data source chính: `GET /api/v1/consultation-bookings/my-bookings`
- Component dùng API:
  - Section "Buổi đặt lịch của tôi"
  - Card: Đang trong quá trình đặt (`Thanh toán ngay`)
  - Card: Đã đặt, đến giờ (`Vào phòng ngay`)
  - Booking status badge (`PendingPayment/Confirmed/Completed...`)

#### Screen: Danh sách chuyên gia

- Data source chính: `GET /api/v1/experts`
- Realtime source:
  - SignalR `/hubs/expert`
  - Member gọi `JoinAsMember`
  - Event `OnlineExpertsSnapshot` để hydrate online map ban đầu
  - Event `ExpertPresenceChanged` để cập nhật online/offline realtime
- Query phục vụ UI UX:
  - `pageNumber`, `pageSize`
  - `specialization`
  - `isOnline`
  - `sortBy` (`isOnline` | `rating` | `consultationFee`)
  - `sortOrder` (`asc` | `desc`)
- Dùng cho:
  - Họ tên
  - Chuyên ngành
  - Trạng thái xác minh
  - Đánh giá và số lượng đánh giá
  - Đơn giá tư vấn
  - Trạng thái online/offline
- Ghi chú:
  - REST trả snapshot dữ liệu list/filter/sort.
  - SignalR dùng để cập nhật tức thời badge online, tránh polling liên tục.
  - `OnlineExpertsSnapshot` lấy từ SignalR connected memory (SignalR-first), không query DB presence.

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
  - CTA `Chọn Tư Vấn Ngay`: `POST /api/v1/consultations/emergency` với `expertId` user đã chọn
  - CTA `Chọn Đặt Lịch`: đi flow đặt lịch ở Screen "Tài liệu tư vấn" (`POST /api/v1/consultation-bookings`)

#### Screen: Tư vấn ngay (User tạo request khẩn cấp)

- Data source:
  - `POST /api/v1/consultations/emergency`
- Component dùng API:
  - Tạo emergency request theo expert đã chọn trước
  - Nhận `requestId` để theo dõi trạng thái xử lý
  - Join SignalR request-room bằng `JoinEmergencyRequestRoom(requestId)` để nhận event `EmergencyRequestStatusChanged`
  - Với trạng thái `Accepted`, dùng `consultationId`/`roomId` từ event để chuyển sang waiting-room/in-room

#### Screen: Đặt lịch tư vấn (chọn ngày/slot)

- Data source:
  - `GET /api/v1/experts/{expertId}/time-slots`
- Dùng cho:
  - Render danh sách ngày/slot khả dụng
  - Cho user chọn slot trước khi qua bước nhập tài liệu

#### Screen: Tài liệu tư vấn

- Data source:
  - `POST /api/v1/consultation-bookings`
- Component dùng API:
  - Text input: Mô tả vấn đề (`problemDescription`)
  - Text input: Câu hỏi cụ thể (`question`)
  - CTA: Xác nhận đặt lịch

#### Screen: Sảnh phòng chờ

- Data source:
  - `POST /api/videocall/livekit-token/{consultationId}`
- Component dùng API:
  - Nút `Join phòng`
  - Lấy token vào phòng LiveKit cho đúng consultation
  - Chặn truy cập nếu user không phải participant

#### Screen: Trong phòng tư vấn

- Data source:
  - `POST /api/videocall/livekit-token/{consultationId}` (lấy token trước khi join room)
  - `POST /api/v1/consultations/{consultationId}/end`
- Component dùng API:
  - Nút `Kết thúc` buổi tư vấn
  - Các control mic/cam/chat là realtime UI, không gọi thêm REST endpoint trong Operation 03

#### Screen: Hoàn thành

- Data source:
  - `POST /api/v1/consultations/{consultationId}/reviews`
- Component dùng API:
  - Star rating
  - Text input nhận xét
  - Nút gửi đánh giá

### Mapping theo màn hình (Expert Series)

#### Screen: Thiết đặt

- Data source/API:
  - `PUT /api/v1/experts/me/settings`
  - `POST /api/v1/experts/me/time-slots/bulk`
- Dùng cho:
  - Cập nhật hồ sơ tư vấn, phí tư vấn
  - Thiết lập lịch làm việc theo tuần để mở slot cho user đặt

#### Screen: Sảnh phòng chờ / Trong phòng tư vấn

- Data source:
  - `POST /api/videocall/livekit-token/{consultationId}`
  - `POST /api/v1/consultations/{consultationId}/end`
- Component dùng API:
  - Join room theo consultation
  - Kết thúc ca tư vấn khi hoàn tất

#### Screen: Inbox khẩn cấp (Expert)

- Data source:
  - SignalR `/hubs/expert`
  - Client method: `JoinAsExpert`
  - Event nhận request: `EmergencyConsultationRequest`
  - Action endpoint:
    - `POST /api/v1/consultations/emergency-requests/{requestId}/accept`
    - `POST /api/v1/consultations/emergency-requests/{requestId}/reject`
- Component dùng API:
  - Hiển thị incoming emergency request realtime
  - Expert accept/reject đúng request được push

### Emergency Consultation State Transition (Operation 04)

- Mục tiêu: chuẩn hóa lifecycle của emergency request để mobile render đúng trạng thái ở cả User và Expert.
- Trạng thái chính:
  - `Pending`: user vừa tạo request, đang chờ expert phản hồi
  - `Accepted`: expert đã nhận ca, chuyển sang vào phòng tư vấn
  - `Rejected`: expert từ chối request
  - `Expired`: quá thời gian chờ phản hồi (timeout nghiệp vụ)

#### Transition Rules

- `Pending -> Accepted`
  - Trigger: expert gọi `POST /api/v1/consultations/emergency-requests/{requestId}/accept`
  - Kết quả UI:
    - Expert: mở flow vào phòng
    - User: nhận event `EmergencyRequestStatusChanged` và chuyển sang room/waiting-room tùy UX mobile
- `Pending -> Rejected`
  - Trigger: expert gọi `POST /api/v1/consultations/emergency-requests/{requestId}/reject`
  - Kết quả UI:
    - User: nhận event `EmergencyRequestStatusChanged`, hiển thị "chuyên gia đã từ chối", cho phép chọn expert khác
- `Pending -> Expired`
  - Trigger: backend timeout request pending
  - Kết quả UI:
    - User: hiển thị "yêu cầu hết hạn", cho phép tạo yêu cầu mới
- `Accepted|Rejected|Expired -> (terminal)`
  - Không cho phép accept/reject lại
  - Backend trả `409` nếu client thao tác trên request không còn `Pending`

## Mobile Handoff Checklist (Operation 01-04)

### 1. Screen: Danh sách chuyên gia

- API: `GET /api/v1/experts`
- Realtime:
  - Hub endpoint: `/hubs/expert`
  - Member method: `JoinAsMember`
  - Event: `OnlineExpertsSnapshot`, `ExpertPresenceChanged`
- Query:
  - `pageNumber` (>= 1)
  - `pageSize` (1..100)
- `specialization` (optional)
- `isOnline` (optional)
- `sortBy` (`isOnline` | `rating` | `consultationFee`) (optional)
- `sortOrder` (`asc` | `desc`) (optional)
- Request DTO: `ExpertDirectoryQueryRequest` (query params)
- Response DTO: `ApiResponse<PagingResponse<ExpertProfileResponse>>`
- Error cần handle:
  - `400` cho query không hợp lệ
- Trạng thái: Ready

### 2. Screen: Thông tin profile chuyên gia

- API: `GET /api/v1/experts/{expertId}`
- Request DTO: Không có body
- Response DTO: `ApiResponse<ExpertProfileResponse>`
- Error cần handle:
  - `404` khi expert không tồn tại
- Trạng thái: Ready

### 3. Screen: Review chuyên gia (tab/section trong profile)

- API: `GET /api/v1/experts/{expertId}/reviews`
- Query:
  - `pageNumber` (>= 1)
  - `pageSize` (1..100)
- Request DTO: `PaginationRequest` (query params)
- Response DTO: `ApiResponse<PagingResponse<UserFeedbackResponse>>`
- Ghi chú nghiệp vụ:
  - Chỉ trả `FeedbackType = Consultation`
- Error cần handle:
  - `400` cho query không hợp lệ
  - `404` khi expert không tồn tại
- Trạng thái: Ready

### 4. Screen: Chọn thời gian đặt lịch

- API: `GET /api/v1/experts/{expertId}/time-slots`
- Request DTO: Không có body
- Response DTO: `ApiResponse<IEnumerable<ExpertTimeSlotResponse>>`
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

### 7. Screen: Consulting Homepage (buổi đặt lịch của tôi)

- API: `GET /api/v1/consultation-bookings/my-bookings`
- Request DTO: Không có body
- Response DTO: `ApiResponse<IEnumerable<ConsultationBookingResponse>>`
- Error cần handle:
  - `401/403` khi chưa đăng nhập hoặc sai role
- Trạng thái: Ready

### 8. Screen: Tài liệu tư vấn (tạo booking đặt lịch)

- API: `POST /api/v1/consultation-bookings`
- Request DTO: `CreateConsultationBookingRequest`
- Response DTO: `ApiResponse<ConsultationBookingResponse>`
- Error cần handle:
  - `400/422` payload không hợp lệ
  - `404` expert/slot không tồn tại
  - `409` slot đã bị user khác đặt trước (race condition)
  - `401/403` khi chưa đăng nhập hoặc sai role
- Trạng thái: Ready

### 9. Screen: Sảnh phòng chờ / Trong phòng tư vấn (join room)

- API: `POST /api/videocall/livekit-token/{consultationId}`
- Request DTO: Không có body
- Response DTO: `ApiResponse<VideoTokenResponse>`
- Error cần handle:
  - `404` consultation không tồn tại
  - `403` không phải participant của consultation
  - `401` chưa đăng nhập
- Trạng thái: Ready

### 10. Screen: Trong phòng tư vấn (kết thúc buổi)

- API: `POST /api/v1/consultations/{consultationId}/end`
- Request DTO: Không có body
- Response DTO: `ApiResponse<object>` (message only)
- Error cần handle:
  - `401/403` không có quyền
  - `404` consultation không tồn tại
- Trạng thái: Ready

### 11. Screen: Hoàn thành (gửi đánh giá)

- API: `POST /api/v1/consultations/{consultationId}/reviews`
- Request DTO: `CreateConsultationReviewRequest`
- Response DTO: `ApiResponse<UserFeedbackResponse>`
- Error cần handle:
  - `400/422` payload không hợp lệ
  - `401/403` sai role hoặc không có quyền review consultation này
  - `404` consultation không tồn tại
- Trạng thái: Ready

### 12. Screen: Chọn loại tư vấn - nhánh tư vấn ngay

- API: `POST /api/v1/consultations/emergency`
- Request DTO: `CreateEmergencyConsultationRequest`
- Response DTO: `ApiResponse<EmergencyConsultationRequestResponse>`
- Error cần handle:
  - `401/403` chưa đăng nhập hoặc sai role
  - `404` expert không tồn tại
  - `409` expert offline hoặc đã có request pending cùng user-expert
- Ghi chú mobile:
  - Response create request có `requestId`, `status`, `expiresAt`.
  - Sau khi tạo request cần gọi `JoinEmergencyRequestRoom(requestId)` để nhận `EmergencyRequestStatusChanged` cho nhánh accept/reject.
  - Trường hợp timeout tự nhiên không có action expert: vẫn cần UX countdown theo `expiresAt`.
- Trạng thái: Ready

### 13. Screen: Expert inbox khẩn cấp

- Realtime:
  - Hub endpoint: `/hubs/expert`
  - Expert gọi `JoinAsExpert`
  - Nhận event `EmergencyConsultationRequest`
- Action API:
  - `POST /api/v1/consultations/emergency-requests/{requestId}/accept`
  - `POST /api/v1/consultations/emergency-requests/{requestId}/reject`
- Error cần handle:
  - `401/403` chưa đăng nhập hoặc sai role expert
  - `404` request không tồn tại
  - `409` request không còn pending
- State transition cần bám:
  - Chỉ cho phép action khi request đang `Pending`
  - Sau `Accepted/Rejected/Expired` phải disable action button và sync trạng thái terminal
- Trạng thái: Ready

### 14. Các màn hình Mobile Not Covered (Operation 01-04)

- Xác nhận thanh toán
- Chat
- Trạng thái: Mobile Not Covered (đợi Operation 05 backend APIs)

## Mobile Build Guidance (Không phụ thuộc operation)

### Build được ngay

- Danh sách chuyên gia
- Profile chuyên gia bản cơ bản (không gồm thống kê nâng cao)
- Chọn slot đặt lịch + gửi tài liệu + tạo booking
- Homepage danh sách booking của user
- Join phòng LiveKit
- End consultation + gửi review
- Thiết đặt chuyên gia (settings + bulk time slots)

### Cần placeholder hoặc feature flag

- Xác nhận thanh toán/checkout đầy đủ
- Chat API riêng cho consultation
