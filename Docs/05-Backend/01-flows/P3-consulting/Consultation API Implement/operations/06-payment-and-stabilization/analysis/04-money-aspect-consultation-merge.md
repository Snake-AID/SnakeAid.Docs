# Money Aspect Consultation Merge Notes

This analysis file captures the consultation-specific current truths merged from `02-layers/money-aspect` into the P3 consultation flow docs.

## Merged conclusions

- consultation remains the only escrow-style business flow among the compared money flows
- consultation escrow is now ledger-driven, not system-wallet-driven
- consultation settlement is no longer `gross amount -> expert 100%`
- consultation settlement now emits:
  - `ExpertPayout`
  - `PlatformFee`
- `ConsultationPaymentResponse.SystemWalletBalanceAfter` has been removed from the client contract
- transaction/reporting for `transType=consultation` must expect `PlatformFee`
- `PlatformFee` is platform-owned, so transaction owner fields may be `null`

## Flow-level implications

- consultation baseline docs must describe escrow via transaction types, not via `system wallet`
- consultation payment usage guide must not show `systemWalletBalanceAfter`
- flow reporting and mobile integration must not assume a single expert payout equal to gross amount

## Source layer references

- `money-aspect.changelog.md`
- `money-aspect.sourcemap.md`
- `money-aspect.refactoring.md`

The flow docs now carry only the consultation-specific outcomes; the broader cross-flow reasoning remains in the money-aspect layer.
