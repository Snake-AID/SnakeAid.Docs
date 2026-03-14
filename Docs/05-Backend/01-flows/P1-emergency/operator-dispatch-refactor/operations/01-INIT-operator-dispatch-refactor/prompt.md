---
doc_role: operation
operation_id: 01-INIT-operator-dispatch-refactor
generated_from: plan.md
status: done
created_at: 2026-03-14
---

# Prompt - INIT operator-dispatch-refactor

Create a new backend flow documentation module named `operator-dispatch-refactor` under `P1-emergency`.

Required outputs:

1. Baseline introduction describing why this refactor is documented as a separate flow
2. Baseline sourcecode file reflecting current implemented backend truth
3. Baseline usage guide for current endpoints, hubs, and realtime events
4. Operation folders to record completed and pending refactor slices

Constraints:

- Do not rewrite legacy `rescue-trigger` baseline
- Do not pretend unfinished refactor gaps are complete
- Keep the new module focused on operator-first dispatch
