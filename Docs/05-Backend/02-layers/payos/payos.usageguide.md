---
doc_role: baseline
module: payos
kind: layer
doc_type: usageguide
status: active
last_updated: 2026-03-28
owners: [backend-team]
---

# PayOS Layer Usage Guide

## Purpose

Mo ta external contract cua PayOS layer. Layer nay phuc vu 3 domain: snake catching, consultation, wallet top-up.

## Authentication

- `POST /api/v1/PayOs/webhook`: anonymous (PayOS callback)
- `GET /api/v1/PayOs/return`: anonymous (browser return)
- `GET /api/v1/PayOs/cancel`: anonymous (browser cancel)
- Tat ca endpoint khac: require authenticated access (Bearer JWT)

## Snake Catching Endpoints

### `POST /api/v1/PayOs/snakecatching/paylink/create`

Tao payment link cho snake catching request.

**Request**:
```json
{
  "snakeCatchingRequestId": "11111111-2222-3333-4444-555555555555",
  "amount": 150000,
  "description": "Payment for snake rescue",
  "transactionType": "CatchingPayment"
}
```

**Response**:
```json
{
  "data": {
    "transactionId": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
    "snakeCatchingRequestId": "11111111-2222-3333-4444-555555555555",
    "amount": 150000,
    "status": "Pending",
    "checkoutUrl": "https://pay.payos.vn/web/...",
    "orderCode": 1741499999123,
    "paymentLinkId": "plink_xxx",
    "provider": "PayOS"
  }
}
```

### `POST /api/v1/PayOs/snakecatching/paylink/cancel/{orderCode}`

Huy payment link.

**Request**: `{ "cancellationReason": "User cancelled checkout" }`

### `POST /api/v1/PayOs/confirm-payment`

Manual confirm payment (snake catching).

**Request**: `{ "transactionId": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" }`

### `POST /api/v1/PayOs/transfer-to-rescuer`

Chuyen tien tu system wallet sang rescuer wallet sau khi hoan thanh.

**Request**: `{ "snakeCatchingRequestId": "11111111-2222-3333-4444-555555555555" }`

## Consultation Payment Endpoints

Consultation payment su dung PayOS thong qua `ConsultationPaymentService`, khong di qua `PayOsController`.

Chi tiet contract xem: `05-Backend/01-flows/P3-consulting/Consultation API Implement/consultation.payment.md`

Endpoints:
- `POST /api/consultations/scheduled/{bookingId}/payments` (paymentMethod: "PayOs")
- `POST /api/consultations/instant/{requestId}/payments` (paymentMethod: "PayOs")
- `POST /api/consultations/payments/confirm`

## Wallet Top-up Endpoints

Wallet top-up su dung PayOS thong qua `WalletTopupService`.

(Endpoint contract chua co usageguide rieng — can bo sung khi mobile integrate.)

## Shared Callback Endpoints

### `GET /api/v1/PayOs/return`

Browser return sau khi user thanh toan tren PayOS portal. Backend tu auto-confirm neu status = PAID.

Tra ve HTML, khong phai JSON.

### `GET /api/v1/PayOs/cancel`

Browser cancel khi user huy tren PayOS portal.

### `POST /api/v1/PayOs/webhook`

PayOS webhook callback. Backend tu detect order code thuoc domain nao (snake-catching, consultation, wallet top-up) va route xu ly tuong ung.

Anonymous endpoint — khong can JWT.

## Gateway Interface

`IPaymentGateway` expose 4 methods:

| Method | Purpose |
|--------|---------|
| `CreatePaymentLinkAsync` | Tao payment link tren PayOS |
| `CancelPaymentLinkAsync` | Huy payment link |
| `GetPaymentLinkInformationAsync` | Lay thong tin payment link |
| `VerifyWebhook` | Verify webhook payload tu PayOS |

Domain services goi gateway thong qua interface nay. Khong co them abstraction layer.

## Error Patterns

- `409 CONFLICT`: payment da ton tai, request khong con o trang thai cho thanh toan
- `404 NOT_FOUND`: transaction hoac request khong ton tai
- `400 VALIDATION_ERROR`: payment method khong hop le
- `500 INTERNAL_SERVER_ERROR`: PayOS SDK error hoac unexpected failure
