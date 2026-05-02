---
doc_role: implementation
module: expert-avatar
kind: response-contract
doc_type: roadmap
status: implemented
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-verified
---

# Expert Avatar Roadmap

## Current Status Snapshot

- module status: `Implemented`
- backend code changed: `Yes`
- baseline docs initialized: `Yes`
- primary endpoint requested: `GET /api/experts/me/consultations`
- second endpoint in scope: `GET /api/users/me/consultations`
- unresolved business decisions: `No`

## Current Truth To Resume From

Verified code facts:

- `Account.AvatarUrl` exists.
- `MyConsultationResponse` has `ExpertId`, `ExpertName`, and nullable `ExpertAvatarUrl`.
- `ExpertConsultationResponse` has `UserId`, `UserName`, and nullable `ExpertAvatarUrl`.
- user decision on 2026-05-02 limits implementation to `GET /api/experts/me/consultations` and `GET /api/users/me/consultations`.

## Outcome

Implemented result:

1. both in-scope consultation-history endpoints return `expertAvatarUrl`
2. existing endpoint behavior, filters, pagination, and authorization stay unchanged
3. existing response fields remain backward compatible
4. tests protect avatar mapping for scheduled and emergency rows inside both in-scope history endpoints
5. docs reflect active code after the contract change

## Task Plan

### Task 1. Add Member Consultation Avatar Contract

Files:

- Modify: `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- Modify: `SnakeAid.Service/Implements/ConsultationService.cs`
- Test: `SnakeAid.Tests/Integration/Consultation*`

Steps:

- [x] Add nullable `string? ExpertAvatarUrl` beside `ExpertName` in `MyConsultationResponse`.
- [x] Update scheduled user consultation mapping in `BuildMyConsultationResponse(...)`.
- [x] Update emergency user consultation mapping in `GetMyConsultationsAsync(...)`.
- [x] Map from `Account.AvatarUrl` through `booking.Expert`, `consultation.Callee`, or `ping.Expert`.
- [x] Add or update tests proving `ExpertAvatarUrl` is populated for scheduled member consultation rows.
- [x] Add or update tests proving `ExpertAvatarUrl` is populated for emergency member consultation rows.
- [x] Sync `expert-avatar.useguide.md` with active response examples.

### Task 2. Add Expert Consultation Avatar Contract

Files:

- Modify: `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- Modify: `SnakeAid.Service/Implements/ConsultationService.cs`
- Test: expert consultation history tests

Steps:

- [x] Add nullable `string? ExpertAvatarUrl` to `ExpertConsultationResponse`.
- [x] Map `ExpertAvatarUrl` from the authenticated expert account for scheduled expert-history rows.
- [x] Map `ExpertAvatarUrl` from the authenticated expert account for emergency expert-history rows.
- [x] Map `ExpertAvatarUrl` for orphaned scheduled fallback rows if still emitted.
- [x] Avoid adding fields that are not part of the two endpoint contracts.
- [x] Add tests proving `ExpertAvatarUrl` is populated for scheduled expert consultation rows.
- [x] Add tests proving `ExpertAvatarUrl` is populated for emergency expert consultation rows.
- [x] Sync `expert-avatar.useguide.md` with active response examples.

### Task 3. Final Documentation Sync

Files:

- Modify: `expert-avatar.introduction.md`
- Modify: `expert-avatar.roadmap.md`
- Modify: `expert-avatar.hallucination.md`
- Modify: `expert-avatar.sourcecode.md`
- Modify: `expert-avatar.useguide.md`

Steps:

- [x] Update `Current Truth To Resume From` after code lands.
- [x] Mark completed roadmap checkboxes.
- [x] Keep resolved hallucination risks as decision records.
- [x] Update class and sequence diagrams to match final code.
- [x] Update all API response examples to match actual JSON fields.

## Verification Strategy

Run focused tests first:

```powershell
dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "FullyQualifiedName~Consultation"
```

Run broader tests after focused pass:

```powershell
dotnet test
```

Manual API verification targets:

- `GET /api/experts/me/consultations`
- `GET /api/users/me/consultations`

## Resume Checklist

- [ ] Read `expert-avatar.hallucination.md` first.
- [ ] Confirm scope remains limited to the two in-scope endpoints before coding.
- [ ] Use graph tools before grep for code discovery.
- [ ] Write or update tests before changing mappings.
- [ ] Update baseline docs after every backend contract change.
- [ ] Keep useguide frontend/mobile focused.

## Change Log

### 2026-05-03

- Implemented nullable `ExpertAvatarUrl` on member and expert consultation-history responses.
- Added focused integration coverage for scheduled and emergency avatar mapping.
- Removed context not directly related to the two requested endpoints.
- Kept roadmap focused on `GET /api/experts/me/consultations` and `GET /api/users/me/consultations`.

### 2026-05-02

- Created baseline planning docs.
- Confirmed `Account.AvatarUrl` exists.
- Identified the two consultation-history DTOs missing expert avatar.
- Logged ambiguity around `GET /api/experts/me/consultations`.
- Adjusted scope by user decision: only `GET /api/experts/me/consultations` and `GET /api/users/me/consultations` remain in scope.
