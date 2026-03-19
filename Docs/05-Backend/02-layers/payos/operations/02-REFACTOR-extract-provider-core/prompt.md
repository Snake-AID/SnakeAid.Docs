---
doc_role: operation
operation_id: 02-REFACTOR-extract-provider-core
generated_from: plan.md
status: done
created_at: 2026-03-09
---

# Prompt - 02-REFACTOR-extract-provider-core

## Goal

Refactor the PayOS module so a reusable `PayOsProvider` is separated from snake-catching business orchestration.

## Required Outputs

- `IPayOsProvider` / `PayOsProvider`
- PayOS-specific but domain-neutral provider DTOs
- updated baseline docs after implementation

## Required Rules

- keep webhook verification and payment-link transport behavior working
- move snake-catching validations out of `PayOsProvider`
- do not silently preserve snake-catching naming inside new provider contracts

## Forbidden Changes

- do not duplicate the current PayOS orchestration for new domains
- do not keep `SnakeCatchingRequestId` inside `PayOsProvider` request/response contracts
