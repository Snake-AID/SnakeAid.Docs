---
doc_role: operation-specific
module: consultation.payment
operation: 05-payment-and-stabilization
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-21
api_version: v1
owners: [backend-team, mobile-team]
---

# Consultation Payment API Guide

## Overview

Tài liệu này dành cho mobile dev tích hợp payment cho consultation flow.

Backend hiện hỗ trợ 2 cách thanh toán cho consultation:

- `WalletBalance`: trừ tiền trực tiếp từ ví hệ thống của user
- `PayOs`: tạo checkout link để user thanh toán qua PayOS

Phạm vi tài liệu:

- scheduled consultation booking payment
- emergency consultation request payment
- consultation PayOS manual confirm
- hành vi callback của PayOS mà mobile cần biết để build flow

Tiền quy ước:

- tất cả request/response JSON đều dùng UTF-8
- enum request/response được serialize dưới dạng chuỗi
- datetime dùng ISO 8601 UTC

## Authentication & Authorization

### REST endpoints cho consultation payment

- Authentication: `Bearer JWT`
- Required role: `User`
- Header:

```http
Authorization: Bearer {{TOKEN}}
Content-Type: application/json
```

Nếu thiếu token:

- HTTP `401 Unauthorized`
- message: `Authentication required. Please provide a valid token.`

Nếu token hợp lệ nhưng không đúng role:

- HTTP `403 Forbidden`
- message: `Access denied. You don't have permission to access this resource.`

### PayOS callback endpoints

- `GET /api/v1/PayOs/return`: anonymous
- `POST /api/v1/PayOs/webhook`: anonymous

Mobile app không cần gắn JWT cho 2 endpoint này.

## Response Envelope

Tất cả consultation payment REST success/error response đều nằm trong `ApiResponse<T>`:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {}
}
```

Response error:

```json
{
  "status_code": 409,
  "message": "Consultation booking is no longer waiting for payment.",
  "is_success": false,
  "data": null,
  "error": {
    "errorCode": "CONFLICT",
    "timestamp": "2026-03-21T11:30:00Z",
    "validationErrors": null
  }
}
```

## Payment Flow Summary

### Wallet flow

1. Mobile gọi payment endpoint với `paymentMethod = "WalletBalance"`.
2. Backend trừ tiền ví user ngay lập tức và đưa tiền vào escrow.
3. API trả về `status = "Escrowed"`.
4. Mobile có thể coi payment đã thành công, không cần mở external checkout.

### PayOS flow

1. Mobile gọi payment endpoint với `paymentMethod = "PayOs"`.
2. Backend tạo pending transaction và trả về `status = "Pending"`.
3. Response có `checkoutUrl`, `orderCode`, `paymentLinkId`.
4. Mobile mở `checkoutUrl` trong webview hoặc browser.
5. Sau khi user thanh toán:
   - PayOS có thể gọi `webhook`
   - browser của user sẽ đi qua `return`
6. Backend sẽ tự động cố gắng confirm payment.
7. Nếu mobile cần fallback chủ động, gọi `POST /api/consultations/payments/confirm` với `transactionId`.

## Endpoints

### `POST /api/consultations/scheduled/{bookingId}/payments`

**Description**: Thanh toán cho scheduled consultation booking.

**Authentication**: Required

**Required Permissions**: `User`

**Request Headers**

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | `Bearer {{TOKEN}}` |
| Content-Type | string | Yes | `application/json` |

**Path Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| bookingId | guid | Yes | ID của booking đang ở trạng thái `PendingPayment` |

**Request Body**

```json
{
  "paymentMethod": "WalletBalance"
}
```

Hoặc:

```json
{
  "paymentMethod": "PayOs"
}
```

**Request Body Schema**

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| paymentMethod | string | Yes | `WalletBalance` hoặc `PayOs` | Payment option user chọn |

**Success Response** (`200 OK`, wallet)  
Payment đã vào escrow ngay:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "8ff2b7ea-321e-4a55-b0f9-29914bc804f3",
    "referenceType": "ScheduledBooking",
    "transactionId": "73d5f609-c1e6-476e-85be-e9f6cfae4d61",
    "amount": 150000,
    "currency": "VND",
    "paymentMethod": "WalletBalance",
    "status": "Escrowed",
    "userWalletBalanceAfter": 350000,
    "systemWalletBalanceAfter": 2150000,
    "paidAtUtc": "2026-03-21T11:45:10Z",
    "provider": "Wallet",
    "checkoutUrl": null,
    "orderCode": null,
    "paymentLinkId": null,
    "externalTransactionId": null
  }
}
```

**Success Response** (`200 OK`, PayOS)  
Payment mới ở trạng thái chờ thanh toán:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "8ff2b7ea-321e-4a55-b0f9-29914bc804f3",
    "referenceType": "ScheduledBooking",
    "transactionId": "73d5f609-c1e6-476e-85be-e9f6cfae4d61",
    "amount": 150000,
    "currency": "VND",
    "paymentMethod": "PayOs",
    "status": "Pending",
    "userWalletBalanceAfter": null,
    "systemWalletBalanceAfter": null,
    "paidAtUtc": null,
    "provider": "PayOS",
    "checkoutUrl": "https://pay.payos.vn/web/3b1d7d1f6b0d4d84b72b2b2f6f3a9f91",
    "orderCode": 240321000123,
    "paymentLinkId": "3b1d7d1f6b0d4d84b72b2b2f6f3a9f91",
    "externalTransactionId": null
  }
}
```

### `POST /api/consultations/instant/{requestId}/payments`

**Description**: Thanh toán cho emergency consultation request.

**Authentication**: Required

**Required Permissions**: `User`

**Request Headers**

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | `Bearer {{TOKEN}}` |
| Content-Type | string | Yes | `application/json` |

**Path Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| requestId | guid | Yes | ID của emergency request đang ở trạng thái `PendingPayment` |

**Request Body**

```json
{
  "paymentMethod": "PayOs"
}
```

**Success Response** (`200 OK`, wallet)

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "22593c11-d276-4cf5-beb3-5b2b3cb5e4b7",
    "referenceType": "EmergencyRequest",
    "transactionId": "8bfc2fb7-6f01-4a1d-9a41-71a0b9c28948",
    "amount": 250000,
    "currency": "VND",
    "paymentMethod": "WalletBalance",
    "status": "Escrowed",
    "userWalletBalanceAfter": 100000,
    "systemWalletBalanceAfter": 2400000,
    "paidAtUtc": "2026-03-21T11:50:00Z",
    "provider": "Wallet",
    "checkoutUrl": null,
    "orderCode": null,
    "paymentLinkId": null,
    "externalTransactionId": null
  }
}
```

**Success Response** (`200 OK`, PayOS)

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "22593c11-d276-4cf5-beb3-5b2b3cb5e4b7",
    "referenceType": "EmergencyRequest",
    "transactionId": "8bfc2fb7-6f01-4a1d-9a41-71a0b9c28948",
    "amount": 250000,
    "currency": "VND",
    "paymentMethod": "PayOs",
    "status": "Pending",
    "userWalletBalanceAfter": null,
    "systemWalletBalanceAfter": null,
    "paidAtUtc": null,
    "provider": "PayOS",
    "checkoutUrl": "https://pay.payos.vn/web/421fdde4d0bb4a3ca0a2ea43d93cf921",
    "orderCode": 240321000124,
    "paymentLinkId": "421fdde4d0bb4a3ca0a2ea43d93cf921",
    "externalTransactionId": null
  }
}
```

### `POST /api/consultations/payments/confirm`

**Description**: Manual fallback để mobile tự confirm consultation PayOS payment khi chưa thấy state được đồng bộ.

**Authentication**: Required

**Required Permissions**: `User`

**Request Headers**

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | `Bearer {{TOKEN}}` |
| Content-Type | string | Yes | `application/json` |

**Request Body**

```json
{
  "transactionId": "73d5f609-c1e6-476e-85be-e9f6cfae4d61"
}
```

**Request Body Schema**

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| transactionId | guid | Yes | non-empty GUID | Consultation payment transaction đã nhận từ response `Pending` |

**Success Response** (`200 OK`)

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "referenceId": "8ff2b7ea-321e-4a55-b0f9-29914bc804f3",
    "referenceType": "ScheduledBooking",
    "transactionId": "73d5f609-c1e6-476e-85be-e9f6cfae4d61",
    "amount": 150000,
    "currency": "VND",
    "paymentMethod": "PayOs",
    "status": "Escrowed",
    "userWalletBalanceAfter": 350000,
    "systemWalletBalanceAfter": 2150000,
    "paidAtUtc": "2026-03-21T11:52:30Z",
    "provider": "PayOS",
    "checkoutUrl": null,
    "orderCode": 240321000123,
    "paymentLinkId": "3b1d7d1f6b0d4d84b72b2b2f6f3a9f91",
    "externalTransactionId": "PAYOS-TXN-983721"
  }
}
```

### `GET /api/v1/PayOs/return`

**Description**: Browser return URL của PayOS. Backend tự động cố gắng confirm payment nếu PayOS báo `PAID`.

**Authentication**: None

**Required Permissions**: None

**Usage note for mobile**

- Không phải REST JSON endpoint cho app parse.
- Endpoint trả về HTML page.
- Thường được mở thông qua webview hoặc browser sau khi user complete checkout.
- Mobile nên coi đây là browser callback, không phải data API.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| code | string | Yes | PayOS result code |
| id | string | Yes | PayOS transaction id |
| cancel | bool | Yes | User có hủy thanh toán hay không |
| status | string | Yes | Ví dụ `PAID` |
| orderCode | int64 | Yes | Order code đã nhận từ response `Pending` |

**Success Response**

- HTTP `200 OK`
- Content-Type: `text/html`
- Hiển thị trang thành công hoặc thất bại

### `POST /api/v1/PayOs/webhook`

**Description**: Webhook backend-to-backend từ PayOS. Dùng để đồng bộ payment status bất đồng bộ.

**Authentication**: None

**Required Permissions**: None

**Usage note for mobile**

- Mobile app không gọi endpoint này.
- Đây là callback từ PayOS vào backend.
- Sau webhook thành công, mobile có thể refresh UI hoặc dùng manual confirm nếu cần.

## Data Models

### ProcessConsultationPaymentRequest

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| paymentMethod | string | Yes | `WalletBalance` hoặc `PayOs` |

### ConsultationPaymentResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| referenceId | guid | Yes | ID booking hoặc emergency request |
| referenceType | string | Yes | `ScheduledBooking` hoặc `EmergencyRequest` |
| transactionId | guid | Yes | Internal payment transaction id |
| amount | decimal | Yes | Số tiền thanh toán |
| currency | string | Yes | Hiện tại luôn là `VND` |
| paymentMethod | string | Yes | `WalletBalance` hoặc `PayOs` |
| status | string | Yes | `Pending` hoặc `Escrowed` |
| userWalletBalanceAfter | decimal \| null | No | Số dư ví user sau giao dịch wallet hoặc confirm |
| systemWalletBalanceAfter | decimal \| null | No | Số dư escrow hệ thống sau giao dịch |
| paidAtUtc | datetime \| null | No | Thời điểm backend ghi nhận payment thành công |
| provider | string \| null | No | `Wallet` hoặc `PayOS` |
| checkoutUrl | string \| null | No | URL checkout PayOS, chỉ có khi `status = "Pending"` |
| orderCode | int64 \| null | No | PayOS order code |
| paymentLinkId | string \| null | No | PayOS payment link id |
| externalTransactionId | string \| null | No | Transaction id bên ngoài sau khi payment được xác nhận |

## Error Catalog

| Status Code | Error Code | Message | Description | Resolution |
|-------------|------------|---------|-------------|------------|
| 400 | VALIDATION_ERROR | Unsupported payment method: ... | `paymentMethod` không hợp lệ | Gửi `WalletBalance` hoặc `PayOs` |
| 400 | VALIDATION_ERROR | Transaction is not a consultation payment. | Gọi manual confirm với transaction không thuộc consultation | Dùng `transactionId` trả về từ consultation payment API |
| 401 | UNAUTHORIZED | Authentication required. Please provide a valid token. | Thiếu hoặc sai bearer token | Đăng nhập lại và gửi token hợp lệ |
| 403 | FORBIDDEN | Access denied. You don't have permission to access this resource. | User không có role `User` | Dùng đúng tài khoản member token |
| 403 | FORBIDDEN | You are not allowed to pay for this booking. | User không phải owner booking | Gọi bằng đúng account tạo booking |
| 403 | FORBIDDEN | You are not allowed to pay for this emergency request. | User không phải requester của emergency flow | Gọi bằng đúng account tạo request |
| 404 | NOT_FOUND | Consultation booking was not found. | Booking id không tồn tại | Kiểm tra lại booking id |
| 404 | NOT_FOUND | Emergency consultation request was not found. | Request id không tồn tại | Kiểm tra lại request id |
| 404 | NOT_FOUND | Consultation payment transaction was not found. | `transactionId` không tồn tại | Dùng lại transaction id backend đã trả |
| 409 | CONFLICT | Consultation booking is no longer waiting for payment. | Booking đã paid, expired, hoặc không còn pending | Refresh lại booking state |
| 409 | CONFLICT | Consultation booking has already been paid. | Booking đã có giao dịch thành công | Không tạo payment mới |
| 409 | CONFLICT | Emergency consultation request is no longer waiting for payment. | Request không còn ở `PendingPayment` | Refresh emergency state |
| 409 | CONFLICT | Emergency consultation request has already been paid. | Request đã có giao dịch thành công | Không tạo payment mới |
| 409 | CONFLICT | Selected expert is currently offline for immediate consultation. | Emergency request chỉ cho thanh toán khi expert đang online | Chọn expert khác hoặc thử lại sau |
| 409 | CONFLICT | Consultation already has a pending payment transaction. | Đang tồn tại PayOS transaction chưa xong | Dùng lại `checkoutUrl` cũ hoặc confirm transaction hiện tại |
| 409 | CONFLICT | PayOS reports status '...'. Payment cannot be confirmed. | Manual confirm được gọi khi PayOS chưa báo thành công | Chờ webhook hoặc return hoặc đợi user thanh toán xong |
| 500 | INTERNAL_SERVER_ERROR | An unexpected error occurred | Lỗi hệ thống ngoài dự kiến | Retry sau, nếu lặp lại thì escalate backend |

## Webhooks & Return Handling

### Webhook behavior

- PayOS webhook được route qua `POST /api/v1/PayOs/webhook`
- Backend tự detect order code này thuộc consultation hay snake-catching
- Nếu là consultation, backend sẽ cập nhật payment và move money vào escrow

### Return behavior

- Browser return đi qua `GET /api/v1/PayOs/return`
- Nếu query cho thấy thanh toán thành công, backend tự auto-confirm
- Response là HTML, không phải JSON

### Mobile recommendation

- Sau khi đóng webview hoặc browser, mobile nên refresh consultation state
- Nếu UI vẫn chưa thấy payment thành công, gọi:
  - `POST /api/consultations/payments/confirm`
- Dùng `transactionId` đã nhận từ response `Pending`

## Examples

### Example 1: Scheduled booking thanh toán bằng wallet

```bash
curl -X POST "https://api.example.com/api/consultations/scheduled/8ff2b7ea-321e-4a55-b0f9-29914bc804f3/payments" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "paymentMethod": "WalletBalance"
  }'
```

Mobile expectation:

- nhận `status = "Escrowed"`
- đóng payment UI ngay

### Example 2: Emergency request thanh toán bằng PayOS

```bash
curl -X POST "https://api.example.com/api/consultations/instant/22593c11-d276-4cf5-beb3-5b2b3cb5e4b7/payments" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "paymentMethod": "PayOs"
  }'
```

Mobile expectation:

1. lấy `checkoutUrl`
2. mở browser hoặc webview
3. sau khi user quay lại app, refresh state
4. nếu cần, gọi manual confirm

### Example 3: Manual confirm consultation PayOS payment

```bash
curl -X POST "https://api.example.com/api/consultations/payments/confirm" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "transactionId": "8bfc2fb7-6f01-4a1d-9a41-71a0b9c28948"
  }'
```

## Current Limits

- Chưa có dedicated endpoint để query payment status riêng cho consultation
- `GET /api/v1/PayOs/return` trả HTML, không phải JSON contract cho mobile
- Mobile nên lưu `transactionId`, `orderCode`, `checkoutUrl` ngay khi nhận response `Pending`

## Recommended Mobile Integration Pattern

### Cho wallet

- user bấm pay
- gọi payment endpoint
- nếu `status = "Escrowed"` thì hiển thị payment success ngay

### Cho PayOS

- user bấm pay
- gọi payment endpoint
- lưu `transactionId`
- mở `checkoutUrl`
- khi app focus lại:
  - refresh consultation screen
  - nếu chưa thấy state mới, gọi manual confirm

Tài liệu này nên được dùng như contract để mobile build payment option selector cho consultation flow.
