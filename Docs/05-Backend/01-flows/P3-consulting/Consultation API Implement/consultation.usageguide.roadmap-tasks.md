---
doc_role: baseline
module: consultation.roadmap-tasks
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-28
owners: [backend-team, mobile-team]
---

# Consultation Roadmap Tasks — Usage Guide

## Scope

Tai lieu nay mo ta ket qua cua 6 task trong consultation roadmap. Danh cho mobile dev tich hop.

## Task Overview

| # | Task | Ket qua | Doc tham chieu |
|---|------|---------|----------------|
| 1 | PayOS cho consultation | DA IMPLEMENT | `consultation.payment.md` |
| 2 | Lock schedule time khi emergency | DA IMPLEMENT | Giai thich ben duoi |
| 3 | Topup tien vao vi | ENDPOINT MOI | Section 1 ben duoi |
| 4 | Cuoc hop hien tai + lich su bao gom emergency | ENDPOINT MOI | Section 2 ben duoi |
| 5 | Get consultation feedback | ENDPOINT MOI | Section 3 ben duoi |

---

## Task 1: PayOS cho consultation (da implement)

Consultation payment da ho tro 2 phuong thuc: `WalletBalance` va `PayOs`.

Chi tiet contract, request/response, error catalog, webhook/return handling: xem `consultation.payment.md`.

---

## Task 2: Lock schedule time khi co emergency request (da implement)

Khi expert accept mot emergency consultation, backend tu dong reserve cac `ExpertTimeSlot` chong lan trong 30 phut tiep theo. Dieu nay ngan user khac book slot cua expert do trong khi emergency dang dien ra.

Logic: trong transaction accept, backend query tat ca slot cua expert co `Status = Available` va overlap voi window `[now, now + 30min]`, chuyen sang `Reserved`.

Mobile khong can lam gi dac biet — day la backend-side protection. Neu user co gang book slot da bi reserve, se nhan `409 Conflict`.

---

## Endpoints moi

| Endpoint | Muc dich |
|----------|---------|
| `POST /api/wallet/topup` | Nap tien vao vi qua PayOS |
| `GET /api/consultations/{id}/reviews` | Xem review cua consultation |
| `GET /api/users/me/consultations` | Lich su consultation (scheduled + emergency) |

---

## 1. `POST /api/wallet/topup`

Tao PayOS payment link de user nap tien vao vi he thong.

**Auth**: Bearer JWT, role `User`

**Request**:
```json
{
  "amount": 100000,
  "description": "Nap tien vi"
}
```

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| amount | decimal | Yes | 1,000 - 10,000,000 VND |
| description | string | No | Max 200 chars |

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

**Mobile flow**:
1. Goi `POST /api/wallet/topup`
2. Lay `checkoutUrl` tu response
3. Mo browser/webview
4. Sau khi user thanh toan, PayOS webhook tu dong credit wallet
5. Neu can fallback: refresh wallet balance qua `GET /api/wallet/me`

**Errors**:
- `400`: amount <= 0 hoac > 10,000,000
- `400`: da co pending top-up transaction chua hoan thanh
- `404`: wallet chua ton tai cho user

---

## 2. `GET /api/users/me/consultations`

Tra tat ca consultations cua user (ca scheduled va emergency), co filter va pagination.

**Auth**: Bearer JWT, role `User`

**Query params**:

| Param | Type | Required | Values | Default |
|-------|------|----------|--------|---------|
| status | string | No | `Ongoing`, `Completed` | all |
| type | string | No | `Scheduled`, `Emergency` | all |
| pageNumber | int | No | >= 1 | 1 |
| pageSize | int | No | 1-100 | 10 |

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

**Response fields**:

| Field | Type | Notes |
|-------|------|-------|
| type | string | `Scheduled` hoac `Emergency` |
| status | string | `Ongoing` hoac `Completed` |
| price | decimal? | Co cho scheduled, null cho emergency |
| problemDescription | string? | Co cho scheduled, null cho emergency |
| bookingId | guid? | Co cho scheduled, null cho emergency |
| slotStartTime/slotEndTime | datetime? | Co cho scheduled, null cho emergency |
| emergencyRequestId | guid? | Co cho emergency, null cho scheduled |

**Luu y cho mobile**:
- Endpoint nay KHONG thay the `GET /api/users/me/consultation-bookings` (endpoint cu van hoat dong)
- Endpoint cu chi tra scheduled bookings (ke ca chua co consultation, vi du `PendingPayment`)
- Endpoint moi chi tra consultations da duoc tao (da co `consultationId`)
- Dung endpoint moi cho man hinh "cuoc hop hien tai" va "lich su cuoc hop"
- Dung endpoint cu cho man hinh "booking cua toi" (bao gom ca booking chua thanh toan)

---

## 3. `GET /api/consultations/{consultationId}/reviews`

Lay review cua mot consultation cu the. Chi participant (caller hoac callee) moi xem duoc.

**Auth**: Bearer JWT

**Path params**: `consultationId` (guid)

**Response khi co review** (`200 OK`):
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

**Response khi chua co review** (`200 OK`):
```json
{
  "status_code": 200,
  "message": "No review found for this consultation.",
  "is_success": true,
  "data": null
}
```

**Errors**:
- `404`: consultation khong ton tai
- `403`: user khong phai participant cua consultation

**Luu y cho mobile**:
- Endpoint nay tra review cho 1 consultation cu the, khong phai list
- Dung de hien thi review tren man hinh ket thuc consultation hoac lich su
- Ca user (caller) va expert (callee) deu xem duoc
- `updatedAverageRating` va `updatedRatingCount` tra 0 vi day la read endpoint, khong phai create
