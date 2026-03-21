---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
---

# DocPlan: Step 03 Confirmation and Webhook Routing

## Goal

Add a safe path that turns verified PayOS payment into consultation payment success.

## Scope

- consultation payment confirm/status sync endpoint and/or service path
- webhook routing strategy for consultation references
- provider verification through shared gateway layer

## Constraints

- verification can be shared
- consultation state transition cannot live only in controller/webhook glue
- idempotency must be explicit

## Deliverable

Verified PayOS payments can be dispatched into consultation payment service reliably.
