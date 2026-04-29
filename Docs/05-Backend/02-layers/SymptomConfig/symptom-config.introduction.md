---
doc_role: implementation
module: symptom-config
kind: layer
doc_type: introduction
status: active
last_updated: 2026-04-29
owners: [backend-team]
verification_status: code-verified
---

# Symptom Config Introduction

## Goal

This document set records the current backend contract and implementation direction for `SymptomConfig`.

The immediate product goal is to let mobile and frontend clients render snakebite symptom dropdowns from backend-managed configuration, while giving admin users a clear CRUD surface for maintaining the options.

## Resume Summary

If this work is resumed later without prior chat memory, the current code-verified state is:

1. `SymptomConfig` exists as a persisted domain entity in `SnakeAid.Core/Domains/SymptomConfig.cs`.
2. CRUD endpoints exist under `api/symptom-configs`.
3. `GET /api/symptom-configs/grouped-for-ui` already returns active configs grouped as UI questions and options.
4. `VenomTypeId` is nullable in code and data; frontend/admin UI should not block on it.
5. Current backend validation only enforces basic request attributes such as required fields, max lengths, and display-order range.
6. Current backend does not enforce the business matrix that restricts valid `AttributeKey` values by `GroupName`; the user decision is to keep that validation in frontend/admin UI for this scope.
7. The prod mirror data lives in `symtom.csv` in this folder and should be treated as the current reference snapshot for docs.
8. `GET /api/symptom-configs` currently returns active and inactive rows because it has no `IsActive` predicate.
9. `GET /api/symptom-configs` remains flat and is ordered by `DisplayOrder`, then `Id`.
10. `DELETE /api/symptom-configs/{id}` soft-deletes by setting `IsActive = false`.
11. Admin read and write endpoints now enforce `[Authorize(Roles = "Admin")]`.
12. `GET /api/symptom-configs/grouped` and `GET /api/symptom-configs/grouped-for-ui` remain active-only public read surfaces.

## Code-Verified Current State

### Domain entity

`SymptomConfig` fields:

- `Id`
- `GroupName`
- `AttributeKey`
- `AttributeLabel`
- `DisplayOrder`
- `Name`
- `Description`
- `IsCritical`
- `AlertMessage`
- `Category`
- `TimeScoresJson`
- `VenomTypeId`
- `IsActive`
- `VenomType`

`TimeScoresJson` is stored as `jsonb` and exposed through the not-mapped `TimeScoreList`.

### Current enum values

`SymptomCategory`:

- `Core = 1`
- `Modifier = 2`

### Current API surface

Verified controller route:

- `api/symptom-configs`

Verified endpoints:

- `POST /api/symptom-configs`
- `GET /api/symptom-configs/{id}`
- `GET /api/symptom-configs/filter`
- `GET /api/symptom-configs`
- `GET /api/symptom-configs/grouped`
- `GET /api/symptom-configs/grouped-for-ui`
- `PUT /api/symptom-configs/{id}`
- `DELETE /api/symptom-configs/{id}`

### Current data snapshot

The mirrored prod data currently contains these question groups:

| GroupName | AttributeKey | AttributeLabel | Current option count |
|---|---|---|---:|
| `BACKGROUND` | `AGE_GROUP` | `Độ tuổi của người bị cắn` | 3 |
| `BACKGROUND` | `MEDICAL_HISTORY` | `Bệnh nền của người bị cắn` | 4 |
| `BACKGROUND` | `BITE_LOCATION` | `Rắn cắn vào vị trí nào trên cơ thể?` | 5 |
| `LOCAL` | `SYMPTOM_LOCAL` | `Có triệu chứng gì ở vết cắn?` | 7 |
| `CRITICAL` | `CORE_SIGNS` | `Có dấu hiệu nguy hiểm nào sau đây không?` | 8 |

`VenomTypeId` is present in the CSV but can be null and is not required for the frontend dropdown contract.

## Target Business Matrix

The requested target matrix is:

| GroupName | Allowed AttributeKey values |
|---|---|
| `BACKGROUND` | `AGE_GROUP`, `MEDICAL_HISTORY`, `BITE_LOCATION` |
| `LOCAL` | `SYMPTOM_LOCAL` |
| `CRITICAL` | `CORE_SIGNS` |

This matrix should be used by frontend/admin UI immediately. Backend enforcement is intentionally not planned for this scope to avoid backend technical debt.

## Recommended UI Direction

The optimal frontend direction is not a rigid one-dropdown-per-table-column CRUD screen.

Recommended client pattern:

1. Use `GET /api/symptom-configs/grouped-for-ui` for patient-facing symptom collection.
2. Render a dynamic question list sorted by `displayOrder`.
3. Treat each response item as one question:
   - `groupName` is the section.
   - `attributeKey` is the stable programmatic key.
   - `attributeLabel` is the question text.
   - `options` are the selectable answers.
4. For admin CRUD, use controlled selects for `GroupName` and `AttributeKey`.
5. When admin selects `GroupName`, filter the `AttributeKey` select using the target business matrix.
6. Auto-fill `AttributeLabel` and a default `DisplayOrder` from the chosen `AttributeKey`, while keeping `AttributeLabel` editable.
7. If admin starts from a known label, auto-detect the related `GroupName`, `AttributeKey`, and canonical `AttributeLabel`.
8. Treat delete as soft delete: `DELETE` sets `IsActive = false` instead of physically removing the row.

This gives frontend a resilient metadata-driven form and reduces coupling to hard-coded screens.

## Scope Boundary

In scope:

- document current `SymptomConfig` API contracts for mobile/frontend integration
- document the current grouped UI read contract
- document admin CRUD behavior and constraints
- document frontend-owned validation around `GroupName` and `AttributeKey`
- document backend soft delete behavior
- document backend authorization enforcement
- keep baseline docs synced whenever backend contract changes

Out of scope for the current baseline:

- redesigning the scoring algorithm
- requiring `VenomTypeId`
- changing the database schema only because the docs were created
- inventing unverified auth requirements that are not visible in the current controller

## Expected Impacted Areas

Future implementation or verification may touch:

- `SnakeAid.Core/Domains/SymptomConfig.cs`
- `SnakeAid.Core/Requests/SymptomConfig/*.cs`
- `SnakeAid.Core/Responses/SymptomConfig/*.cs`
- `SnakeAid.Api/Controllers/SymptomConfigController.cs`
- `SnakeAid.Service/Interfaces/ISymptomConfigService.cs`
- `SnakeAid.Service/Implements/SymptomConfigService.cs`
- `SnakeAid.Repository/Data/Configurations/SymptomConfigConfiguration.cs`
- tests under `SnakeAid.Tests`
- docs under `SnakeAid.Docs/Docs/05-Backend/02-layers/SymptomConfig`
