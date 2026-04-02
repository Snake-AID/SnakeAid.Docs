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