---
module: expert-certs
last_updated: 2026-04-22
status: planning
---

# Expert Certificates Sourcecode Notes

## Verified Existing Code Elements

### Domain and Persistence

- `SnakeAid.Core/Domains/ExpertCertificate.cs`
- `SnakeAid.Core/Domains/ExpertProfile.cs`
- `SnakeAid.Core/Domains/ReportMedia.cs`
- `SnakeAid.Repository/Data/SnakeAidDbContext.cs`

### Existing Service Touchpoints

- `SnakeAid.Service/Implements/ExpertService.cs`
- `SnakeAid.Service/Implements/MyProfileService.cs`
- `SnakeAid.Service/Implements/AdminUserService.cs`
- `SnakeAid.Service/Implements/MediaService.cs`
- `SnakeAid.Service/Extensions/NoSqlReportMediaExtensions.cs`

## Current Structural Observations

- `ExpertCertificate` is currently standalone and not wired into any application flow
- `ReportMedia` is intentionally polymorphic and attachment-like, not a normal FK-bound relational child
- `ExpertProfile` is the correct place for a cached verification flag because multiple profile responses already need that state
- current public expert directory mapping is inconsistent because `IsVerified` exists in the response but is hard-coded to `false`

## Chosen Design Decisions

- use `MediaReferenceType.ExpertCertificate`
- for certificate media, `ReportMedia.ReferenceId = ExpertCertificate.Id`
- long-term ownership of certificate files belongs to `ReportMedia`, not `ExpertCertificate.CertificateUrl`
- any expert-side update resets `VerificationStatus` to `Pending`
- `ExpertProfile.IsVerified = true` only when all active certificates are `Verified`
- expose persisted `IsVerified` in public expert directory, expert my-profile, and admin user detail

## Current Class Diagram

```mermaid
classDiagram
    class ExpertProfile {
        +Guid AccountId
        +string Biography
        +bool IsOnline
        +decimal ConsultationFee
        +decimal? EmergencyConsultationFee
        +decimal Rating
        +int RatingCount
    }

    class ExpertCertificate {
        +Guid Id
        +Guid ExpertId
        +string CertificateName
        +string IssuingOrganization
        +DateTime IssueDate
        +DateTime? ExpiryDate
        +string CertificateUrl
        +VerificationStatus VerificationStatus
        +string RejectionReason
    }

    class ReportMedia {
        +Guid Id
        +Guid? ReferenceId
        +MediaReferenceType ReferenceType
        +string FileName
        +string MediaUrl
        +string ContentType
        +long FileSize
        +MediaPurpose Purpose
        +Guid? UploadBatchId
        +int? SequenceOrder
    }

    class ExpertProfileResponse {
        +bool IsVerified
    }

    ExpertProfileResponse ..> ExpertProfile
```

## Planned Target Class Diagram

```mermaid
classDiagram
    class ExpertProfile {
        +Guid AccountId
        +string Biography
        +bool IsOnline
        +decimal ConsultationFee
        +decimal? EmergencyConsultationFee
        +decimal Rating
        +int RatingCount
        +bool IsVerified
    }

    class ExpertCertificate {
        +Guid Id
        +Guid ExpertId
        +string CertificateName
        +string IssuingOrganization
        +DateTime IssueDate
        +DateTime? ExpiryDate
        +string CertificateUrl
        +VerificationStatus VerificationStatus
        +string RejectionReason
    }

    class ReportMedia {
        +Guid Id
        +Guid? ReferenceId
        +MediaReferenceType ReferenceType
    }

    class IExpertCertificateService {
        +CreateMyAsync()
        +GetMyListAsync()
        +GetMyDetailAsync()
        +UpdateMyAsync()
        +DeleteMyAsync()
        +AdminCreateAsync()
        +AdminGetListAsync()
        +AdminGetDetailAsync()
        +AdminUpdateAsync()
        +AdminDeleteAsync()
    }

    class ExpertCertificateService
    class ExpertCertificatesController
    class AdminExpertCertificatesController

    ExpertProfile "1" <-- "*" ExpertCertificate : AccountId/ExpertId
    ReportMedia ..> ExpertCertificate : ExpertCertificate.Id + MediaReferenceType.ExpertCertificate
    IExpertCertificateService <|.. ExpertCertificateService
    ExpertCertificatesController --> IExpertCertificateService
    AdminExpertCertificatesController --> IExpertCertificateService
```

## Sequence Diagram: Expert Creates Certificate

```mermaid
sequenceDiagram
    actor Expert
    participant Api as ExpertCertificatesController
    participant Service as ExpertCertificateService
    participant DB as SnakeAidDbContext

    Expert->>Api: POST /api/experts/me/certificates
    Api->>Service: CreateMyAsync(currentExpertId, request)
    Service->>DB: validate expert profile exists
    Service->>DB: insert ExpertCertificate
    Service->>DB: insert ReportMedia for ExpertCertificate.Id
    Service->>DB: commit
    Service-->>Api: certificate response
    Api-->>Expert: 201 Created
```

## Sequence Diagram: Expert Updates Certificate

```mermaid
sequenceDiagram
    actor Expert
    participant Api as ExpertCertificatesController
    participant Service as ExpertCertificateService
    participant DB as SnakeAidDbContext

    Expert->>Api: PUT /api/experts/me/certificates/{certificateId}
    Api->>Service: UpdateMyAsync(currentExpertId, certificateId, request)
    Service->>DB: load owned certificate
    Service->>DB: update certificate metadata and media
    Service->>DB: reset VerificationStatus to Pending
    Service->>DB: recalculate ExpertProfile.IsVerified
    Service->>DB: commit
    Api-->>Expert: 200 OK
```

## Sequence Diagram: Admin Reviews Certificate

```mermaid
sequenceDiagram
    actor Admin
    participant Api as AdminExpertCertificatesController
    participant Service as ExpertCertificateService
    participant DB as SnakeAidDbContext

    Admin->>Api: PUT /api/admin/expert/certificates/{certificateId}
    Api->>Service: AdminUpdateAsync(certificateId, request)
    Service->>DB: load certificate
    Service->>DB: update verification status
    Service->>DB: recalculate ExpertProfile.IsVerified
    Service->>DB: commit
    Service-->>Api: updated certificate response
    Api-->>Admin: 200 OK
```

## Sequence Diagram: Admin Direct-Recruit Flow

```mermaid
sequenceDiagram
    actor Admin
    participant Api as AdminExpertCertificatesController
    participant Service as ExpertCertificateService
    participant DB as SnakeAidDbContext

    Admin->>Api: POST /api/admin/expert/certificates
    Api->>Service: AdminCreateAsync(request)
    Service->>DB: validate existing expert account
    Service->>DB: create ExpertCertificate
    Service->>DB: attach ReportMedia
    Service->>DB: set VerificationStatus = Verified
    Service->>DB: recalculate ExpertProfile.IsVerified
    Service->>DB: commit
    Api-->>Admin: 201 Created
```

## Sequence Diagram: Verification Recalculation

```mermaid
sequenceDiagram
    participant Service as ExpertCertificateService
    participant DB as SnakeAidDbContext

    Service->>DB: query certificates for expert
    DB-->>Service: certificate set
    Service->>Service: determine whether all active certificates are Verified
    Service->>DB: update ExpertProfile.IsVerified
    Service->>DB: commit
```

## Code Notes For Future Implementation

### Mapping updates needed

- `ExpertService.MapExpertProfilesAsync` should read persisted `ExpertProfile.IsVerified`
- `MyProfileService.MapExpert` should expose persisted `IsVerified`
- `AdminUserService.GetAdminUserDetailAsync` should include it in `AdminExpertProfileResponse`

### Persistence updates needed

- add `IsVerified` column to `ExpertProfiles`
- update model snapshot and migration

### Media updates needed

- add `MediaReferenceType.ExpertCertificate`
- query certificate media by `ReferenceId = ExpertCertificate.Id` plus `ReferenceType = MediaReferenceType.ExpertCertificate`
- prepare the removal path for direct URL ownership on `ExpertCertificate`

## Resume Pointers

When resuming implementation, inspect these first:

- `SnakeAid.Core/Domains/ExpertCertificate.cs`
- `SnakeAid.Core/Domains/ExpertProfile.cs`
- `SnakeAid.Core/Domains/ReportMedia.cs`
- `SnakeAid.Service/Implements/ExpertService.cs`
- `SnakeAid.Service/Implements/MyProfileService.cs`
- `SnakeAid.Service/Implements/AdminUserService.cs`
