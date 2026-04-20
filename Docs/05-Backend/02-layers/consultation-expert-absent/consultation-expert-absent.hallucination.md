---
doc_role: planning
module: consultation-expert-absent
kind: decision-log
doc_type: hallucination
status: closed
last_updated: 2026-04-21
owners: [backend-team]
verification_status: mixed
---

# Consultation Expert Absent Hallucination Log

This file captures the confirmed product decisions for the module.

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

## Confirmed Decision 7: Audit Metadata

Confirmed by product direction:

- v1 metadata should include:
  - `CustomerReport`
  - `CustomerReportSubmittedAt`

Implementation direction:

- store both fields on `Consultation`
- do not add `CustomerReportSubmittedBy` in v1
- do not add admin resolution fields in this module

### Baseline direction after this review

Closed decisions:

- [x] OD1 `Customer Report`
- [x] OD2 store on `Consultation`
- [x] OD3 set `Status = ExpertAbsent`
- [x] OD4 allow after `StartTime`
- [x] OD5 reject duplicates
- [x] OD6 return updated consultation object
- [x] OD7 use `CustomerReport` and `CustomerReportSubmittedAt`
