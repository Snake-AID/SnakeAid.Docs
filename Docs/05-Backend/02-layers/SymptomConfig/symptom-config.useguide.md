---
doc_role: integration
module: symptom-config
kind: layer
doc_type: useguide
status: active
last_updated: 2026-04-29
api_version: v1
owners: [backend-team]
verification_status: code-verified
---

# Symptom Config Useguide

## 1. Table Of Contents

- [1. Table Of Contents](#1-table-of-contents)
- [2. Overview](#2-overview)
- [3. Authentication & Authorization](#3-authentication--authorization)
- [4. Mobile/Frontend Business + APIs](#4-mobilefrontend-business--apis)
- [5. Admin Business + Admin APIs](#5-admin-business--admin-apis)
- [6. Shared Data Models](#6-shared-data-models)
- [7. Verified Endpoint List](#7-verified-endpoint-list)
- [8. Changelog](#8-changelog)

## 2. Overview

This document records the current verified backend contract for `SymptomConfig`.

Use this module for:

- rendering patient-facing snakebite background, local symptom, and critical sign questions
- reading grouped active options for dropdowns or checkbox lists
- admin CRUD of symptom options

Recommended mobile/frontend read endpoint:

- `GET /api/symptom-configs/grouped-for-ui`

Recommended admin listing endpoint:

- `GET /api/symptom-configs/filter`

Current target group/key matrix:

| GroupName | Allowed AttributeKey values |
|---|---|
| `BACKGROUND` | `AGE_GROUP`, `MEDICAL_HISTORY`, `BITE_LOCATION` |
| `LOCAL` | `SYMPTOM_LOCAL` |
| `CRITICAL` | `CORE_SIGNS` |

Current canonical labels:

| AttributeKey | AttributeLabel |
|---|---|
| `AGE_GROUP` | `Độ tuổi của người bị cắn` |
| `MEDICAL_HISTORY` | `Bệnh nền của người bị cắn` |
| `BITE_LOCATION` | `Rắn cắn vào vị trí nào trên cơ thể?` |
| `SYMPTOM_LOCAL` | `Có triệu chứng gì ở vết cắn?` |
| `CORE_SIGNS` | `Có dấu hiệu nguy hiểm nào sau đây không?` |

Important integration notes:

- `VenomTypeId` is optional and may be `null`.
- `TimeScoreList` is a list of time windows and scores.
- `IsCritical = true` means the client should show the alert message immediately when the option is selected.
- The backend does not enforce the target group/key matrix in this scope; frontend/admin UI owns this validation.
- `GET /api/symptom-configs` currently returns active and inactive rows.
- `GET /api/symptom-configs` returns a flat list ordered by `displayOrder`, then `id`.
- `GET /api/symptom-configs/filter` can return inactive rows when `isActive=false`.
- `GET /api/symptom-configs/grouped` and `GET /api/symptom-configs/grouped-for-ui` return active rows only.
- `DELETE /api/symptom-configs/{id}` soft-deletes by setting `IsActive = false`.

## 3. Authentication & Authorization

### Current Verified State

Admin read/write endpoints enforce `[Authorize(Roles = "Admin")]`.

Admin-only endpoints:

- `POST /api/symptom-configs`
- `PUT /api/symptom-configs/{id}`
- `DELETE /api/symptom-configs/{id}`
- `GET /api/symptom-configs`
- `GET /api/symptom-configs/filter`
- `GET /api/symptom-configs/{id}`

Public active-only read endpoints:

- `GET /api/symptom-configs/grouped`
- `GET /api/symptom-configs/grouped-for-ui`

### Recommended Client Behavior

- public patient-facing reads should use only the grouped UI endpoint unless product requires another active-only shape
- admin CRUD screens should require an admin session in the app
- do not expose create, update, or delete controls in normal member/mobile symptom collection screens

## 4. Mobile/Frontend Business + APIs

### 4.1 Recommended Rendering Flow

1. Call `GET /api/symptom-configs/grouped-for-ui`.
2. Sort by `displayOrder` if the client wants defensive sorting.
3. Render each item as one question.
4. Use `groupName` to group questions into sections:
   - `BACKGROUND`
   - `LOCAL`
   - `CRITICAL`
5. Use `attributeKey` as the stable programmatic key in local state.
6. Use `attributeLabel` as question text.
7. Render `options` as dropdown items, chips, radio buttons, or checkboxes depending on UX.
8. If selected option has `isCritical = true`, show `alertMessage` when present.

### 4.2 Get Grouped Symptom Configs For UI

#### `GET /api/symptom-configs/grouped-for-ui`

Purpose:

- get active symptom configs grouped as frontend-ready questions with options

Status:

- `Active`
- Code-verified

Auth:

- not explicitly verified in controller snippet

Request:

```http
GET /api/symptom-configs/grouped-for-ui
```

Success response shape:

```json
{
  "success": true,
  "message": "Symptom configurations retrieved successfully for UI display",
  "data": [
    {
      "groupName": "BACKGROUND",
      "attributeKey": "AGE_GROUP",
      "attributeLabel": "Độ tuổi của người bị cắn",
      "displayOrder": 1,
      "options": [
        {
          "id": 101,
          "name": "Trẻ em (dưới 12 tuổi)",
          "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
          "isCritical": true,
          "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
          "category": 2,
          "categoryDisplay": "Modifier",
          "timeScoreList": [
            {
              "minMinutes": 0,
              "maxMinutes": 1440,
              "score": 20
            }
          ],
          "venomTypeId": null,
          "venomType": null,
          "isActive": true
        }
      ]
    }
  ]
}
```

Field notes:

- only active options are returned
- `groupName` is the UI section key
- `attributeKey` is stable and should be used in client state
- `attributeLabel` is the question title
- `displayOrder` controls question order
- `options[].id` is the option ID to submit in downstream symptom flows if they require selected config IDs
- `options[].venomTypeId` can be `null`
- `options[].venomType` can be `null`

### 4.3 Get Symptom Configs Grouped By AttributeKey

#### `GET /api/symptom-configs/grouped`

Purpose:

- get active symptom configs grouped by `AttributeKey`

Status:

- `Active`
- Code-verified

Request:

```http
GET /api/symptom-configs/grouped
```

Success response shape:

```json
{
  "success": true,
  "data": {
    "AGE_GROUP": [
      {
        "id": 101,
        "groupName": "BACKGROUND",
        "attributeKey": "AGE_GROUP",
        "attributeLabel": "Độ tuổi của người bị cắn",
        "displayOrder": 1,
        "name": "Trẻ em (dưới 12 tuổi)",
        "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
        "isCritical": true,
        "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
        "category": 2,
        "categoryDisplay": "Modifier",
        "timeScoreList": [
          {
            "minMinutes": 0,
            "maxMinutes": 1440,
            "score": 20
          }
        ],
        "venomTypeId": null,
        "venomType": null,
        "isActive": true,
        "createdAt": "2026-03-07T14:22:21.897471Z",
        "updatedAt": "2026-03-07T14:22:21.897471Z"
      }
    ]
  }
}
```

Frontend recommendation:

- prefer `grouped-for-ui` for rendering forms because it already returns question objects with `options`

## 5. Admin Business + Admin APIs

### 5.1 Recommended Admin CRUD UI

Use controlled fields:

| Field | UI control | Client rule |
|---|---|---|
| `GroupName` | select | one of `BACKGROUND`, `LOCAL`, `CRITICAL` |
| `AttributeKey` | dependent select | filtered by selected `GroupName` |
| `AttributeLabel` | auto-filled text input | default from selected label/key; still editable |
| `DisplayOrder` | number input | `1` to `999` |
| `Name` | text input | required option display name |
| `Description` | textarea | optional |
| `IsCritical` | toggle | when true, enable alert message |
| `AlertMessage` | textarea | recommended when `IsCritical = true` |
| `Category` | segmented control | `Core = 1`, `Modifier = 2` |
| `TimeScoreList` | repeatable table | rows of `minMinutes`, `maxMinutes`, `score` |
| `VenomTypeId` | optional select | nullable; do not block save when empty |
| `IsActive` | toggle | use false to hide from mobile grouped UI |

Admin UI group/key behavior:

- when admin selects `GroupName`, filter `AttributeKey` to the allowed values for that group
- when admin selects a known `AttributeKey`, auto-fill the canonical `AttributeLabel`
- when admin starts from a known label, auto-detect the related `GroupName`, `AttributeKey`, and canonical `AttributeLabel`
- keep `AttributeLabel` editable because backend accepts it as an editable field
- block save in the admin UI if the selected `GroupName` and `AttributeKey` do not match the documented matrix

### 5.2 Filter Symptom Configs

#### `GET /api/symptom-configs/filter`

Purpose:

- get paginated admin list with optional filters

Status:

- `Active`
- Code-verified

Query parameters:

| Name | Type | Required | Notes |
|---|---|---:|---|
| `groupName` | string | no | partial match in current service code |
| `attributeKey` | string | no | partial match in current service code |
| `name` | string | no | partial match in current service code |
| `category` | number | no | `1 = Core`, `2 = Modifier` |
| `isActive` | boolean | no | filter active/inactive |
| `isCritical` | boolean | no | filter critical options |
| `venomTypeId` | number | no | nullable in data |
| `pageNumber` | number | no | inherited pagination field |
| `pageSize` | number | no | inherited pagination field |

Example request:

```http
GET /api/symptom-configs/filter?groupName=BACKGROUND&attributeKey=AGE_GROUP&pageNumber=1&pageSize=20
```

Success response shape:

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 101,
        "groupName": "BACKGROUND",
        "attributeKey": "AGE_GROUP",
        "attributeLabel": "Độ tuổi của người bị cắn",
        "displayOrder": 1,
        "name": "Trẻ em (dưới 12 tuổi)",
        "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
        "isCritical": true,
        "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
        "category": 2,
        "categoryDisplay": "Modifier",
        "timeScoreList": [
          {
            "minMinutes": 0,
            "maxMinutes": 1440,
            "score": 20
          }
        ],
        "venomTypeId": null,
        "venomType": null,
        "isActive": true,
        "createdAt": "2026-03-07T14:22:21.897471Z",
        "updatedAt": "2026-03-07T14:22:21.897471Z"
      }
    ],
    "meta": {
      "pageNumber": 1,
      "pageSize": 20,
      "totalCount": 3,
      "totalPages": 1
    }
  }
}
```

Ordering:

- `displayOrder` ascending
- `attributeKey` ascending
- `name` ascending

### 5.3 Get All Symptom Configs

#### `GET /api/symptom-configs`

Purpose:

- get all symptom configs without pagination

Status:

- `Active`
- Code-verified

Request:

```http
GET /api/symptom-configs
```

Response:

- `ApiResponse<List<SymptomConfigResponse>>`

Admin recommendation:

- use the paginated filter endpoint for large admin tables
- current verified behavior: this endpoint returns active and inactive rows because `GetAllSymptomConfigAsync()` has no `IsActive` filter
- current verified ordering:
  - `displayOrder` ascending
  - `id` ascending
- this endpoint is the all-data admin read endpoint that includes inactive rows

### 5.4 Get Symptom Config By ID

#### `GET /api/symptom-configs/{id}`

Purpose:

- get one symptom config by ID

Status:

- `Active`
- Code-verified
- Admin role required

Request:

```http
GET /api/symptom-configs/101
```

Success response shape:

```json
{
  "success": true,
  "data": {
    "id": 101,
    "groupName": "BACKGROUND",
    "attributeKey": "AGE_GROUP",
    "attributeLabel": "Độ tuổi của người bị cắn",
    "displayOrder": 1,
    "name": "Trẻ em (dưới 12 tuổi)",
    "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
    "isCritical": true,
    "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
    "category": 2,
    "categoryDisplay": "Modifier",
    "timeScoreList": [
      {
        "minMinutes": 0,
        "maxMinutes": 1440,
        "score": 20
      }
    ],
    "venomTypeId": null,
    "venomType": null,
    "isActive": true,
    "createdAt": "2026-03-07T14:22:21.897471Z",
    "updatedAt": "2026-03-07T14:22:21.897471Z"
  }
}
```

Not found:

- returns not found when ID does not exist

### 5.5 Create Symptom Config

#### `POST /api/symptom-configs`

Purpose:

- create one symptom config option

Status:

- `Active`
- Code-verified

Request constraints:

| Field | Required | Constraint |
|---|---:|---|
| `groupName` | yes | max length 100 |
| `attributeKey` | yes | max length 100 |
| `attributeLabel` | yes | max length 500 |
| `displayOrder` | yes | 1 to 999 |
| `name` | yes | max length 300 |
| `description` | no | max length 1000 |
| `isCritical` | no | defaults false |
| `alertMessage` | no | max length 1000 |
| `isActive` | no | defaults true |
| `category` | yes | `1 = Core`, `2 = Modifier` |
| `timeScoreList` | no | list of time score rows |
| `venomTypeId` | no | nullable; when supplied, backend checks it exists |

Example request:

```http
POST /api/symptom-configs
Content-Type: application/json
```

```json
{
  "groupName": "BACKGROUND",
  "attributeKey": "AGE_GROUP",
  "attributeLabel": "Độ tuổi của người bị cắn",
  "displayOrder": 1,
  "name": "Phụ nữ mang thai",
  "description": "Cần theo dõi sát do nguy cơ biến chứng cao",
  "isCritical": true,
  "alertMessage": "Cần ưu tiên tư vấn y tế ngay cho phụ nữ mang thai bị rắn cắn.",
  "isActive": true,
  "category": 2,
  "timeScoreList": [
    {
      "minMinutes": 0,
      "maxMinutes": 1440,
      "score": 25
    }
  ],
  "venomTypeId": null
}
```

Success response:

```json
{
  "success": true,
  "message": "Symptom configuration created successfully!",
  "data": {
    "id": 999,
    "groupName": "BACKGROUND",
    "attributeKey": "AGE_GROUP",
    "attributeLabel": "Độ tuổi của người bị cắn",
    "displayOrder": 1,
    "name": "Phụ nữ mang thai",
    "description": "Cần theo dõi sát do nguy cơ biến chứng cao",
    "isCritical": true,
    "alertMessage": "Cần ưu tiên tư vấn y tế ngay cho phụ nữ mang thai bị rắn cắn.",
    "category": 2,
    "categoryDisplay": "Modifier",
    "timeScoreList": [
      {
        "minMinutes": 0,
        "maxMinutes": 1440,
        "score": 25
      }
    ],
    "venomTypeId": null,
    "venomType": null,
    "isActive": true,
    "createdAt": "2026-04-29T08:30:00Z",
    "updatedAt": "2026-04-29T08:30:00Z"
  }
}
```

Important current behavior:

- backend currently accepts any string combination that passes basic validation
- admin UI must prevent invalid group/key combinations
- this frontend-owned validation is intentional for this scope
- non-null unknown `venomTypeId` returns not found

### 5.6 Update Symptom Config

#### `PUT /api/symptom-configs/{id}`

Purpose:

- update one existing symptom config option

Status:

- `Active`
- Code-verified

Example request:

```http
PUT /api/symptom-configs/101
Content-Type: application/json
```

```json
{
  "groupName": "BACKGROUND",
  "attributeKey": "AGE_GROUP",
  "attributeLabel": "Độ tuổi của người bị cắn",
  "displayOrder": 1,
  "name": "Trẻ em (dưới 12 tuổi)",
  "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
  "isCritical": true,
  "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
  "isActive": true,
  "category": 2,
  "timeScoreList": [
    {
      "minMinutes": 0,
      "maxMinutes": 1440,
      "score": 20
    }
  ],
  "venomTypeId": null
}
```

Success response:

```json
{
  "success": true,
  "message": "Symptom configuration updated successfully!",
  "data": {
    "id": 101,
    "groupName": "BACKGROUND",
    "attributeKey": "AGE_GROUP",
    "attributeLabel": "Độ tuổi của người bị cắn",
    "displayOrder": 1,
    "name": "Trẻ em (dưới 12 tuổi)",
    "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
    "isCritical": true,
    "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
    "category": 2,
    "categoryDisplay": "Modifier",
    "timeScoreList": [
      {
        "minMinutes": 0,
        "maxMinutes": 1440,
        "score": 20
      }
    ],
    "venomTypeId": null,
    "venomType": null,
    "isActive": true,
    "createdAt": "2026-03-07T14:22:21.897471Z",
    "updatedAt": "2026-04-29T08:45:00Z"
  }
}
```

Current update notes:

- backend applies only provided non-null or non-empty fields
- `groupName` is required by DTO validation
- `attributeKey`, `attributeLabel`, `displayOrder`, `name`, `description`, `isCritical`, `alertMessage`, `isActive`, `category`, `timeScoreList`, and `venomTypeId` are optional in update DTO
- send the full editable form from admin UI to avoid accidental mismatch between group, key, and label

### 5.7 Deactivate Symptom Config

Admin can deactivate an option with update:

```http
PUT /api/symptom-configs/101
Content-Type: application/json
```

```json
{
  "groupName": "BACKGROUND",
  "isActive": false
}
```

Effect:

- the option remains stored
- `GET /api/symptom-configs/grouped-for-ui` no longer returns it
- `GET /api/symptom-configs` can still return it
- `GET /api/symptom-configs/filter?isActive=false` can fetch inactive options

### 5.8 Delete Symptom Config

#### `DELETE /api/symptom-configs/{id}`

Purpose:

- delete one symptom config

Status:

- `Active`
- Code-verified behavior: soft delete by setting `IsActive = false`
- Admin role required

Request:

```http
DELETE /api/symptom-configs/101
```

Success response shape:

```json
{
  "success": true,
  "data": "Symptom configuration deleted successfully!"
}
```

Admin warning:

- this endpoint deactivates the row and keeps it fetchable through all-data admin reads
- mobile/patient grouped endpoints will no longer return the deactivated option

## 6. Shared Data Models

### 6.1 GroupedSymptomConfigResponse

```json
{
  "groupName": "BACKGROUND",
  "attributeKey": "AGE_GROUP",
  "attributeLabel": "Độ tuổi của người bị cắn",
  "displayOrder": 1,
  "options": []
}
```

### 6.2 SymptomOptionResponse

```json
{
  "id": 101,
  "name": "Trẻ em (dưới 12 tuổi)",
  "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
  "isCritical": true,
  "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
  "category": 2,
  "categoryDisplay": "Modifier",
  "timeScoreList": [
    {
      "minMinutes": 0,
      "maxMinutes": 1440,
      "score": 20
    }
  ],
  "venomTypeId": null,
  "venomType": null,
  "isActive": true
}
```

### 6.3 SymptomConfigResponse

```json
{
  "id": 101,
  "groupName": "BACKGROUND",
  "attributeKey": "AGE_GROUP",
  "attributeLabel": "Độ tuổi của người bị cắn",
  "displayOrder": 1,
  "name": "Trẻ em (dưới 12 tuổi)",
  "description": "Cơ thể trẻ nhỏ, tỷ lệ độc/cân nặng cao",
  "isCritical": true,
  "alertMessage": "CẢNH BÁO: Trẻ em có cơ thể nhỏ, nọc độc lan nhanh và nguy hiểm gấp nhiều lần người lớn. Cần theo dõi sát!",
  "category": 2,
  "categoryDisplay": "Modifier",
  "timeScoreList": [
    {
      "minMinutes": 0,
      "maxMinutes": 1440,
      "score": 20
    }
  ],
  "venomTypeId": null,
  "venomType": null,
  "isActive": true,
  "createdAt": "2026-03-07T14:22:21.897471Z",
  "updatedAt": "2026-03-07T14:22:21.897471Z"
}
```

### 6.4 TimeScorePoint

```json
{
  "minMinutes": 0,
  "maxMinutes": 1440,
  "score": 20
}
```

Field notes:

- `minMinutes`: lower bound after bite time
- `maxMinutes`: upper bound after bite time
- `score`: score used by backend scoring logic

### 6.5 SymptomCategory

| Value | Name | Meaning |
|---:|---|---|
| 1 | `Core` | max-score category |
| 2 | `Modifier` | additive modifier category |

### 6.6 VenomTypeInfo

```json
{
  "id": 1,
  "name": "Neurotoxic"
}
```

Notes:

- `venomType` can be `null`
- `venomTypeId` can be `null`

## 7. Verified Endpoint List

| Method | Path | Primary client | Status | Notes |
|---|---|---|---|---|
| `GET` | `/api/symptom-configs/grouped-for-ui` | mobile/frontend | active | public recommended patient-facing UI endpoint |
| `GET` | `/api/symptom-configs/grouped` | mobile/frontend/admin | active | public grouped by `AttributeKey` dictionary |
| `GET` | `/api/symptom-configs/filter` | admin | active | admin-only paginated list with filters; can fetch inactive rows |
| `GET` | `/api/symptom-configs` | admin | active | admin-only flat all-data list including inactive rows; ordered by `displayOrder`, then `id` |
| `GET` | `/api/symptom-configs/{id}` | admin | active | admin-only detail by ID |
| `POST` | `/api/symptom-configs` | admin | active | admin-only create option |
| `PUT` | `/api/symptom-configs/{id}` | admin | active | admin-only update option |
| `DELETE` | `/api/symptom-configs/{id}` | admin | active | admin-only soft delete by setting `isActive = false` |

## 8. Changelog

### 2026-04-29 Baseline

- created frontend/mobile useguide for `SymptomConfig`
- documented grouped UI endpoint as preferred rendering endpoint
- documented CRUD request and response examples
- documented target group/key matrix
- documented `VenomTypeId` as optional
- documented frontend-owned group/key matrix validation

### 2026-04-29 User Decision Update

- recorded that frontend/admin UI owns group/key matrix validation
- recorded that backend keeps `AttributeLabel` editable while admin UI auto-fills related metadata
- recorded target backend delete behavior as soft delete through `IsActive = false`
- recorded current all-data read behavior: `GET /api/symptom-configs` includes inactive rows
- recorded `GET /api/symptom-configs` ordering as `displayOrder`, then `id`
- recorded authorization enforcement decision before CRUD was documented as code-verified admin-only

### 2026-04-29 Backend Hardening Update

- updated delete contract to code-verified soft delete
- updated admin read/write endpoints to code-verified `Admin` role requirement
- documented that grouped read endpoints remain public and active-only
