---
doc_role: baseline
module: wallet-withdrawal
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-04-02
api_version: v1
owners: [backend-team]
verification_status: code-verified
---

# Wallet Withdrawal API

## Overview

Flow withdrawal hiện tại trong code hỗ trợ:
- User tạo yêu cầu rút tiền từ ví
- User xem danh sách và chi tiết withdrawal của chính mình
- User hủy withdrawal khi còn `Pending`
- Admin xem toàn bộ withdrawal hoặc chỉ `Pending`
- Admin `approve`, `reject`, `complete`, `fail`
- QR VietQR được sinh khi admin `approve`
- Số dư ví chỉ bị trừ khi admin `complete`

---

## Authentication & Authorization

### User Operations
- JWT Bearer token bắt buộc
- Bất kỳ user đã đăng nhập đều gọi được user endpoints

### Admin Operations
- JWT Bearer token bắt buộc
- Bắt buộc role `Admin`

---

## Actual Status Lifecycle

Các trạng thái có trong code:
- `Pending`
- `Approved`
- `Rejected`
- `Completed`
- `Failed`

Các transition đã được code support:
- `Pending -> Approved`
- `Pending -> Rejected`
- `Pending -> Cancel` được lưu thành `Rejected` với reason `Cancelled by user`
- `Approved -> Rejected`
- `Approved -> Completed`
- `Approved -> Failed`

Lưu ý:
- `ProcessedAt` được set khi cancel / approve / reject / complete / fail

---

## Business Rules

### Create Withdrawal
- Validate request model:
  - `amount`: `10000` đến `50000000`
  - `bankAccount`: `8-20` chữ số
  - `bankName`: bắt buộc, tối đa `100` ký tự
  - `bankBin`: bắt buộc, đúng `6` chữ số
- Service kiểm tra số dư ví hiện tại phải đủ để tạo request
- Khi tạo request:
  - withdrawal được tạo với trạng thái `Pending`
  - chưa trừ tiền khỏi ví
  - chưa sinh QR

### Approve
- Chỉ approve được withdrawal đang `Pending`
- Khi approve:
  - sinh `VietQrPayload`
  - sinh `VietQrImageBase64`
  - set status `Approved`

### Complete
- Chỉ complete được withdrawal đang `Approved`
- Khi complete:
  - trừ tiền khỏi `Wallet.Balance`
  - tạo `Transaction` với `TransactionType = WalletWithdraw`
  - set status `Completed`

### Cancel
- User chỉ cancel được withdrawal của chính mình
- Chỉ cancel được withdrawal đang `Pending`
- Cancel được lưu thành:
  - `Status = Rejected`
  - `RejectionReason = "Cancelled by user"`

### Reject / Fail
- `Reject`: admin reject được trạng thái `Pending` hoặc `Approved`
- `Fail`: admin fail được trạng thái `Approved`

---

## User Workflow

### 1. Lấy ví hiện tại

Endpoint thực tế để lấy ví là:
- `GET /api/wallet/me`

Lưu ý:
- Codebase hiện tại không có `GET /api/wallet/balance`

### 2. Lấy danh sách ngân hàng

Endpoint:
- `GET /api/wallet/banks`

Lưu ý:
- Controller trả về `ApiResponse` wrapper, không phải raw array
- Dữ liệu ngân hàng hiện tại là mock data cache trong service

### 3. Tạo yêu cầu rút tiền

Endpoint:
- `POST /api/withdrawals/create`

Request body:
```json
{
  "amount": 50000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "bankBin": "970400"
}
```

Response:
- trả về object withdrawal
- `bankAccount` bị mask ở user endpoints
- status ban đầu là `Pending`

### 4. Xem withdrawal của chính mình

Endpoints:
- `GET /api/withdrawals/me`
- `GET /api/withdrawals/{id}`

Authorization rule:
- User chỉ xem được withdrawal có `UserId` là của chính mình

### 5. Hủy withdrawal

Endpoint:
- `POST /api/withdrawals/{id}/cancel`

Điều kiện:
- chỉ owner mới hủy được
- chỉ hủy được khi đang `Pending`

---

## API Contracts

Phần này giữ lại contract để frontend/mobile có thể tích hợp API.

### Common Authentication Header

```http
Authorization: Bearer {token}
```

### Wrapper Response Convention

Một số endpoint trong module wallet dùng `ApiResponse<T>` wrapper:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {},
  "error": null
}
```

Lưu ý:
- `GET /api/wallet/me` dùng wrapper
- `GET /api/wallet/banks` dùng wrapper
- nhóm `/api/withdrawals/*` và `/api/admin/withdrawals/*` hiện trả object / array trực tiếp, không bọc `ApiResponse<T>`

### 1. `GET /api/wallet/me`

Mục đích:
- Lấy ví hiện tại của user để đọc số dư trước khi tạo withdrawal

Authentication:
- Required

Success response:
```json
{
  "status_code": 200,
  "message": "Wallet retrieved successfully",
  "is_success": true,
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa7",
    "balance": 150000,
    "createdAt": "2026-04-02T12:00:00Z",
    "updatedAt": "2026-04-02T12:30:00Z"
  },
  "error": null
}
```

### 2. `GET /api/wallet/banks`

Mục đích:
- Lấy danh sách ngân hàng hỗ trợ hiển thị cho user

Authentication:
- Required

Success response:
```json
{
  "status_code": 200,
  "message": "Bank directory retrieved successfully",
  "is_success": true,
  "data": [
    {
      "bin": "970400",
      "name": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
      "vietQrStatus": "TransferSupported"
    },
    {
      "bin": "970405",
      "name": "Ngân hàng TMCP Quốc Tế (VIB)",
      "vietQrStatus": "TransferSupported"
    }
  ],
  "error": null
}
```

Ghi chú:
- Dữ liệu hiện tại là mock data từ `BankDirectoryService`

### 3. `POST /api/withdrawals/create`

Mục đích:
- Tạo yêu cầu rút tiền

Authentication:
- Required

Request body:
```json
{
  "amount": 50000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "bankBin": "970400"
}
```

Request constraints:
- `amount`: `10000` đến `50000000`
- `bankAccount`: `8-20` chữ số
- `bankName`: tối đa `100` ký tự
- `bankBin`: đúng `6` chữ số

Success response:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 50000,
  "bankAccount": "******7890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Pending",
  "processedAt": null,
  "rejectionReason": null,
  "vietQrPayload": null,
  "vietQrImageBase64": null,
  "createdAt": "2026-04-02T12:30:00Z"
}
```

Lưu ý:
- User response có mask `bankAccount`
- QR fields sẽ là `null` khi mới tạo

### 4. `GET /api/withdrawals/me`

Mục đích:
- Lấy toàn bộ withdrawal của user hiện tại

Authentication:
- Required

Success response:
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "amount": 50000,
    "bankAccount": "******7890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "status": "Pending",
    "processedAt": null,
    "rejectionReason": null,
    "vietQrPayload": null,
    "vietQrImageBase64": null,
    "createdAt": "2026-04-02T12:30:00Z"
  }
]
```

### 5. `GET /api/withdrawals/{id}`

Mục đích:
- Lấy chi tiết một withdrawal cụ thể của chính user

Authentication:
- Required

Authorization:
- Chỉ owner mới xem được

Success response sau khi approve:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 50000,
  "bankAccount": "******7890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Approved",
  "processedAt": "2026-04-02T12:45:00Z",
  "rejectionReason": null,
  "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN1234567890540650000.005802VN6304XXXX",
  "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "createdAt": "2026-04-02T12:30:00Z"
}
```

### 6. `POST /api/withdrawals/{id}/cancel`

Mục đích:
- User hủy withdrawal của chính mình

Authentication:
- Required

Success response:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 50000,
  "bankAccount": "******7890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Rejected",
  "processedAt": "2026-04-02T12:35:00Z",
  "rejectionReason": "Cancelled by user",
  "vietQrPayload": null,
  "vietQrImageBase64": null,
  "createdAt": "2026-04-02T12:30:00Z"
}
```

### 7. `GET /api/admin/withdrawals`

Mục đích:
- Admin lấy toàn bộ withdrawal

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "amount": 50000,
    "bankAccount": "1234567890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "status": "Pending",
    "processedAt": null,
    "rejectionReason": null,
    "vietQrPayload": null,
    "vietQrImageBase64": null,
    "createdAt": "2026-04-02T12:30:00Z"
  }
]
```

Lưu ý:
- Admin response hiện không mask `bankAccount`

### 8. `GET /api/admin/withdrawals/pending`

Mục đích:
- Admin lấy danh sách withdrawal đang `Pending`

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
- cùng shape với `GET /api/admin/withdrawals`

### 9. `POST /api/admin/withdrawals/{id}/approve`

Mục đích:
- Approve withdrawal và generate QR

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 50000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Approved",
  "processedAt": "2026-04-02T12:45:00Z",
  "rejectionReason": null,
  "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN1234567890540650000.005802VN6304XXXX",
  "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "createdAt": "2026-04-02T12:30:00Z"
}
```

### 10. `POST /api/admin/withdrawals/{id}/reject`

Request body:
```json
{
  "reason": "Invalid bank account"
}
```

Success response:
- cùng shape `WithdrawalResponse`
- `status = "Rejected"`
- `rejectionReason` chứa giá trị từ request

### 11. `POST /api/admin/withdrawals/{id}/complete`

Mục đích:
- Đánh dấu hoàn tất withdrawal
- Đây là bước trừ tiền khỏi ví

Success response:
- cùng shape `WithdrawalResponse`
- `status = "Completed"`

### 12. `POST /api/admin/withdrawals/{id}/fail`

Request body:
```json
{
  "reason": "Bank transfer failed"
}
```

Success response:
- cùng shape `WithdrawalResponse`
- `status = "Failed"`
- `rejectionReason` chứa giá trị từ request

---

## Admin Workflow

### 1. Xem danh sách withdrawal

Endpoints:
- `GET /api/admin/withdrawals`
- `GET /api/admin/withdrawals/pending`

### 2. Duyệt / từ chối / hoàn tất / fail

Endpoints:
- `POST /api/admin/withdrawals/{id}/approve`
- `POST /api/admin/withdrawals/{id}/reject`
- `POST /api/admin/withdrawals/{id}/complete`
- `POST /api/admin/withdrawals/{id}/fail`

Request body cho reject:
```json
{
  "reason": "Invalid bank account"
}
```

Request body cho fail:
```json
{
  "reason": "Bank transfer failed"
}
```

Lưu ý:
- Codebase hiện tại không có endpoint `GET /api/admin/withdrawals/{id}`

---

## Data Model Snapshot

### CreateWithdrawalRequest
| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| amount | decimal | Yes | 10000-50000000 |
| bankAccount | string | Yes | 8-20 digits |
| bankName | string | Yes | max 100 chars |
| bankBin | string | Yes | 6 digits |

### WithdrawalResponse
| Field | Type | Description |
|-------|------|-------------|
| id | Guid | Withdrawal ID |
| userId | Guid | Owner user ID |
| amount | decimal | Withdrawal amount |
| bankAccount | string | User endpoints mask số tài khoản |
| bankName | string | Bank name |
| status | enum | `Pending/Approved/Rejected/Completed/Failed` |
| processedAt | datetime? | Thời điểm xử lý |
| rejectionReason | string? | Lý do reject/fail/cancel |
| vietQrPayload | string? | QR payload sau khi approve |
| vietQrImageBase64 | string? | Base64 QR image sau khi approve |
| createdAt | datetime | Thời điểm tạo |

### BankDirectoryResponse
| Field | Type | Description |
|-------|------|-------------|
| bin | string | Bank BIN |
| name | string | Bank name |
| vietQrStatus | enum | `TransferSupported/ReceiveOnly/NotSupported` |

### WalletResponse
| Field | Type | Description |
|-------|------|-------------|
| id | Guid | Wallet ID |
| userId | Guid | Owner user ID |
| balance | decimal | Current balance |
| createdAt | datetime | Created timestamp |
| updatedAt | datetime? | Updated timestamp |

### ApiResponse<T>
| Field | Type | Description |
|-------|------|-------------|
| status_code | int | HTTP-like status code in body |
| message | string | Human-readable message |
| is_success | bool | Success flag |
| data | T | Payload |
| error | object? | Error detail when failed |

---

## Important Implementation Notes

### 1. `bankBin` hiện chưa đi hết flow

Mặc dù request bắt buộc có `bankBin`, implementation hiện tại:
- không lưu `bankBin` vào entity `WalletWithdraw`
- khi approve lại hardcode bank BIN là `970400`

Vì vậy tài liệu này không khẳng định QR luôn được sinh theo đúng `bankBin` mà client gửi lên.

### 2. User response và admin response khác nhau về masking

- User endpoints mask `bankAccount`
- Admin endpoints hiện trả raw `bankAccount`

### 3. Số dư chỉ bị trừ ở bước `complete`

Điểm này rất quan trọng:
- `create`: chưa trừ tiền
- `approve`: chưa trừ tiền
- `complete`: mới trừ tiền và tạo transaction

---

## Verified Endpoint List

### User
- `GET /api/wallet/me`
- `GET /api/wallet/banks`
- `POST /api/withdrawals/create`
- `GET /api/withdrawals/me`
- `GET /api/withdrawals/{id}`
- `POST /api/withdrawals/{id}/cancel`

### Admin
- `GET /api/admin/withdrawals`
- `GET /api/admin/withdrawals/pending`
- `POST /api/admin/withdrawals/{id}/approve`
- `POST /api/admin/withdrawals/{id}/reject`
- `POST /api/admin/withdrawals/{id}/complete`
- `POST /api/admin/withdrawals/{id}/fail`

---

## Changelog

### 2026-04-02
- Removed all undocumented / unverified business claims
- Corrected wallet balance endpoint to `GET /api/wallet/me`
- Corrected bank directory response behavior to `ApiResponse` wrapper
- Documented actual status transitions from code
- Documented actual deduction timing at `complete`
- Documented current `bankBin` implementation gap
