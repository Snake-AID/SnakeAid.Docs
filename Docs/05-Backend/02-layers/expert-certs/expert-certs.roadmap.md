---
module: expert-certs
last_updated: 2026-04-22
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

- [ ] Add `MediaReferenceType.ExpertCertificate`
- [ ] Add `ExpertProfile.IsVerified`
- [ ] Define active-certificate rule for verification queries and filters
- [ ] Plan transition away from `ExpertCertificate.CertificateUrl`
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
- [ ] Implement direct-recruit admin flow: create certificate and approve immediately for an existing account
- [ ] Implement `ExpertProfile.IsVerified` recalculation helper

### M4. API Layer

- [ ] Add expert routes under `/api/experts/me/certificates`
- [ ] Add admin routes under `/api/admin/expert/certificates`
- [ ] Support admin-side create for an existing expert account in the direct-recruit flow
- [ ] Apply role authorization
- [ ] Add swagger summaries and response contracts
- [ ] Add route convention tests if this module adopts the same pattern as `MyProfile`

### M5. Verification Propagation

- [ ] On admin set `Verified`, recalculate using the "all active certificates must be verified" rule
- [ ] On admin set `Rejected`, recalculate using the same rule
- [ ] On admin set `Pending`, recalculate using the same rule
- [ ] On delete of any active certificate, recalculate profile verification
- [ ] On any expert-side update, reset the certificate to `Pending`
- [ ] Expose persisted `IsVerified` in public directory, expert my-profile, and admin user detail

### M6. Tests

- [ ] Integration test: expert can create own certificate
- [ ] Integration test: expert cannot access another expert's certificate
- [ ] Integration test: admin can list certificates globally
- [ ] Integration test: admin direct-recruit flow can create and immediately approve certificates for an existing expert account
- [ ] Integration test: any expert-side update resets `VerificationStatus` to `Pending`
- [ ] Integration test: profile stays unverified while any active certificate is still `Pending`
- [ ] Integration test: profile stays unverified while any active certificate is `Rejected`
- [ ] Integration test: profile becomes verified only when all active certificates are `Verified`
- [ ] Unit test: profile response mapping returns persisted verification state instead of hard-coded `false`

### M7. Docs Sync After Code Change

- [ ] Update `expert-certs.introduction.md`
- [ ] Update `expert-certs.roadmap.md`
- [ ] Update `expert-certs.hallucination.md`
- [ ] Update `expert-certs.sourcecode.md`
- [ ] Update `expert-certs.useguide.md`

## Proposed Execution Order

1. confirm the remaining open decision from `expert-certs.hallucination.md`
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

- add `MediaReferenceType.ExpertCertificate`
- add `ExpertProfile.IsVerified`
- prepare the transition path away from `ExpertCertificate.CertificateUrl`
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
- update expert my-profile response
- update admin user detail expert response
- apply the "all active certificates must be verified" rule consistently

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
