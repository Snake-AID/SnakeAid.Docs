---
doc_role: baseline
module: wallet-withdrawal-processing
kind: flow
doc_type: introduction
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: code-verified
---

# Wallet Withdrawal Processing Introduction

## Goal

This doc set prepares the next wallet-withdrawal processing change:

1. move wallet deduction from `Approved/Complete-era processing` to `Pending`
2. add an explicit refund/release path when admin `Reject`s a withdrawal
3. keep the docs resumable without relying on prior chat context

## Current Code-Verified State

Current withdrawal flow is implemented in:

- `SnakeAid.Api/Controllers/WithdrawalsController.cs`
- `SnakeAid.Api/Controllers/AdminWithdrawalsController.cs`
- `SnakeAid.Service/Implements/WalletWithdrawService.cs`
- `SnakeAid.Tests/Unit/WalletWithdrawServiceTests.cs`
- `SnakeAid.Tests/Integration/WalletWithdrawalFlowIntegrationTests.cs`

Current verified behavior:

- user creates a withdrawal in `Pending`
- wallet balance is not reduced at create time
- create-time validation already subtracts the sum of existing `Pending` withdrawals from available balance
- admin `Approve` generates VietQR and deducts wallet balance
- admin `Complete` only marks the transfer as completed
- admin `Reject` refunds only when the withdrawal was already `Approved`
- admin `Fail` refunds because it only applies to `Approved`
- user `Cancel` changes `Pending` to `Rejected`

## Important Clarification

The exact oversubscription example from the business note is not fully current anymore.

Current code already blocks repeated pending-withdrawal stacking inside `CreateWithdrawalRequestAsync(...)` by using:

- `availableBalance = wallet.Balance - sum(Pending withdrawals)`

That means the reservation is currently enforced only inside withdrawal-create validation, not as a real wallet hold recorded in the wallet balance itself.

## Why This Change Still Matters

The target business rule is stronger and cleaner:

- once a withdrawal becomes `Pending`, the amount should already be held
- admin review should not perform the first wallet deduction
- reject/cancel/fail paths should release the held amount consistently
- the reserve/release behavior should be visible in persisted wallet state and transaction history

## Planned Direction

Planned implementation direction for this workstream:

1. debit wallet balance when the withdrawal is created as `Pending`
2. create the withdrawal ledger record at the same time as the hold
3. make `Approve` a review-and-QR step only
4. keep `Complete` as a transfer confirmation step only
5. release held funds on `Reject`
6. release held funds on `Cancel`
7. keep release on `Fail`
8. update tests and docs in the same change set

Chosen ledger direction:

- create a dedicated hold transaction type when the withdrawal enters `Pending`
- create a dedicated release transaction type when funds are returned from `Reject`, `Cancel`, or `Fail`
- stop overloading `WalletWithdraw` or `AdminAdjustment` for withdrawal hold/release semantics

## Target Transaction Timeline

The new direction assumes two dedicated transaction types:

- `TransactionType.WalletWithdrawHold`
- `TransactionType.WalletWithdrawRelease`

### Happy Path: Create -> Approve -> Complete

Example starting state:

- `Wallet.Balance = 500000`
- withdrawal amount = `100000`

Step 1. User creates the withdrawal

- create `WalletWithdraw` with `Status = Pending`
- reduce `Wallet.Balance` from `500000` to `400000`
- insert one transaction:
  - `TransactionType = WalletWithdrawHold`
  - `Amount = 100000`
  - `ReferenceId = withdrawalId`

Step 2. Admin approves the withdrawal

- move withdrawal from `Pending` to `Approved`
- generate and store VietQR data
- do not change `Wallet.Balance`
- do not insert a new financial transaction

Step 3. Admin completes the withdrawal

- move withdrawal from `Approved` to `Completed`
- do not change `Wallet.Balance`
- do not insert a new financial transaction

Net result:

- one hold transaction exists
- no release transaction exists
- the wallet balance stays reduced after completion

### Reject Path: Create -> Reject

Step 1. User creates the withdrawal

- create `Pending`
- reduce wallet balance
- insert `WalletWithdrawHold`

Step 2. Admin rejects the withdrawal

- move withdrawal to `Rejected`
- restore `Wallet.Balance`
- insert one transaction:
  - `TransactionType = WalletWithdrawRelease`
  - `Amount = 100000`
  - `ReferenceId = withdrawalId`

Net result:

- one hold transaction
- one release transaction
- net wallet effect is `0`

### Cancel Path: Create -> Cancel

Step 1. User creates the withdrawal

- create `Pending`
- reduce wallet balance
- insert `WalletWithdrawHold`

Step 2. User cancels the withdrawal

- move withdrawal to `Rejected`
- set `RejectionReason = "Cancelled by user"`
- restore `Wallet.Balance`
- insert `WalletWithdrawRelease`

### Fail Path: Create -> Approve -> Fail

Step 1. Create

- create `Pending`
- reduce wallet balance
- insert `WalletWithdrawHold`

Step 2. Approve

- review and QR only
- no wallet mutation
- no new financial transaction

Step 3. Fail

- move withdrawal to `Failed`
- clear QR fields
- restore `Wallet.Balance`
- insert `WalletWithdrawRelease`

### Reject After Approval Path: Create -> Approve -> Reject

Step 1. Create

- reduce wallet balance
- insert `WalletWithdrawHold`

Step 2. Approve

- review and QR only
- no wallet mutation

Step 3. Reject

- move withdrawal to `Rejected`
- restore `Wallet.Balance`
- insert `WalletWithdrawRelease`

### Lifecycle Summary

| Flow | First transaction | Second transaction | Final wallet effect |
|---|---|---|---|
| Create -> Approve -> Complete | `WalletWithdrawHold` | none | `-amount` |
| Create -> Reject | `WalletWithdrawHold` | `WalletWithdrawRelease` | `0` |
| Create -> Cancel | `WalletWithdrawHold` | `WalletWithdrawRelease` | `0` |
| Create -> Approve -> Fail | `WalletWithdrawHold` | `WalletWithdrawRelease` | `0` |
| Create -> Approve -> Reject | `WalletWithdrawHold` | `WalletWithdrawRelease` | `0` |

### Guardrails Required In Code

- only `Create` may insert `WalletWithdrawHold`
- only `Reject`, `Cancel`, and `Fail` may insert `WalletWithdrawRelease`
- `Approve` and `Complete` must remain non-financial transitions
- one withdrawal may be released at most once
- any second release attempt for the same `withdrawalId` must be blocked

## Scope Boundary

In scope:

- withdrawal state-processing changes
- wallet balance mutation timing
- refund/release behavior for admin rejection
- unit and integration tests
- baseline docs in this folder

Out of scope unless a later decision adds them:

- major API route redesign
- new withdrawal statuses
- notification transport redesign
- bank directory redesign

## Baseline Docs In This Folder

- `wallet-withdrawal-processing.introduction.md`
- `wallet-withdrawal-processing.roadmap.md`
- `wallet-withdrawal-processing.hallucination.md`
- `wallet-withdrawal-processing.sourcecode.md`
- `wallet-withdrawal-processing.useguide.md`

## Resume Note

When code changes land, this doc set must be updated so that:

- `introduction` reflects the new current truth
- `roadmap` shows completed and remaining work
- `hallucination` keeps decision history for closed risks
- `sourcecode` matches actual method names and lifecycle
- `useguide` reflects only active code-verified API behavior
