---
doc_role: operation
operation_id: 02-REFACTOR-core-operator-dispatch
generated_from: plan.md
status: done
created_at: 2026-03-14
---

# Prompt - REFACTOR core operator dispatch

Refactor the emergency rescue flow from automatic rescuer pairing into an operator-centered dispatch model.

Required outcomes:

1. Operator claims incidents
2. Operator confirms incidents
3. Operator dispatches a chosen rescuer
4. Rescuer accepts or declines
5. Mission opens only after acceptance

Constraints:

- Keep emergency API and mission flow connected
- Preserve decline path for re-dispatch
- Avoid coupling legacy session-based semantics into the new operator flow
