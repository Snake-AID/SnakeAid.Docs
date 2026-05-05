---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: roadmap
status: implemented
last_updated: 2026-05-06
owners: [backend-team]
verification_status: verified
---

# Consultation Instant Booking Cancel Roadmap

## Current Status Snapshot

- module status: `Implemented and verified`
- current instant request endpoint: `Available`
- current expert accept endpoint: `Available`
- current expert reject endpoint: `Available`
- current member/expert history inclusion for accepted instant requests: `Available`
- current member/expert history inclusion for expert-rejected instant requests: `Available as kind = instant`
- current member/expert history inclusion for expired instant requests: `Available as kind = instant`
- selected direction: `Union history contract with kind = consultation | instant`
- status filter decision: `status filters consultation rows only; instant rows appear only when status is omitted`
- DTO boundary decision: `History usecase has its own typed union DTOs; expert absent keeps MyConsultationResponse`
- admin endpoint compatibility with union format: `Not compatible (consultation-centric AdminConsultationResponse)`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Code-verified state:

- `ConsultationInstantController` creates, accepts, and rejects instant/emergency requests.
- `EmergencyConsultationService.AcceptEmergencyRequestAsync(...)` creates the `Consultation` row.
- `EmergencyConsultationService.RejectEmergencyRequestAsync(...)` does not create a `Consultation` row.
- `ConsultationPaymentService.ExpireEmergencyRequestsAsync(...)` expires pending instant/emergency requests without creating a `Consultation` row.
- `ConsultationService.GetMyConsultationsAsync(...)` returns `PagingResponse<MyConsultationHistoryUnionResponse>`.
- `ConsultationService.GetExpertConsultationsAsync(...)` returns `PagingResponse<ExpertConsultationHistoryUnionResponse>`.
- `ReportExpertAbsentAsync(...)` must remain `Task<MyConsultationResponse>` and must not inherit or reuse history DTOs.
- `RescuerCancelled` exists in `ConsultationPingStatus` but no production flow currently sets it.

Decision-verified state:

- Do not create Fake `Consultation` rows for rejected/expired instant requests.
- Do not fake `consultationId`.
- Use a union response contract.
- `kind = consultation` represents real `Consultation` rows.
- `kind = instant` represents terminal request-level rows without `Consultation`.
- `kind = instant` is a separate DTO with flat fields.
- `kind = instant` currently covers `DeclinedByExpert` and `Expired`.
- Do not use `object` or `dynamic` for public history response contracts.
- Do not use `JsonIgnore(WhenWritingNull)` to hide non-applicable fields in reused DTOs.
- Do not add instant request fields to `MyConsultationResponse` or `ExpertConsultationResponse`.
- History DTOs live under `SnakeAid.Core/Responses/Consultation/History/`.

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
- full admin union-contract redesign until explicitly picked up

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

- [x] add `Responses/Consultation/History/MyConsultationHistoryUnionResponse.cs`
- [x] add `Responses/Consultation/History/MyConsultationHistoryResponse.cs`
- [x] add `Responses/Consultation/History/MyInstantConsultationRequestHistoryResponse.cs`
- [x] add `Responses/Consultation/History/ExpertConsultationHistoryUnionResponse.cs`
- [x] add `Responses/Consultation/History/ExpertConsultationHistoryResponse.cs`
- [x] add `Responses/Consultation/History/ExpertInstantConsultationRequestHistoryResponse.cs`
- [x] keep `MyConsultationResponse` and `ExpertConsultationResponse` free of `kind`, instant fields, and `JsonIgnore` changes
- [x] change `GetMyConsultationsAsync(...)` to `PagingResponse<MyConsultationHistoryUnionResponse>`
- [x] change `GetExpertConsultationsAsync(...)` to `PagingResponse<ExpertConsultationHistoryUnionResponse>`
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
- [x] serialization test proves derived history DTO properties appear at runtime
- [x] expert absent tests prove `MyConsultationResponse` contract remains unchanged

### Phase 5. Docs Sync

- [x] close H-001 strategy/DTO decisions in hallucination doc
- [x] close H-002 filter decision in hallucination doc
- [x] update introduction with selected union contract
- [x] update roadmap with selected direction and implementation checklist
- [x] update sourcecode doc with target union flow
- [x] update useguide with target frontend/mobile contract

### Phase 6. Admin Compatibility Assessment (requested 2026-05-06)

- [x] verify route contracts for `GET /api/admin/consultations` and `GET /api/admin/consultations/{consultationId}`
- [x] verify service return types and DTO shape for admin consultation APIs
- [x] verify emergency query predicates for request-level terminal statuses
- [x] verify behavior against admin unit/integration tests
- [x] update all baseline docs with compatibility result and follow-up decision points

Evidence summary:

- Controller contract: `ApiResponse<PagingResponse<AdminConsultationResponse>>` and `ApiResponse<AdminConsultationResponse>`.
- Admin history query includes emergency requests only with `ConsultationId.HasValue` and `Status == AcceptedByExpert`.
- Terminal request-only rows (`DeclinedByExpert`, `Expired` without linked consultation) are not represented as independent union rows.
- Verification: `AdminConsultationsControllerTests` + `AdminConsultationHistoryIntegrationTests` passed.

## Next Resume Step

Next resume step: decide whether admin endpoints should remain consultation-centric or adopt a typed union contract for terminal request-only rows, then implement/update tests and admin docs accordingly.

## Change Log

### 2026-05-06

- Per request, checked compatibility of:
  - `GET /api/admin/consultations`
  - `GET /api/admin/consultations/{consultationId}`
- Confirmed both endpoints are not compatible with member/expert history union format (`kind = consultation | instant`).
- Recorded evidence from controller signatures, service query predicates, and DTO shape.
- Re-verified admin behavior by tests:
  - `dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "AdminConsultationsControllerTests|AdminConsultationHistoryIntegrationTests"` (17 passed)
- Opened follow-up decision point for potential admin union alignment.

### 2026-05-05

- Synced baseline docs to code-verified state after implementation completion (removed stale `target/paused` wording from runtime contract narrative).
- Re-verified implementation after sync:
  - `rtk dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter ConsultationInstantHistoryIntegrationTests` (5 passed)
  - `rtk dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "ConsultationPriceBugConditionTests|ConsultationPricePreservationTests|ExpertConsultationPriceResponseTests|ConsultationExpertAbsentIntegrationTests|ConsultationPropertyTests"` (31 passed)
- Noted current dependency warnings from test run:
  - `MailKit 3.2.0` advisory `GHSA-9j88-vvj5-vhgr`
  - `MimeKit 3.2.0` advisories `GHSA-g7hc-96xr-gvvx`, `GHSA-gmc6-fwg3-75m5`

- Implemented typed member/expert history union DTOs under `SnakeAid.Core/Responses/Consultation/History/`.
- Updated member/expert history service contracts and API action response types.
- Added terminal `DeclinedByExpert` and `Expired` instant request rows for member/expert history when `status` is omitted.
- Preserved accepted emergency requests as `kind = consultation`.
- Added focused integration tests for instant history, status filter behavior, serialization, and expert-absent DTO separation.
- Verification passed:
  - `dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter ConsultationInstantHistoryIntegrationTests`
  - `dotnet test SnakeAid.Tests\SnakeAid.Tests.csproj --no-restore --filter "ConsultationPriceBugConditionTests|ConsultationPricePreservationTests|ExpertConsultationPriceResponseTests|ConsultationExpertAbsentIntegrationTests|ConsultationPropertyTests"`

- Updated DTO design decision after review: history gets its own typed union DTOs, expert absent keeps `MyConsultationResponse`.
- Rejected `JsonIgnore(WhenWritingNull)` as a contract-shaping mechanism.
- Rejected `object`/`dynamic` for public history response contracts.
- Chosen DTO folder: `SnakeAid.Core/Responses/Consultation/History/`.
- Chosen DTO names:
  - `MyConsultationHistoryUnionResponse`
  - `MyConsultationHistoryResponse`
  - `MyInstantConsultationRequestHistoryResponse`
  - `ExpertConsultationHistoryUnionResponse`
  - `ExpertConsultationHistoryResponse`
  - `ExpertInstantConsultationRequestHistoryResponse`

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
