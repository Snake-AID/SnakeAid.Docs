---
doc_role: implementation
module: expert-avatar
kind: response-contract-amendment
doc_type: introduction
status: amendment-planning
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-inspected
---

# Expert Avatar Introduction

## Goal

Frontend needs avatar data on the two consultation-history screens:

- member screen: `GET /api/users/me/consultations`
- expert screen: `GET /api/experts/me/consultations`

This document is an amendment over the currently implemented backend contract. Do not treat it as a fresh rewrite from the original state.

## Frontend Clarification

Decision from Anh Khoa on 2026-05-03:

- `GET /api/users/me/consultations` should keep returning the consulted expert avatar.
- `GET /api/experts/me/consultations` should expose the member/rescuer avatar, because expert screens need to show the other participant.
- The expert does not need to fetch their own avatar from `GET /api/experts/me/consultations`.

## Current Implemented Backend State

Code-inspected on 2026-05-03:

- `MyConsultationResponse` already has nullable `ExpertAvatarUrl`.
- `GET /api/users/me/consultations` already maps expert avatar from the consulted expert account.
- `ExpertConsultationResponse` already has nullable `ExpertAvatarUrl`.
- `GET /api/experts/me/consultations` currently maps `ExpertAvatarUrl` from the authenticated expert account.

Important code locations:

- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Tests/Integration/ConsultationPriceBugConditionTests.cs`
- `SnakeAid.Tests/Integration/ExpertConsultationPriceResponseTests.cs`

## Required Amendment

Keep implemented member endpoint behavior:

- `GET /api/users/me/consultations`
- response keeps `expertAvatarUrl`
- value means consulted expert avatar

Amend expert endpoint behavior:

- `GET /api/experts/me/consultations`
- add nullable `userAvatarUrl`
- value means other participant avatar:
  - scheduled consultation: member account avatar
  - emergency consultation: rescuer account avatar
- remove `expertAvatarUrl` from this response because expert screen does not need the authenticated expert's own avatar

## Implementation Direction

Do this as a contract correction over the implemented backend:

1. Add nullable `UserAvatarUrl` to `ExpertConsultationResponse`.
2. Map scheduled expert-history `UserAvatarUrl` from `ConsultationBooking.User.AvatarUrl` or fallback `Consultation.Caller.AvatarUrl`.
3. Map emergency expert-history `UserAvatarUrl` from `ConsultationPingRequest.Rescuer.AvatarUrl`.
4. Remove `ExpertAvatarUrl` from `ExpertConsultationResponse` and its mappings/tests.
5. Keep `MyConsultationResponse.ExpertAvatarUrl` unchanged.
6. Update tests to assert member/rescuer avatar on expert history.
7. Update useguide examples after code changes.

## Non-Goals

- Do not rewrite existing `GET /api/users/me/consultations` avatar implementation.
- Do not add avatar fields to endpoints outside the two consultation-history endpoints.
- Do not change avatar upload/storage behavior.

## Resume Rule

When resuming this task, start from the current implemented backend state, then apply the amendment. Do not repeat the old implementation plan as if no avatar work has landed.
