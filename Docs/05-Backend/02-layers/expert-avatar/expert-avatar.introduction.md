---
doc_role: implementation
module: expert-avatar
kind: response-contract
doc_type: introduction
status: implemented
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-verified
---

# Expert Avatar Introduction

## Goal

Mobile and frontend screens that display expert information should receive the expert avatar from backend responses without doing extra profile lookups.

Implemented target:

- add avatar data to `GET /api/experts/me/consultations`
- add avatar data to `GET /api/users/me/consultations`

Implemented scope:

- add avatar data only to the two consultation-history endpoints:
  - `GET /api/experts/me/consultations`
  - `GET /api/users/me/consultations`

## Current Codebase State

Verified from backend code on 2026-05-03:

- `Account` already stores `AvatarUrl`.
- consultation history response DTOs now expose nullable `ExpertAvatarUrl` for the two in-scope endpoints.

Important verified response classes:

- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`

Important verified services:

- `ConsultationService.GetExpertConsultationsAsync(Guid expertId, MyConsultationsQueryRequest query)`
- `ConsultationService.GetMyConsultationsAsync(Guid userId, MyConsultationsQueryRequest query)`
- `ConsultationService.BuildMyConsultationResponse(Consultation consultation, ConsultationBooking? booking = null)`

## Verified Endpoint Surfaces

- `GET /api/experts/me/consultations`
- `GET /api/users/me/consultations`

## Implementation Direction

Implemented contract shape:

- nullable `ExpertAvatarUrl` exists on `MyConsultationResponse`
- nullable `ExpertAvatarUrl` exists on `ExpertConsultationResponse`
- mappings read from `Account.AvatarUrl`
- existing response fields remain unchanged for backward compatibility
- avatar nullability remains explicit: `null` means account has no avatar or the relation is not loaded/available

Implemented verification:

- `ConsultationPriceBugConditionTests` covers member scheduled and emergency avatar mapping.
- `ExpertConsultationPriceResponseTests` covers expert scheduled and emergency avatar mapping.

## Non-Goals

- Do not change avatar upload/storage behavior.
- Do not introduce new media endpoints.
- Do not remove existing name/id fields.

## Current Risk Summary

Resolved scope decision:

- user decision on 2026-05-02: only add avatar to the two get-consultation endpoints for expert and member.
- `GET /api/experts/me/consultations` should add `expertAvatarUrl` for the authenticated expert.
- `GET /api/users/me/consultations` should add `expertAvatarUrl` for the consulted expert.

See `expert-avatar.hallucination.md`.
