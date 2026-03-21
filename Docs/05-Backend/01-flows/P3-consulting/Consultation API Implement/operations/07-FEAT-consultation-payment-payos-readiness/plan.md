---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
affects:
  - SnakeAid.Api/Controllers/ConsultationPaymentsController.cs
  - SnakeAid.Api/Controllers/PayOsController.cs
  - SnakeAid.Service/Implements/ConsultationPaymentService.cs
  - SnakeAid.Service/Interfaces/IConsultationPaymentService.cs
  - SnakeAid.Core/Requests/Consultation/ProcessConsultationPaymentRequest.cs
  - SnakeAid.Core/Responses/Consultation/ConsultationPaymentResponse.cs
  - SnakeAid.Core/Domains/Transaction.cs
  - SnakeAid.Api/Program.cs
  - 05-Backend/01-flows/P3-consulting/Consultation API Implement/consultation.sourcecode.md
  - 05-Backend/02-layers/payos/payos.sourcecode.md
---

# Plan: Implement PayOS Option for Consultation Payment

## Analysis References

Implementation of this operation must follow:

- `analysis/01-architecture-decision.md`
- `analysis/02-state-machine.md`
- `analysis/03-sequence-flows.md`

## Goal

Add `PayOS` as an additional payment option for consultation beside existing `WalletBalance`, while preserving the same consultation escrow lifecycle:

- scheduled booking is confirmed only after payment success
- emergency request is activated only after payment success
- reject / expire still refund correctly
- consultation completion still settles correctly

## As-Is

Current code:

- consultation only supports `WalletBalance`
- `ConsultationPaymentService` owns consultation payment, refund, expire, and settlement
- PayOS layer already exists in backend but consultation does not use it
- `ConsultationPaymentResponse` currently assumes money is already escrowed

## To-Be

The implementation will introduce a dual-mode consultation payment flow:

### 1. Wallet mode

Keep current behavior unchanged:

- validate wallet balance
- materialize escrow immediately
- update business state immediately

### 2. PayOS mode

Add a new provider-backed flow:

- create payment intent / paylink for consultation reference
- return checkout metadata to client
- wait for webhook or explicit confirm/status sync
- only then materialize escrow and advance consultation state

## Implementation Tracks

This operation is split into step folders:

1. `01-contract-shape`
2. `02-payos-intent-creation`
3. `03-confirmation-and-webhook-routing`
4. `04-escrow-state-transition`
5. `05-doc-baseline-update`

Each step contains:

- `docplan.md`
- `promptplan.md`

## Core Constraints

1. `ConsultationPaymentService` remains the domain owner.
2. `IPaymentGateway` / `PayOsGateway` are reused only for provider-facing actions.
3. Refund and settlement stay in consultation domain code.
4. `WalletBalance` path must remain backward compatible.
5. PayOS path must not mark escrow as complete before verified payment success.
6. Idempotency must continue to rely on transaction semantics and explicit provider confirmation.

## Validation Plan

Implementation is complete only when:

1. request contract supports choosing wallet or PayOS
2. PayOS create-intent returns enough data for frontend redirect
3. webhook or confirm path can complete consultation payment safely
4. scheduled booking only becomes `Confirmed` after verified success
5. emergency request only becomes `PendingExpertResponse` after verified success
6. reject and expire refund still work with both wallet-origin and PayOS-origin successful payments
7. settlement path remains unchanged from a domain perspective

## Execution Order

Recommended implementation order:

1. contract shape
2. create PayOS intent
3. add confirm / webhook dispatch
4. connect escrow materialization to confirmed external payment
5. update baseline docs
