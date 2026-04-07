---
doc_role: operation
module: myprofile
kind: layer
doc_type: plan
status: active
last_updated: 2026-04-07
owners: [backend-team]
---

# MyProfile Implementation Plan

## Task

Implement authenticated self-profile endpoints for three client-facing roles:

- `GET /api/members/me/profile`
- `PUT /api/members/me/profile`
- `GET /api/experts/me/profile`
- `PUT /api/experts/me/profile`
- `GET /api/rescuers/me/profile`
- `PUT /api/rescuers/me/profile`

The endpoint family supports the mobile/frontend "edit my personal profile" use case. The usage guide must be written only after code is implemented and verified.

## Current Codebase State

- `Account` stores shared identity fields: `FullName`, `AvatarUrl`, `PhoneNumber`, `Email`, `Role`, active/reputation metadata.
- `MemberProfile` stores member-specific editable fields: `EmergencyContacts`, `HasUnderlyingDisease`; rating fields are system-managed.
- `ExpertProfile` stores expert-specific editable business/profile fields: `Biography`, `ConsultationFee`, `EmergencyConsultationFee`; rating and online status are system-managed or handled by other flows.
- `RescuerProfile` stores rescuer-specific operational fields: `Type`, `IsOnline`, `IsAvailable`, location, mission stats, ratings. For this task, only identity fields are self-editable; operational fields remain read-only in this endpoint.
- `ExpertController` already exposes `PUT /api/experts/me/settings`, but that endpoint only updates expert settings and does not cover shared identity profile fields.
- `RescuerController` is currently restricted at controller level to `Operator,Admin`, so adding a rescuer self endpoint there would require care. A dedicated profile controller avoids breaking existing operator endpoints.
- Service registration uses Scrutor for `*Service` classes as implemented interfaces.
- Response envelope convention uses `ApiResponseBuilder.BuildSuccessResponse(...)`.

## Implementation Strategy

- Add dedicated request/response DTOs under `SnakeAid.Core/Requests/MyProfile` and `SnakeAid.Core/Responses/MyProfile`.
- Add `IMyProfileService` and `MyProfileService`.
- Use `UserManager<Account>` for identity account updates and `IUnitOfWork<SnakeAidDbContext>` for role profile updates in one service boundary.
- Keep role-specific profile validation in service methods:
  - Member: update `FullName`, `PhoneNumber`, `AvatarUrl`, `EmergencyContacts`, `HasUnderlyingDisease`.
  - Expert: update `FullName`, `PhoneNumber`, `AvatarUrl`, `Biography`, `ScheduledConsultationFee`, `EmergencyConsultationFee`.
  - Rescuer: update `FullName`, `PhoneNumber`, `AvatarUrl`; return rescuer operational fields as read-only.
- Add one dedicated `MyProfileController` with method-level routes:
  - `api/members/me/profile`, role `User`.
  - `api/experts/me/profile`, role `Expert`.
  - `api/rescuers/me/profile`, role `Rescuer`.
- Write tests before implementation for service update behavior and role/profile-not-found guardrails.
- Fill `myprofile.usageguide.md` only after implementation and verification.

## Tasklist

- [x] Read usageguide rules and current code patterns.
- [x] Inspect profile domains, existing expert/rescuer controllers, service registration, response envelopes.
- [x] Add red tests for profile service behavior.
- [x] Add request/response DTOs.
- [x] Add `IMyProfileService`.
- [x] Add `MyProfileService`.
- [x] Add consolidated `MyProfileController` for member/expert/rescuer self profile routes.
- [x] Run targeted tests and fix failures.
- [x] Update `myprofile.sourcemap.md` with final file-level map.
- [x] Write `myprofile.usageguide.md` from verified endpoints only.
- [x] Run final verification.

## Verification Plan

- Run targeted test class for myprofile behavior.
- Run project build if targeted tests pass.
- Re-read usage guide checklist before finalizing docs.
