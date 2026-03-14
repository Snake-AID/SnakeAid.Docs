---
doc_role: operation
operation_id: 04-REFACTOR-gap-closure-and-state-alignment
generated_from: plan.md
status: draft
created_at: 2026-03-14
---

# Prompt - REFACTOR gap closure and state alignment

Close the remaining production gaps of the operator dispatch refactor.

Required focus:

1. false alarm flow
2. redispatch flow
3. operator disconnect release logic
4. stale pending escalation
5. state naming and transition alignment

Constraints:

- preserve existing working dispatch behavior while closing gaps
- treat current code as partially migrated, not greenfield
- update baseline docs after each completed mutation
