---
module: expert-certs
last_updated: 2026-04-22
status: planning
source_of_truth: SnakeAid.Backend
---

# Expert Certificates Introduction

## Goal

Implement the `ExpertCert` workstream so the system can support expert qualification review through certificate submission and admin approval before an expert is treated as verified inside the platform.

This workstream currently contains three requested tasks:

1. extend `MediaReferenceType` in `ReportMedia` with a certificate-specific enum value
2. add persisted `IsVerified` to `ExpertProfile`
3. build CRUD APIs for `ExpertCertificate` for both `Expert` and `Admin`, including admin-side create for direct-recruit onboarding

## Verified Current Codebase State

The following points are verified from the backend repository as of 2026-04-22:

- `ExpertCertificate` already exists as a domain entity in `SnakeAid.Core/Domains/ExpertCertificate.cs`
- `ExpertCertificate` already has `Id`, `ExpertId`, certificate metadata fields, `CertificateUrl`, `VerificationStatus`, and `RejectionReason`
- `VerificationStatus` already exists with values `Pending`, `Verified`, and `Rejected`
- `SnakeAidDbContext` already exposes `DbSet<ExpertCertificate> ExpertCertificates`
- `ReportMedia` already follows a polymorphic, NoSQL-style attachment pattern using `ReferenceId` and `MediaReferenceType`
- current `MediaReferenceType` values are only:
  - `CommunityReport`
  - `SnakebiteIncident`
  - `RescueMission`
  - `SnakeCatchingRequest`
  - `SnakeCatchingMission`
- `ExpertProfile` does not currently persist `IsVerified`
- public expert directory response already contains `ExpertProfileResponse.IsVerified`
- current `ExpertService.MapExpertProfilesAsync` hard-codes `IsVerified = false`
- current `MyProfile` expert response does not expose `IsVerified`
- current admin user detail expert profile response does not expose `IsVerified`
- there is no active `ExpertCertificate` controller, request DTO set, response DTO set, service, or automated API test surface

## Implementation Direction

The intended backend direction is:

- keep `ExpertCertificate` as the business entity for expert credential review
- reuse the existing `ReportMedia` polymorphic pattern instead of introducing relational certificate-media tables
- add `MediaReferenceType.ExpertCertificate`
- for certificate media, use `ReferenceId = ExpertCertificate.Id`
- use `ReportMedia` as the long-term source of truth for certificate attachments
- make `ExpertProfile.IsVerified` the cached user-facing verification flag
- let admin review outcomes drive `ExpertProfile.IsVerified`
- expose certificate CRUD for:
  - expert self-service on own certificates
  - admin global management and review
  - admin-side create for an existing expert account during direct-recruit onboarding

## Scope Boundaries

In scope for this workstream:

- persistence changes
- migration changes
- service logic
- admin and expert certificate CRUD APIs
- verification recalculation logic
- route contract tests and integration tests
- baseline docs in `SnakeAid.Docs`

Out of scope unless explicitly added later:

- onboarding role changes
- approval workflows outside certificate review
- file upload infrastructure redesign
- mobile UI implementation

## Planned Architecture Shape

### Data Model

- add `ExpertProfile.IsVerified : bool`
- add `MediaReferenceType.ExpertCertificate`
- certificate attachment lookup key becomes:
  - `ReferenceType = MediaReferenceType.ExpertCertificate`
  - `ReferenceId = ExpertCertificate.Id`
- keep `ExpertCertificate` as a separate table/entity

### Business Flow

1. expert uploads certificate file through existing media/upload strategy
2. expert creates or updates `ExpertCertificate`
3. admin reviews certificate and sets `VerificationStatus`
4. service recalculates `ExpertProfile.IsVerified`
5. expert-facing and admin-facing APIs expose the resulting state consistently

Chosen rule for verification:

- `ExpertProfile.IsVerified = true` only when all active certificates are `Verified`
- if any active certificate is `Pending` or `Rejected`, `ExpertProfile.IsVerified = false`

Chosen rule for expert-side edit:

- any expert-side update resets that certificate's `VerificationStatus` to `Pending`

Chosen rule for profile exposure:

- expose persisted `IsVerified` in:
  - public expert directory
  - expert my-profile
  - admin user detail

Known transition note:

- current code still has `ExpertCertificate.CertificateUrl`
- the chosen long-term direction is to pivot fully to `ReportMedia`
- implementation may need an intermediate compatibility step before removing direct URL ownership

## Current Gaps That Must Be Closed

### Gap 1. Persisted verification state does not exist

Current code exposes `IsVerified` in some responses but does not store it on `ExpertProfile`.

### Gap 2. Public response and admin response are inconsistent

- public expert directory already has `IsVerified`
- current implementation always returns `false`
- my-profile expert response does not currently expose it
- admin user detail expert profile response does not currently expose it

### Gap 3. Certificate CRUD is missing

`ExpertCertificate` exists only as a persistence model today.

### Gap 4. Certificate attachment model is not wired yet

`ReportMedia` is already polymorphic and non-relational, but certificate-specific reference handling is not implemented yet.

## Recommended Delivery Order

1. finalize the remaining open decision in `expert-certs.hallucination.md`
2. add persistence changes and migration
3. add request/response DTOs
4. implement service layer
5. add controllers and route tests
6. add integration tests
7. update baseline docs to match shipped code

## Resume Notes

If work resumes later, start with these files first:

- `expert-certs.roadmap.md`
- `expert-certs.hallucination.md`
- `expert-certs.sourcecode.md`
- `expert-certs.useguide.md`
