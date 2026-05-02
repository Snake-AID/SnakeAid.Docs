---
doc_role: implementation
module: expert-avatar
kind: diagrams
doc_type: sourcecode
status: implemented
last_updated: 2026-05-03
owners: [backend-team]
verification_status: code-verified
---

# Expert Avatar Source Code Notes

## Implemented Class Diagram

```mermaid
classDiagram
class Account {
  +Guid Id
  +string FullName
  +string? AvatarUrl
  +AccountRole Role
}

class MyConsultationResponse {
  +Guid ConsultationId
  +string Type
  +string Status
  +Guid ExpertId
  +string? ExpertName
  +string? ExpertAvatarUrl
  +decimal? Price
}

class ExpertConsultationResponse {
  +Guid ConsultationId
  +string Type
  +string Status
  +Guid UserId
  +string? UserName
  +string? ExpertAvatarUrl
  +decimal? GrossPrice
  +decimal? NetPrice
}

Account <.. MyConsultationResponse : ExpertAvatarUrl
Account <.. ExpertConsultationResponse : ExpertAvatarUrl
```

## Sequence: User Consultation History

```mermaid
sequenceDiagram
participant M as Mobile
participant C as ConsultationsController
participant S as ConsultationService
participant DB as Database

M->>C: GET /api/users/me/consultations
C->>S: GetMyConsultationsAsync(userId, query)
S->>DB: Load scheduled bookings with Expert, TimeSlot, Consultation
S->>DB: Load emergency ping requests with Expert, Consultation
S->>S: Map ExpertAvatarUrl from Expert.AvatarUrl
S-->>C: PagingResponse<MyConsultationResponse>
C-->>M: expertId, expertName, expertAvatarUrl
```

## Sequence: Expert Consultation History

```mermaid
sequenceDiagram
participant M as Mobile
participant C as ExpertController
participant S as ConsultationService
participant DB as Database

M->>C: GET /api/experts/me/consultations
C->>S: GetExpertConsultationsAsync(expertId, query)
S->>DB: Load current expert Account
S->>DB: Load scheduled and emergency consultation rows
S->>S: Map ExpertAvatarUrl from Account.AvatarUrl
S-->>C: PagingResponse<ExpertConsultationResponse>
C-->>M: userId, userName, expertAvatarUrl
```

## Implementation Hotspots

Use these code locations during implementation:

- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
