---
doc_role: baseline
module: wallet-withdrawal-processing
kind: flow
doc_type: useguide
status: active
last_updated: 2026-04-21
api_version: v1
owners: [backend-team]
verification_status: code-verified
---

# Wallet Withdrawal Processing API

## Overview

This guide documents the current active API contract verified from code.

Current user-visible behavior:

- a withdrawal is created with status `Pending`
- user responses mask `bankAccount`
- admin responses return full `bankAccount`
- QR fields are generated on `Approved`
- wallet balance is not deducted at create time
- wallet balance is deducted at admin `Approve`
- admin `Complete` confirms the payout and does not deduct again
- user `Cancel` stores the withdrawal as `Rejected`
- admin `Reject` refunds only when the withdrawal had already reached `Approved`

## Authentication & Authorization

### Expert/Member Business + Expert/Member APIs

- all routes require JWT Bearer authentication
- `Expert` and `Member` use the same withdrawal routes

### Admin Business + Admin APIs

- all routes require JWT Bearer authentication
- admin routes require role `Admin`

## Expert/Member Business + Expert/Member APIs

### Business Scope

- read current wallet balance
- read the supported bank directory
- create a withdrawal request
- list personal withdrawals
- read personal withdrawal detail
- cancel a pending withdrawal

### Business Rules

#### Create Withdrawal

- request validation:
  - `amount`: `50000` to `5000000`
  - `bankAccount`: `8` to `20` digits
  - `bankName`: required, max `100`
  - `accountHolderName`: required, max `150`
  - `bankBin`: required, exactly `6` digits
- service validation:
  - wallet must exist
  - available balance is checked as `wallet balance - sum(pending withdrawals)`
  - same-day withdrawal total excluding `Rejected` and `Failed` must not exceed `10000000`
- success result:
  - status is `Pending`
  - QR fields are `null`
  - wallet balance is unchanged at this step

#### Cancel Withdrawal

- user can cancel only their own withdrawal
- only `Pending` can be cancelled
- successful cancel returns:
  - `status = Rejected`
  - `rejectionReason = "Cancelled by user"`

### Response Envelope

All routes use `ApiResponse<T>`.

Example success envelope:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {},
  "error": null
}
```

### User APIs

#### `GET /api/wallet/me`

Purpose:

- return the authenticated user's wallet

Success response:

- `ApiResponse<WalletResponse>`

Example:

```json
{
  "status_code": 200,
  "message": "Wallet retrieved successfully",
  "is_success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440010",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "balance": 2500000,
    "createdAt": "2026-03-25T08:00:00Z",
    "updatedAt": "2026-04-03T12:45:00Z"
  },
  "error": null
}
```

#### `GET /api/wallet/banks`

Purpose:

- return supported banks for withdrawal input

Success response:

- `ApiResponse<List<BankDirectoryResponse>>`

Example:

```json
{
  "status_code": 200,
  "message": "Bank directory retrieved successfully",
  "is_success": true,
  "data": [
    {
      "key": "vietcombank",
      "code": "VCB",
      "shortName": "Vietcombank",
      "bin": "970400",
      "name": "Joint Stock Commercial Bank for Foreign Trade of Vietnam",
      "vietQrStatus": "TransferSupported",
      "lookupSupported": true,
      "swiftCode": "BFTVVNVX"
    }
  ],
  "error": null
}
```

#### `POST /api/withdrawals/create`

Purpose:

- create a new withdrawal request

Request body:

```json
{
  "amount": 50000,
  "bankAccount": "1234567890",
  "bankName": "Vietcombank",
  "accountHolderName": "NGUYEN VAN A",
  "bankBin": "970400"
}
```

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
    "bankName": "Vietcombank",
    "accountHolderName": "NGUYEN VAN A",
    "bankBin": "970400",
    "status": "Pending",
    "processedAt": null,
    "rejectionReason": null,
    "vietQrPayload": null,
    "vietQrImageBase64": null,
    "createdAt": "2026-04-03T12:30:00Z"
  },
  "error": null
}
```

#### `GET /api/withdrawals/me`

Purpose:

- list the authenticated user's withdrawals

Success response:

- `ApiResponse<IEnumerable<WithdrawalResponse>>`

Behavior notes:

- order is newest first by `CreatedAt`
- `bankAccount` is masked

#### `GET /api/withdrawals/{id}`

Purpose:

- return one withdrawal owned by the authenticated user

Behavior notes:

- if the record does not exist, response is `404 WITHDRAWAL_NOT_FOUND`
- if the record exists but belongs to another user, response is also `404 WITHDRAWAL_NOT_FOUND`

Approved-state example:

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
    "bankName": "Vietcombank",
    "accountHolderName": "NGUYEN VAN A",
    "bankBin": "970400",
    "status": "Approved",
    "processedAt": "2026-04-03T12:45:00Z",
    "rejectionReason": null,
    "vietQrPayload": "000201010211...",
    "vietQrImageBase64": "iVBORw0KGgoAAAANSUhEUgAA...",
    "createdAt": "2026-04-03T12:30:00Z"
  },
  "error": null
}
```

#### `POST /api/withdrawals/{id}/cancel`

Purpose:

- cancel a pending withdrawal

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
    "bankName": "Vietcombank",
    "accountHolderName": "NGUYEN VAN A",
    "bankBin": "970400",
    "status": "Rejected",
    "processedAt": "2026-04-03T12:35:00Z",
    "rejectionReason": "Cancelled by user",
    "vietQrPayload": null,
    "vietQrImageBase64": null,
    "createdAt": "2026-04-03T12:30:00Z"
  },
  "error": null
}
```

## Admin Business + Admin APIs

### Business Scope

- list all withdrawals
- list pending withdrawals
- read withdrawal detail
- approve withdrawal
- reject withdrawal
- complete withdrawal
- fail withdrawal

### Business Rules

#### Approve

- source status must be `Pending`
- current code deducts `Wallet.Balance`
- current code inserts `TransactionType.WalletWithdraw`
- current code generates `VietQrPayload`
- current code stores optional `AdminNotes`

#### Reject

- source status may be `Pending` or `Approved`
- if source status is `Approved`, current code refunds wallet balance
- if source status is `Approved`, current code inserts `TransactionType.AdminAdjustment`
- QR fields are cleared

#### Complete

- source status must be `Approved`
- current code only changes status to `Completed`

#### Fail

- source status must be `Approved`
- current code refunds wallet balance
- current code inserts `TransactionType.AdminAdjustment`
- QR fields are cleared

### Admin APIs

#### `GET /api/admin/withdrawals`

- returns `ApiResponse<IEnumerable<AdminWithdrawalResponse>>`
- order is newest first by `CreatedAt`

#### `GET /api/admin/withdrawals/pending`

- returns `ApiResponse<IEnumerable<AdminWithdrawalResponse>>`
- order is oldest first by `CreatedAt`

#### `GET /api/admin/withdrawals/{id}`

- returns `ApiResponse<AdminWithdrawalResponse>`

#### `POST /api/admin/withdrawals/{id}/approve`

Request body:

```json
{
  "adminNotes": "Verified bank details"
}
```

Notes:

- body is optional
- response returns unmasked `bankAccount`

#### `POST /api/admin/withdrawals/{id}/reject`

Request body:

```json
{
  "reason": "Invalid bank account",
  "adminNotes": "Name mismatch during review"
}
```

Notes:

- `reason` is required, max `500`
- `adminNotes` is optional, max `1000`

#### `POST /api/admin/withdrawals/{id}/complete`

Request body:

```json
{
  "adminNotes": "Transfer confirmed in banking app"
}
```

Notes:

- body is optional

#### `POST /api/admin/withdrawals/{id}/fail`

Request body:

```json
{
  "reason": "Bank transfer failed",
  "adminNotes": "Destination bank unavailable"
}
```

Notes:

- `reason` is required, max `500`
- `adminNotes` is optional, max `1000`

## Shared Data Models

### `CreateWithdrawalRequest`

| Field | Type | Required | Constraints |
|---|---|---|---|
| amount | decimal | Yes | `50000` to `5000000` |
| bankAccount | string | Yes | `8` to `20` digits |
| bankName | string | Yes | max `100` |
| accountHolderName | string | Yes | max `150` |
| bankBin | string | Yes | exactly `6` digits |

### `WithdrawalResponse`

| Field | Type | Notes |
|---|---|---|
| id | Guid | withdrawal id |
| userId | Guid | owner |
| amount | decimal | requested amount |
| bankAccount | string | masked on user routes |
| bankName | string | bank name |
| accountHolderName | string | account holder name |
| bankBin | string? | bank BIN |
| status | enum | `Pending`, `Approved`, `Rejected`, `Completed`, `Failed` |
| processedAt | datetime? | set after review/cancel/fail/complete |
| rejectionReason | string? | reject, fail, or cancel reason |
| vietQrPayload | string? | present after approval |
| vietQrImageBase64 | string? | may be `null` even when payload exists |
| createdAt | datetime | creation timestamp |

### `AdminWithdrawalResponse`

Adds:

- `processedByAdminId`
- `adminNotes`

### `BankDirectoryResponse`

| Field | Type |
|---|---|
| key | string? |
| code | string? |
| shortName | string? |
| bin | string |
| name | string |
| vietQrStatus | enum |
| lookupSupported | bool |
| swiftCode | string? |

### Common Public Error Codes

| Code | Meaning |
|---|---|
| `WITHDRAWAL_NOT_FOUND` | withdrawal not found |
| `WITHDRAWAL_FORBIDDEN` | forbidden access |
| `WITHDRAWAL_INVALID_STATUS` | invalid status transition |
| `WITHDRAWAL_INSUFFICIENT_BALANCE` | insufficient balance |
| `WITHDRAWAL_DAILY_LIMIT_EXCEEDED` | daily limit exceeded |
| `WITHDRAWAL_BANK_BIN_MISSING` | QR cannot be generated because `bankBin` is missing |

## Verified Endpoint List

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

## Changelog

### 2026-04-21

- created the wallet-withdrawal-processing baseline use guide
- documented the current code-verified contract
- explicitly recorded that wallet deduction still happens at admin approval in the current baseline
