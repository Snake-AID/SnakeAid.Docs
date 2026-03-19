---
doc_role: baseline
module: payos
kind: layer
status: active
last_updated: 2026-03-19
owners: [backend-team]
---

# PayOS Layer Introduction

## Domain Context

The `payos` layer is the current external payment-gateway integration used by `SnakeAid.Backend`.

In theory, this layer should play the role of a provider integration:
- create checkout links
- receive asynchronous gateway callbacks
- confirm payment status
- cancel payment links

In current code, the PayOS layer does not yet behave like a reusable `PayOsProvider` for multiple business domains. It is materially coupled to the `snake catching` domain and currently exposes snake-catching-specific contracts and behaviors.

## Current Scope

- Create PayOS payment link for snake catching payments.
- Cancel PayOS payment link.
- Confirm PayOS payment manually or by order code.
- Process PayOS webhook callbacks.
- Transfer paid snake-catching funds from system wallet to rescuer wallet.
- Refund from system wallet back to a receiver wallet (implemented internally, not exposed via public API).

## Current Consumers

The current PayOS layer is used by:
- snake catching request payment flow
- snake catching mission settlement / rescuer payout flow

The current PayOS layer is **not yet** a reusable `PayOsProvider` for:
- consultation payment
- wallet top-up
- other future business domains

## Core Invariants

1. The gateway provider is PayOS.
2. Payment records are persisted in the shared `Transaction` table.
3. `ReferenceId` is currently interpreted using snake-catching business semantics.
4. Order-code extraction depends on a string pattern embedded in transaction description.
5. The current integration mixes:
   - provider concerns
   - wallet movement concerns
   - snake-catching business rules

## Architectural Reality

Current implementation should be understood as:
- `PayOsClient + snake-catching-specific PayOsPaymentService`

not as:
- `PayOsClient + PayOsProvider + DomainService`

This distinction matters because future reuse for consultation or wallet top-up would otherwise push the team toward DTO duplication or forced misuse of snake-catching contracts.

## Out of Scope

This layer currently does not provide:
- `PayOsProvider` as a domain-neutral provider façade
- consultation PayOS orchestration
- wallet top-up orchestration

## Why This Module Matters

Payment provider integrations are cross-domain infrastructure. If the provider layer is allowed to stay coupled to one business domain, later domains tend to either:
- duplicate the provider integration
- or corrupt the existing contract by stuffing unrelated domain meaning into old DTOs

The `payos` module therefore needs to be treated as shared backend infrastructure rather than a snake-catching-only convenience wrapper.
