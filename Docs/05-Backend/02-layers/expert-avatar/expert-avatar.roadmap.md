---
doc_role: implementation
module: expert-avatar
kind: response-contract-amendment
doc_type: roadmap
status: amendment-planning
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-inspected
---

# Expert Avatar Amendment Roadmap

## Current Status Snapshot

- module status: `Amendment required`
- backend already implemented first avatar pass: `Yes`
- member endpoint current behavior: `Correct`
- expert endpoint current behavior: `Needs replacement`
- docs must be resumed from implemented state: `Yes`

## Current Truth To Resume From

Verified code facts:

- `Account.AvatarUrl` exists.
- `MyConsultationResponse` has `ExpertId`, `ExpertName`, and nullable `ExpertAvatarUrl`.
- `GET /api/users/me/consultations` maps `ExpertAvatarUrl` from the consulted expert account.
- `ExpertConsultationResponse` has `UserId`, `UserName`, and nullable `ExpertAvatarUrl`.
- `GET /api/experts/me/consultations` currently maps `ExpertAvatarUrl` from the authenticated expert account.

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

- [ ] Add nullable `string? UserAvatarUrl` beside `UserName` in `ExpertConsultationResponse`.
- [ ] Remove nullable `string? ExpertAvatarUrl` from `ExpertConsultationResponse`.
- [ ] In scheduled expert-history mapping, set `UserAvatarUrl = b.User?.AvatarUrl`.
- [ ] In emergency expert-history mapping, set `UserAvatarUrl = p.Rescuer?.AvatarUrl`.
- [ ] In orphaned scheduled fallback mapping, set `UserAvatarUrl = c.Caller?.AvatarUrl`.
- [ ] Remove authenticated expert-account avatar lookup when it is only used for expert-history `ExpertAvatarUrl`.
- [ ] Do not change `MyConsultationResponse`.

### Task 2. Amend Tests

Files:

- Modify: `SnakeAid.Tests/Integration/ExpertConsultationPriceResponseTests.cs`
- Leave as-is unless failing: `SnakeAid.Tests/Integration/ConsultationPriceBugConditionTests.cs`

Steps:

- [ ] Add scheduled expert-history assertion for `UserAvatarUrl`.
- [ ] Add emergency expert-history assertion for `UserAvatarUrl`.
- [ ] Keep existing member-history assertions for `ExpertAvatarUrl`.
- [ ] Remove existing expert-history `ExpertAvatarUrl` assertions.
- [ ] Add assertions that `ExpertConsultationResponse` does not expose `ExpertAvatarUrl` if route/contract tests exist.
- [ ] Run focused consultation tests.

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

- [ ] Mark this amendment as implemented only after backend tests pass.
- [ ] Update useguide response example for `GET /api/experts/me/consultations` to include `userAvatarUrl`.
- [ ] Keep member useguide example with `expertAvatarUrl`.
- [ ] Record final code verification commands and results in roadmap changelog.

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

- Added amendment plan from Anh Khoa frontend clarification.
- Recorded that member endpoint implementation is correct.
- Recorded that expert endpoint needs participant avatar via `userAvatarUrl`.
- Updated decision: expert endpoint must remove `expertAvatarUrl` and keep only `userAvatarUrl`.
