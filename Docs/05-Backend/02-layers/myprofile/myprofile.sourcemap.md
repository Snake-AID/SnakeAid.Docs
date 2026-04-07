---
doc_role: operation
module: myprofile
kind: layer
doc_type: sourcemap
status: active
last_updated: 2026-04-07
owners: [backend-team]
---

# MyProfile Source Map

## File-Level Mapping

Backend files:

| Layer | File | Purpose |
|---|---|---|
| API | `SnakeAid.Api/Controllers/MyProfileController.cs` | Consolidated self-profile controller for member, expert, and rescuer routes. |
| Core Request | `SnakeAid.Core/Requests/MyProfile/UpdateMemberProfileRequest.cs` | Member profile update contract. |
| Core Request | `SnakeAid.Core/Requests/MyProfile/UpdateExpertProfileRequest.cs` | Expert profile update contract. |
| Core Request | `SnakeAid.Core/Requests/MyProfile/UpdateRescuerProfileRequest.cs` | Rescuer profile update contract. |
| Core Response | `SnakeAid.Core/Responses/MyProfile/MemberMyProfileResponse.cs` | Member self-profile response contract. |
| Core Response | `SnakeAid.Core/Responses/MyProfile/ExpertMyProfileResponse.cs` | Expert self-profile response contract. |
| Core Response | `SnakeAid.Core/Responses/MyProfile/RescuerMyProfileResponse.cs` | Rescuer self-profile response contract. |
| Service Interface | `SnakeAid.Service/Interfaces/IMyProfileService.cs` | Service contract for role-specific self-profile reads/updates. |
| Service Implementation | `SnakeAid.Service/Implements/MyProfileService.cs` | Loads account + role profile, validates ownership by current user id, updates editable fields. |
| Tests | `SnakeAid.Tests/Integration/MyProfileServiceTests.cs` | Regression tests for update behavior and role/profile guardrails. |
| Tests | `SnakeAid.Tests/Unit/MyProfileRouteConventionTests.cs` | Route and role convention tests for the consolidated controller. |

## Class Diagram

```mermaid
classDiagram
    class MyProfileController {
        +GetMemberProfile()
        +UpdateMemberProfile(UpdateMemberProfileRequest)
        +GetExpertProfile()
        +UpdateExpertProfile(UpdateExpertProfileRequest)
        +GetRescuerProfile()
        +UpdateRescuerProfile(UpdateRescuerProfileRequest)
    }

    class IMyProfileService {
        +GetMemberProfileAsync(Guid)
        +UpdateMemberProfileAsync(Guid, UpdateMemberProfileRequest)
        +GetExpertProfileAsync(Guid)
        +UpdateExpertProfileAsync(Guid, UpdateExpertProfileRequest)
        +GetRescuerProfileAsync(Guid)
        +UpdateRescuerProfileAsync(Guid, UpdateRescuerProfileRequest)
    }

    class MyProfileService
    class Account
    class MemberProfile
    class ExpertProfile
    class RescuerProfile

    MyProfileController --> IMyProfileService
    IMyProfileService <|.. MyProfileService
    MyProfileService --> Account
    MyProfileService --> MemberProfile
    MyProfileService --> ExpertProfile
    MyProfileService --> RescuerProfile
```

## Sequence Diagram

```mermaid
sequenceDiagram
    participant Client
    participant Controller
    participant MyProfileService
    participant UnitOfWork
    participant Database

    Client->>Controller: GET/PUT /api/{role}/me/profile with Bearer token
    Controller->>Controller: Read current user id from JWT claims
    Controller->>MyProfileService: Get/Update role profile(userId, request)
    MyProfileService->>UnitOfWork: Load Account by userId
    MyProfileService->>UnitOfWork: Load role profile by AccountId
    UnitOfWork->>Database: Query account/profile tables
    MyProfileService->>Database: Persist account/profile changes when PUT
    MyProfileService-->>Controller: Role-specific response DTO
    Controller-->>Client: ApiResponse<T>
```

## Function Calling Graph

```mermaid
flowchart TD
    A[MyProfileController.GetMemberProfile] --> S[MyProfileService.GetMemberProfileAsync]
    B[MyProfileController.UpdateMemberProfile] --> T[MyProfileService.UpdateMemberProfileAsync]
    C[MyProfileController.GetExpertProfile] --> U[MyProfileService.GetExpertProfileAsync]
    D[MyProfileController.UpdateExpertProfile] --> V[MyProfileService.UpdateExpertProfileAsync]
    E[MyProfileController.GetRescuerProfile] --> W[MyProfileService.GetRescuerProfileAsync]
    F[MyProfileController.UpdateRescuerProfile] --> X[MyProfileService.UpdateRescuerProfileAsync]
    S --> M[MapMemberProfile]
    T --> M
    U --> N[MapExpertProfile]
    V --> N
    W --> O[MapRescuerProfile]
    X --> O
```
