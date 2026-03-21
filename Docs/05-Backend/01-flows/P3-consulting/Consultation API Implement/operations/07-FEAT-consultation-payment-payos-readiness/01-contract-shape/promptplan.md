---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
generated_from: docplan.md
status: approved
created_at: 2026-03-21
---

# PromptPlan: Step 01 Contract Shape

Update consultation payment request/response contracts to support both wallet and PayOS.

Requirements:

1. add `PayOS` to consultation payment method enum
2. make response capable of returning provider intent metadata
3. keep wallet response backward compatible where possible
4. distinguish pending external payment from already escrowed payment
