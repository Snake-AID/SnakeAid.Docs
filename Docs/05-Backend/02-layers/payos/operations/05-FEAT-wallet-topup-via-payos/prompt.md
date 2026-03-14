---
doc_role: operation
operation_id: 05-FEAT-wallet-topup-via-payos
generated_from: plan.md
status: draft
created_at: 2026-03-09
---

# Prompt - 05-FEAT-wallet-topup-via-payos

## Goal

Add a real wallet top-up flow using the decoupled PayOS provider.

## Required Rules

- top-up is a distinct domain flow
- do not reuse snake-catching payment DTOs
- successful webhook must credit the user's wallet directly
