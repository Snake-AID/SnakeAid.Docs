---
doc_role: baseline
module: consultation.expert-history
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-30
owners: [backend-team, mobile-team]
---

# Expert Consultation History & Room Expiry — Usage Guide

## Scope

Hai tính năng mới:
1. Endpoint lịch sử tư vấn cho Expert (scheduled + emergency)
2. Tự động kick user khi phòng hết giờ (SignalR signal `RoomExpiring`)

---

## Mục 1: `GET /api/experts/me/consultations`

Trả tất cả consultations mà expert là callee (cả scheduled và emergency), có lọc và phân trang.

**Xác thực**: Bearer JWT, role `Expert`

**Query params**:

| Tham số | Kiểu | Bắt buộc | Giá trị | Mặc định |
|---------|------|----------|---------|----------|
| status | string | Không | `Ongoing`, `Completed` | tất cả |
| type | string | Không | `Scheduled`, `Emergency` | tất cả |
| pageNumber | int | Không | >= 1 | 1 |
| pageSize | int | Không | 1-100 | 10 |

**Ví dụ gọi**:
- Cuộc họp hiện tại: `GET /api/experts/me/consultations?status=Ongoing`
- Lịch sử: `GET /api/experts/me/consultations?status=Completed`
- Chỉ emergency: `GET /api/experts/me/consultations?type=Emergency`
- Kết hợp: `GET /api/experts/me/consultations?status=Completed&type=Scheduled&pageNumber=1&pageSize=20`

**Response** (`200 OK`):
```json
{
  "status_code": 200,
  "is_success": true,
  "data": {
    "items": [
      {
        "consultationId": "bfa25be0-...",
        "type": "Scheduled",
        "status": "Completed",
        "userId": "2a6d6f8f-...",
        "userName": "Alice Smith",
        "roomId": "consultation-bfa25be0...",
        "startTime": "2026-03-28T10:00:00Z",
        "endTime": "2026-03-28T10:30:00Z",
        "price": 150000,
        "bookingId": "7da8e6c6-...",
        "slotStartTime": "2026-03-28T10:00:00Z",
        "slotEndTime": "2026-03-28T10:30:00Z",
        "emergencyRequestId": null
      },
      {
        "consultationId": "ccc11111-...",
        "type": "Emergency",
        "status": "Completed",
        "userId": "d90d01f7-...",
        "userName": "Bob Johnson",
        "roomId": "consultation-ccc11111...",
        "startTime": "2026-03-27T14:00:00Z",
        "endTime": "2026-03-27T14:28:00Z",
        "price": null,
        "bookingId": null,
        "slotStartTime": null,
        "slotEndTime": null,
        "emergencyRequestId": "e12e34f5-..."
      }
    ],
    "meta": {
      "currentPage": 1,
      "pageSize": 10,
      "totalItems": 2,
      "totalPages": 1
    }
  }
}
```

**Giải thích các trường**:

| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| consultationId | guid | ID consultation |
| type | string | `Scheduled` hoặc `Emergency` |
| status | string | `Ongoing` hoặc `Completed` |
| userId | guid | CallerId — user đã gọi tư vấn |
| userName | string? | Tên user (caller) |
| roomId | string? | Tên phòng LiveKit |
| startTime | datetime? | Thời điểm bắt đầu consultation |
| endTime | datetime? | Thời điểm kết thúc (null nếu đang Ongoing) |
| price | decimal? | Có cho scheduled (từ booking), null cho emergency |
| bookingId | guid? | Có cho scheduled, null cho emergency |
| slotStartTime | datetime? | Có cho scheduled, null cho emergency |
| slotEndTime | datetime? | Có cho scheduled, null cho emergency |
| emergencyRequestId | guid? | Có cho emergency, null cho scheduled |

### So sánh với endpoint User

| | User endpoint | Expert endpoint |
|---|-------------|----------------|
| URL | `GET /api/users/me/consultations` | `GET /api/experts/me/consultations` |
| Role | `User` | `Expert` |
| Đối tác hiển thị | `expertId`, `expertName` | `userId`, `userName` |
| `problemDescription` | Có | Không |
| Filter/Pagination | Giống nhau | Giống nhau |
| Sort | `startTime` descending | `startTime` descending |

### Lưu ý cho mobile

- Endpoint này KHÔNG thay thế `GET /api/experts/me/consultations/scheduled` (endpoint cũ vẫn hoạt động)
- Endpoint cũ chỉ trả scheduled bookings (Confirmed + Completed), response shape là `ConsultationBookingResponse`
- Endpoint mới trả tất cả consultations (cả emergency), response shape là `ExpertConsultationResponse`
- Dùng endpoint mới cho màn hình "lịch sử tư vấn" và "cuộc họp hiện tại" của expert
- Dùng endpoint cũ nếu cần xem chi tiết booking (bao gồm `problemDescription`)

### Lỗi

- `401`: thiếu JWT
- `403`: role không phải Expert
- `400`: `pageNumber < 1` hoặc `pageSize` ngoài 1-100

---

## Mục 2: Room Expiry — SignalR `RoomExpiring` Signal

Khi consultation hết giờ (scheduled: slot hết giờ, emergency: hết 30 phút), backend tự động:
1. Gửi signal `RoomExpiring` qua ConsultationHub
2. Xóa phòng LiveKit (kick tất cả participant)
3. Cập nhật status `Completed`
4. Settlement escrow → expert

### Signal `RoomExpiring`

**Hub**: `/hubs/consultation`

**Event name**: `RoomExpiring`

**Payload**:
```json
{
  "ConsultationId": "bfa25be0-...",
  "Reason": "slot_elapsed"
}
```

| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| ConsultationId | guid | ID consultation sắp bị đóng |
| Reason | string | Luôn là `"slot_elapsed"` |

### Luồng mobile nên xử lý

```
1. Nhận event "RoomExpiring"
2. Hiển thị thông báo "Phòng sắp đóng do hết giờ"
3. Sau vài giây, LiveKit sẽ ngắt kết nối (backend xóa phòng)
4. Navigate về màn hình kết thúc / đánh giá
```

### Timing

- Signal được gửi TRƯỚC KHI xóa phòng LiveKit
- Giữa signal và room deletion có thể có vài trăm ms
- Signal là best-effort: nếu user đã disconnect thì không nhận được
- Sau khi phòng bị xóa, LiveKit SDK sẽ tự fire disconnect event

### Khi nào xảy ra

| Loại | Điều kiện hết giờ | Tần suất check |
|------|-------------------|----------------|
| Scheduled | `TimeSlot.EndTime <= DateTime.UtcNow` | Mỗi 30 giây |
| Emergency | `StartTime + 30 phút <= DateTime.UtcNow` | Mỗi 30 giây |

### Lưu ý cho mobile

- Mobile KHÔNG cần gọi API nào để trigger room expiry — hoàn toàn tự động từ backend
- Nếu user đang trong phòng khi hết giờ, sẽ nhận `RoomExpiring` signal → sau đó LiveKit disconnect
- Nếu user đã rời phòng trước khi hết giờ, không ảnh hưởng gì
- Consultation status sẽ chuyển sang `Completed` sau khi phòng bị xóa
- Settlement (giải ngân cho expert) xảy ra tự động sau khi status update

---

## Screen Delivery Status

- Expert History (Expert): `Build Now`
- Room Expiry Handling (User + Expert): `Build Now`
