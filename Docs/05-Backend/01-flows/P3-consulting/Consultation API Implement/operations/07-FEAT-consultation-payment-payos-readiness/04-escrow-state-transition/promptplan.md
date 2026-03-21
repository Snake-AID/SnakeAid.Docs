---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
generated_from: docplan.md
status: approved
created_at: 2026-03-21
---

# PromptPlan: Step 04 Escrow State Transition

Implement the state transition from confirmed PayOS payment to consultation escrow success.

Requirements:

1. materialize escrow exactly once
2. apply the correct business transition by reference type
3. make paid PayOS references compatible with existing refund and settlement flows
