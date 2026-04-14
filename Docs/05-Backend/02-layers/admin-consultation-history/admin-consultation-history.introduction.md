---
doc_role: planning
module: admin-consultation-history
kind: layer
doc_type: introduction
status: implemented
last_updated: 2026-04-14
last_updated: 2026-04-15
owners: [backend-team]
verification_status: code-verified
---

# Admin Consultation History Introduction

## Goal

This document defines the implemented backend scope for the admin consultation history module.

Business goal:
- Admin can browse consultations across the whole system
- Admin can open one consultation and inspect the same normalized data shape in more depth
- Admin can work with both scheduled and emergency consultations without switching API contracts

## Current Codebase Status

The codebase already has two actor-specific consultation history APIs:

- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

The admin module now adds a dedicated admin surface:

- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`
- `AdminConsultationsController`
- `IConsultationService.GetAllConsultationsForAdminAsync(...)`
- `IConsultationService.GetConsultationByIdForAdminAsync(...)`
- one shared admin DTO: `AdminConsultationResponse`
- Mapster registration in `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`

## Implemented Contract Shape

The admin module uses one normalized response model:

- `AdminConsultationResponse`

This shape is shared by:

- the list endpoint, wrapped in `PagingResponse`
- the single-item endpoint, returned directly in `ApiResponse<T>`

This means:

- `GetAll` and `GetOne` expose the same consultation fields
- the only payload difference is collection vs single item
- booking and emergency-request metadata are available in both endpoints when related data exists

## Code-Verified Data Sources

Admin consultation data is assembled from these sources:

1. `Consultation`
   - canonical source for consultation identity, type, status, room, start/end time
2. `ConsultationBooking`
   - scheduled consultation enrichment
   - source for price, problem description, booking metadata, slot metadata
3. `ConsultationPingRequest`
   - emergency consultation enrichment
   - source for emergency request metadata
4. `Transaction`
   - source for emergency pricing resolution

## Implemented Behavior

The current implementation has two retrieval modes, but one unified contract:

### List

- Scheduled and emergency consultations are queried separately
- Results are merged in memory
- Sort order is `StartTime desc`
- Pagination is applied after merge

### Single Item

- Lookup starts from `Consultation`
- Enrichment then branches by `Consultation.Type`
- Scheduled consultations are enriched from `ConsultationBooking`
- Emergency consultations are enriched from `ConsultationPingRequest`
- Emergency price uses the same resolution rule as the list endpoint

## Mapping Implementation

Admin consultation response mapping is now split between:

- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
  - base mapping from `Consultation`
  - enrichment mapping from `ConsultationBooking`
  - enrichment mapping from `ConsultationPingRequest`
- `ConsultationService`
  - source selection
  - orphan fallback handling
  - emergency price resolution
  - final protection of consultation-owned semantics such as consultation `Status`

## Important Mapping Rules

### Scheduled Consultation

- base fields come from `Consultation`
- booking fields come from `ConsultationBooking`
- slot fields come from `ExpertTimeSlot`
- `Price` comes from `ConsultationBooking.Price`

### Emergency Consultation

- base fields come from `Consultation`
- request fields come from `ConsultationPingRequest`
- `Price` is resolved from `Transaction`
  - prefer `TransactionType = ConsultationPayment` by `ConsultationPingRequest.Id`
  - fallback `TransactionType = ExpertPayout` by `Consultation.Id`

### Orphan Consultation Handling

The module still returns consultations even when the expected business-side record is missing:

- orphan scheduled consultation:
  - consultation exists but booking is missing
  - booking fields are `null`
- orphan emergency consultation:
  - consultation exists but ping request is missing
  - emergency-request fields are `null`

## Design Direction Now Closed

The codebase originally considered separating admin list and admin detail into different response shapes.

That is no longer the chosen direction.

The implemented contract now standardizes on:

- one shared DTO for admin consultation payloads
- one list endpoint
- one single-item endpoint
- consistent field names across both entry points

## Scope Boundary

In scope:

- admin list consultations across the system
- admin get one consultation by `consultationId`
- pagination
- filters by `status` and `type`
- normalized scheduled + emergency response contract

Out of scope:

- admin update consultation state
- export
- free-text search
- analytics / aggregates

## Delivered Artifacts

- `admin-consultation-history.introduction.md`
- `admin-consultation-history.roadmap.md`
- `admin-consultation-history.sourcecode.md`
- `admin-consultation-history.useguide.md`
- backend implementation
- service and controller test coverage
