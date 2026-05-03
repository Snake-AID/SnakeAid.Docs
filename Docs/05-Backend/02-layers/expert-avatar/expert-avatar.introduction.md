---
doc_role: implementation
module: expert-avatar
kind: response-contract-amendment
doc_type: introduction
status: implemented
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-verified
---

# Expert Avatar Introduction

## Goal

Frontend needs avatar data on the two consultation-history screens:

- member screen: `GET /api/users/me/consultations`
- expert screen: `GET /api/experts/me/consultations`

This document records the implemented amendment over the first backend avatar pass.

## Frontend Clarification

Decision from Anh Khoa on 2026-05-03:

- `GET /api/users/me/consultations` should keep returning the consulted expert avatar.
- `GET /api/experts/me/consultations` should expose the avatar of the other user shown to the expert.
- The expert does not need to fetch their own avatar from `GET /api/experts/me/consultations`.

## Current Implemented Backend State

Code-verified on 2026-05-03:

- `MyConsultationResponse` already has nullable `ExpertAvatarUrl`.
- `GET /api/users/me/consultations` already maps expert avatar from the consulted expert account.
- `ExpertConsultationResponse` has nullable `UserAvatarUrl`.
- `GET /api/experts/me/consultations` maps `UserAvatarUrl` from the other user account.
- `ExpertConsultationResponse` no longer exposes `ExpertAvatarUrl`.

Important code locations:

- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Tests/Integration/ConsultationPriceBugConditionTests.cs`
- `SnakeAid.Tests/Integration/ExpertConsultationPriceResponseTests.cs`

## Implemented Amendment

Keep implemented member endpoint behavior:

- `GET /api/users/me/consultations`
- response keeps `expertAvatarUrl`
- value means consulted expert avatar

Amended expert endpoint behavior:

- `GET /api/experts/me/consultations`
- add nullable `userAvatarUrl`
- value means other participant avatar:
  - scheduled consultation: member account avatar
  - emergency consultation: emergency requester account avatar
- code note: emergency requester is currently stored on `ConsultationPingRequest.Rescuer`.
- removes `expertAvatarUrl` from this response because expert screen does not need the authenticated expert's own avatar

## Implementation Result

Completed as a contract correction over the implemented backend:

1. Added nullable `UserAvatarUrl` to `ExpertConsultationResponse`.
2. Mapped scheduled expert-history `UserAvatarUrl` from `ConsultationBooking.User.AvatarUrl` or fallback `Consultation.Caller.AvatarUrl`.
3. Mapped emergency expert-history `UserAvatarUrl` from the emergency requester account via `ConsultationPingRequest.Rescuer.AvatarUrl`.
4. Removed `ExpertAvatarUrl` from `ExpertConsultationResponse` and its mappings/tests.
5. Kept `MyConsultationResponse.ExpertAvatarUrl` unchanged.
6. Updated tests to assert the other user's avatar on expert history.
7. Updated useguide examples after code changes.

## Non-Goals

- Do not rewrite existing `GET /api/users/me/consultations` avatar implementation.
- Do not add avatar fields to endpoints outside the two consultation-history endpoints.
- Do not change avatar upload/storage behavior.

## Resume Rule

When resuming this module, treat the amendment as implemented. Do not restore the old expert self-avatar behavior.
