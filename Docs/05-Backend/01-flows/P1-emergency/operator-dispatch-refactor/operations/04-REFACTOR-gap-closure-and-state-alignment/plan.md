---
doc_role: operation
operation_id: 04-REFACTOR-gap-closure-and-state-alignment
type: REFACTOR
status: partial
created_at: 2026-03-14
affects:
  - SnakeAid.Service/Implements/SnakebiteIncidentService.cs
  - SnakeAid.Api/Hubs/RescuerHub.cs
  - SnakeAid.Api/Controllers/SnakebiteIncidentController.cs
  - SnakeAid.Api/Services/SignalROperatorRealtimeNotificationService.cs
---

# Plan - REFACTOR gap closure and state alignment

## 1. As-Is

- Core operator dispatch exists
- Realtime and shift snapshots exist
- Sprint work has already landed part of the gap-closure slice
- Several target behaviors still remain incomplete

## 2. Gap Analysis

- redispatch endpoint absent
- operator disconnect recovery absent
- pending escalation background worker absent
- current state naming still diverges from target design
- false alarm and no-answer paths exist, but the broader recovery/state-alignment story is still incomplete
- rescuer who already `Declined` a dispatch can still be reconsidered unless exclusion policy is made explicit
- rescuer who already aborted a mission can still appear as a candidate unless redispatch filtering is made explicit
- operator web UI does not yet have a guaranteed server-backed rule for excluding prior `abort` / `deny` rescuers from candidate selection

## 3. To-Be Design

- close parity gaps against the refactor target state
- normalize state language and transition semantics
- harden operational recovery paths

Sprint trace from commit analysis:

- `0c478d61aa8cfbb9c3eb76a809ff793450b71590`
- `e410b263e9b10224d7f39c9914093396fc01e5ea`
- `be5f4af9fb40cc8b156ccadca29445ec53765a7c`
- `db1d8f603cd538e39354074b36d5d7952cf293f0`
- `792526d90e78b3da0c5f16bbf2148d95f8f3620f`

## 4. Impacted Components

- incident command APIs
- operator realtime lifecycle
- background services
- state transition rules

## 5. Risks & Constraints

- production behavior already depends on `Verified` and `Assigned`
- renaming or remapping states may impact clients and docs together
- recovery automation must not accidentally steal active incidents
- excluding prior rescuers too aggressively may block legitimate manual reassignment when operator intentionally wants to retry
- filtering only in web UI is insufficient; the server must enforce the same redispatch rule
- abort and deny need distinct business semantics if the team wants different retry behavior later

## 6. Validation Plan

- verify false alarm and no-answer endpoints remain wired to service + realtime notifications
- verify false alarm end-to-end
- verify redispatch flow
- verify operator disconnect release semantics
- verify escalation behavior for stale pending incidents
- verify a rescuer who already declined the current incident cannot be redispatched unintentionally
- verify a rescuer who aborted the current incident cannot be redispatched unintentionally
- verify operator candidate snapshot or dispatch validation excludes the above rescuers consistently
- verify web UI candidate selection and backend dispatch validation enforce the same rule

## 7. Implementation Prep

Recommended implementation slices:

1. Define the redispatch exclusion rule
   - decide whether `Declined` and `MissionAborted` both mean "exclude rescuer for this incident"
   - decide whether operator override is allowed

2. Enforce the rule in backend dispatch validation
   - reject dispatch when the rescuer has already declined the same incident
   - reject dispatch when the rescuer has already aborted the same incident

3. Surface the rule in candidate queries
   - exclude prior `Declined` rescuers from snapshot / candidate lists
   - exclude prior aborted rescuers from snapshot / candidate lists

4. Sync operator UI behavior
   - make dropdown / map candidate list reflect the same backend exclusion rule
   - show clear reason why a rescuer is not selectable

5. Add regression coverage
   - decline then redispatch same rescuer
   - abort then redispatch same rescuer
   - operator tries to bypass UI and call dispatch API directly
