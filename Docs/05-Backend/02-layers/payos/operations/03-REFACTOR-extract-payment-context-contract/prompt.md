---
doc_role: operation
operation_id: 03-REFACTOR-extract-payment-context-contract
generated_from: plan.md
status: draft
created_at: 2026-03-09
---

# Prompt - 03-REFACTOR-extract-payment-context-contract

## Goal

Create a shared payment-context contract so PayOS can serve multiple domains without DTO duplication.

## Target Domains

- snake catching
- wallet top-up
- consultation

## Required Outputs

- shared request/response contracts for provider orchestration
- clear mapping strategy from domain aggregate to payment context

## Forbidden Changes

- do not add one more parallel DTO family per domain if the contract can be normalized
