---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
generated_from: plan.md
status: approved
created_at: 2026-03-21
---

# Prompt: Implement PayOS as an Additional Consultation Payment Option

## Objective

Implement PayOS for consultation as an additional payment option beside `WalletBalance`.

## Required Outcome

The final behavior must be:

1. user can choose wallet or PayOS when paying for:
   - scheduled consultation booking
   - emergency consultation request
2. wallet path remains as-is
3. PayOS path creates an external payment intent first
4. only confirmed PayOS payment can move money into escrow
5. business transitions remain unchanged:
   - scheduled booking -> `Confirmed`
   - emergency request -> `PendingExpertResponse`
   - reject/expire -> refund
   - consultation completion -> settlement

## Required Technical Rules

1. Keep `ConsultationPaymentService` as orchestration owner.
2. Reuse `IPaymentGateway` / `PayOsGateway`.
3. Do not duplicate consultation business rules in `PayOsController`.
4. Do not push refund/settlement logic into gateway code.
5. Keep `WalletBalance` fully backward compatible.

## Required Implementation Sequence

1. update request/response/payment method contracts
2. implement create-intent path for PayOS
3. implement confirm/webhook path for consultation
4. materialize escrow only after verified external payment success
5. update docs after code is complete

## Forbidden Changes

1. Do not replace wallet flow with PayOS.
2. Do not mark consultation payment as escrowed immediately after paylink creation.
3. Do not make consultation state transitions directly inside raw webhook controller logic.
4. Do not break current refund/settlement behavior for wallet path.
