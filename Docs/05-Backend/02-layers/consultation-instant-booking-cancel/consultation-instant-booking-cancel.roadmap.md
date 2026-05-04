---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: roadmap
status: current
last_updated: 2026-05-05
owners: [backend-team]
verification_status: code-investigated
---

# Consultation Instant Booking Cancel Roadmap

## Current Status Snapshot

- module status: `Contract decision locked, implementation pending`
- current instant request endpoint: `Available`
- current expert accept endpoint: `Available`
- current expert reject endpoint: `Available`
- current member/expert history inclusion for accepted instant requests: `Available`
- current member/expert history inclusion for expert-rejected instant requests: `Not implemented`
- current member/expert history inclusion for expired instant requests: `Not implemented`
- selected direction: `Union history contract with kind = consultation | instant`
- remaining open decision: `Status filter mapping for kind = instant`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Code-verified state:

- `ConsultationInstantController` creates, accepts, and rejects instant/emergency requests.
- `EmergencyConsultationService.AcceptEmergencyRequestAsync(...)` creates the `Consultation` row.
- `EmergencyConsultationService.RejectEmergencyRequestAsync(...)` does not create a `Consultation` row.
- `ConsultationPaymentService.ExpireEmergencyRequestsAsync(...)` expires pending instant/emergency requests without creating a `Consultation` row.
- `ConsultationService.GetMyConsultationsAsync(...)` returns emergency rows only when `ConsultationPingRequest.ConsultationId.HasValue` and `Status == AcceptedByExpert`.
- `ConsultationService.GetExpertConsultationsAsync(...)` has the same accepted-only emergency history filter.
- `RescuerCancelled` exists in `ConsultationPingStatus` but no production flow currently sets it.

Decision-verified state:

- Do not create Fake `Consultation` rows for rejected/expired instant requests.
- Do not fake `consultationId`.
- Use a union response contract.
- `kind = consultation` represents real `Consultation` rows.
- `kind = instant` represents terminal request-level rows without `Consultation`.
- `kind = instant` is a separate DTO with flat fields.
- `kind = instant` currently covers `DeclinedByExpert` and `Expired`.

## Scope

In scope:

- instant/emergency request rejection by expert
- instant/emergency request expiration
- member history endpoint impact
- expert history endpoint impact
- union response contract for request-level rows
- status and filter semantics for instant request rows

Out of scope:

- scheduled booking cancellation behavior
- scheduled booking refund policy
- scheduled booking push notification copy
- admin consultation history implementation until explicitly picked up

## Implementation Checklist

### Phase 1. Decision Lock

- [x] choose union API contract instead of Fake `Consultation`
- [x] define discriminator values: `consultation`, `instant`
- [x] define `kind = instant` as separate DTO
- [x] define member history flat actor fields
- [x] define expert history flat actor fields
- [x] verify terminal request statuses without `Consultation`
- [ ] close status filter mapping decision

### Phase 2. Contract Design

- [x] define `kind = instant` id field: `instantRequestId`
- [x] define `kind = instant` status field: `requestStatus`
- [x] define `kind = instant` timestamp fields: `requestedAt`, `respondedAt`
- [x] exclude `consultationId`, `roomId`, `startTime`, `endTime`, `rescueMissionId`, `expiresAt`, and price fields from `kind = instant`
- [x] document that `DeclinedByExpert` and `Expired` are request-level terminal history statuses
- [ ] define final status filter mapping for mixed `kind = consultation | instant` rows

### Phase 3. Service Implementation

- [ ] introduce/adjust member history response model for union rows
- [ ] introduce/adjust expert history response model for union rows
- [ ] update `GetMyConsultationsAsync(...)` to query `DeclinedByExpert` and `Expired` pings for `kind = instant`
- [ ] update `GetExpertConsultationsAsync(...)` to query `DeclinedByExpert` and `Expired` pings for `kind = instant`
- [ ] preserve accepted emergency consultation behavior as `kind = consultation`
- [ ] sort `kind = instant` rows by `respondedAt`
- [ ] implement status filter behavior after H-002 is closed

### Phase 4. Tests

- [ ] member history includes `DeclinedByExpert` instant request as `kind = instant`
- [ ] expert history includes `DeclinedByExpert` instant request as `kind = instant`
- [ ] member history includes `Expired` instant request as `kind = instant`
- [ ] expert history includes `Expired` instant request as `kind = instant`
- [ ] accepted emergency consultation history still maps from linked `Consultation` as `kind = consultation`
- [ ] `kind = instant` rows do not expose consultation-scoped action ids
- [ ] sorting behavior uses `respondedAt` for terminal request rows
- [ ] status filtering behavior matches the selected H-002 contract

### Phase 5. Docs Sync

- [x] close H-001 strategy/DTO decisions in hallucination doc
- [x] keep H-002 filter decision open in hallucination doc
- [x] update introduction with locked union contract
- [x] update roadmap with selected direction and implementation checklist
- [x] update sourcecode doc with planned union flow
- [x] update useguide with planned frontend/mobile contract

## Next Resume Step

Close H-002 by deciding how the `status` query filter should apply to `kind = instant` rows, then implement the union response DTOs and member/expert history query changes.

## Change Log

### 2026-05-05

- Locked H-001 to union response contract with `kind = consultation | instant`.
- Documented `kind = instant` as a separate flat DTO.
- Added `Expired` alongside `DeclinedByExpert` as a request-level terminal history row.
- Verified `RescuerCancelled` is enum-only in production code and is not currently covered.
- Kept status filter mapping open as H-002.
- Synced the baseline pack to the selected contract decision.

### 2026-05-04

- Created isolated documentation pack for instant/emergency cancellation history.
- Moved instant/emergency history analysis out of `consultation-scheduled-booking-cancel`.
- Recorded current root cause and proposed implementation impact for request-only history rows.
- Reframed H-001 around three user-defined approaches: split contract with mobile two-screen work, contract-preserving response merge, and Fake `Consultation` creation.
- Reset the roadmap from a locked implementation path back to decision selection.
