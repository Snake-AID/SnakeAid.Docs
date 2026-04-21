---
doc_role: implementation
module: wallet-withdrawal-processing
kind: flow
doc_type: hallucination
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: mixed
---

# Wallet Withdrawal Processing Decision Log

## Purpose

This file records only the items that still need product or owner input.

When a risk is resolved:

1. keep the original option list
2. record the chosen decision
3. sync the decision into `introduction`, `roadmap`, `sourcecode`, and `useguide` when relevant
4. mark the risk as closed

## Open Risks

No open risk remains in the current baseline.

## Closed Risks

### Risk 1. Ledger Semantics For Hold And Release

Status: `Closed`

Why this needs a decision:

- current code originally used `TransactionType.WalletWithdraw` when admin approved
- current refund paths originally used `TransactionType.AdminAdjustment`
- after moving deduction to `Pending`, the team must decide how the ledger should describe the hold and later release

Options:

1. Recommended: create `TransactionType.WalletWithdraw` at `Pending`, keep release entries as `TransactionType.AdminAdjustment`
2. Add new transaction types for explicit withdrawal debit and refund, finalized as `WithdrawalInitiated` and `WithdrawalRefund`
3. Do not create any financial transaction at `Pending`; only mutate `Wallet.Balance` and keep a release transaction on reject/cancel/fail

Tradeoff summary:

- Option 1 is the smallest code change and stays closest to current model
- Option 2 gives the clearest ledger semantics but requires enum, mapping, and downstream reporting changes
- Option 3 is the weakest audit trail because the first balance mutation would not have a matching transaction row

Decision record:

- chosen option: `2`
- owner decision date: `2026-04-21`
- final decision:
  - add dedicated transaction types `WithdrawalInitiated` and `WithdrawalRefund`
  - use `WithdrawalInitiated` when a withdrawal is created as `Pending`
  - use `WithdrawalRefund` when funds are returned by `Reject`, `Cancel`, or `Fail`
  - avoid overloading `TransactionType.WalletWithdraw` and `TransactionType.AdminAdjustment` for the new hold/release lifecycle
