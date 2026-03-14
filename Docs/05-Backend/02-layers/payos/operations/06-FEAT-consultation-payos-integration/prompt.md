---
doc_role: operation
operation_id: 06-FEAT-consultation-payos-integration
generated_from: plan.md
status: draft
created_at: 2026-03-09
---

# Prompt - 06-FEAT-consultation-payos-integration

## Goal

Add consultation support on top of the decoupled PayOS provider without duplicating the old snake-catching-specific PayOS domain.

## Required Rules

- consultation state machine remains authoritative
- external provider success must feed consultation escrow flow
- do not clone `CreateSnakeCatchingPaymentRequest` / `SnakeCatchingPaymentResponse` into consultation equivalents at provider layer
