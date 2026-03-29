---
doc_role: baseline
module: consultation.emergency
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-30
owners: [backend-team, mobile-team]
---

# Emergency Consultation — Usage Guide

## Scope

Tư vấn ngay: create request → pay → expert accept/reject → video room. Bao gồm SignalR presence và request room.

## User Flow

### Step 1: `POST /api/consultations/instant`

Tạo emergency request cho expert đang online.

**Request**: `{ "expertId": "3fa85f64-..." }`

**Response**: `ApiResponse<EmergencyConsultationRequestResponse>`

```json
{
  "data": {
    "requestId": "d90d01f7-...",
    "requesterId": "2a6d6f8f-...",
    "expertId": "3fa85f64-...",
    "status": "PendingPayment",
    "requestedAt": "2026-03-08T09:31:00Z",
    "expiresAt": null,
    "respondedAt": null,
    "consultationId": null,
    "roomId": null
  }
}
```

### Step 2: Join request room

Ngay sau create success, gọi SignalR:

```
JoinEmergencyRequestRoom(requestId)
```

Để nhận `EmergencyRequestStatusChanged` cho request vừa tạo.

### Step 3: `POST /api/consultations/instant/{requestId}/payments`

Thanh toán request. Chi tiết payment contract xem `consultation.payment.md`.

**Request**: `{ "paymentMethod": "WalletBalance" }` hoặc `{ "paymentMethod": "PayOs" }`

Sau payment success:
- Request chuyển `PendingExpertResponse`
- `ExpiresAt` được set (TTL 2 phút)
- Backend push `EmergencyConsultationRequest` sang expert

### Step 4: Chờ expert response

User nhận `EmergencyRequestStatusChanged` qua SignalR:

```json
{
  "requestId": "d90d01f7-...",
  "requesterId": "2a6d6f8f-...",
  "expertId": "3fa85f64-...",
  "status": "AcceptedByExpert",
  "requestedAt": "2026-03-08T09:31:10Z",
  "expiresAt": "2026-03-08T09:33:10Z",
  "respondedAt": "2026-03-08T09:31:30Z",
  "consultationId": "bfa25be0-...",
  "roomId": "consultation-bfa25be0..."
}
```

**Status values mobile phải handle**: `PendingPayment`, `PendingExpertResponse`, `AcceptedByExpert`, `DeclinedByExpert`, `Expired`

### Step 5: Vào phòng

Nếu `AcceptedByExpert` → dùng `consultationId` để lấy LiveKit token (xem scheduled flow).

## Expert Flow

### Nhận request

Expert connect `/hubs/expert` → gọi `JoinAsExpert` → nhận `EmergencyConsultationRequest`:

```json
{
  "requestId": "d90d01f7-...",
  "requesterId": "2a6d6f8f-...",
  "expertId": "3fa85f64-...",
  "requestedAt": "2026-03-08T09:31:10Z",
  "expiresAt": "2026-03-08T09:33:10Z"
}
```

### `POST /api/consultations/instant/{requestId}/accept`

Không có body. Chỉ targeted expert mới gọi được.

**Response**: `EmergencyConsultationRequestResponse` với `status: "AcceptedByExpert"`, `consultationId`, `roomId`.

Side effects: tạo consultation `Ongoing`, reserve overlapping `ExpertTimeSlot` trong 30 phút (Slot Paradox).

### `POST /api/consultations/instant/{requestId}/reject`

Không có body. Chỉ targeted expert mới gọi được.

**Response**: `EmergencyConsultationRequestResponse` với `status: "DeclinedByExpert"`.

Side effects: refund escrow ngay lập tức về member wallet.

## State Machine

```
PendingPayment → (payment) → PendingExpertResponse → AcceptedByExpert → (consultation)
                                                    → DeclinedByExpert → refund
                                                    → Expired → refund
```

## Money Flow

- Payment success → escrow vào system wallet
- Reject/Expired → refund escrow → member wallet
- Consultation complete → settle escrow → expert wallet

## Room Expiry (auto-complete khi hết 30 phút)

Emergency consultation có thời lượng cố định 30 phút (tính từ `StartTime`). Khi hết giờ, backend tự động:
1. Gửi signal `RoomExpiring` qua ConsultationHub (payload: `{ "ConsultationId": guid, "Reason": "slot_elapsed" }`)
2. Xóa phòng LiveKit → kick tất cả participant
3. Cập nhật status `Completed`, `EndTime = DateTime.UtcNow`
4. Settlement escrow → expert

Mobile nên listen event `RoomExpiring` trên `/hubs/consultation` để hiển thị thông báo trước khi phòng bị đóng. Chi tiết: xem `consultation.usageguide.expert-history.md` Mục 2.

## Error Notes

- `POST .../payments`: `409` nếu request đã paid, không còn `PendingPayment`, hoặc expert offline
- `POST .../accept|reject`: `403` nếu không phải targeted expert, `409` nếu request không còn pending
