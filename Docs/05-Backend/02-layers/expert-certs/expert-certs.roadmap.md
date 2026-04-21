---
module: expert-certs
last_updated: 2026-04-21
status: planning
---

# Expert Certificates Roadmap

## Current Status Snapshot

- domain entity `ExpertCertificate`: available
- persistence table and `DbSet`: available
- certificate CRUD service: not started
- expert certificate controller: not started
- admin certificate controller: not started
- certificate DTO contracts: not started
- `ExpertProfile.IsVerified` persistence: not started
- certificate media reference enum: not started
- route contract tests: not started
- integration tests: not started
- baseline docs in `expert-certs`: initialized

## Verified Current State

- `ExpertCertificate` is present in the backend domain and database context
- no active API endpoints exist for certificate CRUD
- no dedicated certificate service exists
- no dedicated certificate DTO set exists
- no verification recalculation logic exists
- current public expert directory response returns `IsVerified = false`

## Milestones

### M1. Data Model Alignment

- [ ] Add certificate-specific `MediaReferenceType` enum member
- [ ] Add `ExpertProfile.IsVerified`
- [ ] Update EF configuration if needed
- [ ] Generate migration
- [ ] Verify snapshot and schema output

### M2. DTO Contract Layer

- [ ] Add expert create request
- [ ] Add expert update request
- [ ] Add admin create request
- [ ] Add admin update request
- [ ] Add certificate detail response
- [ ] Add certificate list item response
- [ ] Add admin list query/filter request if paging or filtering is required
- [ ] Add verification fields to relevant profile responses

### M3. Service Layer

- [ ] Add `IExpertCertificateService`
- [ ] Implement expert self create
- [ ] Implement expert self list
- [ ] Implement expert self detail
- [ ] Implement expert self update with ownership guard
- [ ] Implement expert self delete with ownership guard
- [ ] Implement admin create
- [ ] Implement admin list
- [ ] Implement admin detail
- [ ] Implement admin update/review
- [ ] Implement admin delete
- [ ] Implement `ExpertProfile.IsVerified` recalculation helper

### M4. API Layer

- [ ] Add expert routes under `/api/experts/me/certificates`
- [ ] Add admin routes under `/api/admin/expert/certificates`
- [ ] Apply role authorization
- [ ] Add swagger summaries and response contracts
- [ ] Add route convention tests if this module adopts the same pattern as `MyProfile`

### M5. Verification Propagation

- [ ] On admin set `Verified`, set `ExpertProfile.IsVerified = true`
- [ ] On admin set `Rejected`, recalculate whether any verified certificate remains
- [ ] On admin set `Pending`, recalculate whether any verified certificate remains
- [ ] On delete of a verified certificate, recalculate profile verification
- [ ] Ensure expert-side update cannot silently keep stale verified state

### M6. Tests

- [ ] Integration test: expert can create own certificate
- [ ] Integration test: expert cannot access another expert's certificate
- [ ] Integration test: admin can list certificates globally
- [ ] Integration test: admin verification updates profile verification flag
- [ ] Integration test: deleting the last verified certificate resets `ExpertProfile.IsVerified`
- [ ] Unit test: profile response mapping returns persisted verification state instead of hard-coded `false`

### M7. Docs Sync After Code Change

- [ ] Update `expert-certs.introduction.md`
- [ ] Update `expert-certs.roadmap.md`
- [ ] Update `expert-certs.hallucination.md`
- [ ] Update `expert-certs.sourcecode.md`
- [ ] Update `expert-certs.useguide.md`

## Proposed Execution Order

1. confirm open risks from `expert-certs.hallucination.md`
2. ship persistence changes
3. ship service layer and verification recalculation
4. ship controller layer
5. ship tests
6. sync docs to actual implementation

## Dependencies

### Internal Code Dependencies

- `SnakeAid.Core/Domains/ExpertCertificate.cs`
- `SnakeAid.Core/Domains/ExpertProfile.cs`
- `SnakeAid.Core/Domains/ReportMedia.cs`
- `SnakeAid.Repository/Data/SnakeAidDbContext.cs`
- `SnakeAid.Service/Implements/ExpertService.cs`
- `SnakeAid.Service/Implements/MyProfileService.cs`
- `SnakeAid.Service/Implements/AdminUserService.cs`

### Architectural Dependency

Certificate attachments must respect the existing polymorphic `ReportMedia` pattern:

- no relational FK from `ReportMedia` to parent certificate entity
- parent identity flows through `ReferenceId`
- parent type flows through `MediaReferenceType`

## Suggested Task Breakdown

### Task Group A. Schema

- add enum member for certificate media
- add `ExpertProfile.IsVerified`
- create migration

### Task Group B. Certificate Domain Application Layer

- add DTOs
- add service interface and implementation
- add ownership and validation rules

### Task Group C. API Surface

- add expert controller
- add admin controller
- add route tests

### Task Group D. Verification Consistency

- update public expert directory mapping
- update my-profile expert response if chosen
- update admin user detail expert response if chosen

### Task Group E. Docs

- keep this roadmap current
- close resolved risks in `expert-certs.hallucination.md`
- reflect shipped contract in `expert-certs.useguide.md`

## Resume Checklist

Before resuming implementation, verify:

- current migration state
- whether `expert-certs.hallucination.md` still contains unresolved risks
- whether any certificate endpoint has already been added
- whether public/admin/my-profile responses have already been synchronized
