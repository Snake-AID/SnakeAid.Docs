---
doc_role: implementation
module: expert-certificates
kind: flow
doc_type: introduction
status: planning
last_updated: 2026-04-20
owners: [backend-team]
verification_status: current-state-code-verified-plan-not-yet-implemented
---

# Expert Certificates Introduction

## Goal

This module plans the expert certification flow in three incremental steps:

1. add `CertExpert` to `ReportMedia.MediaReferenceType`
2. add persisted `IsVerified` to `ExpertProfile`
3. build CRUD APIs for `ExpertCertificate` for both `Expert` and `Admin`

The target outcome is:

- experts can submit and manage their own certificates
- admins can review and manage certificates across all experts
- public expert-facing profile data can expose a real verification flag instead of a hardcoded placeholder

## Resume Summary

If this work is resumed later without prior chat history, the current code-verified state is:

1. `ExpertCertificate` entity already exists in the backend domain and database context.
2. `ExpertProfile` entity does not currently persist `IsVerified`.
3. public `ExpertProfileResponse` already exposes `IsVerified`, but `ExpertService` currently sets it to `false` for every expert.
4. `ExpertMyProfileResponse` does not currently expose `IsVerified`.
5. `ReportMedia.MediaReferenceType` does not currently contain `CertExpert`.
6. there is no current controller, service, request, response, or test surface for `ExpertCertificate` CRUD.

## Code-Verified Current State

### Expert certificate domain

Current domain entity:

- `SnakeAid.Core/Domains/ExpertCertificate.cs`
- fields:
  - `Id`
  - `ExpertId`
  - `CertificateName`
  - `IssuingOrganization`
  - `IssueDate`
  - `ExpiryDate`
  - `CertificateUrl`
  - `VerificationStatus`
  - `RejectionReason`

Current domain enum:

- `VerificationStatus.Pending = 0`
- `VerificationStatus.Verified = 1`
- `VerificationStatus.Rejected = 2`

Current persistence state:

- `SnakeAidDbContext` already exposes `DbSet<ExpertCertificate> ExpertCertificates`
- the table already exists in migrations and snapshot output

### Expert profile verification gap

Current code-verified mismatch:

- `SnakeAid.Core/Domains/ExpertProfile.cs` has no `IsVerified`
- `SnakeAid.Core/Responses/Expert/ExpertProfileResponse.cs` already includes `IsVerified`
- `SnakeAid.Service/Implements/ExpertService.cs` currently maps `IsVerified = false`
- `SnakeAid.Core/Responses/MyProfile/ExpertMyProfileResponse.cs` does not currently include `IsVerified`

This means the public expert response already has a contract field, but the backend currently has no real persisted verification source behind it.

### Report media state

Current `ReportMedia.MediaReferenceType` values:

- `CommunityReport`
- `SnakebiteIncident`
- `RescueMission`
- `SnakeCatchingRequest`
- `SnakeCatchingMission`

Current gap:

- no `CertExpert` enum value exists yet

Current upload surface:

- `POST /api/media/report` creates a `ReportMedia` row
- `POST /api/media/upload-file` uploads a generic file but does not create `ReportMedia`

### API surface state

Current code-verified expert-related endpoints:

- `GET /api/experts`
- `GET /api/experts/{id}`
- `GET /api/experts/me/profile`
- `PUT /api/experts/me/profile`

Current code-verified admin route style:

- admin resources are typically exposed under `/api/admin/...`

Current module gap:

- no `ExpertCertificate` endpoint exists yet for either `Expert` or `Admin`

## Planned Implementation Direction

### Step 1. Extend report media taxonomy

Add:

- `MediaReferenceType.CertExpert`

Primary reason:

- align expert certificate attachments with the existing polymorphic media model
- avoid inventing a separate ad hoc file-category vocabulary for certificate-related media

### Step 2. Persist profile verification

Add:

- `ExpertProfile.IsVerified`

Default direction:

- DB default: `false`
- initial state for all existing experts: `false`

Planned synchronization rule:

- if an expert has at least one `ExpertCertificate` with `VerificationStatus.Verified`, set `ExpertProfile.IsVerified = true`
- if no verified certificates remain, set `ExpertProfile.IsVerified = false`

### Step 3. Add CRUD APIs

Planned backend additions:

- `IExpertCertificateService`
- `ExpertCertificateService`
- expert-facing controller endpoints under `/api/experts/me/certificates`
- admin-facing controller endpoints under `/api/admin/expert/certificates`
- request/response DTOs for create, update, detail, list
- integration tests for role boundaries and verification-state synchronization

## Locked Functional Direction

- `ExpertCertificate` remains the primary business entity
- `ExpertProfile.IsVerified` becomes a fast-read snapshot for profile APIs
- certificate verification authority belongs to `Admin`
- expert-created or expert-updated certificates should not become auto-verified
- public expert APIs should stop hardcoding `IsVerified = false` after this module is implemented

## Risks And Design Notes

Detailed ambiguity and risk buckets for this module are tracked in:

- `expert-certificates.hallucination.md`

The current introduction file keeps only decision-level direction.

### Contract drift to clean up

This module should also close two current contract drifts:

1. public expert responses expose `isVerified` but always return `false`
2. self expert profile responses currently have no `isVerified` field even though verification becomes a user-visible state once admin review exists
