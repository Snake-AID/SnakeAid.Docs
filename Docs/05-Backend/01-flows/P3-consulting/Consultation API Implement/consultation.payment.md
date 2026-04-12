---
doc_role: operation-specific
module: consultation.payment
operation: 06-FEAT-payment-and-stabilization
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-04-12
api_version: v1
owners: [backend-team, mobile-team]
---

# Consultation Payment API Guide

## Overview

Tài liệu này dành cho mobile/frontend tích hợp payment cho consultation flow.

Backend hiện hỗ trợ 2 cách thanh toán cho consultation:

- `WalletBalance`: trừ tiền trực tiếp từ ví hệ thống của user
- `PayOs`: tạo checkout link để user thanh toán qua PayOS

Phạm vi tài liệu:

- scheduled consultation booking payment
- emergency consultation request payment
- consultation PayOS manual confirm
- callback/return behavior mà client cần biết
- money semantics sau khi consultation payment thành công

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
    "timestamp": "2026-04-12T11:30:00Z",
    "validationErrors": null
  }
}
```

## Payment Flow Summary

### Wallet flow

1. Mobile gọi payment endpoint với `paymentMethod = "WalletBalance"`.
2. Backend trừ tiền ví user ngay lập tức.
3. Backend ghi `ConsultationPayment` transaction và trả `status = "Escrowed"`.
4. Mobile có thể coi payment đã thành công, không cần mở external checkout.

### PayOS flow

1. Mobile gọi payment endpoint với `paymentMethod = "PayOs"`.
2. Backend tạo pending transaction và trả `status = "Pending"`.
3. Response có `checkoutUrl`, `orderCode`, `paymentLinkId`.
4. Mobile mở `checkoutUrl` trong webview hoặc browser.
5. Sau khi user thanh toán:
   - PayOS có thể gọi `webhook`
   - browser của user sẽ đi qua `return`
6. Backend sẽ tự động cố gắng confirm payment.
7. Nếu mobile cần fallback chủ động, gọi `POST /api/consultations/payments/confirm` với `transactionId`.
8. Sau confirm thành công, consultation chuyển sang trạng thái escrowed.

## Consultation Money Semantics

### Escrow

- consultation vẫn là escrow flow
- escrow không còn được biểu diễn bằng `system wallet`
- escrow được suy ra từ transaction thật của consultation
- transaction giữ vai trò escrow hold là `ConsultationPayment`
- transaction đóng vai trò sink của escrow là:
  - `ExpertPayout`
  - `ConsultationRefund`
  - `PlatformFee`

### Settlement

- consultation kết thúc không còn đồng nghĩa với expert nhận 100% gross amount
- settlement tạo:
  - `ExpertPayout`: phần expert thực nhận
  - `PlatformFee`: phần nền tảng giữ lại
- khi client đọc transaction/reporting theo consultation, phải expect `PlatformFee`

### Breaking contract already applied

- `ConsultationPaymentResponse.SystemWalletBalanceAfter` đã bị xóa khỏi contract
- client không được suy luận escrow amount từ balance của `system wallet`
- với transaction `PlatformFee`, `UserName` và `FullName` có thể là `null`

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
    "paidAtUtc": "2026-04-12T11:45:10Z",
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
    "referenceId": "8ff2b7ea-321e-4a55-b0f9-29914bc804f3",
    "referenceType": "ScheduledBooking",
    "transactionId": "73d5f609-c1e6-476e-85be-e9f6cfae4d61",
    "amount": 150000,
    "currency": "VND",
    "paymentMethod": "PayOs",
    "status": "Pending",
    "userWalletBalanceAfter": null,
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
    "paidAtUtc": "2026-04-12T11:50:00Z",
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
    "paidAtUtc": "2026-04-12T11:52:30Z",
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

- không phải REST JSON endpoint cho app parse
- endpoint trả về HTML page
- thường được mở thông qua webview hoặc browser sau khi user complete checkout
- mobile nên coi đây là browser callback, không phải data API

### `POST /api/v1/PayOs/webhook`

**Description**: Webhook backend-to-backend từ PayOS. Dùng để đồng bộ payment status bất đồng bộ.

**Authentication**: None

**Required Permissions**: None

**Usage note for mobile**

- mobile app không gọi endpoint này
- đây là callback từ PayOS vào backend
- sau webhook thành công, mobile có thể refresh UI hoặc dùng manual confirm nếu cần

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
| userWalletBalanceAfter | decimal \| null | No | Số dư ví user sau giao dịch wallet hoặc sau confirm |
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

## Transaction / Reporting Notes For Client

- nếu app dùng `GET /api/transactions?transType=consultation`, phải expect thêm `PlatformFee`
- không assume consultation settlement chỉ có một `ExpertPayout` bằng toàn bộ gross amount
- với `PlatformFee`, transaction response có thể không có owner user:
  - `UserName` có thể `null`
  - `FullName` có thể `null`

## Mobile Integration Pattern

### Scheduled consultation, wallet

1. `POST /api/consultations/scheduled/{bookingId}/payments`
2. không cần gọi confirm endpoint
3. khi consultation kết thúc, gọi `POST /api/consultations/{consultationId}/end`
4. backend tự settle escrow sau bước end

### Scheduled consultation, PayOS

1. `POST /api/consultations/scheduled/{bookingId}/payments`
2. user hoàn tất PayOS
3. backend confirm qua PayOS return/webhook; nếu app đang dùng fallback thủ công thì gọi `POST /api/consultations/payments/confirm`
4. khi consultation kết thúc, gọi `POST /api/consultations/{consultationId}/end`
5. backend tự settle escrow sau bước end

### Instant consultation

1. `POST /api/consultations/instant/{requestId}/payments`
2. confirm nếu là PayOS
3. `POST /api/consultations/{consultationId}/end` khi kết thúc
4. backend tự settle escrow

### What mobile must stop doing

- không parse hoặc map `ConsultationPaymentResponse.SystemWalletBalanceAfter`
- không build UI state kiểu `system wallet increased => escrowed`
- không dùng `EscrowHold` / `EscrowRelease` để mô tả consultation escrow state
- không assume consultation settlement = chỉ có một `ExpertPayout` bằng toàn bộ gross amount

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

- chưa có dedicated endpoint để query payment status riêng cho consultation
- `GET /api/v1/PayOs/return` trả HTML, không phải JSON contract cho mobile
- mobile nên lưu `transactionId`, `orderCode`, `checkoutUrl` ngay khi nhận response `Pending`

Tài liệu này nên được dùng như contract để mobile/frontend build payment option selector, payment status handling, và consultation settlement reporting đúng semantic hiện tại.
