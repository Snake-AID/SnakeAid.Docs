---
doc_role: backlog
module: wallet-withdrawal
kind: implementation-tracker
doc_type: backlog
status: active
last_updated: 2026-04-04
owners: [backend-team]
---

# Wallet Withdrawal Backlog

## Scope

Internal backend tracker for withdrawal work.
User-facing API usage stays in `wallet-withdrawal.usageguide.md`.

Current assumption:
- Withdrawal flow is still being finalized at feature level
- Some environments may already carry interim withdrawal migrations, so migration cleanup is deferred to Phase 5
- Backward compatibility is still secondary to correctness for the feature contract, but migration history can no longer be treated as purely dev-only

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

Status: Completed on backend 2026-04-03

- [x] Replace mock `BankDirectoryService` data with real `VietQRHelper` bank directory integration
- [x] Add caching strategy for bank directory lookup
- [x] Review whether withdrawal create contract should expose richer bank metadata beyond `bin` + `name`
- [x] Add withdrawal notification flow
- [x] Decide to keep only final withdrawal state on `WalletWithdraw`
- [x] Add stable public error codes for FE/mobile
- [x] Add unit tests for create/approve/reject/complete/fail/cancel
- [x] Add integration tests for status transitions and balance effects
- [x] Update `wallet-withdrawal.usageguide.md` if operational changes affect public API/error contract
- [x] Update `wallet-withdrawal.implementation.md` for operational flow changes
- [x] Update this backlog after Phase 3 completion

Notes:
- Bash scripts are aligned with current response envelope
- Phase 3 test pass also caught and fixed:
  - untracked admin state transitions caused by repository default `asNoTracking`
  - negative withdrawal transaction amount violating current transaction validation
- Create-withdrawal checks now run atomically inside one transaction to reduce concurrent oversubscription races
- Notification dispatch is now best-effort after commit so publish failures do not turn committed withdrawal state changes into API errors
- Audit trail was explicitly dropped after review; no separate withdrawal history table is kept in this phase

### Phase 4. Production Guardrails

Status: Not started

- [ ] Add rate limiting if product requires it
- [ ] Apply `ValidateModel` consistently to withdrawal endpoints so request validation behavior is explicit and production-safe
- [ ] Decide whether realtime status update is needed
- [ ] Add push/webhook/event flow if realtime is required
- [ ] Add fraud / suspicious pattern rules if required
- [ ] Add duplicate request prevention if required
- [ ] Add account holder validation if required
- [ ] Update `wallet-withdrawal.usageguide.md` if Phase 4 changes client integration behavior
- [ ] Update `wallet-withdrawal.implementation.md` for Phase 4 production guardrails
- [ ] Update this backlog after Phase 4 completion

### Phase 5. Migration Consolidation

Status: Completed on backend 2026-04-04

Goal: only after the withdrawal flow is fully stabilized, collapse fragmented withdrawal migrations into a single clean breaking migration path.

- [x] Freeze withdrawal schema changes before consolidation starts
- [x] Identify the nearest migration baseline not related to withdrawal flow
- [x] Inventory all withdrawal-related migrations currently layered on top of that baseline
- [x] Verify no production data depends on withdrawal-specific schema introduced by those mini migrations
- [x] Prepare rollback notes for local and production databases back to the non-withdrawal baseline
- [x] Revert database state to the chosen baseline before rewriting source migration history
- [x] Remove fragmented withdrawal migrations from source
- [x] Generate one consolidated withdrawal migration from the finalized model
- [x] Verify upgrade path on a clean database: baseline -> consolidated migration
- [x] Verify upgrade path on an existing reverted database: baseline -> consolidated migration
- [x] Update `wallet-withdrawal.implementation.md` with the final migration strategy
- [x] Update this backlog after Phase 5 completion

Phase 5 result:
- Chosen baseline: `20260402100313_UpdateAppNotification`
- Removed fragmented withdrawal migrations:
  - `20260402141955_AddWalletWithdrawFields_PostgreSQL`
  - `20260403091500_AddWalletWithdrawBankBin_PostgreSQL`
  - `20260403133000_FinalizeWalletWithdrawPhase2_PostgreSQL`
- Added consolidated migration:
  - `20260403200823_SnakeaidWalletWithdraw`
- Verified database path by reverting to the chosen baseline first, then applying the consolidated migration

Breaking-change note:
- This phase intentionally rewrites the withdrawal migration path
- It must only happen after feature work is complete, otherwise new schema churn will fragment the path again
- Database execution remains manual; backend only prepares the plan, source changes, and validation steps

## Parking Lot

These are known issues but not on the immediate critical path:

- `WalletWithdrawService` currently depends on `IWalletService` DTO output instead of a domain-level wallet model
- Old-data remediation is optional for now and should only be added if the team decides dev/staging data must be preserved

## Recommended Next Order

- [x] Phase 1. Contract Cleanup
- [x] Phase 2. Domain Finalization
- [x] Phase 3. Operational Hardening
- [x] Phase 5. Migration Consolidation
- [ ] Phase 4. Production Guardrails

Current sequencing note:
- `Migration Consolidation` is now prioritized before `Production Guardrails`
- Reason: Phase 3 contract is already stable enough for frontend/mobile integration, while migration history should be collapsed before broader handoff and environment rollout
- Constraint: keep Phase 5 backward-compatibility assumptions explicit because migration rewriting can affect deployment workflow even if it does not change the public API contract
