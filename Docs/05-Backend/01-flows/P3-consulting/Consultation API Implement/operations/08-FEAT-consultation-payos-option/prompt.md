---
doc_role: operation
operation_id: 08-FEAT-consultation-payos-option
generated_from: plan.md
status: draft
created_at: 2026-03-21
---

# Prompt: Document PayOS as a Consultation Payment Option

## Goal

Write documentation that explains PayOS for consultation with this framing:

- from the user perspective, PayOS is an additional payment option beside the system wallet
- from the backend perspective, PayOS is a new execution branch that must still converge into the same consultation escrow lifecycle

## Required Messaging

The documentation must say clearly:

1. users choose between wallet and PayOS
2. consultation business flow stays the same
3. payment method changes how money is collected
4. payment method does not change refund or settlement rules

## Required Technical Clarity

The documentation must distinguish:

- wallet path = internal immediate escrow
- PayOS path = external payment intent then confirmed escrow

## Forbidden Framing

1. Do not describe PayOS as a separate consultation flow.
2. Do not imply that consultation already uses PayOS today.
3. Do not imply that webhook logic should own consultation business transitions directly.
4. Do not collapse payment option design into only enum-level changes.
