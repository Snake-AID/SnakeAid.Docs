---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: roadmap
status: implemented
last_updated: 2026-05-05
owners: [backend-team]
verification_status: code-and-tests-verified
---

# Consultation Instant Booking Cancel Roadmap

## Current Status Snapshot

- module status: `Implemented for member/expert history`
- current instant request endpoint: `Available`
- current expert accept endpoint: `Available`
- current expert reject endpoint: `Available`
- current member/expert history inclusion for accepted instant requests: `Available`
- current member/expert history inclusion for expert-rejected instant requests: `Available as kind = instant`
- current member/expert history inclusion for expired instant requests: `Available as kind = instant`
- selected direction: `Union history contract with kind = consultation | instant`
- status filter decision: `status filters consultation rows only; instant rows appear only when status is omitted`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Code-verified state:

- `ConsultationInstantController` creates, accepts, and rejects instant/emergency requests.
- `EmergencyConsultationService.AcceptEmergencyRequestAsync(...)` creates the `Consultation` row.
- `EmergencyConsultationService.RejectEmergencyRequestAsync(...)` does not create a `Consultation` row.
- `ConsultationPaymentService.ExpireEmergencyRequestsAsync(...)` expires pending instant/emergency requests without creating a `Consultation` row.
- `ConsultationService.GetMyConsultationsAsync(...)` returns accepted emergency rows as `kind = consultation` and terminal `DeclinedByExpert`/`Expired` pings without `ConsultationId` as `kind = instant` when `status` is omitted.
- `ConsultationService.GetExpertConsultationsAsync(...)` has the same implemented union behavior.
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
- [x] close status filter mapping decision

### Phase 2. Contract Design

- [x] define `kind = instant` id field: `instantRequestId`
- [x] define `kind = instant` status field: `requestStatus`
- [x] define `kind = instant` timestamp fields: `requestedAt`, `respondedAt`
- [x] exclude `consultationId`, `roomId`, `startTime`, `endTime`, `rescueMissionId`, `expiresAt`, and price fields from `kind = instant`
- [x] document that `DeclinedByExpert` and `Expired` are request-level terminal history statuses
- [x] define final status filter mapping for mixed `kind = consultation | instant` rows

### Phase 3. Service Implementation

- [x] introduce/adjust member history response model for union rows
- [x] introduce/adjust expert history response model for union rows
- [x] update `GetMyConsultationsAsync(...)` to query `DeclinedByExpert` and `Expired` pings for `kind = instant`
- [x] update `GetExpertConsultationsAsync(...)` to query `DeclinedByExpert` and `Expired` pings for `kind = instant`
- [x] preserve accepted emergency consultation behavior as `kind = consultation`
- [x] sort `kind = instant` rows by `respondedAt`
- [x] implement status filter behavior after H-002 is closed

### Phase 4. Tests

- [x] member history includes `DeclinedByExpert` instant request as `kind = instant`
- [x] expert history includes `DeclinedByExpert` instant request as `kind = instant`
- [x] member history includes `Expired` instant request as `kind = instant`
- [x] expert history includes `Expired` instant request as `kind = instant`
- [x] accepted emergency consultation history still maps from linked `Consultation` as `kind = consultation`
- [x] `kind = instant` rows do not expose consultation-scoped action ids
- [x] sorting behavior uses `respondedAt` for terminal request rows
- [x] status filtering behavior matches the selected H-002 contract

### Phase 5. Docs Sync

- [x] close H-001 strategy/DTO decisions in hallucination doc
- [x] close H-002 filter decision in hallucination doc
- [x] update introduction with implemented union contract
- [x] update roadmap with selected direction and implementation checklist
- [x] update sourcecode doc with implemented union flow
- [x] update useguide with implemented frontend/mobile contract

## Next Resume Step

Next resume step: decide whether a future `requestStatus` query parameter is needed for frontend/mobile filtering of request-level instant rows, especially before extending the same union model to admin history.

## Change Log

### 2026-05-05

- Implemented member/expert emergency history union rows for terminal instant requests.
- Added `kind`, `instantRequestId`, `requestStatus`, `requestedAt`, and `respondedAt` to member/expert history response models.
- Made consultation-scoped fields nullable and JSON-ignored when null so `kind = instant` rows do not expose fake consultation identifiers.
- Implemented `DeclinedByExpert` and `Expired` request-level rows for `GET /api/users/me/consultations` and `GET /api/experts/me/consultations`.
- Closed H-002 with conservative filter behavior: `status` filters consultation rows only; instant rows appear when `status` is omitted.
- Added integration coverage in `ConsultationInstantHistoryIntegrationTests`.
- Verified with `dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter ConsultationInstantHistoryIntegrationTests`.

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
