---
doc_role: planning
module: consultation-expert-absent
kind: decision-log
doc_type: hallucination
status: partially-resolved
last_updated: 2026-04-21
owners: [backend-team]
verification_status: mixed
---

# Consultation Expert Absent Hallucination Log

This file captures items that required product confirmation and the remaining design area that still needs caution.

## Confirmed Decision 1: Canonical Field Name

Confirmed by product direction:

- use `Customer Report` as the canonical field name in baseline docs

Practical interpretation for implementation:

- business concept is still member-authored report text
- docs should stop mixing `Customer Report` and `Member Report`
- implementation should keep one consistent property name across persistence and DTOs where possible

Status:

- [x] Confirmed

## Confirmed Decision 2: Persistence Location

Confirmed by product direction:

- store the new field on `Consultation`

Why this is acceptable:

- admin list/detail already read consultation-centric data
- consultation status already carries `ExpertAbsent`
- avoids duplicating the same business fact into `ConsultationBooking`

Status:

- [x] Confirmed

## Confirmed Decision 3: Status Transition Rule

Confirmed by product direction:

- report submission should also set `Consultation.Status = ExpertAbsent`

Impact:

- admin can filter by `ExpertAbsent` immediately using the existing status model
- the report action becomes a visible state transition, not just hidden note storage

Status:

- [x] Confirmed

## Confirmed Decision 4: Eligibility Window

Confirmed by product direction:

- member may report any time after `Consultation.StartTime`

Implementation note:

- this is broader than a slot-window-only rule
- backend should still reject impossible ownership/state cases
- docs should reflect that the lower time boundary is `StartTime`

Status:

- [x] Confirmed

## Confirmed Decision 5: Idempotency

Confirmed by product direction:

- duplicate report submission should be rejected

Why this is useful:

- prevents silent overwrites
- keeps the first accepted report as the authoritative one unless a later edit flow is explicitly introduced

Status:

- [x] Confirmed

## Confirmed Decision 6: API Response Contract

Confirmed by product direction:

- the report endpoint should return an updated consultation object

Recommended implementation interpretation:

- return the updated member-facing consultation DTO or a dedicated updated consultation response
- response should at minimum include:
  - `consultationId`
  - `status`
  - `customerReport`

Status:

- [x] Confirmed

## Decision 7: Audit Metadata Needs Deeper Design

Current product direction:

- metadata may be needed
- but there is concern about adding too many new fields into `Consultation`

This part should not be over-simplified.

### What metadata is actually useful

Potential metadata candidates:

- `CustomerReportSubmittedAt`
- `CustomerReportSubmittedBy`
- `ExpertAbsentResolvedAt`
- `ExpertAbsentResolvedBy`
- `ExpertAbsentResolutionNote`

### Complexity analysis

If these are all added directly into `Consultation`, the entity starts carrying:

- consultation lifecycle state
- room lifecycle timestamps
- absent-report business note
- audit timestamps
- future admin resolution fields

That is acceptable only if the absent-report workflow remains small and consultation-owned.

The downside is clear:

- `Consultation` becomes broader and harder to reason about
- each new field increases mapper/test/doc surface
- future resolution workflows may introduce asymmetric state that does not belong to the main consultation aggregate root cleanly

### Practical design options

Option A: Minimal metadata on `Consultation`

- add only:
  - `CustomerReport`
  - `CustomerReportSubmittedAt`
- do not add resolver fields yet

Pros:

- low migration cost
- enough to know what was reported and when
- keeps current implementation simple

Cons:

- admin resolution history is still missing

Option B: Medium metadata on `Consultation`

- add:
  - `CustomerReport`
  - `CustomerReportSubmittedAt`
  - `CustomerReportSubmittedBy`

Pros:

- better audit clarity
- still moderate complexity

Cons:

- `SubmittedBy` is partly redundant today because caller/member already exists as `CallerId`
- extra field may not add much value unless future delegated reporting is allowed

Option C: Full workflow fields on `Consultation`

- add report fields and admin resolution fields directly on `Consultation`

Pros:

- single-table read for the whole story

Cons:

- highest coupling
- grows the entity fastest
- least flexible if resolution later becomes multi-step or comment-based

Option D: Keep `Consultation` minimal, move future audit trail to a separate entity later

- v1 adds only:
  - `CustomerReport`
  - maybe `CustomerReportSubmittedAt`
- v2 introduces a separate entity if workflow expands

Pros:

- best balance for the current scope
- avoids overfitting the entity too early
- preserves upgrade path for richer admin handling later

Cons:

- if v2 arrives quickly, a second migration will still be needed

### Recommended path

Recommended baseline:

- confirm `CustomerReport` on `Consultation`
- add `CustomerReportSubmittedAt` in v1
- do not add `SubmittedBy` now unless reporting may be done by someone other than `CallerId`
- do not add admin resolution fields into `Consultation` yet

Why this is the best trade-off:

- `CustomerReport` alone is too weak for auditability
- `SubmittedAt` gives strong business value with low entity bloat
- `SubmittedBy` is redundant in the current model because the reporting actor is constrained to the member/caller
- resolution fields belong to a future admin workflow, which is explicitly out of current scope

### Baseline direction after this review

Closed decisions:

- [x] OD1 `Customer Report`
- [x] OD2 store on `Consultation`
- [x] OD3 set `Status = ExpertAbsent`
- [x] OD4 allow after `StartTime`
- [x] OD5 reject duplicates
- [x] OD6 return updated consultation object

Partially open design area:

- [ ] OD7 final v1 metadata shape

Current recommended v1 metadata shape:

- `CustomerReport`
- `CustomerReportSubmittedAt`
