---
doc_role: implementation
module: expert-certificates
kind: flow
doc_type: roadmap
status: planning
last_updated: 2026-04-20
owners: [backend-team]
verification_status: current-state-code-verified-plan-not-yet-implemented
---

# Expert Certificates Roadmap

## Current Status Snapshot

- module status: `Planned`
- `ExpertCertificate` domain entity: `Available`
- `ExpertCertificate` CRUD API: `Not started`
- `ExpertProfile.IsVerified` persistence: `Not available`
- `ExpertProfileResponse.isVerified` contract field: `Available but placeholder-only`
- `MediaReferenceType.CertExpert`: `Not available`
- docs status: `Planning baseline created`

## Current Truth To Resume From

This roadmap is written to resume from zero memory.

Current verified state:

- `ExpertCertificate` exists only as a domain/persistence model
- `ExpertService.MapExpertProfilesAsync(...)` currently hardcodes `IsVerified = false`
- `MyProfileService` has no expert certificate logic
- no expert certificate controller exists
- no expert certificate service exists
- no certificate request/response DTO set exists
- no tests exist for certificate verification flow

## Planned Outcome

1. experts can create, view, update, and delete their own certificates
2. admins can list, view, create, update, and delete certificates across experts
3. admins can set certificate verification state through the update contract
4. `ExpertProfile.IsVerified` becomes a real persisted field
5. expert profile APIs return a real verification value
6. the docs set can be resumed and implemented without relying on prior chat memory

## Locked Functional Direction

- [x] `ExpertCertificate` stays as a separate entity instead of being folded into `ExpertProfile`
- [x] `ExpertProfile.IsVerified` is persisted explicitly
- [x] admin owns verification state transitions
- [x] public expert profile APIs must stop returning placeholder verification values
- [x] expert self-service routes should follow `/api/experts/me/...`
- [x] admin routes should follow `/api/admin/...`
- [x] request/response docs must distinguish verified current state from planned contract

## Implementation Checklist

### Phase 1. Domain And Migration

- [ ] Add `CertExpert` to `MediaReferenceType`
- [ ] Add `IsVerified` to `ExpertProfile`
- [ ] Generate EF migration for the `ExpertProfiles` schema change
- [ ] Verify migration default value for existing rows is `false`

### Phase 2. Contract Models

- [ ] Add expert certificate create request
- [ ] Add expert certificate update request
- [ ] Add expert certificate response
- [ ] Add expert certificate list item response
- [ ] Add admin certificate filter/query request if paging/filtering is needed
- [ ] Add `IsVerified` to `ExpertMyProfileResponse`

### Phase 3. Service Layer

- [ ] Add `IExpertCertificateService`
- [ ] Implement create certificate for expert self-service
- [ ] Implement get my certificates
- [ ] Implement get my certificate detail
- [ ] Implement update my certificate with ownership validation
- [ ] Implement delete my certificate with ownership validation
- [ ] Implement admin list/detail/update/delete
- [ ] Implement centralized profile verification recalculation

### Phase 4. API Layer

- [ ] Add expert controller routes under `/api/experts/me/certificates`
- [ ] Add admin controller routes under `/api/admin/expert/certificates`
- [ ] Apply `Expert` authorization to self-service routes
- [ ] Apply `Admin` authorization to admin routes
- [ ] Keep response envelope style consistent with `ApiResponse<T>`

### Phase 5. Verification Rules

- [ ] On create by expert, default `VerificationStatus = Pending`
- [ ] On update by expert, decide whether status resets to `Pending`
- [ ] On admin set `Verified`, recalculate `ExpertProfile.IsVerified = true`
- [ ] On admin set `Rejected`, recalculate based on remaining verified certificates
- [ ] On delete of a verified certificate, recalculate profile verification

### Phase 6. Tests

- [ ] Integration test: expert can create own certificate
- [ ] Integration test: expert cannot edit another expert's certificate
- [ ] Integration test: admin can list certificates globally
- [ ] Integration test: admin verify action updates `ExpertProfile.IsVerified`
- [ ] Integration test: deleting the last verified certificate resets `ExpertProfile.IsVerified`
- [ ] Integration test: public expert profile returns persisted verification state

### Phase 7. Docs Sync

- [ ] Update introduction when implementation starts
- [ ] update roadmap phase status during development
- [ ] replace planned diagrams with implemented diagrams in sourcecode
- [ ] activate planned API contracts in useguide once endpoints are code-verified

## Proposed Route Set

Expert self-service:

- `POST /api/experts/me/certificates`
- `GET /api/experts/me/certificates`
- `GET /api/experts/me/certificates/{certificateId}`
- `PUT /api/experts/me/certificates/{certificateId}`
- `DELETE /api/experts/me/certificates/{certificateId}`

Admin:

- `POST /api/admin/expert/certificates`
- `GET /api/admin/expert/certificates`
- `GET /api/admin/expert/certificates/{certificateId}`
- `PUT /api/admin/expert/certificates/{certificateId}`
- `DELETE /api/admin/expert/certificates/{certificateId}`

## Verification Strategy

Minimum end-to-end verification:

1. expert uploads or prepares a certificate URL
2. expert creates a pending certificate
3. expert reads and updates own certificate
4. admin lists the certificate
5. admin updates status to `Verified`
6. `ExpertProfile.IsVerified` becomes `true`
7. `GET /api/experts/{id}` returns `isVerified = true`
8. admin rejects or deletes the last verified certificate
9. `ExpertProfile.IsVerified` becomes `false` again

## Open Decision To Confirm During Implementation

One business rule still needs to be locked in code while implementing:

- when an expert edits a previously verified certificate, should the certificate remain `Verified`, or should the edit reset it to `Pending` for re-review

Recommended default:

- reset to `Pending` on material expert edits

## Change Log

### 2026-04-20

- created the initial planning set for expert certificates
- documented the current code-verified gap between response contract and persisted verification data
- proposed the baseline route set for expert and admin CRUD
