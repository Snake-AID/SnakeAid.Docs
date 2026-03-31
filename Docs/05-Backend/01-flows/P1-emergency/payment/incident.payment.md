---
doc_role: handoff
module: incident-payment
kind: api-integration-guide
status: active
last_updated: 2025-07-15
owners: [backend-team, mobile-team]
---

# Incident Payment — Mobile Integration Guide

## 1. Overview

Tài liệu này mô tả payment flow cho **Flow 1 — Snakebite Incident** (cứu hộ rắn cắn), dành cho mobile developer tích hợp thanh toán.

### Sự khác biệt với Consultation Payment (Flow 3)

| Đặc điểm | Flow 1 — Incident | Flow 3 — Consultation |
|---|---|---|
| Actors | 2: **user ↔ system** | 3: **user ↔ system ↔ expert** |
| Escrow release | System **giữ tiền** (không transfer cho rescuer qua escrow) | `TransferEscrowToExpertAsync` → expert wallet |
| Khi nào thanh toán | Sau khi mission hoàn thành (`Finished → Completed`) | Trước khi bắt đầu tư vấn (`PendingPayment → Confirmed`) |
| Description prefix | `INCIDENT-{orderCode}` | `CONSULTPAY-{orderCode}` |
| TransactionType | `SnakebiteIncidentPayment` (40) | `ConsultationPayment` |
| Refund TransactionType | `SnakebiteIncidentRefund` (41) | `ConsultationRefund` |

**Điểm quan trọng**: Incident payment chỉ có 2 actor. Tiền user vào escrow (System_Wallet), system giữ lại — không có bước chuyển tiền cho rescuer qua escrow như consultation chuyển cho expert.

## 2. Authentication

Tất cả endpoint yêu cầu **Bearer JWT** trong header:

```
Authorization: Bearer <jwt_token>
```

- Endpoint payment (tạo link, wallet, cancel): yêu cầu user đã đăng nhập (`[Authorize]`)
- Endpoint refund: yêu cầu role `Operator` hoặc `Admin` (`[Authorize(Roles = "Operator,Admin")]`)
- Endpoint webhook: không yêu cầu auth (PayOS gọi trực tiếp, verify bằng signature)

## 3. Payment Flow Summary

### 3.1 Wallet Flow

```
POST /api/incidents/{id}/payment/wallet
    → Debit user wallet
    → Credit System_Wallet (escrow)
    → status = "Escrowed"
    → Incident status → Completed
```

Wallet payment là atomic — một request hoàn tất toàn bộ flow.

### 3.2 PayOS Flow

```
POST /api/incidents/{id}/payment/payos
    → Tạo pending transaction (DB)
    → Tạo payment link (PayOS)
    → status = "Pending"
    → Trả về checkoutUrl

User mở checkoutUrl → thanh toán trên PayOS

PayOS callback (webhook hoặc return URL):
    → Credit System_Wallet (escrow)
    → status = "Escrowed"
    → Incident status → Completed
```

PayOS payment là 2 bước: tạo link → user thanh toán → webhook/confirm xác nhận.

## 4. Endpoint Contracts

### 4.1 Tạo PayOS Payment Link

```
POST /api/incidents/{incidentId}/payment/payos
Authorization: Bearer <token>
```

**Request Body:**

```json
{
  "snakebiteIncidentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "amount": 150000,
  "description": "Thanh toán phí cứu hộ rắn cắn",
  "transactionType": 40
}
```

| Field | Type | Bắt buộc | Mô tả |
|---|---|---|---|
| `snakebiteIncidentId` | `Guid` | Có | ID incident (phải trùng với `{incidentId}` trên URL) |
| `amount` | `decimal` | Có | Số tiền thanh toán (phải khớp `ActualCost` hoặc `Price` của mission) |
| `description` | `string?` | Không | Mô tả tùy chọn |
| `transactionType` | `int` | Có | Luôn = `40` (`SnakebiteIncidentPayment`) |

**Response 200 OK:**

```json
{
  "snakebiteIncidentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "transactionId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "orderCode": 1234567890,
  "amount": 150000,
  "currency": "VND",
  "status": "Pending",
  "provider": "PayOS",
  "checkoutUrl": "https://pay.payos.vn/web/abc123...",
  "paymentLinkId": "plk_abc123",
  "expiresAt": "2025-07-15T10:30:00Z",
  "externalTransactionId": null,
  "paidAt": null,
  "userWalletBalanceAfter": null,
  "systemWalletBalanceAfter": null
}
```

### 4.2 Thanh toán bằng Wallet

```
POST /api/incidents/{incidentId}/payment/wallet
Authorization: Bearer <token>
```

**Request Body:** (giống 4.1)

```json
{
  "snakebiteIncidentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "amount": 150000,
  "description": "Thanh toán phí cứu hộ bằng ví",
  "transactionType": 40
}
```

**Response 200 OK:**

```json
{
  "snakebiteIncidentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "transactionId": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
  "orderCode": null,
  "amount": 150000,
  "currency": "VND",
  "status": "Escrowed",
  "provider": "Wallet",
  "checkoutUrl": null,
  "paymentLinkId": null,
  "expiresAt": null,
  "externalTransactionId": null,
  "paidAt": "2025-07-15T09:00:00Z",
  "userWalletBalanceAfter": 350000,
  "systemWalletBalanceAfter": 1500000
}
```

### 4.3 Hủy PayOS Payment Link

```
DELETE /api/incidents/{incidentId}/payment/payos/{orderCode}
Authorization: Bearer <token>
```

**Request Body:**

```json
{
  "cancellationReason": "Đổi sang thanh toán bằng ví"
}
```

| Field | Type | Bắt buộc | Mô tả |
|---|---|---|---|
| `cancellationReason` | `string?` | Không | Lý do hủy |

**Response 200 OK:**

```json
{
  "success": true,
  "referenceId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "orderCode": 1234567890,
  "status": "Cancelled",
  "amount": 150000,
  "amountPaid": 0,
  "amountRemaining": 150000,
  "message": "Payment link cancelled successfully"
}
```

### 4.4 Hoàn tiền (Refund) — Chỉ Operator/Admin

```
POST /api/incidents/{incidentId}/payment/refund
Authorization: Bearer <token>  (role: Operator hoặc Admin)
```

**Request Body:**

```json
{
  "receiverId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "referenceId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "amount": 150000,
  "description": "Hoàn tiền do hủy ca cứu hộ",
  "transactionType": 41
}
```

| Field | Type | Bắt buộc | Mô tả |
|---|---|---|---|
| `receiverId` | `Guid` | Có | ID user nhận hoàn tiền |
| `referenceId` | `Guid` | Có | ID incident |
| `amount` | `decimal` | Có | Số tiền hoàn (≤ số tiền gốc) |
| `description` | `string` | Có | Mô tả lý do hoàn tiền |
| `transactionType` | `int` | Có | Luôn = `41` (`SnakebiteIncidentRefund`) |

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Refund processed successfully",
  "receiverId": "c3d4e5f6-a7b8-9012-cdef-123456789012",
  "refundAmount": 150000,
  "refundTransactionId": "d4e5f6a7-b890-1234-defg-234567890123",
  "systemWalletBalanceBefore": 1500000,
  "systemWalletBalanceAfter": 1350000,
  "receiverWalletBalanceBefore": 350000,
  "receiverWalletBalanceAfter": 500000,
  "refundedAt": "2025-07-15T11:00:00Z"
}
```

### 4.5 Manual Confirm Payment

Endpoint dùng chung cho tất cả payment flow. Routing dựa trên description prefix của transaction.

```
POST /api/v1/PayOs/confirm-payment
Authorization: Bearer <token>
```

**Request Body:**

```json
{
  "transactionId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Payment confirmed successfully",
  "data": {
    "success": true,
    "message": "Payment confirmed",
    "orderCode": 1234567890,
    "amount": 150000
  }
}
```

**Lưu ý**: Nếu transaction đã được xác nhận (qua webhook), endpoint trả về kết quả đã xử lý (idempotent). Nếu chưa xác nhận, backend sẽ gọi PayOS API để verify trạng thái trước khi confirm.

### 4.6 Return URL Callback

PayOS redirect user về URL này sau khi thanh toán. Backend tự động confirm nếu thành công.

```
GET /api/v1/PayOs/return?code=00&id={paymentLinkId}&cancel=false&status=PAID&orderCode={orderCode}
```

| Param | Mô tả |
|---|---|
| `code` | `00` = thành công |
| `id` | PayOS payment link ID |
| `cancel` | `true` nếu user hủy |
| `status` | `PAID`, `CANCELLED`, etc. |
| `orderCode` | Order code từ PayOS |

**Response**: HTML page (hiển thị kết quả cho user trong WebView).

Mobile không cần gọi endpoint này trực tiếp — PayOS tự redirect. Mobile chỉ cần detect URL pattern trong WebView để biết khi nào đóng WebView.

### 4.7 Webhook

PayOS gọi endpoint này khi thanh toán thành công. Mobile không cần quan tâm endpoint này.

```
POST /api/v1/PayOs/webhook
(No auth — PayOS gọi trực tiếp, verify bằng signature)
```

Backend xử lý:
1. Verify webhook signature
2. Kiểm tra description prefix `INCIDENT-` → route đến incident handler
3. Credit System_Wallet (escrow)
4. Update incident status → `Completed`
5. Idempotent: nếu đã xử lý → trả về kết quả cũ

## 5. Error Catalog

### 400 Bad Request — Validation Errors

| Trường hợp | Message mẫu |
|---|---|
| Amount không khớp mission cost | `Payment amount must equal completed mission amount ({expected})` |
| Webhook signature không hợp lệ | `Invalid webhook payload` |
| TransactionId rỗng (confirm) | `TransactionId is required` |
| Transaction không tìm thấy (confirm) | `Transaction not found` |
| Unknown payment flow (confirm) | `Unknown payment flow for the given transaction` |

### 403 Forbidden

| Trường hợp | Message mẫu |
|---|---|
| User không phải owner của incident | `You are not allowed to create payment link for this incident` |
| Role không đủ (refund) | HTTP 403 (ASP.NET tự trả) |

### 404 Not Found

| Trường hợp | Message mẫu |
|---|---|
| Incident không tồn tại | `Snakebite incident was not found` |
| Transaction không tồn tại (refund) | `Original payment transaction not found for this incident` |
| Wallet không tồn tại | `User wallet was not found` |

### 409 Conflict

| Trường hợp | Message mẫu |
|---|---|
| Incident chưa ở trạng thái `Finished` | `Payment is allowed only after the rescue mission is completed` |
| Incident đã thanh toán (`Completed`) | `This incident has already been paid` |
| Số dư ví không đủ (wallet payment) | `Insufficient wallet balance` |
| Số dư system wallet không đủ (refund) | `System wallet has insufficient balance for refund` |
| Refund amount > original amount | `Refund amount exceeds original payment amount` |

### 500 Internal Server Error

| Trường hợp | Message mẫu |
|---|---|
| PayOS gateway lỗi | `Failed to create payment link: {error}` |
| Lỗi hủy payment link | `Failed to cancel payment link: {error}` |

## 6. Webhook/Return Handling

### Routing qua Description Prefix

Backend sử dụng `PayOsDescriptionLookup` để route webhook/confirm/return đến đúng payment handler dựa trên **description prefix** trong transaction:

| Prefix | Handler | Flow |
|---|---|---|
| `INCIDENT-{orderCode}` | `ProcessSnakebiteIncidentWebhookAsync` | Flow 1 — Incident |
| `CONSULTPAY-{orderCode}` | `ProcessConsultationWebhookAsync` | Flow 3 — Consultation |
| `SNAKEAID-{orderCode}` | `ProcessSnakeCatchingWebhookAsync` | Flow 2 — Snake Catching |

### Luồng xử lý callback

```
PayOS → POST /api/v1/PayOs/webhook
    → VerifyWebhook(rawPayload) — kiểm tra signature
    → Đọc description từ webhook data
    → description.StartsWith("INCIDENT-") ?
        → Gọi ProcessSnakebiteIncidentWebhookAsync
        → Credit System_Wallet + tạo WalletTopup transaction
        → Update incident status → Completed
```

### Idempotency

Webhook và return URL có thể đến **đồng thời** cho cùng một order code. Backend đảm bảo idempotent:
- Kiểm tra `ExternalTransactionId` trên pending transaction
- Nếu đã có giá trị → trả về kết quả đã xử lý, không tạo duplicate
- Nếu chưa có → xử lý bình thường

## 7. Recommended Mobile Integration Pattern

### PayOS Payment Flow

```
1. Gọi POST /api/incidents/{id}/payment/payos
   → Nhận response với checkoutUrl, transactionId, orderCode

2. Mở checkoutUrl trong WebView (in-app browser)
   → User thanh toán trên trang PayOS

3. Detect return URL trong WebView
   → URL pattern: /api/v1/PayOs/return?code=00&status=PAID...
   → Đóng WebView

4. (Fallback) Gọi POST /api/v1/PayOs/confirm-payment
   → Gửi transactionId nhận được ở bước 1
   → Đảm bảo payment được ghi nhận dù webhook chậm

5. Refresh incident detail
   → Kiểm tra incident status = Completed
```

### Wallet Payment Flow

```
1. Gọi POST /api/incidents/{id}/payment/wallet
   → Nhận response với status = "Escrowed"
   → Payment hoàn tất ngay lập tức

2. Cập nhật UI:
   → Hiển thị số dư mới từ userWalletBalanceAfter
   → Incident status = Completed
```

### Xử lý lỗi trên mobile

```
- 400 → Hiển thị message lỗi cho user
- 403 → "Bạn không có quyền thực hiện thao tác này"
- 404 → "Không tìm thấy thông tin" → quay lại màn trước
- 409 → Hiển thị message cụ thể (đã thanh toán, không đủ số dư, ...)
- 500 → "Đã xảy ra lỗi, vui lòng thử lại sau"
```

### Tips

- Luôn gọi **confirm-payment** sau khi đóng WebView PayOS như fallback — webhook có thể bị delay
- Kiểm tra `status` trong response để xác định bước tiếp theo: `"Pending"` → chờ thanh toán, `"Escrowed"` → hoàn tất
- Nếu user tạo payment link mới khi đã có link cũ chưa thanh toán, backend tự động hủy link cũ và tạo mới
- `checkoutUrl` có thời hạn — nếu hết hạn, tạo link mới
