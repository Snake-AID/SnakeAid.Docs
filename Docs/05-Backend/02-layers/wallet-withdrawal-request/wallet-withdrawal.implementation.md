---
doc_role: implementation
module: wallet-withdrawal
kind: design-record
doc_type: implementation
status: active
last_updated: 2026-04-04
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

Concurrency note:
- create-withdrawal checks and insert now run inside one database transaction with `Serializable` isolation
- the service recomputes wallet, pending total, and daily total inside that transaction before inserting the new `WalletWithdraw`
- this closes the race window where concurrent create requests could otherwise bypass balance or daily-limit checks

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

### 8. Bank directory source

- `BankDirectoryService` now reads from `VietQRHelper.BankApp.BanksObject`
- Results are cached in-memory for `24` hours
- Backend now exposes richer bank metadata:
  - `Key`
  - `Code`
  - `ShortName`
  - `LookupSupported`
  - `SwiftCode`

Implementation goal:
- keep FE/mobile bank picker aligned with the same source used for VietQR-related metadata

### 9. Stable public error codes

Withdrawal flow now uses stable codes for FE/mobile-visible failures:
- `WITHDRAWAL_NOT_FOUND`
- `WITHDRAWAL_FORBIDDEN`
- `WITHDRAWAL_INVALID_STATUS`
- `WITHDRAWAL_INSUFFICIENT_BALANCE`
- `WITHDRAWAL_DAILY_LIMIT_EXCEEDED`
- `WITHDRAWAL_BANK_BIN_MISSING`

These codes are propagated through `ApiException` and the exception middleware so clients do not need to parse exception text.

### 10. Notification flow

Current notification behavior:
- `create`: broadcast admin notification `WITHDRAWAL_REQUEST_CREATED`
- `cancel`: broadcast admin notification `WITHDRAWAL_CANCELLED`
- `approve`: publish user notification `WITHDRAWAL_APPROVED`
- `reject`: publish user notification `WITHDRAWAL_REJECTED`
- `complete`: publish user notification `WITHDRAWAL_COMPLETED`
- `fail`: publish user notification `WITHDRAWAL_FAILED`

Deep-link generation is handled in `NotificationQueueService`.

Decision note:
- notification remains inside `WalletWithdrawService` for now
- backend does not add a withdrawal-specific notification orchestration layer in this phase
- `INotificationQueueService` remains a transport dependency only

Reliability note:
- database state change is committed before notification dispatch is attempted
- notification publish/broadcast is best-effort after commit
- notification failures are logged and swallowed so a successful withdrawal state change does not become a `500` only because transport failed

### 11. Persistence fix discovered during Phase 3

While adding tests, Phase 3 uncovered two real implementation bugs:
- approval path was writing a negative `Transaction.Amount`, which violated the current transaction validation attributes
- approve/reject/complete/fail were loading `WalletWithdraw` with repository default `asNoTracking = true`, so status and wallet changes were not reliably persisted

Current fix:
- withdrawal debit transaction now stores a positive amount and relies on `TransactionType` + wallet balance mutation to indicate direction
- state-changing admin queries now use tracked entities (`asNoTracking: false`)

### 12. Automated coverage added

Current added coverage:
- unit tests for create/cancel/approve/fail service behavior
- integration tests for `create -> approve -> complete`
- integration tests for `create -> approve -> fail`

### 13. Service contract notes

- `GetWithdrawalByIdAsync` is now nullable in the service contract: `Task<WalletWithdraw?>`
- controllers remain responsible for converting `null` into the appropriate `404` API response
- this matches the current controller flow more accurately than a non-null signature with post-call null checks

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
- realtime updates
- fraud checks / rate limiting
- consistent `ValidateModel` coverage across withdrawal endpoints
- layering cleanup for `IWalletService`

Sequencing note after Phase 3:
- The current public withdrawal contract is considered stable enough for frontend/mobile integration.
- Migration consolidation is prioritized before production guardrails so the team can collapse withdrawal migration history before broader handoff and environment rollout.
- This sequencing does not imply Phase 4 is unimportant; it only means Phase 4 should stay backward-compatible with the already-shared client contract.

## Migration Strategy

### Phase 5 consolidation result

Withdrawal migration path is now consolidated.

Chosen baseline:
- `20260402100313_UpdateAppNotification`

Removed fragmented withdrawal migrations:
- `20260402141955_AddWalletWithdrawFields_PostgreSQL`
- `20260403091500_AddWalletWithdrawBankBin_PostgreSQL`
- `20260403133000_FinalizeWalletWithdrawPhase2_PostgreSQL`

Current consolidated migration:
- `20260403200823_SnakeaidWalletWithdraw`

The consolidated migration contains the finalized withdrawal schema delta on top of the chosen baseline:
- `AccountHolderName`
- `AdminNotes`
- `BankBin`
- `ProcessedByAdminId`
- `VietQrPayload`
- `VietQrImageBase64`
- index + foreign key for `ProcessedByAdminId`

Verification performed during Phase 5:
- revert database state back to `20260402100313_UpdateAppNotification`
- remove fragmented withdrawal migrations from source history
- generate one consolidated migration from the finalized model
- apply `20260403200823_SnakeaidWalletWithdraw` on the reverted database
- verify migration list now shows one consolidated withdrawal migration after the chosen baseline

## Change Log

### 2026-04-03

- Finalized Phase 2 domain contract
- Added `AccountHolderName`, `ProcessedByAdminId`, and `AdminNotes`
- Changed amount range to `50000-5000000`
- Added daily withdrawal limit `10000000`
- Moved balance deduction from `complete` to `approve`
- Changed `complete` into transfer-confirmation step
- Formalized QR fallback behavior
- Phase 3 replaced mock bank directory with `VietQRHelper` integration + caching
- Added stable public withdrawal error codes
- Added withdrawal notifications for create/cancel/approve/reject/complete/fail
- Fixed tracked-entity persistence bug in admin status transitions
- Added automated test coverage for main withdrawal state transitions
- Decided not to persist withdrawal audit history; only final state is kept on `WalletWithdraw`

### 2026-04-04

- Completed Phase 5 migration consolidation for wallet withdrawal
- Replaced the three fragmented April withdrawal migrations with one consolidated migration: `20260403200823_SnakeaidWalletWithdraw`
- Verified the revert-and-reapply path against the configured database target
- Made create-withdrawal limit checks atomic by running wallet/pending/daily-limit checks and insert inside one transaction
- Made withdrawal notifications best-effort after commit so publish failures no longer turn successful DB commits into API errors
- Aligned `GetWithdrawalByIdAsync` service contract with controller behavior by returning nullable results
