---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
generated_from: docplan.md
status: approved
created_at: 2026-03-21
---

# PromptPlan: Step 02 PayOS Intent Creation

Implement PayOS create-intent logic inside consultation payment orchestration.

Requirements:

1. branch on payment method
2. for PayOS, create pending provider payment
3. store enough data to resolve later confirmation
4. return intent metadata to frontend
5. do not escrow or advance consultation state yet
