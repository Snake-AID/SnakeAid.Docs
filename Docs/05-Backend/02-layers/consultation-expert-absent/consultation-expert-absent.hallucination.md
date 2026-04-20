---
doc_role: planning
module: consultation-expert-absent
kind: layer
doc_type: hallucination
status: open
last_updated: 2026-04-20
owners: [backend-team]
verification_status: decision-buckets-open
---
# Consultation Expert Absent Hallucination Buckets

## Purpose

This file captures decision points that are ambiguous in the business request or have multiple valid implementation options.

Rule:

- do not silently auto-decide these items during implementation
- confirm decisions here first
- after decision, update roadmap + useguide + sourcecode together

## Bucket A - Canonical Naming (`CustomerReport` vs `MemberReport`)

### Code-verified fact

- there is no existing report field in current consultation entities/DTOs.
- requirement text mentions both:
  - `Customer Report` (Task1, Task3)
  - `Member Report` (Task2)

### Why this is risky

- inconsistent naming can leak into DB, DTO, and API contract with avoidable mismatch.

### Candidate options

1. Canonical everywhere = `customerReport`
2. Canonical everywhere = `memberReport`
3. Input = `memberReport`, persisted + output = `customerReport`

### Default recommendation

- option 3 for this iteration, because it fits the task wording directly while keeping admin-visible field as `CustomerReport`.

### User decision required

- confirm the canonical naming strategy before code is implemented.

## Bucket B - Report Endpoint Route and Method

### Code-verified fact

- no existing absent-report endpoint in `ConsultationsController`.

### Candidate options

1. `POST /api/consultations/{consultationId}/expert-absence-report`
2. `POST /api/consultations/{consultationId}/report-expert-absent`
3. `PATCH /api/consultations/{consultationId}/customer-report`

### Default recommendation

- option 1 (`POST /expert-absence-report`) because it is explicit, action-oriented, and easy for mobile teams.

### User decision required

- confirm route style and HTTP method.

## Bucket C - Status Side Effect On Report Submission

### Code-verified fact

- current lifecycle marks elapsed consultations as `Completed`.
- `ExpertAbsent` enum exists but is currently unused.

### Candidate options

1. report only stores text; status unchanged
2. report sets `Consultation.Status = ExpertAbsent` immediately
3. report creates pending-review state (requires extra state model, not currently available)

### Default recommendation

- option 1 for initial release (low-risk): store report first, let admin review process decide status transitions.

### User decision required

- decide whether status should change on report submission in this phase.

## Bucket D - Report Write Policy

### Candidate options

1. one-time write only (cannot edit)
2. update allowed until consultation is completed/cancelled
3. unlimited overwrite (last write wins)

### Default recommendation

- option 2: allow update only during active report window, lock afterward.

### User decision required

- confirm write policy and lock condition.

## Bucket E - Applicable Consultation Types

### Code-verified fact

- consultation domain supports both `Scheduled` and `Emergency`.

### Candidate options

1. scheduled only
2. emergency only
3. both scheduled and emergency

### Default recommendation

- option 1 (scheduled only) for initial rollout unless business confirms emergency no-show reporting is also required.

### User decision required

- confirm type scope.

## Bucket F - Report Time Window

### Candidate options

1. allow only from `StartTime` until `StartTime + X minutes`
2. allow any time before consultation reaches terminal state
3. allow always

### Default recommendation

- option 1 with configurable `X` (example: 30 minutes) to align with operational relevance.

### User decision required

- confirm report window policy.

## Bucket G - Admin Processing Scope In This Task Set

### Code-verified fact

- requested tasks explicitly include admin visibility update, not admin action endpoints.

### Candidate options

1. visibility only (add field in list/detail)
2. visibility + admin action endpoint (`approve/reject absent report`)

### Default recommendation

- option 1 now; keep moderation actions for next task group.

### User decision required

- confirm whether action endpoints are out of scope.

## Bucket H - Validation Constraints For Report Text

### Candidate options

1. required, min 10, max 2000
2. required, min 1, max 1000
3. optional, max 2000

### Default recommendation

- required, max 2000, with trimmed non-empty input.

### User decision required

- confirm exact constraints.

## Decision Log Template

Use this section to record final decisions before coding:

- Naming strategy: `TBD`
- Endpoint route/method: `TBD`
- Status side effect: `TBD`
- Write policy: `TBD`
- Consultation type scope: `TBD`
- Time window: `TBD`
- Admin scope: `TBD`
- Validation constraints: `TBD`

## Change Log

### 2026-04-20

- Created hallucination buckets for ConsultaionExpertAbsent planning.
- Flagged naming conflict (`CustomerReport` vs `MemberReport`) as the highest-priority decision.
