---
doc_role: handoff
module: consultation
kind: api-mapping
status: active
last_updated: 2026-03-18
owners: [backend-team, mobile-team]
---

# Consultation Screen API

## Mục tiêu tài liệu

Tài liệu này chỉ trả lời 4 câu hỏi cho mobile dev:
- màn hình này gọi REST/WS gì
- gọi vào thời điểm nào
- thành công thì chuyển sang màn hình nào
- màn hình đó hiện `Build Now`, `Build With Placeholder`, hay `Wait Backend`

Payload chi tiết của từng endpoint xem tại:
- [consultation.usageguide.md](D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend\01-flows\P3-consulting\Consultation%20API%20Implement\consultation.usageguide.md) (toàn diện tất cả operations)
- [consultation.operation5.usageguide.md](D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend\01-flows\P3-consulting\Consultation%20API%20Implement\consultation.operation5.usageguide.md) (chỉ operation 5 - tối ưu context window)

Screen flow chi tiết cho từng operation:
- [consultation.operation5.screen-api.md](D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend\01-flows\P3-consulting\Consultation%20API%20Implement\consultation.operation5.screen-api.md) (chỉ operation 5 - tối ưu context window)

## Delivery Legend

- `Build Now`: Backend đã đủ để mobile đi end-to-end cho bước đó.
- `Build With Placeholder`: Backend đã có phần lõi, nhưng mobile vẫn phải ẩn bớt hoặc placeholder một phần UI.
- `Wait Backend`: Backend chưa có contract đủ để build flow thật.

## Canonical Status Names

Dùng đúng các tên trạng thái backend sau trong code mobile:
- Booking: `PendingPayment`, `Confirmed`, `Completed`
- Emergency Request: `PendingPayment`, `PendingExpertResponse`, `AcceptedByExpert`, `DeclinedByExpert`, `Expired`

Không tự rút gọn thành `Accepted`, `Rejected`, `Declined`, `Pending` trong code xử lý state. Nếu cần text hiển thị UI, map riêng ở tầng presentation.

## Quick Endpoint Index

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

### In-Room Features

- **Search Snake Species**: `GET /api/snake-species/search?q={query}`

### SignalR

- **Hub**: `/hubs/expert`
- **User Methods**:
  - `JoinAsMember`
  - `JoinEmergencyRequestRoom(requestId)`
- **Expert Methods**:
  - `JoinAsExpert`
- **Events**:
  - `OnlineExpertsSnapshot`
  - `ExpertPresenceChanged`
  - `EmergencyConsultationRequest`
  - `EmergencyRequestStatusChanged`

- **Hub**: `/hubs/consultation`
- **Methods** (both Caller and Callee):
  - `ReceiveMessage(string content, string? attachmentUrl)`
  - `Signal(string eventType, string payload)`
- **Events**:
  - `MessageReceived`
  - `SignalReceived`

## User Journey

### 1. Screen: Consulting Homepage

**Status**: `Build Now`

**Khi mở màn hình**
- `GET /api/users/me/consultation-bookings`

**Mục đích**
- render các booking của user
- biết booking nào còn `PendingPayment`
- biết booking nào đã `Confirmed` để vào phòng

**Đi tiếp**
- `Thanh toán ngay` -> `Screen: Xác nhận thanh toán (Scheduled)`
- `Vào phòng ngay` -> `Screen: Sảnh phòng chờ`

### 2. Screen: Danh sách chuyên gia

**Status**: `Build Now`

**Khi mở màn hình**
- `GET /api/experts`
- connect `/hubs/expert`
- invoke `JoinAsMember`

**WS cần nghe**
- `OnlineExpertsSnapshot`
- `ExpertPresenceChanged`

**Mục đích**
- render list chuyên gia
- cập nhật online/offline realtime

**Đi tiếp**
- chọn expert -> `Screen: Thông tin profile chuyên gia`

### 3. Screen: Thông tin profile chuyên gia

**Status**: `Build With Placeholder`

**Khi mở màn hình**
- `GET /api/experts/{expertId}`
- `GET /api/experts/{expertId}/reviews`
- `GET /api/experts/{expertId}/time-slots`

**Mục đích**
- render profile, review, slot, pricing, stats

**Lưu ý**
- `IsVerified` vẫn là deferred MVP

**Đi tiếp**
- `Đặt lịch / Tư vấn ngay` -> `Screen: Chọn loại tư vấn`

### 4. Screen: Chọn loại tư vấn

**Status**: `Build Now`

**Dữ liệu dùng lại**
- profile expert
- slot khả dụng
- presence realtime từ SignalR

**Đi tiếp**
- `Chọn Tư Vấn Ngay` -> `Usecase: Tư vấn ngay`
- `Chọn Đặt Lịch` -> `Usecase: Đặt lịch tư vấn`

## Usecase: Tư vấn ngay

### 5. Screen: Tư vấn ngay

**Status**: `Build Now`

**Khi user bấm gửi yêu cầu**
- `POST /api/consultations/emergency-requests`
- sau khi create thành công -> invoke `JoinEmergencyRequestRoom(requestId)`

**Mục đích**
- tạo request ở trạng thái `PendingPayment`
- chuẩn bị room để nghe trạng thái request

**Đi tiếp**
- create thành công -> `Screen: Xác nhận thanh toán (Emergency)`

### 6. Screen: Xác nhận thanh toán (Emergency)

**Status**: `Build With Placeholder`

**Khi user bấm thanh toán**
- `POST /api/consultations/emergency-requests/{requestId}/payments`

**WS cần nghe sau payment**
- `EmergencyRequestStatusChanged`

**Mục đích**
- payment thành công -> `PendingExpertResponse`
- backend mới push request sang expert

**Đi tiếp**
- nhận `AcceptedByExpert` -> `Screen: Sảnh phòng chờ`
- nhận `DeclinedByExpert` hoặc `Expired` -> quay lại chọn expert / chọn loại tư vấn theo UX app

**Lưu ý**
- hiện runtime chỉ support `WalletBalance`

### 7. Screen: Sảnh phòng chờ

**Status**: `Build Now`

**Khi vào màn hình**
- `POST /api/videocall/livekit-token/{consultationId}`

**Mục đích**
- lấy token để vào LiveKit room

**Đi tiếp**
- `Join phòng` -> `Screen: Trong phòng tư vấn`

### 8. Screen: Trong phòng tư vấn

**Status**: `Build Now`

**Khi kết thúc buổi tư vấn**
- `POST /api/consultations/{consultationId}/end`

**Mục đích**
- kết thúc consultation

**Đi tiếp**
- end thành công -> `Screen: Hoàn thành`

### 9. Screen: Chat

**Status**: `Build Now`

**Khi vào màn hình chat**
- connect `/hubs/consultation`
- invoke `OnConnectedAsync` (automatic authorization check)
- listen `MessageReceived`, `SignalReceived`

**Khi user gửi tin nhắn**
- upload image trước nếu có: `POST /api/media/upload-image` (domain: "chat-media")
- invoke `ReceiveMessage(string content, string? attachmentUrl)`

**Mục đích**
- chat realtime với expert trong consultation room
- gửi hình ảnh qua attachmentUrl
- rate limit: 10 messages/minute per user

**Đi tiếp**
- chat flow chạy song song với video call
- user có thể gửi signal UI như typing indicator qua `Signal`

### 10. Screen: Hoàn thành

**Status**: `Build With Placeholder`

**Khi user gửi review**
- `POST /api/consultations/{consultationId}/reviews`

**Mục đích**
- gửi đánh giá consultation

**Lưu ý**
- payment summary đầy đủ của màn completion chưa có contract riêng hoàn chỉnh

## Usecase: Đặt lịch tư vấn

### 11. Screen: Chọn thời gian

**Status**: `Build Now`

**Khi mở màn hình**
- `GET /api/experts/{expertId}/time-slots`

**Mục đích**
- render ngày/slot khả dụng

**Đi tiếp**
- chọn slot -> `Screen: Tài liệu tư vấn`

### 12. Screen: Tài liệu tư vấn

**Status**: `Build Now`

**Khi user bấm xác nhận đặt lịch**
- `POST /api/consultation-bookings`

**Mục đích**
- tạo booking ở trạng thái `PendingPayment`

**Đi tiếp**
- create booking thành công -> `Screen: Xác nhận thanh toán (Scheduled)`

### 13. Screen: Xác nhận thanh toán (Scheduled)

**Status**: `Build With Placeholder`

**Khi user bấm thanh toán**
- `POST /api/consultation-bookings/{bookingId}/payments`

**Mục đích**
- payment thành công -> booking `Confirmed`

**Đi tiếp**
- payment thành công -> quay lại homepage hoặc waiting state theo UX app
- tới giờ consultation -> `Screen: Sảnh phòng chờ`

**Lưu ý**
- hiện runtime chỉ support `WalletBalance`

### 14. Screen: Sảnh phòng chờ

**Status**: `Build Now`

**Khi vào màn hình**
- `POST /api/videocall/livekit-token/{consultationId}`

**Đi tiếp**
- `Join phòng` -> `Screen: Trong phòng tư vấn`

### 15. Screen: Trong phòng tư vấn

**Status**: `Build Now`

**Khi kết thúc buổi tư vấn**
- `POST /api/consultations/{consultationId}/end`

**Đi tiếp**
- end thành công -> `Screen: Hoàn thành`

**Lưu ý**
- nếu không ai bấm end, backend vẫn có background lifecycle auto-complete khi qua slot end

### 16. Screen: Chat

**Status**: `Wait Backend`

### 17. Screen: Hoàn thành

**Status**: `Build With Placeholder`

**Khi user gửi review**
- `POST /api/consultations/{consultationId}/reviews`

**Lưu ý**
- payment summary đầy đủ của màn completion chưa có contract riêng hoàn chỉnh

## Expert Journey

### 18. Supporting Screen: Lịch tư vấn đã chốt của expert

**Status**: `Build Now`

**Khi expert mở danh sách ca scheduled đã chốt**
- `GET /api/experts/me/consultation-bookings`

**Mục đích**
- load các ca scheduled đã được member thanh toán thành công
- lấy `consultationId` và `roomId` để expert biết ca nào sắp diễn ra và vào đúng phòng khi tới giờ

**Lưu ý**
- endpoint này là REST pull, chưa có realtime inbox cho scheduled consultation

**Đi tiếp**
- chọn ca đã đến giờ -> `Screen: Sảnh phòng chờ`

### 19. Screen: Thiết đặt

**Status**: `Build Now`

**Khi expert cập nhật hồ sơ**
- `PUT /api/experts/me/settings`

**Khi expert set lịch khả dụng**
- `POST /api/experts/me/time-slots/bulk`

**Mục đích**
- cập nhật pricing, biography, weekly working slots

### 20. Screen: Các tư vấn khẩn cấp

**Status**: `Build Now`

**Khi expert sẵn sàng nhận ca**
- connect `/hubs/expert`
- invoke `JoinAsExpert`
- listen `EmergencyConsultationRequest`

**Khi expert thao tác**
- `POST /api/consultations/emergency-requests/{requestId}/accept`
- `POST /api/consultations/emergency-requests/{requestId}/reject`

**Đi tiếp**
- accept thành công -> `Screen: Sảnh phòng chờ`
- reject thành công -> ở lại inbox / chờ request khác

### 21. Screen: Sảnh phòng chờ

**Status**: `Build Now`

**Khi vào màn hình**
- `POST /api/videocall/livekit-token/{consultationId}`

**Đi tiếp**
- `Join phòng` -> `Screen: Trong phòng tư vấn`

### 22. Screen: Trong phòng tư vấn

**Status**: `Build Now`

**Khi expert kết thúc buổi tư vấn**
- `POST /api/consultations/{consultationId}/end`

**Đi tiếp**
- end thành công -> `Screen: Hoàn tất tư vấn`

### 23. Screen: Chat

**Status**: `Build Now`

**Khi vào màn hình chat**
- connect `/hubs/consultation`
- invoke `OnConnectedAsync` (automatic authorization check)
- listen `MessageReceived`, `SignalReceived`

**Khi expert gửi tin nhắn**
- upload image trước nếu có: `POST /api/media/upload-image` (domain: "chat-media")
- invoke `ReceiveMessage(string content, string? attachmentUrl)`

**Khi expert tìm kiếm snake species**
- `GET /api/snake-species/search?q={query}`

**Mục đích**
- chat realtime với user trong consultation room
- gửi hình ảnh qua attachmentUrl
- search thông tin về snake species với venom/antivenom data
- rate limit: 10 messages/minute per user

**Đi tiếp**
- chat flow chạy song song với video call
- expert có thể gửi signal UI như typing indicator qua `Signal`

### 24. Screen: Hoàn tất tư vấn

**Status**: `Build With Placeholder`

**Đã có**
- end consultation
- settlement backend cho expert

**Chưa có đủ**
- payment breakdown contract riêng cho expert completion screen

## Realtime Checklist

### User app

- vào danh sách chuyên gia:
  - connect hub
  - `JoinAsMember`
- tạo emergency request thành công:
  - `JoinEmergencyRequestRoom(requestId)`
- vào phòng tư vấn (cả scheduled và emergency):
  - connect `/hubs/consultation`
  - automatic `OnConnectedAsync` authorization
  - listen `MessageReceived`, `SignalReceived`
- khi reconnect:
  - invoke lại join methods

### Expert app

- vào trạng thái sẵn sàng:
  - connect hub
  - `JoinAsExpert`
- vào phòng tư vấn (cả scheduled và emergency):
  - connect `/hubs/consultation`
  - automatic `OnConnectedAsync` authorization
  - listen `MessageReceived`, `SignalReceived`
- khi reconnect:
  - invoke lại `JoinAsExpert`

## Coverage Summary

### Build Now

- Consulting Homepage
- Danh sách chuyên gia
- Chọn loại tư vấn
- Tư vấn ngay: create request
- Chọn thời gian đặt lịch
- Tài liệu tư vấn / create booking
- Sảnh phòng chờ
- Trong phòng tư vấn
- Chat (User)
- Chat (Expert)
- Thiết đặt expert
- Lịch tư vấn đã chốt của expert
- Các tư vấn khẩn cấp

### Build With Placeholder

- Thông tin profile chuyên gia
- Xác nhận thanh toán (Emergency)
- Xác nhận thanh toán (Scheduled)
- Hoàn thành
- Hoàn tất tư vấn

### Wait Backend



