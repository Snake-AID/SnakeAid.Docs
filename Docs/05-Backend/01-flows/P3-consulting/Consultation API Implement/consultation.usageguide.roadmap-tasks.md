---
doc_role: baseline
module: consultation.roadmap-tasks
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-28
owners: [backend-team, mobile-team]
---

# Consultation Roadmap Tasks — Hướng dẫn tích hợp

## LƯU Ý CHUYỂN ĐỔI (Mobile dev đọc đây trước)

Màn hình "Cuộc họp hiện tại" và "Lịch sử cuộc họp" hiện đang gọi `GET /api/users/me/consultation-bookings`.

Endpoint đó CHỈ trả scheduled bookings. Emergency consultations KHÔNG xuất hiện.

Chuyển sang endpoint mới: `GET /api/users/me/consultations`

| | Endpoint cũ | Endpoint mới |
|---|------------|-------------|
| URL | `/api/users/me/consultation-bookings` | `/api/users/me/consultations` |
| Scheduled | Có | Có |
| Emergency | KHÔNG | Có |
| Lọc theo trạng thái | Không | `?status=Ongoing` hoặc `?status=Completed` |
| Lọc theo loại | Không | `?type=Scheduled` hoặc `?type=Emergency` |
| Phân trang | Không | `?pageNumber=1&pageSize=10` |
| Response shape | `ConsultationBookingResponse` | `MyConsultationResponse` (xem Mục 2) |

Endpoint cũ vẫn hoạt động, không bị break. Nhưng nếu mobile muốn hiện emergency thì PHẢI dùng endpoint mới.

Giữ endpoint cũ cho màn hình "Booking của tôi" (bao gồm cả booking chưa thanh toán, chưa có consultation).

---

## Phạm vi

Tài liệu này mô tả kết quả của 6 task trong consultation roadmap. Dành cho mobile dev tích hợp.

## Tổng quan các task

| # | Task | Kết quả | Tài liệu tham chiếu |
|---|------|---------|----------------------|
| 1 | PayOS cho consultation | ĐÃ IMPLEMENT | `consultation.payment.md` |
| 2 | Khóa slot khi có emergency request | ĐÃ IMPLEMENT | Giải thích bên dưới |
| 3 | Nạp tiền vào ví | ENDPOINT MỚI | Mục 1 bên dưới |
| 4 | Cuộc họp hiện tại + lịch sử bao gồm emergency | ENDPOINT MỚI | Mục 2 bên dưới |
| 5 | Xem đánh giá consultation | ENDPOINT MỚI | Mục 3 bên dưới |

---

## Task 1: PayOS cho consultation (đã implement)

Consultation payment đã hỗ trợ 2 phương thức: `WalletBalance` và `PayOs`.

Chi tiết contract, request/response, error catalog, webhook/return handling: xem `consultation.payment.md`.

---

## Task 2: Khóa slot khi có emergency request (đã implement)

Khi expert chấp nhận một emergency consultation, backend tự động reserve các `ExpertTimeSlot` chồng lấn trong 30 phút tiếp theo. Điều này ngăn user khác book slot của expert đó trong khi emergency đang diễn ra.

Logic: trong transaction accept, backend query tất cả slot của expert có `Status = Available` và overlap với window `[now, now + 30min]`, chuyển sang `Reserved`.

Mobile không cần làm gì đặc biệt — đây là backend-side protection. Nếu user cố gắng book slot đã bị reserve, sẽ nhận `409 Conflict`.

---

## Các endpoint mới

| Endpoint | Mục đích |
|----------|---------|
| `POST /api/wallet/topup` | Nạp tiền vào ví qua PayOS |
| `GET /api/users/me/consultations` | Lịch sử consultation (scheduled + emergency) |
| `GET /api/consultations/{id}/reviews` | Xem đánh giá của consultation |

---

## Mục 1: `POST /api/wallet/topup`

Tạo PayOS payment link để user nạp tiền vào ví hệ thống.

**Xác thực**: Bearer JWT, role `User`

**Request**:
```json
{
  "amount": 100000,
  "description": "Nạp tiền ví"
}
```

| Trường | Kiểu | Bắt buộc | Ràng buộc |
|--------|------|----------|-----------|
| amount | decimal | Có | 1,000 - 10,000,000 VND |
| description | string | Không | Tối đa 200 ký tự |

**Response** (`200 OK`):
```json
{
  "status_code": 200,
  "message": "Wallet top-up payment link created successfully",
  "is_success": true,
  "data": {
    "transactionId": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
    "userId": "11111111-2222-3333-4444-555555555555",
    "amount": 100000,
    "status": "Pending",
    "checkoutUrl": "https://pay.payos.vn/web/...",
    "orderCode": 1741499999123,
    "paymentLinkId": "plink_xxx",
    "expiresAt": null,
    "provider": "PayOS",
    "gatewayRawResponse": { ... }
  }
}
```

**Luồng mobile**:
1. Gọi `POST /api/wallet/topup`
2. Lấy `checkoutUrl` từ response
3. Mở browser/webview
4. Sau khi user thanh toán, PayOS webhook tự động cộng tiền vào ví
5. Nếu cần fallback: refresh số dư ví qua `GET /api/wallet/me`

**Lỗi**:
- `400`: số tiền <= 0 hoặc > 10,000,000
- `400`: đã có pending top-up transaction chưa hoàn thành
- `404`: ví chưa tồn tại cho user

---

## Mục 2: `GET /api/users/me/consultations`

Trả tất cả consultations của user (cả scheduled và emergency), có lọc và phân trang.

**Xác thực**: Bearer JWT, role `User`

**Query params**:

| Tham số | Kiểu | Bắt buộc | Giá trị | Mặc định |
|---------|------|----------|---------|----------|
| status | string | Không | `Ongoing`, `Completed` | tất cả |
| type | string | Không | `Scheduled`, `Emergency` | tất cả |
| pageNumber | int | Không | >= 1 | 1 |
| pageSize | int | Không | 1-100 | 10 |

**Ví dụ gọi**:
- Cuộc họp hiện tại: `GET /api/users/me/consultations?status=Ongoing`
- Lịch sử: `GET /api/users/me/consultations?status=Completed`
- Chỉ emergency: `GET /api/users/me/consultations?type=Emergency`

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
        "status": "Ongoing",
        "expertId": "3fa85f64-...",
        "expertName": "Dr. John Doe",
        "roomId": "consultation-bfa25be0...",
        "startTime": "2026-03-28T10:00:00Z",
        "endTime": null,
        "price": 150000,
        "problemDescription": "Snake bite swelling",
        "bookingId": "7da8e6c6-...",
        "slotStartTime": "2026-03-28T10:00:00Z",
        "slotEndTime": "2026-03-28T10:30:00Z",
        "emergencyRequestId": null
      },
      {
        "consultationId": "ccc11111-...",
        "type": "Emergency",
        "status": "Completed",
        "expertId": "3fa85f64-...",
        "expertName": "Dr. John Doe",
        "roomId": "consultation-ccc11111...",
        "startTime": "2026-03-27T14:00:00Z",
        "endTime": "2026-03-27T14:28:00Z",
        "price": null,
        "problemDescription": null,
        "bookingId": null,
        "slotStartTime": null,
        "slotEndTime": null,
        "emergencyRequestId": "d90d01f7-..."
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
| type | string | `Scheduled` hoặc `Emergency` |
| status | string | `Ongoing` hoặc `Completed` |
| price | decimal? | Có cho scheduled, null cho emergency |
| problemDescription | string? | Có cho scheduled, null cho emergency |
| bookingId | guid? | Có cho scheduled, null cho emergency |
| slotStartTime/slotEndTime | datetime? | Có cho scheduled, null cho emergency |
| emergencyRequestId | guid? | Có cho emergency, null cho scheduled |

**Lưu ý cho mobile**:
- Endpoint này KHÔNG thay thế `GET /api/users/me/consultation-bookings` (endpoint cũ vẫn hoạt động)
- Endpoint cũ chỉ trả scheduled bookings (kể cả chưa có consultation, ví dụ `PendingPayment`)
- Endpoint mới chỉ trả consultations đã được tạo (đã có `consultationId`)
- Dùng endpoint mới cho màn hình "cuộc họp hiện tại" và "lịch sử cuộc họp"
- Dùng endpoint cũ cho màn hình "booking của tôi" (bao gồm cả booking chưa thanh toán)

---

## Mục 3: `GET /api/consultations/{consultationId}/reviews`

Lấy đánh giá của một consultation cụ thể. Chỉ participant (caller hoặc callee) mới xem được.

**Xác thực**: Bearer JWT

**Path params**: `consultationId` (guid)

**Response khi có đánh giá** (`200 OK`):
```json
{
  "status_code": 200,
  "is_success": true,
  "data": {
    "id": "5e801b64-...",
    "raterId": "2a6d6f8f-...",
    "targetUserId": "3fa85f64-...",
    "referenceId": "bfa25be0-...",
    "type": "Consultation",
    "rating": 5,
    "comments": "Very helpful consultation.",
    "createdAt": "2026-03-28T10:10:00Z",
    "updatedAt": "2026-03-28T10:10:00Z",
    "raterName": "Alice Smith",
    "targetUserName": "Dr. John Doe",
    "updatedAverageRating": 0,
    "updatedRatingCount": 0
  }
}
```

**Response khi chưa có đánh giá** (`200 OK`):
```json
{
  "status_code": 200,
  "message": "No review found for this consultation.",
  "is_success": true,
  "data": null
}
```

**Lỗi**:
- `404`: consultation không tồn tại
- `403`: user không phải participant của consultation

**Lưu ý cho mobile**:
- Endpoint này trả đánh giá cho 1 consultation cụ thể, không phải danh sách
- Dùng để hiển thị đánh giá trên màn hình kết thúc consultation hoặc lịch sử
- Cả user (caller) và expert (callee) đều xem được
- `updatedAverageRating` và `updatedRatingCount` trả 0 vì đây là read endpoint, không phải create
