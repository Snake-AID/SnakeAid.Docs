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

## 1. Table Of Contents

- [1. Table Of Contents](#1-table-of-contents)
- [2. Overview](#2-overview)
- [3. Authentication & Authorization](#3-authentication--authorization)
- [4. Expert/Member Business + Expert/Member APIs](#4-expertmember-business--expertmember-apis)
- [5. Admin Business + Admin APIs](#5-admin-business--admin-apis)
- [6. Shared Data Models](#6-shared-data-models)
- [7. Verified Endpoint List](#7-verified-endpoint-list)
- [8. Changelog](#8-changelog)

## 2. Overview

Flow withdrawal hiện tại trong code hỗ trợ:
- User tạo yêu cầu rút tiền từ ví
- User xem danh sách và chi tiết withdrawal của chính mình
- User hủy withdrawal khi còn `Pending`
- Admin xem toàn bộ withdrawal hoặc chỉ `Pending`
- Admin `approve`, `reject`, `complete`, `fail`
- QR VietQR được sinh khi admin `approve`
- Số dư ví chỉ bị trừ khi admin `complete`

## 3. Authentication & Authorization

### Expert/Member Operations
- JWT Bearer token bắt buộc
- `Expert` và `Member` dùng cùng nhóm user-facing APIs cho flow này

### Admin Operations
- JWT Bearer token bắt buộc
- Bắt buộc role `Admin`

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Business Scope

- Xem ví hiện tại để đọc số dư
- Lấy danh sách ngân hàng hỗ trợ
- Tạo yêu cầu rút tiền
- Xem danh sách withdrawal của chính mình
- Xem chi tiết từng withdrawal của chính mình
- Hủy withdrawal khi còn `Pending`

### 4.2 Actual Status Lifecycle

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

### 4.3 Business Rules

#### Create Withdrawal
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

#### Cancel
- User chỉ cancel được withdrawal của chính mình
- Chỉ cancel được withdrawal đang `Pending`
- Cancel được lưu thành:
  - `Status = Rejected`
  - `RejectionReason = "Cancelled by user"`

### 4.4 Common Authentication Header

```http
Authorization: Bearer {token}
```

### 4.5 Wrapper Response Convention

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
- nhóm `/api/withdrawals/*` và `/api/admin/withdrawals/*` dùng `ApiResponse<T>` wrapper

### 4.6 Expert/Member APIs

#### 4.6.1 `GET /api/wallet/me`

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

#### 4.6.2 `GET /api/wallet/banks`

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

#### 4.6.3 `POST /api/withdrawals/create`

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
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "amount": 50000,
    "bankAccount": "******7890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "bankBin": "970400",
    "status": "Pending",
    "processedAt": null,
    "rejectionReason": null,
    "vietQrPayload": null,
    "vietQrImageBase64": null,
    "createdAt": "2026-04-02T12:30:00Z"
  },
  "error": null
}
```

Lưu ý:
- User response có mask `bankAccount`
- QR fields sẽ là `null` khi mới tạo

#### 4.6.4 `GET /api/withdrawals/me`

Mục đích:
- Lấy toàn bộ withdrawal của user hiện tại

Authentication:
- Required

Success response:
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "userId": "550e8400-e29b-41d4-a716-446655440001",
      "amount": 50000,
      "bankAccount": "******7890",
      "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
      "bankBin": "970400",
      "status": "Pending",
      "processedAt": null,
      "rejectionReason": null,
      "vietQrPayload": null,
      "vietQrImageBase64": null,
      "createdAt": "2026-04-02T12:30:00Z"
    }
  ],
  "error": null
}
```

#### 4.6.5 `GET /api/withdrawals/{id}`

Mục đích:
- Lấy chi tiết một withdrawal cụ thể của chính user

Authentication:
- Required

Authorization:
- Chỉ owner mới xem được

Success response sau khi approve:
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "amount": 50000,
    "bankAccount": "******7890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "bankBin": "970400",
    "status": "Approved",
    "processedAt": "2026-04-02T12:45:00Z",
    "rejectionReason": null,
    "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN1234567890540650000.005802VN6304XXXX",
    "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
    "createdAt": "2026-04-02T12:30:00Z"
  },
  "error": null
}
```

#### 4.6.6 `POST /api/withdrawals/{id}/cancel`

Mục đích:
- User hủy withdrawal của chính mình

Authentication:
- Required

Success response:
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "amount": 50000,
    "bankAccount": "******7890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "bankBin": "970400",
    "status": "Rejected",
    "processedAt": "2026-04-02T12:35:00Z",
    "rejectionReason": "Cancelled by user",
    "vietQrPayload": null,
    "vietQrImageBase64": null,
    "createdAt": "2026-04-02T12:30:00Z"
  },
  "error": null
}
```

## 5. Admin Business + Admin APIs

### 5.1 Business Scope

- Xem toàn bộ withdrawal
- Lấy riêng các withdrawal đang `Pending`
- Approve request để sinh QR
- Reject request
- Complete request để trừ số dư ví và tạo transaction
- Fail request khi xử lý không thành công

### 5.2 Business Rules

#### Approve
- Chỉ approve được withdrawal đang `Pending`
- Khi approve:
  - sinh `VietQrPayload`
  - sinh `VietQrImageBase64`
  - set status `Approved`

#### Complete
- Chỉ complete được withdrawal đang `Approved`
- Khi complete:
  - trừ tiền khỏi `Wallet.Balance`
  - tạo `Transaction` với `TransactionType = WalletWithdraw`
  - set status `Completed`

#### Reject / Fail
- `Reject`: admin reject được trạng thái `Pending` hoặc `Approved`
- `Fail`: admin fail được trạng thái `Approved`

### 5.3 Common Authentication Header

```http
Authorization: Bearer {token}
```

### 5.4 Admin APIs

#### 5.4.1 `GET /api/admin/withdrawals`

Mục đích:
- Admin lấy toàn bộ withdrawal

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "userId": "550e8400-e29b-41d4-a716-446655440001",
      "amount": 50000,
      "bankAccount": "1234567890",
      "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
      "bankBin": "970400",
      "status": "Pending",
      "processedAt": null,
      "rejectionReason": null,
      "vietQrPayload": null,
      "vietQrImageBase64": null,
      "createdAt": "2026-04-02T12:30:00Z"
    }
  ],
  "error": null
}
```

Lưu ý:
- Admin response hiện không mask `bankAccount`

#### 5.4.2 `GET /api/admin/withdrawals/pending`

Mục đích:
- Admin lấy danh sách withdrawal đang `Pending`

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
- cùng shape với `GET /api/admin/withdrawals`

#### 5.4.3 `GET /api/admin/withdrawals/{id}`

Mục đích:
- Admin lấy chi tiết một withdrawal cụ thể

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
- cùng shape `ApiResponse<WithdrawalResponse>`

#### 5.4.4 `POST /api/admin/withdrawals/{id}/approve`

Mục đích:
- Approve withdrawal và generate QR

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "amount": 50000,
    "bankAccount": "1234567890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "bankBin": "970400",
    "status": "Approved",
    "processedAt": "2026-04-02T12:45:00Z",
    "rejectionReason": null,
    "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN1234567890540650000.005802VN6304XXXX",
    "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
    "createdAt": "2026-04-02T12:30:00Z"
  },
  "error": null
}
```

#### 5.4.5 `POST /api/admin/withdrawals/{id}/reject`

Authentication:
- Required

Authorization:
- Role `Admin`

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

#### 5.4.6 `POST /api/admin/withdrawals/{id}/complete`

Mục đích:
- Đánh dấu hoàn tất withdrawal
- Đây là bước trừ tiền khỏi ví

Authentication:
- Required

Authorization:
- Role `Admin`

Success response:
- cùng shape `WithdrawalResponse`
- `status = "Completed"`

#### 5.4.7 `POST /api/admin/withdrawals/{id}/fail`

Authentication:
- Required

Authorization:
- Role `Admin`

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

## 6. Shared Data Models

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
| bankBin | string? | Bank BIN được lưu cùng withdrawal |
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
| error | object? | Error detail khi thất bại |

## 7. Verified Endpoint List

### Expert/Member
- `GET /api/wallet/me`
- `GET /api/wallet/banks`
- `POST /api/withdrawals/create`
- `GET /api/withdrawals/me`
- `GET /api/withdrawals/{id}`
- `POST /api/withdrawals/{id}/cancel`

### Admin
- `GET /api/admin/withdrawals`
- `GET /api/admin/withdrawals/pending`
- `GET /api/admin/withdrawals/{id}`
- `POST /api/admin/withdrawals/{id}/approve`
- `POST /api/admin/withdrawals/{id}/reject`
- `POST /api/admin/withdrawals/{id}/complete`
- `POST /api/admin/withdrawals/{id}/fail`

## 8. Changelog

### 2026-04-03
- Standardized all withdrawal endpoints to `ApiResponse<T>`
- Added `bankBin` to withdrawal response contract
- Added `GET /api/admin/withdrawals/{id}`
- Kept cancel contract as `POST /api/withdrawals/{id}/cancel`

### 2026-04-02
- Restructured document into actor-based sections
- Merged `Expert` with `Member` because codebase uses the same user-facing flow
- Added TOC as section 1
- Kept only code-verified behavior and API contracts
- Preserved frontend/mobile-usable request and response contracts
