---
doc_role: operation-specific
module: consultation
operation: 05-FEAT-in-room-features
kind: screen-api
status: active
last_updated: 2026-03-18
owners: [backend-team, mobile-team]
---

# Operation 5: In-Room Features Screen API

## Mục tiêu tài liệu

Tài liệu này chỉ trả lời 4 câu hỏi cho mobile dev về **operation 5 (In-Room Features)**:
- màn hình nào gọi REST/WS gì
- gọi vào thời điểm nào
- thành công thì chuyển sang màn hình nào
- màn hình đó hiện `Build Now`, `Build With Placeholder`, hay `Wait Backend`

**Operation 5 bao gồm:**
- Real-time text chat trong phòng tư vấn
- Gửi hình ảnh trong chat
- Tìm kiếm thông tin loài rắn cho chuyên gia
- UI signaling (typing indicators, mic/camera status)

**Tài liệu chính:**
- `Consultation Screen API.md`: Toàn diện tất cả operations
- Tài liệu này: Chỉ operation 5 (tối ưu context window)
- `consultation.operation5.usageguide.md`: Payload chi tiết của endpoints

## Delivery Legend

- `Build Now`: Backend đã đủ để mobile đi end-to-end cho bước đó.
- `Build With Placeholder`: Backend đã có phần lõi, nhưng mobile vẫn phải ẩn bớt hoặc placeholder một phần UI.
- `Wait Backend`: Backend chưa có contract đủ để build flow thật.

## Canonical Status Names

Dùng đúng các tên trạng thái backend sau trong code mobile:
- Booking: `PendingPayment`, `Confirmed`, `Completed`
- Emergency Request: `PendingPayment`, `PendingExpertResponse`, `AcceptedByExpert`, `DeclinedByExpert`, `Expired`

Không tự rút gọn thành `Accepted`, `Rejected`, `Declined`, `Pending` trong code xử lý state.

## Quick Endpoint Index (Operation 5)

### In-Room Features

- **Search Snake Species**: `GET /api/v1/snakes/search?q={query}`
- **Upload Chat Media**: `POST /api/media/upload-image` (domain: "chat-media")

### SignalR Consultation Chat

- **Hub**: `/hubs/consultation`
- **Methods** (both Caller and Callee):
  - `ReceiveMessage(string content, string? attachmentUrl)`
  - `Signal(string eventType, string payload)`
- **Events**:
  - `MessageReceived`
  - `SignalReceived`

## In-Room Features Screens

### Screen: Chat (User)

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
- không có màn hình chuyển tiếp cụ thể - chat là feature bổ sung

### Screen: Chat (Expert)

**Status**: `Build Now`

**Khi vào màn hình chat**
- connect `/hubs/consultation`
- invoke `OnConnectedAsync` (automatic authorization check)
- listen `MessageReceived`, `SignalReceived`

**Khi expert gửi tin nhắn**
- upload image trước nếu có: `POST /api/media/upload-image` (domain: "chat-media")
- invoke `ReceiveMessage(string content, string? attachmentUrl)`

**Khi expert tìm kiếm snake species**
- `GET /api/v1/snakes/search?q={query}`

**Mục đích**
- chat realtime với user trong consultation room
- gửi hình ảnh qua attachmentUrl
- search thông tin về snake species với venom/antivenom data
- rate limit: 10 messages/minute per user

**Đi tiếp**
- chat flow chạy song song với video call
- expert có thể gửi signal UI như typing indicator qua `Signal`
- không có màn hình chuyển tiếp cụ thể - chat là feature bổ sung

## Realtime Checklist (Operation 5)

### User app

- vào phòng tư vấn (cả scheduled và emergency):
  - connect `/hubs/consultation`
  - automatic `OnConnectedAsync` authorization
  - listen `MessageReceived`, `SignalReceived`
- khi reconnect:
  - invoke lại join methods

### Expert app

- vào phòng tư vấn (cả scheduled và emergency):
  - connect `/hubs/consultation`
  - automatic `OnConnectedAsync` authorization
  - listen `MessageReceived`, `SignalReceived`
- khi reconnect:
  - invoke lại methods

## Coverage Summary (Operation 5)

### Build Now

- Chat (User)
- Chat (Expert)

### Build With Placeholder

- None

### Wait Backend

- None

## Integration Notes

- Chat là feature bổ sung cho video call, không phải flow chính
- User và expert đều connect cùng hub với cùng consultationId
- Authorization tự động check dựa trên consultation participants
- Media upload phải dùng domain "chat-media"
- Rate limiting áp dụng per user per consultation
- Chat history được lưu vĩnh viễn (không có retention policy)

## Related Documentation

- `Consultation Screen API.md`: Toàn diện tất cả operations và screens
- `consultation.operation5.usageguide.md`: Payload chi tiết của operation 5 endpoints