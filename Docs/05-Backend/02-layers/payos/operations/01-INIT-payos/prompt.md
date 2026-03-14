---
doc_role: operation
operation_id: 01-INIT-payos
generated_from: plan.md
status: done
created_at: 2026-03-09
---

# Prompt - 01-INIT-payos

## Goal

Create the initial documentation baseline for the `payos` layer.

## Required Outputs

1. `payos.introduction.md`
2. `payos.sourcecode.md`
3. `payos.usageguide.md`

## Required Content

- Describe current PayOS implementation as it exists in code
- state clearly that current orchestration is coupled to snake catching
- do not describe a future generic provider architecture as current truth

## Forbidden Changes

- do not modify backend code
- do not rewrite snake-catching flow behavior
- do not introduce future design into baseline docs
