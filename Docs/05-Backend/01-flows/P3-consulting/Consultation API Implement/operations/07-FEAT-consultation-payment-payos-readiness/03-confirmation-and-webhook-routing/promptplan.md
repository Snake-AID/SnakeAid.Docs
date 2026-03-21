---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
generated_from: docplan.md
status: approved
created_at: 2026-03-21
---

# PromptPlan: Step 03 Confirmation and Webhook Routing

Implement consultation-specific PayOS confirmation handling.

Requirements:

1. verify provider result through shared PayOS gateway
2. route verified payment into consultation payment service
3. ensure repeated webhook/confirm calls are idempotent
4. keep business transition logic inside consultation orchestration
