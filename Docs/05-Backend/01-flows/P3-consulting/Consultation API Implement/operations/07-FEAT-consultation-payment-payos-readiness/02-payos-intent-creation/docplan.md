---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
---

# DocPlan: Step 02 PayOS Intent Creation

## Goal

Create a PayOS payment intent for consultation references without materializing escrow yet.

## Scope

- scheduled booking PayOS intent
- emergency request PayOS intent
- persist pending transaction / provider reference mapping
- return checkout URL and provider metadata

## Constraints

- do not confirm booking/request here
- do not debit wallet or credit system wallet here
- preserve duplicate payment protection

## Deliverable

Consultation payment endpoints can return a PayOS checkout session for valid references.
