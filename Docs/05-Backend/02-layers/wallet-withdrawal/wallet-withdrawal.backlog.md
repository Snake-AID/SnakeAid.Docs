---
doc_role: backlog
module: wallet-withdrawal
kind: implementation-tracker
doc_type: backlog
status: active
last_updated: 2026-04-03
owners: [backend-team]
---

# Wallet Withdrawal Backlog

## Scope

Internal backend tracker for withdrawal work.
User-facing API usage stays in `wallet-withdrawal.usageguide.md`.

Current assumption:
- Withdrawal flow is still in development and has not gone to production
- Backward compatibility is not a primary constraint for this module right now
- Schema/API cleanup may prefer correctness and simplicity over preserving old dev-only contracts

## Current Baseline

- User endpoints exist:
  - `GET /api/wallet/me`
  - `GET /api/wallet/banks`
  - `POST /api/withdrawals/create`
  - `GET /api/withdrawals/me`
  - `GET /api/withdrawals/{id}`
  - `POST /api/withdrawals/{id}/cancel`
- Admin endpoints exist:
  - `GET /api/admin/withdrawals`
  - `GET /api/admin/withdrawals/pending`
  - `GET /api/admin/withdrawals/{id}`
  - `POST /api/admin/withdrawals/{id}/approve`
  - `POST /api/admin/withdrawals/{id}/reject`
  - `POST /api/admin/withdrawals/{id}/complete`
  - `POST /api/admin/withdrawals/{id}/fail`
- Success envelope for wallet + withdrawal endpoints: `ApiResponse<T>`
- `bankBin` is now persisted and used when generating VietQR
- Required create fields now include `accountHolderName`
- Amount rule is now `50000` to `5000000`
- Daily withdrawal limit is `10000000`
- Balance is now deducted at `approve`
- User response masks `bankAccount`; admin response returns full value and admin metadata

## Phase Status

### Phase 1. Contract Cleanup

Status: Completed on backend 2026-04-03

- [x] Persist `bankBin` in `WalletWithdraw`
- [x] Return `bankBin` in withdrawal responses
- [x] Use stored `bankBin` in approve QR flow
- [x] Add `GET /api/admin/withdrawals/{id}`
- [x] Standardize withdrawal endpoints on `ApiResponse<T>`
- [x] Lock cancel contract to `POST /api/withdrawals/{id}/cancel`
- [x] Align docs with current contract
- [x] Update backlog progress after Phase 1 completion
- [x] Update bash scripts to parse `.data`

Residual notes:
- Legacy dev data without `BankBin` is not a blocker for current design decisions
- If local/dev data becomes inconsistent after schema changes, data reset or targeted cleanup is acceptable
- No `DELETE` alias is provided for cancel

### Phase 2. Domain Finalization

Status: Completed on backend 2026-04-03

Goal: settle the actual withdrawal business contract, not just transport/API consistency.

- [x] Finalize `WalletWithdraw` schema
- [x] Require `AccountHolderName`
- [x] Add `ProcessedByAdminId` / `AdminNotes`
- [x] Drop `RenderQrEnabled` from current contract
- [x] Finalize naming across entity, DTO, and docs
- [x] Finalize amount limits
- [x] Add daily withdrawal limit
- [x] Finalize balance deduction strategy
- [x] Decide that balance is not reserved at `create`
- [x] Add oversubscription validation for pending requests at `create`
- [x] Formalize masking policy for admin vs user
- [ ] Formalize QR contract:
  - [x] payload format
  - [x] image is optional
  - [x] fallback behavior when image generation fails
- [x] Update `wallet-withdrawal.usageguide.md` for any API contract change in Phase 2
- [x] Update `wallet-withdrawal.implementation.md` for any design/schema decision in Phase 2
- [x] Update this backlog after Phase 2 completion

Implementation note:
- Because this flow is not yet in production, Phase 2 may change schema and API contract directly without carrying legacy compatibility paths unless explicitly required later

Decision summary:
- `AccountHolderName` is part of the required create contract
- `ProcessedByAdminId` and `AdminNotes` are stored on the withdrawal record
- Balance strategy is:
  - `create`: no deduction
  - `approve`: deduct balance + create withdrawal transaction
  - `complete`: confirm transfer only
  - `reject/fail` from `Approved`: refund balance + create admin adjustment transaction
- QR contract is payload-required for approved withdrawal, image best-effort

### Phase 3. Operational Hardening

Status: Not started

- [ ] Replace mock bank directory with real source
- [ ] Add withdrawal notification flow
- [ ] Add dedicated audit trail beyond the current withdrawal fields
- [ ] Add stable public error codes for FE/mobile
- [ ] Add unit tests for create/approve/reject/complete/fail/cancel
- [ ] Add integration tests for status transitions and balance effects
- [ ] Update `wallet-withdrawal.usageguide.md` if operational changes affect public API/error contract
- [ ] Update `wallet-withdrawal.implementation.md` for operational flow changes
- [ ] Update this backlog after Phase 3 completion

Notes:
- Bash scripts are aligned with current response envelope
- Automated coverage is still shallow for withdrawal service rules

### Phase 4. Production Guardrails

Status: Not started

- [ ] Add rate limiting if product requires it
- [ ] Decide whether realtime status update is needed
- [ ] Add push/webhook/event flow if realtime is required
- [ ] Add fraud / suspicious pattern rules if required
- [ ] Add duplicate request prevention if required
- [ ] Add account holder validation if required
- [ ] Update `wallet-withdrawal.usageguide.md` if Phase 4 changes client integration behavior
- [ ] Update `wallet-withdrawal.implementation.md` for Phase 4 production guardrails
- [ ] Update this backlog after Phase 4 completion

## Parking Lot

These are known issues but not on the immediate critical path:

- `WalletWithdrawService` currently depends on `IWalletService` DTO output instead of a domain-level wallet model
- Stable business error typing is still missing; current flow still relies on exception text in several cases
- Old-data remediation is optional for now and should only be added if the team decides dev/staging data must be preserved

## Recommended Next Order

- [x] Phase 1. Contract Cleanup
- [x] Phase 2. Domain Finalization
- [ ] Phase 3. Operational Hardening
- [ ] Phase 4. Production Guardrails
