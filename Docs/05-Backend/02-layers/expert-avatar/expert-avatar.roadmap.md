---
doc_role: implementation
module: expert-avatar
kind: response-contract-amendment
doc_type: roadmap
status: implemented
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-verified
---

# Expert Avatar Amendment Roadmap

## Current Status Snapshot

- module status: `Implemented`
- backend already implemented first avatar pass: `Yes`
- member endpoint current behavior: `Correct`
- expert endpoint current behavior: `Corrected`
- docs must be resumed from implemented state: `Yes`

## Current Truth To Resume From

Verified code facts:

- `Account.AvatarUrl` exists.
- `MyConsultationResponse` has `ExpertId`, `ExpertName`, and nullable `ExpertAvatarUrl`.
- `GET /api/users/me/consultations` maps `ExpertAvatarUrl` from the consulted expert account.
- `ExpertConsultationResponse` has `UserId`, `UserName`, and nullable `UserAvatarUrl`.
- `GET /api/experts/me/consultations` maps `UserAvatarUrl` from the member/rescuer participant account.
- `GET /api/experts/me/consultations` no longer returns `ExpertAvatarUrl`.

Frontend decision from Anh Khoa:

- expert screen needs avatar of the other participant, not the expert's own avatar.
- member screen behavior remains correct: member history needs expert avatar.

## Amendment Outcome

After this amendment:

1. `GET /api/users/me/consultations` still returns `expertAvatarUrl` for the consulted expert.
2. `GET /api/experts/me/consultations` returns `userAvatarUrl` for the member/rescuer participant.
3. `GET /api/experts/me/consultations` no longer returns `expertAvatarUrl`.
4. Tests prove scheduled and emergency expert-history rows map `userAvatarUrl` from the participant account.
5. Docs describe current implemented behavior plus this correction, so agents remove the old expert self-avatar behavior deliberately.

## Task Plan

### Task 1. Add Expert-History Participant Avatar

Files:

- Modify: `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- Modify: `SnakeAid.Service/Implements/ConsultationService.cs`
- Test: `SnakeAid.Tests/Integration/ExpertConsultationPriceResponseTests.cs`

Steps:

- [x] Add nullable `string? UserAvatarUrl` beside `UserName` in `ExpertConsultationResponse`.
- [x] Remove nullable `string? ExpertAvatarUrl` from `ExpertConsultationResponse`.
- [x] In scheduled expert-history mapping, set `UserAvatarUrl = b.User?.AvatarUrl`.
- [x] In emergency expert-history mapping, set `UserAvatarUrl = p.Rescuer?.AvatarUrl`.
- [x] In orphaned scheduled fallback mapping, set `UserAvatarUrl = c.Caller?.AvatarUrl`.
- [x] Remove authenticated expert-account avatar lookup when it is only used for expert-history `ExpertAvatarUrl`.
- [x] Do not change `MyConsultationResponse`.

### Task 2. Amend Tests

Files:

- Modify: `SnakeAid.Tests/Integration/ExpertConsultationPriceResponseTests.cs`
- Leave as-is unless failing: `SnakeAid.Tests/Integration/ConsultationPriceBugConditionTests.cs`

Steps:

- [x] Add scheduled expert-history assertion for `UserAvatarUrl`.
- [x] Add emergency expert-history assertion for `UserAvatarUrl`.
- [x] Keep existing member-history assertions for `ExpertAvatarUrl`.
- [x] Remove existing expert-history `ExpertAvatarUrl` assertions.
- [x] Add assertions that `ExpertConsultationResponse` does not expose `ExpertAvatarUrl` if route/contract tests exist.
- [x] Run focused consultation tests.

Focused command:

```powershell
dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "FullyQualifiedName~ExpertConsultationPriceResponseTests|FullyQualifiedName~ConsultationPriceBugConditionTests"
```

### Task 3. Sync Baseline Docs After Code Change

Files:

- Modify: `expert-avatar.introduction.md`
- Modify: `expert-avatar.roadmap.md`
- Modify: `expert-avatar.hallucination.md`
- Modify: `expert-avatar.sourcecode.md`
- Modify: `expert-avatar.useguide.md`

Steps:

- [x] Mark this amendment as implemented only after backend tests pass.
- [x] Update useguide response example for `GET /api/experts/me/consultations` to include `userAvatarUrl`.
- [x] Keep member useguide example with `expertAvatarUrl`.
- [x] Record final code verification commands and results in roadmap changelog.

## Manual API Verification Targets

- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

## Resume Checklist

- [ ] Treat current expert-history `ExpertAvatarUrl` implementation as code to replace, not preserve.
- [ ] Add `UserAvatarUrl` for expert history and remove expert-history `ExpertAvatarUrl`.
- [ ] Keep docs written as an amendment until code is patched.
- [ ] Update docs immediately after code changes.

## Changelog

### 2026-05-03

- Implemented amendment in backend.
- Verification: `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "FullyQualifiedName~ExpertConsultationPriceResponseTests|FullyQualifiedName~ConsultationPriceBugConditionTests" --no-restore` passed 5 tests.
- Verification: `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "FullyQualifiedName~Consultation" --no-restore` passed 105 tests.
- Added amendment plan from Anh Khoa frontend clarification.
- Recorded that member endpoint implementation is correct.
- Recorded that expert endpoint needs participant avatar via `userAvatarUrl`.
- Updated decision: expert endpoint must remove `expertAvatarUrl` and keep only `userAvatarUrl`.
