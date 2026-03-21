---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
---

# DocPlan: Step 01 Contract Shape

## Goal

Adjust consultation payment contracts so backend can represent both:

- `WalletBalance`
- `PayOS`

## Scope

- extend `ConsultationPaymentMethod`
- reshape request if PayOS needs extra fields later
- reshape `ConsultationPaymentResponse` so it can represent:
  - escrowed wallet success
  - pending PayOS intent
  - provider metadata such as checkout URL / order code

## Constraints

- do not break current wallet consumers
- avoid provider-specific fields leaking into every successful wallet response unless clearly optional
- response should distinguish:
  - `PendingExternalPayment`
  - `Escrowed`

## Deliverable

A clear API contract that can support both payment paths without ambiguity.
