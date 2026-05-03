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
}

class MyConsultationResponse {
  +Guid ExpertId
  +string? ExpertName
  +string? ExpertAvatarUrl
}

class ExpertConsultationResponse {
  +Guid UserId
  +string? UserName
  +string? UserAvatarUrl
}

Account <.. MyConsultationResponse : consulted expert AvatarUrl
Account <.. ExpertConsultationResponse : member/rescuer AvatarUrl via UserAvatarUrl
```

## Current Implemented Sequence: Member Consultation History

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
S->>S: Map ExpertAvatarUrl from expert Account.AvatarUrl
S-->>C: PagingResponse<MyConsultationResponse>
C-->>M: expertId, expertName, expertAvatarUrl
```

## Implemented Sequence: Expert Consultation History

```mermaid
sequenceDiagram
participant M as Mobile
participant C as ExpertController
participant S as ConsultationService
participant DB as Database

M->>C: GET /api/experts/me/consultations
C->>S: GetExpertConsultationsAsync(expertId, query)
S->>DB: Load scheduled bookings with User
S->>DB: Load emergency requests with Rescuer
S->>S: Map UserAvatarUrl from member/rescuer Account.AvatarUrl
S-->>C: PagingResponse<ExpertConsultationResponse>
C-->>M: userId, userName, userAvatarUrl
```

## Implementation Hotspots

- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Tests/Integration/ExpertConsultationPriceResponseTests.cs`

Member endpoint is already correct:

- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Tests/Integration/ConsultationPriceBugConditionTests.cs`
