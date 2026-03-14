---
doc_role: operation
operation_id: 03-REFACTOR-operator-realtime-and-duty-snapshot
generated_from: plan.md
status: done
created_at: 2026-03-14
---

# Prompt - REFACTOR operator realtime and duty snapshot

Make operator dispatch operationally usable by adding:

1. realtime visibility into new incidents and rescuer presence
2. dispatch feedback events for operator dashboards
3. snapshot APIs for on-duty rescuers
4. shift management APIs to support schedule-aware dispatch

Constraints:

- preserve current emergency mission path
- keep operator-facing data consumable by web dashboard clients
- do not assume a dedicated `OperatorHub` exists yet
