---
doc_role: planning
module: consultation-expert-absent
kind: layer
doc_type: introduction
status: draft
last_updated: 2026-04-20
owners: [backend-team]
verification_status: current-state-code-verified-planned-delta-pending-implementation
---

# Consultation Expert Absent Introduction

## Goal

This module plans the implementation for the ConsultaionExpertAbsent task group:

- ConsultaionAbsent Task1: add `CustomerReport` field so a member can report an absent expert
- ConsultaionAbsent Task2: build a member API to submit the absent-expert report (`Member Report` input)
- ConsultaionAbsent Task3: update admin consultation endpoints to include `CustomerReport`

Business objective:

- when consultation time starts and the member joins the video room but the expert does not join, the member needs a backend-supported way to report the absence so admin can process expert no-show handling.

## Resume Summary

This document is designed so implementation can resume without prior chat context.

Current code-verified situation:

1. Consultation domain already defines absence-related statuses (`UserAbsent`, `ExpertAbsent`, `AllAbsent`) in enum, but no active backend flow sets them.
2. There is currently no `CustomerReport` or `MemberReport` field in `Consultation`, `ConsultationBooking`, `MyConsultationResponse`, or `AdminConsultationResponse`.
3. There is currently no member endpoint to report expert absence.
4. Admin consultation endpoints exist (`GET /api/admin/consultations`, `GET /api/admin/consultations/{consultationId}`), but response does not contain absent-report text.
5. Lifecycle auto-complete currently marks consultations as `Completed` and settles payments; no no-show dispute/report branch exists.

## Code-Verified Current State (As-Is)

### Consultation entity and status

- `Consultation` has core fields: caller/callee, room, start/end, status, type.
- `ConsultationStatus` includes:
  - `Scheduled`
  - `Ongoing`
  - `Completed`
  - `Cancelled`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`

Current behavior:

- services actively use `Scheduled`, `Ongoing`, `Completed`, `Cancelled`
- no active write path currently sets `UserAbsent`, `ExpertAbsent`, or `AllAbsent`

### Member consultation surface

Current active endpoints relevant to this scope:

- `POST /api/consultations/{consultationId}/video-token`
- `GET /api/users/me/consultations`
- `POST /api/consultations/{consultationId}/end`

Current gap:

- member can join/get token and can end consultation, but cannot report expert absence.

### Admin consultation surface

Current active endpoints:

- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`

Current response shape (`AdminConsultationResponse`) includes booking/emergency metadata but does not include any customer/member absent-report field.

### Lifecycle behavior

Current sweep behavior:

- scheduled elapsed consultations: auto-complete to `Completed`
- emergency elapsed consultations: auto-complete to `Completed`
- no current branch to auto-mark absent status based on member report or participation evidence

## Planned Direction (To-Be)

### Task1 - Add report field

Planned baseline direction:

- add nullable report storage on `Consultation` as `CustomerReport` (text)
- expose report back to member consultation history response (`GET /api/users/me/consultations`)

### Task2 - Build member report API

Planned API direction:

- add new member endpoint for absent-expert reporting
- endpoint validates consultation ownership and writable status window
- endpoint writes member-submitted report text into `CustomerReport`

### Task3 - Include report in admin endpoint

Planned admin direction:

- add `CustomerReport` to `AdminConsultationResponse`
- ensure both admin list and detail return the field

## Scope

In scope:

- domain/model field changes for report storage
- member report API contract and validation rules
- admin DTO/projection update
- migration + tests + docs update

Out of scope for this module iteration:

- automatic expert-presence detection from LiveKit participant webhooks
- admin enforcement workflow (penalty, suspension, scoring) beyond data visibility
- attachment/media upload for absence evidence

## Key Risks

1. Naming inconsistency in requirements (`CustomerReport` vs `Member Report`) can produce API confusion.
2. If report write window is too broad, users may report long after consultation ended, reducing operational quality.
3. If report endpoint also mutates status without strict rules, billing/reconciliation side effects can regress existing payment logic.
4. Existing integration tests around consultation history can break if DTO changes are not propagated consistently.

## Initial Mitigation Strategy

- lock naming and contract decisions first in `consultation-expert-absent.hallucination.md`
- keep first implementation focused on report capture + visibility, avoid hidden payment behavior changes
- add explicit test coverage for:
  - member can submit report
  - non-owner cannot submit report
  - admin list/detail include report text
  - existing consultation history fields remain stable

## Deliverables In This Planning Set

- `consultation-expert-absent.introduction.md`
- `consultation-expert-absent.roadmap.md`
- `consultation-expert-absent.hallucination.md`
- `consultation-expert-absent.sourcecode.md`
- `consultation-expert-absent.useguide.md`
