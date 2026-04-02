---
doc_role: baseline
module: wallet-withdrawal
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-04-02
api_version: v1
owners: [backend-team]
---

# Wallet Withdrawal API

## Overview

The Wallet Withdrawal API enables users to withdraw funds from their SnakeAid wallet to Vietnamese bank accounts. The system supports QR code generation for VietQR-compatible transfers and provides a complete workflow from request creation through admin approval to completion.

### Key Capabilities
- Create withdrawal requests with bank account details
- Generate VietQR codes for easy mobile banking transfers
- Admin approval and processing workflow
- Secure bank account masking and validation
- Real-time withdrawal status tracking

### Prerequisites
- Valid JWT authentication token
- Sufficient wallet balance
- Vietnamese bank account (8-20 digits)
- Admin role required for approval operations

---

## Business Context

### When to Use Wallet Withdrawals

Users withdraw funds from their SnakeAid wallet when they want to:
- **Convert earned rewards to cash**: After accumulating points from platform activities (purchases, referrals, promotions)
- **Access liquidity**: Need immediate cash access for personal expenses or reinvestment
- **Bank integration**: Move funds to traditional banking for higher interest rates or financial planning
- **Emergency funds**: Quick access to earned money during financial needs

### Business Value

The withdrawal system enables:
- **User retention**: Allows users to realize value from earned rewards
- **Trust building**: Transparent, secure fund management builds platform credibility
- **Revenue optimization**: Balances reward distribution with platform sustainability
- **Regulatory compliance**: Proper KYC and AML checks for financial operations

### User Journey Overview

```
Earn Rewards → Accumulate in Wallet → Request Withdrawal → Admin Review → Approval → QR Generation → Bank Transfer → Completion
```

**Typical Timeline**: 5-15 minutes for approval, 1-2 business days for bank settlement.

---

## Decision Trees

### Admin Approval Decision Tree

When reviewing a withdrawal request, admins should consider:

```
Is withdrawal amount ≤ wallet balance?
├── YES → Is bank account format valid?
│   ├── YES → Is account verified/recently used?
│   │   ├── YES → APPROVE (generate QR)
│   │   └── NO → Check user history
│   │       ├── Good history → APPROVE
│   │       └── Suspicious activity → REJECT (risk assessment)
│   └── NO → REJECT (invalid format)
└── NO → REJECT (insufficient balance)
```

### Status Transition Logic

```
Pending → Approved (admin approval, QR generated)
    ↓
Pending → Rejected (admin rejection, user cancellation)
    ↓
Approved → Completed (successful bank transfer)
Approved → Failed (transfer failure, system error)
```

### Rejection Scenarios

**Automatic Rejection:**
- Insufficient wallet balance
- Invalid bank account format
- Daily withdrawal limit exceeded (5 requests/hour)

**Manual Rejection:**
- Suspicious account activity
- Unverified bank account
- Regulatory compliance issues
- User request cancellation

---

## Business Rules

### Withdrawal Policies

**Amount Limits:**
- Minimum: 10,000 VND per withdrawal
- Maximum: 50,000,000 VND per withdrawal
- Daily user limit: No specific daily limit (rate limited at 5 requests/hour)

**Frequency Limits:**
- User withdrawal requests: 5 per hour
- Admin processing: 100 operations per hour

**Bank Account Requirements:**
- Must be Vietnamese bank account (8-20 digits)
- Must be from supported banks (see Bank Directory)
- Account must be in user's name (KYC verification may be required)

### Approval Workflow

**Automatic Processing:**
- Balance verification
- Account format validation
- Rate limit checking

**Manual Admin Review:**
- Suspicious activity detection
- Large amount verification (>5,000,000 VND)
- New user verification (first withdrawal)
- Account change verification

**Approval Timeline:**
- Standard requests: Within 15 minutes during business hours
- Large amounts: May require additional verification (up to 24 hours)
- Weekend requests: Processed next business day

### Security Policies

**Data Protection:**
- Bank accounts encrypted at rest
- Responses show masked account numbers (****XXXX)
- Admin access logged and audited

**Fraud Prevention:**
- Rate limiting prevents abuse
- Suspicious pattern detection
- Manual review for high-risk requests

**Compliance:**
- Vietnamese banking regulations
- Anti-money laundering checks
- Transaction reporting requirements

### Completion Requirements

**Successful Transfer:**
- QR code scanned by banking app
- Funds transferred within 2 business days
- Confirmation received from bank

**Failure Handling:**
- Automatic retry for network issues
- Manual intervention for bank errors
- User notification for all status changes

---

## User Experience Workflows

### Complete User Withdrawal Flow

**Bước 1: Kiểm tra số dư ví**
- Gọi `GET /api/wallet/balance` để lấy số dư hiện tại
- Validate: Số dư phải ≥ 10,000 VND và ≥ số tiền muốn rút
- Nếu không đủ: Hiển thị thông báo lỗi và dừng flow

**Bước 2: Chọn ngân hàng và nhập thông tin tài khoản**
- Gọi `GET /api/wallet/banks` để lấy danh sách ngân hàng hỗ trợ
- User chọn ngân hàng từ dropdown và nhập số tài khoản (8-20 chữ số)
- Validate format: Số tài khoản Việt Nam hợp lệ

**Bước 3: Tạo yêu cầu rút tiền**
- Gọi `POST /api/withdrawals/create` với thông tin:
  - amount: Số tiền rút (10,000 - 50,000,000 VND)
  - bankAccount: Số tài khoản (8-20 chữ số)
  - bankName: Tên ngân hàng đầy đủ
  - bankBin: Mã BIN 6 chữ số
- Response trả về withdrawal ID và status "Pending"

**Bước 4: Theo dõi trạng thái yêu cầu**
- Gọi `GET /api/withdrawals/me` để lấy danh sách rút tiền
- Hoặc `GET /api/withdrawals/{id}` để lấy chi tiết cụ thể
- Poll mỗi 30 giây cho đến khi status thay đổi
- Khi status = "Approved": Hiển thị QR code để user scan

**Bước 5: Hoàn tất chuyển khoản ngân hàng**
- User mở app ngân hàng và scan QR VietQR
- Ngân hàng xử lý chuyển khoản (1-2 ngày làm việc)
- Hệ thống nhận confirmation và cập nhật status thành "Completed"

### Admin Processing Workflow

**Bước 1: Xem xét yêu cầu đang chờ xử lý**
- Gọi `GET /api/admin/withdrawals/pending` để lấy danh sách rút tiền chờ duyệt
- Review từng request: kiểm tra số dư, format tài khoản, lịch sử user
- Ưu tiên xử lý theo thời gian tạo request

**Bước 2: Duyệt hoặc từ chối yêu cầu**
- **Duyệt**: Gọi `POST /api/admin/withdrawals/{id}/approve`
  - Hệ thống tự động tạo QR VietQR
  - Status chuyển thành "Approved"
- **Từ chối**: Gọi `POST /api/admin/withdrawals/{id}/reject` với lý do
  - Status chuyển thành "Rejected"

**Bước 3: Giám sát hoàn tất**
- Theo dõi status chuyển thành "Completed" sau khi user scan QR
- Xử lý các trường hợp thất bại: gọi `POST /api/admin/withdrawals/{id}/fail`
- Nhận thông báo cho các transfer thất bại để can thiệp thủ công

---

## Integration Patterns

### Mobile App Flow (Flutter)

**1. Khởi tạo rút tiền:**
- Load danh sách ngân hàng: `GET /api/wallet/banks`
- Hiển thị dropdown chọn ngân hàng
- Nhập số tài khoản và số tiền

**2. Xử lý tạo yêu cầu:**
- Validate input trước khi gọi API
- Call `POST /api/withdrawals/create`
- Xử lý response: Thành công → chuyển màn hình tracking, Thất bại → hiển thị lỗi

**3. Theo dõi trạng thái:**
- Tự động poll `GET /api/withdrawals/{id}` mỗi 30 giây
- Khi status = "Approved" → hiển thị QR code
- Khi status = "Completed" → cập nhật UI và thông báo user

**4. Xử lý lỗi:**
- Network error: Retry với exponential backoff
- Rate limit: Hiển thị countdown timer
- Business error: Validate và hiển thị message cụ thể

### Webhook Integration

**Status Update Webhook:**
```json
{
  "event": "withdrawal.status_changed",
  "withdrawalId": "550e8400-e29b-41d4-a716-446655440000",
  "oldStatus": "Pending",
  "newStatus": "Approved",
  "timestamp": "2026-04-02T12:45:00Z",
  "qrCode": {
    "payload": "00020101021138550010A0000007270125000697045601139704005802VN5412345678905802VN6304XXXX",
    "imageBase64": "iVBORw0KGgoAAAANSUhEUgAA..."
  }
}
```

**Business Logic cho Webhook:**
- Nhận event "withdrawal.status_changed"
- Parse withdrawalId và newStatus
- Nếu status = "Approved": Gửi QR code đến user qua push notification/email
- Nếu status = "Completed": Cập nhật balance cache, gửi confirmation
- Nếu status = "Failed": Thông báo user và cung cấp support contact

### Best Practices

**Quy trình nghiệp vụ:**
- Luôn kiểm tra số dư trước khi cho phép user tạo yêu cầu rút tiền
- Validate format tài khoản ngân hàng trước khi submit
- Hiển thị progress indicator trong suốt flow rút tiền
- Thông báo user ngay khi status thay đổi

**Xử lý lỗi:**
- Rate limit exceeded: Hiển thị thời gian chờ và retry tự động
- Network error: Retry 3 lần với backoff, sau đó hiển thị error message
- Business error: Parse error code và hiển thị message phù hợp cho user

**Bảo mật:**
- Không lưu trữ đầy đủ số tài khoản trong device
- Sử dụng HTTPS cho tất cả API calls
- Validate input ở cả client và server side

**Trải nghiệm người dùng:**
- Cache danh sách ngân hàng để load nhanh
- Poll status mỗi 30 giây, không quá thường để tránh rate limit
- Hiển thị QR code ngay khi được duyệt, hướng dẫn user cách scan

---

---

## Authentication & Authorization

### User Operations
- **Method**: JWT Bearer Token
- **Header**: `Authorization: Bearer {token}`
- **Required Roles**: Any authenticated user

### Admin Operations
- **Method**: JWT Bearer Token
- **Header**: `Authorization: Bearer {token}`
- **Required Roles**: Admin

---

## Endpoints

### User Endpoints

#### `POST /api/withdrawals/create`

**Description**: Creates a new withdrawal request from the user's wallet to a bank account.

**Authentication**: Required

**Required Permissions**: Authenticated user

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |
| Content-Type | string | Yes | application/json |

**Request Body**:
```json
{
  "amount": 50000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "bankBin": "970400"
}
```

**Request Body Schema**:
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| amount | decimal | Yes | 10000-50000000 | Withdrawal amount in VND |
| bankAccount | string | Yes | 8-20 digits | Vietnamese bank account number |
| bankName | string | Yes | max 100 chars | Full bank name |
| bankBin | string | Yes | 6 digits | Bank BIN code |

**Success Response** (200 OK):
```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "amount": 50000,
  "bankAccount": "****67890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Pending",
  "createdAt": "2026-04-02T12:30:00Z"
}
```

**Error Responses**: See Error Catalog section.

#### `GET /api/withdrawals/me`

**Description**: Retrieves all withdrawal requests for the authenticated user.

**Authentication**: Required

**Required Permissions**: Authenticated user

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Query Parameters**: None

**Success Response** (200 OK):
```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "amount": 50000,
    "bankAccount": "****67890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "status": "Pending",
    "createdAt": "2026-04-02T12:30:00Z"
  }
]
```

#### `GET /api/withdrawals/{id}`

**Description**: Retrieves details of a specific withdrawal request.

**Authentication**: Required

**Required Permissions**: Owner of the withdrawal

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Path Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Withdrawal request UUID |

**Success Response** (200 OK):
```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "amount": 50000,
  "bankAccount": "****67890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Approved",
  "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN5412345678905802VN6304XXXX",
  "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "createdAt": "2026-04-02T12:30:00Z",
  "processedAt": "2026-04-02T12:45:00Z"
}
```

#### `POST /api/withdrawals/{id}/cancel`

**Description**: Cancels a pending withdrawal request.

**Authentication**: Required

**Required Permissions**: Owner of the withdrawal

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Path Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Withdrawal request UUID |

**Success Response** (200 OK):
```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "amount": 50000,
  "bankAccount": "****67890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Rejected",
  "rejectionReason": "Cancelled by user",
  "createdAt": "2026-04-02T12:30:00Z",
  "processedAt": "2026-04-02T12:35:00Z"
}
```

### Bank Directory Endpoint

#### `GET /api/wallet/banks`

**Description**: Retrieves list of supported banks for withdrawal operations.

**Authentication**: Required

**Required Permissions**: Authenticated user

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Success Response** (200 OK):
```json
[
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
]
```

### Admin Endpoints

#### `GET /api/admin/withdrawals`

**Description**: Retrieves all withdrawal requests for admin management.

**Authentication**: Required

**Required Permissions**: Admin role

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Success Response** (200 OK):
```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "amount": 50000,
    "bankAccount": "1234567890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "status": "Pending",
    "createdAt": "2026-04-02T12:30:00Z"
  }
]
```

#### `GET /api/admin/withdrawals/pending`

**Description**: Retrieves only pending withdrawal requests for admin processing.

**Authentication**: Required

**Required Permissions**: Admin role

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

#### `POST /api/admin/withdrawals/{id}/approve`

**Description**: Approves a pending withdrawal request and generates QR code.

**Authentication**: Required

**Required Permissions**: Admin role

**Path Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | Withdrawal request UUID |

#### `POST /api/admin/withdrawals/{id}/reject`

**Description**: Rejects a withdrawal request with reason.

**Authentication**: Required

**Required Permissions**: Admin role

**Request Body**:
```json
{
  "reason": "Invalid bank account"
}
```

#### `POST /api/admin/withdrawals/{id}/complete`

**Description**: Marks an approved withdrawal as completed and deducts wallet balance.

**Authentication**: Required

**Required Permissions**: Admin role

#### `POST /api/admin/withdrawals/{id}/fail`

**Description**: Marks a withdrawal as failed with reason.

**Authentication**: Required

**Required Permissions**: Admin role

**Request Body**:
```json
{
  "reason": "Bank transfer failed"
}
```

---

## Data Models

### CreateWithdrawalRequest
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| amount | decimal | Yes | 10000-50000000 | Withdrawal amount in VND |
| bankAccount | string | Yes | 8-20 digits | Vietnamese bank account number |
| bankName | string | Yes | max 100 chars | Full bank name |
| bankBin | string | Yes | 6 digits | Bank BIN code |

### WithdrawalResponse
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Yes | Withdrawal request UUID |
| userId | string | Yes | User UUID |
| amount | decimal | Yes | Withdrawal amount |
| bankAccount | string | Yes | Masked bank account (****XXXX) |
| bankName | string | Yes | Full bank name |
| status | string | Yes | Pending/Approved/Rejected/Completed/Failed |
| rejectionReason | string | No | Reason for rejection/failure |
| vietQrPayload | string | No | VietQR payload string |
| vietQrImageBase64 | string | No | Base64 encoded QR code image |
| createdAt | datetime | Yes | ISO 8601 format |
| processedAt | datetime | No | Processing timestamp |

### BankDirectoryResponse
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| bin | string | Yes | Bank BIN code |
| name | string | Yes | Full bank name |
| vietQrStatus | string | Yes | TransferSupported/ReceiveOnly/NotSupported |

### RejectWithdrawalRequest
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| reason | string | Yes | Rejection reason |

### FailWithdrawalRequest
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| reason | string | Yes | Failure reason |

---

## Error Catalog

| Status Code | Error Code | Message | Description | Resolution |
|-------------|------------|---------|-------------|------------|
| 400 | INVALID_INPUT | Invalid request format | Request validation failed | Check field constraints |
| 400 | INSUFFICIENT_BALANCE | Insufficient wallet balance | User doesn't have enough funds | Top up wallet or reduce amount |
| 400 | INVALID_BANK_ACCOUNT | Invalid bank account format | Account doesn't match Vietnamese format | Use 8-20 digit account number |
| 401 | UNAUTHORIZED | Authentication required | Missing or invalid JWT token | Provide valid Bearer token |
| 403 | FORBIDDEN | Insufficient permissions | User lacks required role | Contact admin for access |
| 403 | ACCESS_DENIED | Not authorized for this withdrawal | User doesn't own the withdrawal | Use correct withdrawal ID |
| 404 | NOT_FOUND | Withdrawal not found | Requested withdrawal doesn't exist | Verify withdrawal ID |
| 409 | INVALID_STATUS | Cannot perform action on withdrawal | Withdrawal is in wrong status | Check current status |
| 500 | INTERNAL_ERROR | Internal server error | Unexpected server error | Contact support |

---

## Examples

### Example: Create Withdrawal Request

**Request**:
```bash
curl -X POST https://api.snakeaid.com/api/withdrawals/create \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 100000,
    "bankAccount": "1234567890",
    "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
    "bankBin": "970400"
  }'
```

**Response**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 100000,
  "bankAccount": "****67890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Pending",
  "createdAt": "2026-04-02T12:30:00Z"
}
```

### Example: Admin Approve Withdrawal

**Request**:
```bash
curl -X POST https://api.snakeaid.com/api/admin/withdrawals/550e8400-e29b-41d4-a716-446655440000/approve \
  -H "Authorization: Bearer {{ADMIN_TOKEN}}"
```

**Response**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 100000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Approved",
  "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN5412345678905802VN6304XXXX",
  "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "createdAt": "2026-04-02T12:30:00Z",
  "processedAt": "2026-04-02T12:45:00Z"
}
```

### Example: Complete Withdrawal

**Request**:
```bash
curl -X POST https://api.snakeaid.com/api/admin/withdrawals/550e8400-e29b-41d4-a716-446655440000/complete \
  -H "Authorization: Bearer {{ADMIN_TOKEN}}"
```

**Response**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "550e8400-e29b-41d4-a716-446655440001",
  "amount": 100000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "status": "Completed",
  "vietQrPayload": "00020101021138550010A0000007270125000697045601139704005802VN5412345678905802VN6304XXXX",
  "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "createdAt": "2026-04-02T12:30:00Z",
  "processedAt": "2026-04-02T13:00:00Z"
}
```

---

## Rate Limiting

The withdrawal API implements rate limiting to prevent abuse:

- **User withdrawals**: Maximum 5 requests per hour
- **Admin operations**: Maximum 100 requests per hour

Rate limit headers are included in responses:
```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 4
X-RateLimit-Reset: 1640995200
```

When rate limit is exceeded, returns 429 Too Many Requests.

---

## Changelog

### v1.0.0 (2026-04-02)
- Initial release of wallet withdrawal API
- VietQR code generation
- Admin approval workflow
- Bank account validation and masking
- Complete withdrawal lifecycle management