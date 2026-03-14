---
doc_role: operation
operation_id: 04-REFACTOR-migrate-snake-catching-to-provider
generated_from: plan.md
status: draft
created_at: 2026-03-09
---

# Prompt - 04-REFACTOR-migrate-snake-catching-to-provider

## Goal

Rewire snake-catching payment flow to consume the decoupled PayOS provider/core abstraction.

## Required Rules

- keep current snake-catching business behavior stable
- keep payout/refund semantics intact
- remove direct dependency on snake-catching-specific provider contracts
