---
doc_role: implementation
module: wallet-withdrawal-processing
kind: flow
doc_type: roadmap
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: code-verified
---

# Wallet Withdrawal Processing Roadmap

## Current Status Snapshot

- withdrawal APIs: `Implemented`
- pending reservation in wallet balance: `Not implemented`
- create-time pending reservation check: `Implemented`
- wallet deduction at approval: `Implemented`
- refund on reject after approval: `Implemented`
- refund on reject while pending hold exists: `Not implemented`

## Current Truth To Resume From

This roadmap is written so work can resume from zero memory.

Current verified baseline:

- `CreateWithdrawalRequestAsync(...)` creates `Pending` withdrawal and does not mutate `Wallet.Balance`
- create flow already subtracts total `Pending` withdrawal amount from available balance validation
- `ApproveWithdrawalAsync(...)` deducts wallet balance, generates QR, and inserts `TransactionType.WalletWithdraw`
- `CompleteWithdrawalAsync(...)` only changes status to `Completed`
- `RejectWithdrawalAsync(...)` accepts `Pending` and `Approved`
- `RejectWithdrawalAsync(...)` refunds only when the source status was `Approved`
- `FailWithdrawalAsync(...)` refunds because it only accepts `Approved`
- `CancelWithdrawalAsync(...)` converts `Pending` to `Rejected` without wallet mutation

## Target Outcome

After this work is complete:

1. creating a withdrawal immediately holds funds by reducing `Wallet.Balance`
2. `Pending` represents a real reserved-money state instead of an implicit validation-only reservation
3. `Approve` does not mutate wallet balance
4. `Complete` does not mutate wallet balance
5. `Reject`, `Cancel`, and `Fail` release held funds when the withdrawal has not been paid out
6. tests prove the new lifecycle and prevent balance double-mutation
7. docs in this folder match the shipped codebase state

## Provisional Decisions

- [x] Keep the existing API routes unless implementation proves a contract change is required
- [x] Keep the existing statuses `Pending`, `Approved`, `Rejected`, `Completed`, `Failed`
- [x] Treat `Pending` as the new wallet-hold state
- [x] Keep `Approve` as admin review plus QR generation
- [x] Keep `Complete` as external payout confirmation
- [x] Keep `Fail` as a post-approval processing failure that releases funds
- [x] Update docs in the same task as code changes
- [x] Lock final ledger strategy for hold and release transactions
- [x] Decision: add dedicated transaction types for withdrawal hold and withdrawal release

## Implementation Checklist

### Phase 1. Contract Lock

- [ ] Confirm that user-facing routes stay unchanged
- [ ] Confirm `Pending` means funds are already held
- [ ] Confirm `Reject` releases funds for both `Pending` and `Approved`
- [ ] Confirm `Cancel` releases funds for `Pending`
- [ ] Confirm `Approve` and `Complete` are non-financial state transitions
- [x] Lock transaction-history semantics for hold and release entries

### Phase 2. Service Layer

- [ ] Update `CreateWithdrawalRequestAsync(...)` to deduct wallet balance inside the same serializable transaction
- [ ] Insert the hold/debit transaction during create
- [ ] Remove wallet deduction from `ApproveWithdrawalAsync(...)`
- [ ] Remove wallet insufficient-balance recheck from `ApproveWithdrawalAsync(...)` if it becomes redundant after hold-at-create
- [ ] Keep QR generation in `ApproveWithdrawalAsync(...)`
- [ ] Update `RejectWithdrawalAsync(...)` so pending rejection releases held funds
- [ ] Update `CancelWithdrawalAsync(...)` so user cancel releases held funds
- [ ] Keep `FailWithdrawalAsync(...)` release logic aligned with the new hold-at-create flow
- [ ] Ensure no path can release funds twice
- [ ] Preserve current transaction boundaries and concurrency protection

### Phase 3. API Layer

- [ ] Verify controller contracts still map correctly after service changes
- [ ] Verify response fields still remain accurate for all statuses
- [ ] Verify no endpoint description in `useguide` claims deduction at approve anymore

### Phase 4. Tests

- [ ] Update unit test: create should reduce wallet balance immediately
- [ ] Update unit test: approve should no longer reduce wallet balance
- [ ] Add unit test: reject from pending releases held amount
- [ ] Add unit test: cancel from pending releases held amount
- [ ] Keep unit test: fail releases held amount
- [ ] Update integration lifecycle test for create -> approve -> complete
- [ ] Add integration lifecycle test for create -> reject
- [ ] Update integration assertions so only one debit and one release happen when expected

### Phase 5. Documentation Sync

- [ ] Update `introduction` from planned direction to implemented baseline
- [ ] Update `sourcecode` diagrams to the final lifecycle
- [ ] Update `useguide` to active contract after code verification
- [ ] Close resolved decision items in `hallucination`

## File Targets

- [ ] `SnakeAid.Service/Implements/WalletWithdrawService.cs`
- [ ] `SnakeAid.Tests/Unit/WalletWithdrawServiceTests.cs`
- [ ] `SnakeAid.Tests/Integration/WalletWithdrawalFlowIntegrationTests.cs`
- [ ] `SnakeAid.Docs/Docs/05-Backend/02-layers/wallet-withdrawal-processing/wallet-withdrawal-processing.introduction.md`
- [ ] `SnakeAid.Docs/Docs/05-Backend/02-layers/wallet-withdrawal-processing/wallet-withdrawal-processing.roadmap.md`
- [ ] `SnakeAid.Docs/Docs/05-Backend/02-layers/wallet-withdrawal-processing/wallet-withdrawal-processing.hallucination.md`
- [ ] `SnakeAid.Docs/Docs/05-Backend/02-layers/wallet-withdrawal-processing/wallet-withdrawal-processing.sourcecode.md`
- [ ] `SnakeAid.Docs/Docs/05-Backend/02-layers/wallet-withdrawal-processing/wallet-withdrawal-processing.useguide.md`

## Verification Strategy

Required verification for the target implementation:

1. create a withdrawal and confirm `Wallet.Balance` decreases immediately
2. approve the same withdrawal and confirm wallet balance does not change again
3. complete the same withdrawal and confirm wallet balance still does not change
4. reject a pending withdrawal and confirm held funds are released
5. cancel a pending withdrawal and confirm held funds are released
6. fail an approved withdrawal and confirm held funds are released exactly once
7. inspect transactions and confirm the ledger matches the chosen hold/release strategy

## Change Log

### 2026-04-21

- initialized the wallet-withdrawal-processing baseline docs
- documented the current code-verified lifecycle
- clarified that create-time pending subtraction already exists in code
- defined the target shift from implicit reservation to explicit wallet hold
- captured implementation phases, test updates, and doc-sync requirements
- locked the ledger direction to dedicated transaction types for withdrawal hold and withdrawal release
