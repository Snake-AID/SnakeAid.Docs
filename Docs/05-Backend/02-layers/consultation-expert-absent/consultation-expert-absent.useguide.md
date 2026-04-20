---
doc_role: planning
module: consultation-expert-absent
kind: layer
doc_type: useguide
status: draft
last_updated: 2026-04-20
api_version: v1
owners: [backend-team]
verification_status: current-contract-code-verified-planned-delta-drafted
---

# Consultation Expert Absent Useguide

## 1. Table Of Contents

- [1. Table Of Contents](#1-table-of-contents)
- [2. Overview](#2-overview)
- [3. Authentication & Authorization](#3-authentication--authorization)
- [4. Expert/Member Business + Expert/Member APIs](#4-expertmember-business--expertmember-apis)
- [5. Admin Business + Admin APIs](#5-admin-business--admin-apis)
- [6. Shared Data Models](#6-shared-data-models)
- [7. Verified Endpoint List](#7-verified-endpoint-list)
- [8. Changelog](#8-changelog)

## 2. Overview

This document describes the absent-expert reporting contract for consultation video calls.

Business objective:

- if member joins at consultation time but expert does not join, member can report the incident so admin can review expert absence.

Current code-verified state:

- consultation APIs exist for join/history/end flows
- there is no dedicated absent-expert report endpoint
- there is no `customerReport` field in member/admin consultation responses

Planned delta in this task group:

- add report field storage on consultation
- add member report submission endpoint
- expose report field in admin consultation list/detail

Important naming note for this draft:

- requirement text uses both `CustomerReport` and `Member Report`
- this draft assumes:
  - request field: `memberReport`
  - stored/admin output field: `customerReport`
- final naming is tracked in `consultation-expert-absent.hallucination.md`

## 3. Authentication & Authorization

### Expert/Member Operations

- JWT Bearer token is required
- report endpoint is intended for role `User` (member) only
- member must be the consultation owner (`CallerId`) for report submission

### Admin Operations

- JWT Bearer token is required
- role `Admin` required for admin consultation history endpoints

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Business Scope

Member-side goals in this module:

- view consultation history including report status/content
- submit absent-expert report for eligible consultation

Expert-side goals in this module:

- no direct expert report API is added in this task group

### 4.2 Current Active Endpoint: `GET /api/users/me/consultations`

Purpose:

- retrieve member consultation history

Status:

- `Active` (code-verified)

Auth:

- JWT Bearer token required
- role `User`

Current response characteristics:

- returns both scheduled and emergency consultation history
- currently does not include absent report field

Current response example (simplified):

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "items": [
      {
        "consultationId": "89f9c73a-1a65-449a-a9e9-5d7f85705e34",
        "type": "Scheduled",
        "status": "Completed",
        "expertId": "d8a7f855-2f56-4fa0-b954-5eab3249a478",
        "expertName": "Dr. Expert",
        "roomId": "consultation-89f9c73a-1a65-449a-a9e9-5d7f85705e34",
        "startTime": "2026-04-20T09:00:00Z",
        "endTime": "2026-04-20T09:30:00Z",
        "price": 150000,
        "problemDescription": "snakebite swelling",
        "bookingId": "f3640677-6e8f-4da4-b5de-bbe58f7b7d8c",
        "slotStartTime": "2026-04-20T09:00:00Z",
        "slotEndTime": "2026-04-20T09:30:00Z",
        "emergencyRequestId": null
      }
    ],
    "meta": {
      "total_pages": 1,
      "total_items": 1,
      "current_page": 1,
      "page_size": 10
    }
  },
  "error": null
}
```

Planned response delta:

- add `customerReport` field in each `MyConsultationResponse` item

### 4.3 Planned Endpoint: `POST /api/consultations/{consultationId}/expert-absence-report`

Purpose:

- member submits absent-expert report for a specific consultation

Status:

- `Planned` (not implemented yet)

Auth:

- JWT Bearer token required
- role `User`
- requester must be the consultation owner/member side

Request:

```http
POST /api/consultations/89f9c73a-1a65-449a-a9e9-5d7f85705e34/expert-absence-report
Authorization: Bearer <member-jwt>
Content-Type: application/json
```

Request body (draft):

```json
{
  "memberReport": "I joined the consultation room at 09:00 UTC and waited 12 minutes. Expert did not join."
}
```

Request constraints (draft):

- `memberReport` required
- trimmed non-empty text
- proposed max length: `2000`
- consultation must be report-eligible (exact status/time-window rule pending final decision)

Success response (draft):

```json
{
  "status_code": 200,
  "message": "Expert absence report submitted successfully.",
  "is_success": true,
  "data": {
    "consultationId": "89f9c73a-1a65-449a-a9e9-5d7f85705e34",
    "customerReport": "I joined the consultation room at 09:00 UTC and waited 12 minutes. Expert did not join.",
    "updatedAt": "2026-04-20T09:12:35Z"
  },
  "error": null
}
```

Expected error responses (draft):

| Status Code | Error Code | When |
|---|---|---|
| 400 | VALIDATION_ERROR | report text invalid / empty / too long |
| 401 | UNAUTHORIZED | missing or invalid token |
| 403 | FORBIDDEN | caller is not eligible to report this consultation |
| 404 | NOT_FOUND | consultation does not exist |
| 409 | CONFLICT | consultation is not in reportable state/window |

### 4.4 Example Curl (Planned)

```bash
curl -X POST "https://api.example.com/api/consultations/89f9c73a-1a65-449a-a9e9-5d7f85705e34/expert-absence-report" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "memberReport": "I joined on time but expert was absent."
  }'
```

## 5. Admin Business + Admin APIs

### 5.1 Business Scope

Admin goals in this module:

- view member-submitted absent-expert report in consultation history list and detail
- no new admin action endpoint is included in this task group

### 5.2 Current Active Endpoint: `GET /api/admin/consultations`

Purpose:

- paged admin consultation history (scheduled + emergency)

Status:

- `Active` (code-verified)

Current response characteristics:

- returns `AdminConsultationResponse`
- currently does not include `customerReport`

Planned response delta:

- include `customerReport` in each item

Planned item snippet:

```json
{
  "consultationId": "89f9c73a-1a65-449a-a9e9-5d7f85705e34",
  "type": "Scheduled",
  "status": "Completed",
  "userId": "2a603164-7038-4f34-9295-5e45ccb8ab5e",
  "userName": "Member One",
  "expertId": "d8a7f855-2f56-4fa0-b954-5eab3249a478",
  "expertName": "Dr. Expert",
  "customerReport": "I joined the room but expert did not join."
}
```

### 5.3 Current Active Endpoint: `GET /api/admin/consultations/{consultationId}`

Purpose:

- admin consultation detail

Status:

- `Active` (code-verified)

Planned response delta:

- include `customerReport` in detail payload

Planned detail snippet:

```json
{
  "consultationId": "89f9c73a-1a65-449a-a9e9-5d7f85705e34",
  "type": "Scheduled",
  "status": "Completed",
  "problemDescription": "snakebite swelling",
  "customerReport": "I joined the room but expert did not join."
}
```

## 6. Shared Data Models

### 6.1 Draft `ReportExpertAbsentRequest`

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| memberReport | string | Yes | non-empty, max 2000 (draft) | member text report for expert absence |

### 6.2 Draft `ReportExpertAbsentResponse`

| Field | Type | Description |
|---|---|---|
| consultationId | Guid | target consultation id |
| customerReport | string? | persisted report text |
| updatedAt | datetime | last update timestamp |

### 6.3 `MyConsultationResponse` (Planned Delta)

| Field | Type | Description |
|---|---|---|
| customerReport | string? | absent-expert report text submitted by member |

### 6.4 `AdminConsultationResponse` (Planned Delta)

| Field | Type | Description |
|---|---|---|
| customerReport | string? | member-submitted report text for admin review |

## 7. Verified Endpoint List

### 7.1 Current Active Endpoints

- `GET /api/users/me/consultations`
- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`
- `POST /api/consultations/{consultationId}/video-token`
- `POST /api/consultations/{consultationId}/end`

### 7.2 Planned Endpoint In This Module

- `POST /api/consultations/{consultationId}/expert-absence-report`

## 8. Changelog

### 2026-04-20

- Created planning useguide for ConsultaionExpertAbsent task group.
- Recorded current active API state (no absent-report contract yet).
- Added draft API contract for member absent-expert reporting.
- Added admin response delta plan for `customerReport` visibility.
