---
doc_role: operation
operation_id: 06-FEAT-payment-and-stabilization
type: FEAT
status: done
created_at: 2026-03-07
merged_from:
  - 06-STAB-mobile-readiness-gap-closure
  - 07-FEAT-consultation-payment-payos-readiness
  - 08-FEAT-consultation-payos-option
  - money-aspect-phase-6B-consultation-escrow
  - money-aspect-phase-7-consultation-platform-fee
  - money-aspect-phase-8-platform-owned-ledger-cleanup
affects:
  - Api/Controllers/ConsultationPaymentsController.cs
  - Api/Controllers/PayOsController.cs
  - Service/Implements/ConsultationPaymentService.cs
  - Service/Implements/ExpertService.cs
  - Service/Implements/BookingService.cs
  - Service/Implements/EmergencyConsultationService.cs
  - Service/Implements/ConsultationLifecycleBackgroundService.cs
  - Core/Domains/ExpertProfile.cs
  - Core/Domains/ConsultationBooking.cs
  - Core/Domains/ConsultationPingRequest.cs
  - Core/Requests/Consultation/ProcessConsultationPaymentRequest.cs
  - Core/Responses/Consultation/ConsultationPaymentResponse.cs
  - Core/Enums/TransactionType.cs
---

# Operation 06: Payment & Stabilization

## 1. As-Is

Consultation had already established scheduled/emergency business flows, but payment and settlement semantics were incomplete and later needed a second stabilization pass.

Initial gaps covered by this merged operation:

- expert profile data still lacked final pricing/stats fields
- PayOS was not yet a stable consultation payment option
- lifecycle automation and settlement idempotency still needed reinforcement
- consultation money semantics still had residue from system-wallet-style escrow thinking

## 2. Gap Analysis

| Gap | Description |
|---|---|
| Payment option gap | consultation needed `PayOs` alongside `WalletBalance` |
| Mobile contract gap | payment contract needed clear pending vs escrowed behavior and confirm fallback |
| Escrow semantic gap | consultation escrow had to move from system-wallet side effects to transaction-sourced semantics |
| Settlement semantic gap | settlement could no longer be modeled as gross amount paid fully to expert |
| Reporting gap | consultation transaction views needed to expect `PlatformFee` and nullable owner fields |
| Stability gap | lifecycle automation and multi-replica safety had to protect refund/settlement paths |

## 3. To-Be Design

### Expert profile completion

- dual pricing:
  - `ScheduledConsultationFee`
  - `EmergencyConsultationFee`
- profile stats:
  - `TotalConsultations`
  - `AverageResponseTimeMinutes`
  - `SuccessRate`
- `IsVerified` remains deferred from MVP

### Payment entrypoints

- `POST /api/consultations/scheduled/{bookingId}/payments`
- `POST /api/consultations/instant/{requestId}/payments`
- `POST /api/consultations/payments/confirm`

### WalletBalance vs PayOS

| Aspect | WalletBalance | PayOs |
|---|---|---|
| Escrow timing | immediate | after confirm |
| Response status | `Escrowed` | `Pending` then `Escrowed` |
| External checkout | no | yes |
| Refund / settlement semantics | same consultation rules | same consultation rules |

### Consultation escrow semantics after money-aspect merge

- consultation remains an escrow flow
- escrow hold is inferred from `ConsultationPayment`
- refund is represented by `ConsultationRefund`
- settlement is represented by:
  - `ExpertPayout`
  - `PlatformFee`
- consultation no longer uses `EscrowHold` / `EscrowRelease` as public semantics
- `ConsultationPaymentResponse.SystemWalletBalanceAfter` is removed from the client contract

### Settlement rules after money-aspect merge

- consultation completion triggers escrow release logic
- expert receives net payout only
- platform keeps `PlatformFee`
- fee percent uses `SystemSettingKeys` and defaults to `20%` if no explicit setting exists
- rounding prioritizes the expert net amount, then derives platform fee from the remainder

### Background automation

- `ConsultationLifecycleBackgroundService` handles:
  - timed-out emergency expiry + refund
  - scheduled auto-complete + settlement
  - elapsed emergency auto-complete + settlement
- PostgreSQL advisory locks protect multi-replica execution

### Analysis references

- `analysis/01-architecture-decision.md`
- `analysis/02-state-machine.md`
- `analysis/03-sequence-flows.md`
- `analysis/04-money-aspect-consultation-merge.md`

## 4. Impacted Components

| Component | Change |
|---|---|
| `ConsultationPaymentsController` | payment entrypoints and confirm flow |
| `PayOsController` | consultation PayOS dispatch integration |
| `ConsultationPaymentService` | wallet payment, PayOS confirm/webhook, ledger-driven escrow, settlement split |
| `ConsultationLifecycleBackgroundService` | timed lifecycle work with refund/settlement safety |
| `ExpertService` | pricing/stat enrichment |
| `ConsultationPaymentResponse` | public payment contract without `SystemWalletBalanceAfter` |
| `TransactionType` | consultation reporting must include `PlatformFee` |

## 5. Risks & Constraints

| Risk | Mitigation |
|---|---|
| Mobile still parsing removed fields | keep flow-level payment guide explicit and update changelog-style notes |
| Transaction/reporting assumes every consultation item has a user owner | document nullable `UserName` / `FullName` for `PlatformFee` |
| Settlement duplication across replicas | keep advisory locks and idempotent settlement checks |
| PayOS callback timing is asynchronous | preserve webhook + return + manual confirm fallback |
| Escrow UI still tied to old `system wallet` mental model | document ledger-driven escrow semantics in consultation docs |

## 6. Validation Plan

- verify wallet payment returns `Escrowed` immediately
- verify PayOS payment returns `Pending` and becomes `Escrowed` after confirm
- verify settlement creates `ExpertPayout + PlatformFee`
- verify refund still creates `ConsultationRefund`
- verify consultation payment contract no longer exposes `SystemWalletBalanceAfter`
- verify transaction/reporting views tolerate `PlatformFee` with nullable owner fields
