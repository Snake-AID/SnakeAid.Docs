---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
---

# DocPlan: Step 04 Escrow State Transition

## Goal

Connect confirmed PayOS payment to the same escrow lifecycle already used by consultation.

## Scope

- materialize escrow only after confirmed payment
- update scheduled booking to `Confirmed`
- update emergency request to `PendingExpertResponse`
- preserve refund and settlement semantics

## Constraints

- avoid double materialization
- keep refund and settlement code reusable across wallet-origin and PayOS-origin paid references

## Deliverable

Once PayOS payment is confirmed, consultation enters the same domain lifecycle as wallet-paid consultation.
