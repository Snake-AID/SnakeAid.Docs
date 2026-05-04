---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: roadmap
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-investigated
---

# Consultation Instant Booking Cancel Roadmap

## Current Status Snapshot

- module status: `Investigation`
- current instant request endpoint: `Available`
- current expert accept endpoint: `Available`
- current expert reject endpoint: `Available`
- current member/expert history inclusion for accepted instant requests: `Available`
- current member/expert history inclusion for expert-rejected instant requests: `Not implemented`
- proposed direction: `Option 2B locked`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Code-verified state:

- `ConsultationInstantController` creates, accepts, and rejects instant/emergency requests.
- `EmergencyConsultationService.AcceptEmergencyRequestAsync(...)` creates the `Consultation` row.
- `EmergencyConsultationService.RejectEmergencyRequestAsync(...)` does not create a `Consultation` row.
- `ConsultationService.GetMyConsultationsAsync(...)` returns emergency rows only when `ConsultationPingRequest.ConsultationId.HasValue` and `Status == AcceptedByExpert`.
- `ConsultationService.GetExpertConsultationsAsync(...)` has the same accepted-only emergency history filter.
- existing response DTOs use non-null `ConsultationId`, which does not fit rejected request-only rows.

## Scope

In scope:

- instant/emergency request rejection by expert
- member history endpoint impact
- expert history endpoint impact
- response contract for request-only rows
- status and filter semantics for declined instant requests

Out of scope:

- scheduled booking cancellation behavior
- scheduled booking refund policy
- scheduled booking push notification copy
- admin consultation history unless later requested

## Proposed Implementation Checklist

### Phase 1. Decision Lock

- [x] decide whether rejected instant requests should appear in existing consultation history endpoints
- [x] decide response shape for request-only rows
- [x] decide status mapping and filtering behavior

Decision:

- expert-rejected instant/emergency requests will appear in existing member/expert consultation history endpoints
- rejected requests will be request-only rows, not fake consultations
- chosen contract is Option 2B:
  - `consultationId = null`
  - `recordKind = "EmergencyRequest"`
  - `status = "Cancelled"`
  - `requestStatus = "DeclinedByExpert"`
  - `roomId = null`

### Phase 2. Contract Design

- [ ] make `consultationId` nullable in member/expert history response models
- [ ] add `recordKind`
- [ ] add exact request status as `requestStatus`
- [ ] document nullability for `roomId`, `startTime`, and `endTime`

### Phase 3. Service Implementation

- [ ] update `GetMyConsultationsAsync(...)` emergency branch
- [ ] update `GetExpertConsultationsAsync(...)` emergency branch
- [ ] map `DeclinedByExpert` request-only rows without fake consultation ids
- [ ] preserve accepted emergency consultation behavior

### Phase 4. Tests

- [ ] user history includes expert-rejected instant request
- [ ] expert history includes expert-rejected instant request
- [ ] accepted emergency consultation history still maps from linked `Consultation`
- [ ] sorting handles request-only timestamps
- [ ] status filtering behavior matches the chosen contract

### Phase 5. Docs Sync

- [ ] update `consultation-instant-booking-cancel.useguide.md` after the contract is chosen
- [ ] update sourcecode diagrams after implementation
- [ ] close the hallucination decision after user confirmation

## Next Resume Step

Implement Option 2B in the backend:

1. update member/expert history response DTOs for nullable `ConsultationId`, `RecordKind`, and `RequestStatus`
2. update `ConsultationService.GetMyConsultationsAsync(...)` to merge `DeclinedByExpert` request-only rows
3. update `ConsultationService.GetExpertConsultationsAsync(...)` with the same request-only mapping
4. add tests for rejected request rows, accepted emergency preservation, sorting, and `status=Cancelled` filtering

## Change Log

### 2026-05-04

- Created isolated documentation pack for instant/emergency cancellation history.
- Moved instant/emergency history analysis out of `consultation-scheduled-booking-cancel`.
- Recorded current root cause and proposed implementation impact for request-only history rows.
- Expanded H-001 option analysis with a decision matrix, code evidence, Option 2A risks, Option 2B filter behavior, and an explicit Option 1/2B/3 decision path.
- Locked H-001 to Option 2B and updated the next resume step from decision selection to backend implementation.
