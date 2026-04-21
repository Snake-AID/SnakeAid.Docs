---
module: expert-certs
last_updated: 2026-04-21
status: open-risks
---

# Expert Certificates Hallucination Log

This file tracks decisions that should not be guessed from incomplete business context.

Rule for maintenance:

- when a risk is resolved, keep the original options
- record the chosen decision explicitly
- merge the chosen direction into the baseline docs
- mark the risk as closed

## Open Risks

### Risk 1. Exact enum member name for certificate media

Status: `Open`

Reason:

The request says "update enum `CertExpert` for entity `ReportMedia`", but the current enum is `MediaReferenceType`. The exact new member name is not fully deterministic from existing code.

Current verified enum values:

- `CommunityReport`
- `SnakebiteIncident`
- `RescueMission`
- `SnakeCatchingRequest`
- `SnakeCatchingMission`

Options:

- Option A: add `ExpertCertificate`
- Option B: add `CertExpert`
- Option C: add `ExpertCert`

Recommended direction:

- Option A because it matches the existing entity name and is the least ambiguous for future maintainers

Decision record:

- No user decision recorded yet

### Risk 2. What should `ReportMedia.ReferenceId` point to for certificate media

Status: `Open`

Reason:

The current attachment model is polymorphic and non-relational. Certificate media can logically attach either to the expert profile or to each certificate record.

Options:

- Option A: `ReferenceId = ExpertProfile.AccountId`
- Option B: `ReferenceId = ExpertCertificate.Id`

Impact:

- Option A simplifies "all media for this expert" queries
- Option B better models one-to-many certificate records and avoids ambiguity when multiple certificates exist

Recommended direction:

- Option B because certificate files are naturally attached to a specific certificate record, not to the profile as a whole

Decision record:

- No user decision recorded yet

### Risk 3. Source of truth for certificate attachment URL

Status: `Open`

Reason:

`ExpertCertificate` already has `CertificateUrl`, while the requested note also points toward `ReportMedia` usage.

Options:

- Option A: keep `ExpertCertificate.CertificateUrl` as the only source of truth and treat `ReportMedia` as out of scope
- Option B: use `ReportMedia` as the attachment registry and copy or derive `CertificateUrl`
- Option C: remove direct URL usage later and fully pivot to `ReportMedia`

Recommended direction:

- Option B for this phase

Reason for recommendation:

- it respects the existing NoSQL-style media design
- it still preserves compatibility with the current domain field
- it reduces migration risk compared with a full URL model redesign

Decision record:

- No user decision recorded yet

### Risk 4. Can admin create certificates on behalf of an expert

Status: `Open`

Reason:

The request asks for CRUD APIs for both admin and expert. CRUD can mean admin has full create/update/delete authority, but that is still a business policy decision.

Options:

- Option A: expert can create own certificates, admin can only review/update/delete existing ones
- Option B: expert and admin both have full CRUD
- Option C: expert can submit, admin can only review state and cannot edit metadata

Recommended direction:

- Option B if the word "CRUD" is interpreted literally for both roles

Decision record:

- No user decision recorded yet

### Risk 5. What makes an expert "verified"

Status: `Open`

Reason:

The target state says admins approve certificates before an expert participates as a verified expert, but the exact aggregation rule is not explicitly stated.

Options:

- Option A: `ExpertProfile.IsVerified = true` if at least one certificate is `Verified`
- Option B: all active certificates must be `Verified`
- Option C: one primary certificate type must be `Verified`, other certificates are informational

Recommended direction:

- Option A as the minimal rule consistent with the current data model

Decision record:

- No user decision recorded yet

### Risk 6. Should expert-side update reset a previously verified certificate

Status: `Open`

Reason:

If an expert edits metadata or replaces the underlying file after admin approval, the previous review may no longer be trustworthy.

Options:

- Option A: any expert-side update resets `VerificationStatus` to `Pending`
- Option B: only file/url changes reset to `Pending`
- Option C: no automatic reset; admin must manually re-review if needed

Recommended direction:

- Option B because it protects review integrity while avoiding unnecessary churn on harmless text edits

Decision record:

- No user decision recorded yet

### Risk 7. Which profile APIs must expose `IsVerified`

Status: `Open`

Reason:

Current code is inconsistent:

- public expert directory response has `IsVerified` but returns hard-coded `false`
- expert my-profile response does not expose it
- admin expert profile response does not expose it

Options:

- Option A: expose persisted `IsVerified` only in public expert directory
- Option B: expose persisted `IsVerified` in public expert directory, expert my-profile, and admin user detail
- Option C: keep only admin visibility and remove from public profile

Recommended direction:

- Option B because it gives a consistent client contract and avoids forcing mobile to infer verification indirectly

Decision record:

- No user decision recorded yet

## Closed Risks

None yet.
