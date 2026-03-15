---
doc_role: operation
operation_id: 04-REFACTOR-gap-closure-and-state-alignment
generated_from: plan.md
status: partial
created_at: 2026-03-14
---

# Prompt - REFACTOR gap closure and state alignment

Close the remaining production gaps of the operator dispatch refactor.

Historical note:

This operation already has partial sprint implementation captured in `commit-analysis-operator-dispatch.md`, especially around false alarm, no-answer, notification expansion, and dashboard actions.

Required focus:

1. consolidate the already-landed false alarm / no-answer slice
2. redispatch flow
3. operator disconnect release logic
4. stale pending escalation
5. state naming and transition alignment
6. exclusion policy for rescuers who already declined or aborted the same incident

Constraints:

- preserve existing working dispatch behavior while closing gaps
- treat current code as partially migrated, not greenfield
- update baseline docs after each completed mutation
- do not rely on frontend-only filtering for rescuer exclusion
- backend dispatch validation and operator candidate selection must agree

Implementation note:

For the current priority, treat these edge cases as part of this operation:

- rescuer already declined an incident should not receive the same dispatch again unless an explicit override rule is designed
- rescuer already aborted the incident should not be selectable again by operator UI unless an explicit override rule is designed
