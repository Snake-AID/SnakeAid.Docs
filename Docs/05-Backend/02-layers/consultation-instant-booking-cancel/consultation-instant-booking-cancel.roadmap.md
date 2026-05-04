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
- proposed direction: `Three approaches under decision`

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

## Proposed Decision And Implementation Checklist

### Phase 1. Decision Lock

- [ ] choose Approach 1: split the API contract and require mobile to build two history screens
- [ ] choose Approach 2: keep the old API contract and force `ConsultationPingRequest` rows into consultation history
- [ ] choose Approach 3: keep the old API contract by creating a Fake `Consultation` when the expert rejects
- [ ] document the selected response behavior for member history
- [ ] document the selected response behavior for expert history

Decision options:

- Approach 1 changes the contract so history can explicitly separate `Consultation` and `ConsultationPingRequest` rows, and mobile must build two history screens or sections.
- Approach 2 keeps the current contract shape and forces rejected `ConsultationPingRequest` records into it without creating `Consultation` rows.
- Approach 3 keeps the current contract shape by creating a Fake cancelled `Consultation` when the expert rejects.

### Phase 2. Contract Design

- [ ] if Approach 1 is selected, define the mixed row contract and discriminator
- [ ] if Approach 2 is selected, define how the existing fields represent request-only rows
- [ ] if Approach 3 is selected, define the Fake `Consultation` status, room id, timestamps, and guards
- [ ] update frontend/mobile contract notes after the selected approach is locked

### Phase 3. Service Implementation

- [ ] update `GetMyConsultationsAsync(...)` according to the selected approach
- [ ] update `GetExpertConsultationsAsync(...)` according to the selected approach
- [ ] include expert-rejected instant/emergency requests in the selected representation
- [ ] preserve accepted emergency consultation behavior

### Phase 4. Tests

- [ ] user history includes expert-rejected instant request
- [ ] expert history includes expert-rejected instant request
- [ ] accepted emergency consultation history still maps from linked `Consultation`
- [ ] sorting behavior matches the selected representation
- [ ] status filtering behavior matches the selected contract
- [ ] consultation-scoped actions are protected from request-only rows or Fake `Consultation` rows

### Phase 5. Docs Sync

- [ ] update `consultation-instant-booking-cancel.useguide.md` after one approach is chosen
- [ ] update sourcecode diagrams after implementation approach is locked
- [ ] close H-001 in `consultation-instant-booking-cancel.hallucination.md` after user confirmation

## Next Resume Step

Ask the user to choose one of the three H-001 approaches:

1. split the API contract to support both `Consultation` and `ConsultationPingRequest`, and require mobile to build two history screens
2. keep the API contract and force rejected `ConsultationPingRequest` rows into history
3. keep the API contract and create a Fake cancelled `Consultation` when the expert rejects

## Change Log

### 2026-05-04

- Created isolated documentation pack for instant/emergency cancellation history.
- Moved instant/emergency history analysis out of `consultation-scheduled-booking-cancel`.
- Recorded current root cause and proposed implementation impact for request-only history rows.
- Reframed H-001 around three user-defined approaches: split contract with mobile two-screen work, contract-preserving response merge, and Fake `Consultation` creation.
- Reset the roadmap from a locked implementation path back to decision selection.
