---
module: expert-certs
last_updated: 2026-04-22
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

### Risk 4. Can admin create certificates on behalf of an expert

Status: `Open`

Reason:

The request asks for CRUD APIs for both admin and expert. CRUD can mean admin has full create/update/delete authority, but that is still a business policy decision.

Options:

- Option A: expert can create own certificates, admin can only review/update/delete existing ones
- Option B: expert and admin both have full CRUD
- Option C: expert can submit, admin can only review state and cannot edit metadata

Researched business cases from user:

- Case 1: `Free expert onboarding`
  - expert submits one or many certificates
  - admin reads and approves or rejects with a reason
  - expert edits and resubmits when rejected
- Case 2: `Direct-recruit expert onboarding`
  - admin provisions the expert account directly
  - admin adds certificates onto the existing account
  - admin can immediately approve those certificates
  - the account becomes `IsVerified = true`
  - admin then hands username/password to the expert

Current assessment:

- the researched cases support admin-side create capability
- what remains open is whether the implementation should support both cases from day one in the same CRUD surface, or phase them separately

Recommended direction:

- support both cases in one module
- admin must be allowed to create certificates for existing expert accounts
- admin must be allowed to approve immediately in the direct-recruit flow

Decision record:

- No user decision recorded yet

## Closed Risks

### Risk 1. Exact enum member name for certificate media

Status: `Closed`

Reason:

The request said "update enum `CertExpert` for entity `ReportMedia`", but the current enum is `MediaReferenceType`.

Current verified enum values before change:

- `CommunityReport`
- `SnakebiteIncident`
- `RescueMission`
- `SnakeCatchingRequest`
- `SnakeCatchingMission`

Options:

- Option A: add `ExpertCertificate`
- Option B: add `CertExpert`
- Option C: add `ExpertCert`

Decision record:

- User selected Option A on 2026-04-22
- Chosen enum member: `ExpertCertificate`

### Risk 2. What should `ReportMedia.ReferenceId` point to for certificate media

Status: `Closed`

Reason:

The current attachment model is polymorphic and non-relational.

Options:

- Option A: `ReferenceId = ExpertProfile.AccountId`
- Option B: `ReferenceId = ExpertCertificate.Id`

Decision record:

- User selected Option B on 2026-04-22
- Chosen reference rule: `ReferenceId = ExpertCertificate.Id`

### Risk 3. Source of truth for certificate attachment URL

Status: `Closed`

Reason:

`ExpertCertificate` already has `CertificateUrl`, while the requested note also points toward `ReportMedia` usage.

Options:

- Option A: keep `ExpertCertificate.CertificateUrl` as the only source of truth and treat `ReportMedia` as out of scope
- Option B: use `ReportMedia` as the attachment registry and copy or derive `CertificateUrl`
- Option C: remove direct URL usage later and fully pivot to `ReportMedia`

Decision record:

- User selected Option C on 2026-04-22
- Long-term direction: remove direct URL ownership from `ExpertCertificate` and pivot fully to `ReportMedia`
- Transitional note: current code still contains `ExpertCertificate.CertificateUrl`, so implementation may need a migration path rather than a one-step removal

### Risk 5. What makes an expert "verified"

Status: `Closed`

Reason:

The target state says admins approve certificates before an expert participates as a verified expert.

Options:

- Option A: `ExpertProfile.IsVerified = true` if at least one certificate is `Verified`
- Option B: all active certificates must be `Verified`
- Option C: one primary certificate type must be `Verified`, other certificates are informational

Decision record:

- User selected Option B on 2026-04-22
- Chosen aggregation rule: all active certificates must be `Verified`

Implementation note:

- a pending or rejected active certificate must force `ExpertProfile.IsVerified = false`

### Risk 6. Should expert-side update reset a previously verified certificate

Status: `Closed`

Reason:

If an expert edits a certificate after admin approval, the previous review may no longer be trustworthy.

Options:

- Option A: any expert-side update resets `VerificationStatus` to `Pending`
- Option B: only file/url changes reset to `Pending`
- Option C: no automatic reset; admin must manually re-review if needed

Decision record:

- User selected Option A on 2026-04-22
- Chosen rule: any expert-side update resets `VerificationStatus` to `Pending`

### Risk 7. Which profile APIs must expose `IsVerified`

Status: `Closed`

Reason:

Current code is inconsistent across public, self, and admin profile surfaces.

Options:

- Option A: expose persisted `IsVerified` only in public expert directory
- Option B: expose persisted `IsVerified` in public expert directory, expert my-profile, and admin user detail
- Option C: keep only admin visibility and remove from public profile

Decision record:

- User selected Option B on 2026-04-22
- Chosen exposure rule: expose persisted `IsVerified` in public expert directory, expert my-profile, and admin user detail
