---
doc_role: implementation
module: wallet-withdrawal
kind: design-record
doc_type: implementation
status: active
last_updated: 2026-04-03
owners: [backend-team]
---

# Wallet Withdrawal Implementation

## Scope

This file records the backend implementation decisions for the current in-development withdrawal flow.
It is not a long-term product spec.

## Current Decisions

### 1. Amount and validation

- Min withdrawal amount: `50000`
- Max withdrawal amount: `5000000`
- Daily withdrawal limit per user: `10000000`
- Bank account format: `8-20` digits
- `bankBin`: required, `6` digits
- `accountHolderName`: required

### 2. Finalized entity shape

`WalletWithdraw` now includes:
- `BankAccount`
- `BankName`
- `AccountHolderName`
- `BankBin`
- `Status`
- `ProcessedAt`
- `ProcessedByAdminId`
- `RejectionReason`
- `AdminNotes`
- `VietQrPayload`
- `VietQrImageBase64`

Not included in the current contract:
- `RenderQrEnabled`

### 3. State model

Supported statuses:
- `Pending`
- `Approved`
- `Rejected`
- `Completed`
- `Failed`

Supported transitions:
- `Pending -> Approved`
- `Pending -> Rejected`
- `Approved -> Rejected`
- `Approved -> Completed`
- `Approved -> Failed`
- User cancel is stored as `Pending -> Rejected` with reason `Cancelled by user`

### 4. Balance strategy

Chosen rule:
- `create`: do not deduct balance
- `approve`: deduct balance and create `TransactionType.WalletWithdraw`
- `complete`: only confirm transfer completed
- `reject/fail` from `Approved`: refund balance and create `TransactionType.AdminAdjustment`

Rationale:
- Prevent user spending the same money after admin has accepted the payout
- Keep `Pending` lightweight and reversible
- Keep `Complete` as an operational confirmation step only

### 5. Available balance rule

At `create`, service validates:
- current wallet balance
- minus total `Pending` withdrawals

This prevents stacking multiple pending withdrawals beyond available funds before admin review.

### 6. QR contract

At `approve`, backend generates:
- `VietQrPayload`: required for approved withdrawal
- `VietQrImageBase64`: best-effort image output

Fallback rule:
- if image generation fails but payload generation succeeds, payload is still returned and image may be `null`

### 7. Masking policy

- User-facing withdrawal responses mask `bankAccount`
- Admin responses return full `bankAccount`
- Admin responses also include:
  - `ProcessedByAdminId`
  - `AdminNotes`

## API Notes

### User create contract

```json
{
  "amount": 50000,
  "bankAccount": "1234567890",
  "bankName": "Ngân hàng TMCP Sài Gòn Thương Tín (Sacombank)",
  "accountHolderName": "NGUYEN VAN A",
  "bankBin": "970400"
}
```

### Admin action payloads

Approve:
```json
{
  "adminNotes": "Verified bank details"
}
```

Reject / Fail:
```json
{
  "reason": "Invalid bank account",
  "adminNotes": "Name mismatch during review"
}
```

Complete:
```json
{
  "adminNotes": "Transfer confirmed in banking app"
}
```

## Out of Scope for Current Implementation

These are intentionally not solved in this phase:
- notification flow
- audit log table
- stable business error codes
- real bank directory source
- realtime updates
- fraud checks / rate limiting
- layering cleanup for `IWalletService`

## Change Log

### 2026-04-03

- Finalized Phase 2 domain contract
- Added `AccountHolderName`, `ProcessedByAdminId`, and `AdminNotes`
- Changed amount range to `50000-5000000`
- Added daily withdrawal limit `10000000`
- Moved balance deduction from `complete` to `approve`
- Changed `complete` into transfer-confirmation step
- Formalized QR fallback behavior
