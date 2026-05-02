---
doc_role: integration
module: expert-avatar
kind: response-contract
doc_type: useguide
status: implemented
last_updated: 2026-05-03
api_version: v1
owners: [backend-team]
verification_status: code-verified
---

# Expert Avatar Useguide

## 1. Table Of Contents

- [1. Table Of Contents](#1-table-of-contents)
- [2. Overview](#2-overview)
- [3. Authentication & Authorization](#3-authentication--authorization)
- [4. Expert/Member Business + Expert/Member APIs](#4-expertmember-business--expertmember-apis)
- [5. Shared Data Models](#5-shared-data-models)
- [6. Verified Endpoint List](#6-verified-endpoint-list)
- [7. Changelog](#7-changelog)

## 2. Overview

This document records the implemented contract for adding expert avatar data to consultation-history responses.

Current active behavior:

- only two endpoints are in scope:
  - `GET /api/users/me/consultations`
  - `GET /api/experts/me/consultations`
- both endpoints should include nullable `expertAvatarUrl`
- existing fields should stay unchanged
- `expertAvatarUrl = null` means no expert avatar is available, and mobile should render its placeholder

## 3. Authentication & Authorization

### Member Consultation History

- JWT Bearer token is required.
- `User` role is required.

### Expert Consultation History

- JWT Bearer token is required.
- `Expert` role is required.

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Member Consultation History

#### `GET /api/users/me/consultations`

Purpose:

- member retrieves consultation history
- mobile can render consulted expert display info without an extra profile request

Auth:

- JWT Bearer token is required
- `User` role is required

Query params:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `pageNumber` | int | No | Default from pagination request |
| `pageSize` | int | No | Default from pagination request |
| `type` | string | No | `Scheduled` or `Emergency` |
| `status` | string | No | Consultation status filter |

Success response item:

```json
{
  "consultationId": "1b1d73f4-a32d-4a3c-95f5-bb0c3b7d8c01",
  "type": "Scheduled",
  "status": "Completed",
  "expertId": "4c86152d-2476-4f91-b4fc-4b46726b81f5",
  "expertName": "Dr. Expert A",
  "expertAvatarUrl": "https://cdn.example.com/avatars/expert-a.png",
  "roomId": "consultation-1b1d73f4-a32d-4a3c-95f5-bb0c3b7d8c01",
  "startTime": "2026-04-20T09:00:00Z",
  "endTime": "2026-04-20T09:25:00Z",
  "price": 500000,
  "problemDescription": "Snake bite concern",
  "customerReport": null,
  "customerReportSubmittedAt": null,
  "bookingId": "c3b5f18d-8871-4ea0-97d2-6ac4b5fd4f2f",
  "slotStartTime": "2026-04-20T09:00:00Z",
  "slotEndTime": "2026-04-20T09:30:00Z",
  "emergencyRequestId": null
}
```

### 4.2 Expert Consultation History

#### `GET /api/experts/me/consultations`

Purpose:

- expert retrieves own consultation history
- mobile can render expert avatar for the authenticated expert without an extra profile request

Auth:

- JWT Bearer token is required
- `Expert` role is required

Query params:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `pageNumber` | int | No | Default from pagination request |
| `pageSize` | int | No | Default from pagination request |
| `type` | string | No | `Scheduled` or `Emergency` |
| `status` | string | No | Consultation status filter |

Success response item:

```json
{
  "consultationId": "1b1d73f4-a32d-4a3c-95f5-bb0c3b7d8c01",
  "type": "Scheduled",
  "status": "Completed",
  "userId": "bd4f9968-53f8-4757-a3a0-53f2049a8c2f",
  "userName": "Nguyen Van A",
  "expertAvatarUrl": "https://cdn.example.com/avatars/expert-a.png",
  "roomId": "consultation-1b1d73f4-a32d-4a3c-95f5-bb0c3b7d8c01",
  "startTime": "2026-04-20T09:00:00Z",
  "endTime": "2026-04-20T09:25:00Z",
  "grossPrice": 500000,
  "netPrice": 400000,
  "bookingId": "c3b5f18d-8871-4ea0-97d2-6ac4b5fd4f2f",
  "slotStartTime": "2026-04-20T09:00:00Z",
  "slotEndTime": "2026-04-20T09:30:00Z",
  "emergencyRequestId": null
}
```

Important client notes:

- `userId` and `userName` still mean the other participant in the expert-history response.
- `expertAvatarUrl` means the authenticated expert's avatar.
- This scope does not add `userAvatarUrl`.

## 5. Shared Data Models

### Member History Expert Display Fields

```json
{
  "expertId": "4c86152d-2476-4f91-b4fc-4b46726b81f5",
  "expertName": "Dr. Expert A",
  "expertAvatarUrl": "https://cdn.example.com/avatars/expert-a.png"
}
```

### Expert History Avatar Field

```json
{
  "expertAvatarUrl": "https://cdn.example.com/avatars/expert-a.png"
}
```

Null behavior:

```json
{
  "expertAvatarUrl": null
}
```

## 6. Verified Endpoint List

| Endpoint | Role | Avatar state |
| --- | --- | --- |
| `GET /api/users/me/consultations` | User | returns nullable `expertAvatarUrl` |
| `GET /api/experts/me/consultations` | Expert | returns nullable `expertAvatarUrl` |

## 7. Changelog

### 2026-05-03

- Implemented nullable `expertAvatarUrl` for `GET /api/users/me/consultations` and `GET /api/experts/me/consultations`.
- Removed information not directly related to `GET /api/users/me/consultations` and `GET /api/experts/me/consultations`.

### 2026-05-02

- Created planning useguide.
- Adjusted scope by user decision: only `GET /api/users/me/consultations` and `GET /api/experts/me/consultations` remain in scope.
