---
doc_role: implementation
module: symptom-config
kind: layer
doc_type: sourcecode
status: active
last_updated: 2026-04-29
owners: [backend-team]
verification_status: code-verified
---

# Symptom Config Source Code Map

## Code-Verified Files

| Area | File | Responsibility |
|---|---|---|
| Domain | `SnakeAid.Core/Domains/SymptomConfig.cs` | Entity, `SymptomCategory`, `TimeScorePoint` |
| Requests | `SnakeAid.Core/Requests/SymptomConfig/CreateSymptomConfigRequest.cs` | Create request contract |
| Requests | `SnakeAid.Core/Requests/SymptomConfig/UpdateSymptomConfigRequest.cs` | Update request contract |
| Requests | `SnakeAid.Core/Requests/SymptomConfig/GetSymptomConfigRequest.cs` | Filter request contract |
| Responses | `SnakeAid.Core/Responses/SymptomConfig/SymptomConfigResponse.cs` | Flat config response |
| Responses | `SnakeAid.Core/Responses/SymptomConfig/GroupedSymptomConfigResponse.cs` | Grouped UI response |
| API | `SnakeAid.Api/Controllers/SymptomConfigController.cs` | HTTP endpoints |
| Service | `SnakeAid.Service/Interfaces/ISymptomConfigService.cs` | Service interface |
| Service | `SnakeAid.Service/Implements/SymptomConfigService.cs` | CRUD and grouped query implementation |
| Repository | `SnakeAid.Repository/Data/Configurations/SymptomConfigConfiguration.cs` | EF table, enum conversion, indexes, optional venom relation |

## Domain Class Diagram

```mermaid
classDiagram
    class BaseEntity {
        DateTime CreatedAt
        DateTime UpdatedAt
    }

    class SymptomConfig {
        int Id
        string GroupName
        string AttributeKey
        string AttributeLabel
        int DisplayOrder
        string Name
        string? Description
        bool IsCritical
        string? AlertMessage
        SymptomCategory Category
        string? TimeScoresJson
        int? VenomTypeId
        bool IsActive
        List~TimeScorePoint~ TimeScoreList
        VenomType? VenomType
    }

    class TimeScorePoint {
        int MinMinutes
        int MaxMinutes
        int Score
    }

    class SymptomCategory {
        <<enumeration>>
        Core = 1
        Modifier = 2
    }

    class VenomType {
        int Id
        string Name
    }

    BaseEntity <|-- SymptomConfig
    SymptomConfig --> SymptomCategory
    SymptomConfig --> TimeScorePoint
    SymptomConfig --> VenomType : optional
```

## API And Service Class Diagram

```mermaid
classDiagram
    class SymptomConfigController {
        CreateSymptomConfig(request)
        GetSymptomConfigById(id)
        FilterSymptomConfigs(request)
        GetAllSymptomConfig()
        GetSymptomConfigsGrouped()
        GetSymptomConfigsGroupedForUI()
        UpdateSymptomConfig(id, request)
        DeleteSymptomConfig(id)
    }

    class ISymptomConfigService {
        CreateSymptomConfigAsync(request)
        GetSymptomConfigByIdAsync(id)
        FilterSymptomConfigsAsync(request)
        GetAllSymptomConfigAsync()
        GetSymptomConfigsGroupedByKeyAsync()
        GetGroupedSymptomConfigsForUIAsync()
        UpdateSymptomConfigAsync(id, request)
        DeleteSymptomConfigAsync(id)
    }

    class SymptomConfigService {
        CreateSymptomConfigAsync(request)
        GetSymptomConfigByIdAsync(id)
        FilterSymptomConfigsAsync(request)
        GetAllSymptomConfigAsync()
        GetSymptomConfigsGroupedByKeyAsync()
        GetGroupedSymptomConfigsForUIAsync()
        UpdateSymptomConfigAsync(id, request)
        DeleteSymptomConfigAsync(id)
    }

    class UnitOfWork {
        GetRepository~T~()
        ExecuteInTransactionAsync()
        CommitAsync()
    }

    SymptomConfigController --> ISymptomConfigService
    ISymptomConfigService <|.. SymptomConfigService
    SymptomConfigService --> UnitOfWork
    SymptomConfigService --> SymptomConfig
```

## Grouped UI Read Sequence

```mermaid
sequenceDiagram
    participant FE as Mobile or Frontend
    participant API as SymptomConfigController
    participant SVC as SymptomConfigService
    participant DB as SymptomConfigs table

    FE->>API: GET /api/symptom-configs/grouped-for-ui
    API->>SVC: GetGroupedSymptomConfigsForUIAsync()
    SVC->>DB: Query active configs ordered by DisplayOrder, Id
    DB-->>SVC: Active SymptomConfig rows with optional VenomType
    SVC->>SVC: Group by AttributeKey, AttributeLabel, GroupName, DisplayOrder
    SVC->>SVC: Map rows to question groups and options
    SVC-->>API: List<GroupedSymptomConfigResponse>
    API-->>FE: ApiResponse<List<GroupedSymptomConfigResponse>>
```

## Create Sequence

```mermaid
sequenceDiagram
    participant Admin as Admin UI
    participant API as SymptomConfigController
    participant SVC as SymptomConfigService
    participant VT as VenomType repository
    participant DB as SymptomConfig repository

    Admin->>API: POST /api/symptom-configs
    API->>SVC: CreateSymptomConfigAsync(request)
    SVC->>SVC: Reject null request
    alt VenomTypeId supplied
        SVC->>VT: Find VenomType by id
        VT-->>SVC: VenomType or null
        SVC->>SVC: Throw NotFound when missing
    end
    SVC->>SVC: Serialize TimeScoreList to TimeScoresJson
    SVC->>DB: Insert SymptomConfig
    SVC->>DB: Commit
    SVC-->>API: SymptomConfigResponse
    API-->>Admin: ApiResponse<SymptomConfigResponse>
```

## Update Sequence

```mermaid
sequenceDiagram
    participant Admin as Admin UI
    participant API as SymptomConfigController
    participant SVC as SymptomConfigService
    participant DB as SymptomConfig repository
    participant VT as VenomType repository

    Admin->>API: PUT /api/symptom-configs/{id}
    API->>SVC: UpdateSymptomConfigAsync(id, request)
    SVC->>SVC: Reject null request
    SVC->>DB: Load SymptomConfig by id with VenomType
    DB-->>SVC: Existing config or null
    SVC->>SVC: Throw NotFound when config is missing
    alt VenomTypeId supplied
        SVC->>VT: Find VenomType by id
        VT-->>SVC: VenomType or null
        SVC->>SVC: Throw NotFound when missing
    end
    SVC->>SVC: Apply only provided fields
    SVC->>SVC: Serialize TimeScoreList when provided
    SVC->>DB: Update SymptomConfig
    SVC->>DB: Commit
    SVC-->>API: SymptomConfigResponse
    API-->>Admin: ApiResponse<SymptomConfigResponse>
```

## Filter Sequence

```mermaid
sequenceDiagram
    participant Admin as Admin UI
    participant API as SymptomConfigController
    participant SVC as SymptomConfigService
    participant DB as SymptomConfig repository

    Admin->>API: GET /api/symptom-configs/filter?groupName=BACKGROUND&pageNumber=1&pageSize=20
    API->>SVC: FilterSymptomConfigsAsync(request)
    SVC->>SVC: Build predicate from optional filters
    SVC->>DB: GetPagingListAsync(predicate, order, include VenomType)
    DB-->>SVC: PagedData<SymptomConfig>
    SVC->>SVC: Map rows to SymptomConfigResponse
    SVC-->>API: PagedData<SymptomConfigResponse>
    API-->>Admin: ApiResponse<PagedData<SymptomConfigResponse>>
```

## Current Implementation Notes

- `GetGroupedSymptomConfigsForUIAsync()` filters `IsActive = true`.
- `GetSymptomConfigsGroupedByKeyAsync()` also filters `IsActive = true`.
- `FilterSymptomConfigsAsync()` can return active and inactive rows depending on query filters.
- `GetAllSymptomConfigAsync()` currently has no `IsActive` predicate, so `GET /api/symptom-configs` returns active and inactive rows.
- `GetAllSymptomConfigAsync()` orders the flat all-data list by `DisplayOrder`, then `Id`.
- `DeleteSymptomConfigAsync()` soft-deletes by setting `IsActive = false` and refreshing `UpdatedAt`.
- `UpdateSymptomConfigAsync()` only applies fields supplied in the request.
- `VenomTypeId` is validated only when a non-null value is supplied.
- The requested group/key matrix is intentionally not enforced in the service for this scope; frontend/admin UI owns that validation.
- Admin read/write actions are protected with `[Authorize(Roles = "Admin")]`.
- `GetSymptomConfigsGrouped()` and `GetSymptomConfigsGroupedForUI()` remain public active-only read actions.

## Soft Delete Sequence

```mermaid
sequenceDiagram
    participant Admin as Admin UI
    participant API as SymptomConfigController
    participant SVC as SymptomConfigService
    participant DB as Repository

    Admin->>API: DELETE /api/symptom-configs/{id}
    API->>SVC: DeleteSymptomConfigAsync(id)
    SVC->>DB: Load SymptomConfig by id
    DB-->>SVC: Existing config or null
    SVC->>SVC: Throw NotFound when config is missing
    SVC->>SVC: Set IsActive = false and UpdatedAt = now
    SVC->>DB: Update SymptomConfig
    SVC->>DB: Commit
    SVC-->>API: Completed
    API-->>Admin: Success response
```
