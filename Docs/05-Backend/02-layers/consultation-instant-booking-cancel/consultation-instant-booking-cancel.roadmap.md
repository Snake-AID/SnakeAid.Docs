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
- proposed direction: `Under decision`

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

- [ ] decide whether rejected instant requests should appear in existing consultation history endpoints
- [ ] decide response shape for request-only rows
- [ ] decide status mapping and filtering behavior

### Phase 2. Contract Design

- [ ] make `consultationId` nullable or introduce a separate response model
- [ ] add a discriminator such as `recordKind`
- [ ] add exact request status such as `requestStatus`
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

Ask the user to choose the exact Option 2 contract shape:

- nullable `consultationId`
- `recordKind`
- unified `status`
- exact `requestStatus`
- filter behavior for `status=Cancelled` and/or `requestStatus=DeclinedByExpert`

## Change Log

### 2026-05-04

- Created isolated documentation pack for instant/emergency cancellation history.
- Moved instant/emergency history analysis out of `consultation-scheduled-booking-cancel`.
- Recorded current root cause and proposed implementation impact for request-only history rows.
