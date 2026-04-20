---
doc_role: planning
module: consultation-expert-absent
kind: flow
doc_type: usageguide
status: partial
last_updated: 2026-04-21
api_version: v1
owners: [backend-team]
verification_status: code-verified-current-state
---

# Consultation Expert Absent API

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

This document records the `current verified backend contract` relevant to the expert-absent reporting use case.

Important status note:

- the absent-report feature is `not implemented yet`
- there is currently `no active API` for a member to report an absent expert in consultation flow
- the field `Customer Report` is `not present yet` in the verified consultation DTOs

This guide therefore does two things only:

- documents the currently active consultation APIs that mobile/admin can rely on now
- clearly states the missing contract pieces that still need implementation

Confirmed target baseline after implementation:

- `Customer Report` will be the exposed report field
- `CustomerReportSubmittedAt` will exist in v1 persistence

## 3. Authentication & Authorization

### Expert/Member Operations

- JWT Bearer token is required
- `User` role is required for member consultation history
- `Expert` role is required for expert consultation history

### Admin Operations

- JWT Bearer token is required
- `Admin` role is required

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Business Scope

Current member consultation behavior:

- member can list their consultations
- member can see consultation status and room information
- member cannot yet submit an expert-absent report through a consultation API

### 4.2 Member Endpoint

#### 4.2.1 `GET /api/users/me/consultations`

Purpose:

- member retrieves their own consultation history

Status:

- `Active`
- Code-verified

Auth:

- JWT Bearer token is required
- `User` role is required

Query params:

| Field      | Type   | Required | Notes                                                                 |
| ---------- | ------ | -------- | --------------------------------------------------------------------- |
| pageNumber | int    | No       | Default `1`, range `>= 1`                                             |
| pageSize   | int    | No       | Default `10`, range `1..100`                                          |
| status     | string | No       | `Scheduled`, `Ongoing`, `Completed`, `Cancelled`, `UserAbsent`, `ExpertAbsent`, `AllAbsent` |
| type       | string | No       | `Scheduled` or `Emergency`                                            |

Example request:

```http
GET /api/users/me/consultations?pageNumber=1&pageSize=10&type=Scheduled
Authorization: Bearer <member-jwt>
```

Success response:

- `ApiResponse<PagingResponse<MyConsultationResponse>>`

Verified example response shape:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {
    "items": [
      {
        "consultationId": "8ce96758-71b5-4310-bc35-d83525b2c54f",
        "type": "Scheduled",
        "status": "Completed",
        "expertId": "ba833fc7-6856-48b2-b032-9c5d985729d1",
        "expertName": "Pham Thi D",
        "roomId": "consultation-8ce96758-71b5-4310-bc35-d83525b2c54f",
        "startTime": "2026-04-09T14:00:00Z",
        "endTime": "2026-04-09T14:30:00Z",
        "price": 150000,
        "problemDescription": "Snakebite on finger",
        "bookingId": "ef54ec06-bb65-47d1-a7c5-db86aad6a49b",
        "slotStartTime": "2026-04-09T14:00:00Z",
        "slotEndTime": "2026-04-09T14:30:00Z",
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

Field notes:

- there is currently `no member/customer report field` in this response
- `problemDescription` is booking problem content, not an expert-absent report
- `status = ExpertAbsent` is supported by enum filtering, but there is currently no verified consultation API that sets it from a member report action

### 4.3 Missing Member Contract For This Module

The following contract is `not active yet` and should not be consumed by mobile yet:

- member report endpoint for absent expert
- `Customer Report` field in `MyConsultationResponse`

## 5. Admin Business + Admin APIs

### 5.1 Business Scope

Current admin consultation behavior:

- admin can list consultations across the system
- admin can open one consultation detail
- admin cannot yet see a dedicated member/customer absent-report field because the backend does not expose one yet

### 5.2 Admin Endpoint

#### 5.2.1 `GET /api/admin/consultations`

Purpose:

- admin retrieves a paged list of consultations across the system

Status:

- `Active`
- Code-verified

Auth:

- JWT Bearer token is required
- `Admin` role is required

Query params:

| Field      | Type   | Required | Notes                                                                 |
| ---------- | ------ | -------- | --------------------------------------------------------------------- |
| pageNumber | int    | No       | Default `1`, range `>= 1`                                             |
| pageSize   | int    | No       | Default `10`, range `1..100`                                          |
| status     | string | No       | `Scheduled`, `Ongoing`, `Completed`, `Cancelled`, `UserAbsent`, `ExpertAbsent`, `AllAbsent` |
| type       | string | No       | `Scheduled` or `Emergency`                                            |

Example request:

```http
GET /api/admin/consultations?pageNumber=1&pageSize=10&type=Scheduled
Authorization: Bearer <admin-jwt>
```

Success response:

- `ApiResponse<PagingResponse<AdminConsultationResponse>>`

Verified field notes:

- there is currently `no customer/member report field`
- `problemDescription` is the scheduled-booking problem description
- scheduled and emergency consultations share one admin response DTO

#### 5.2.2 `GET /api/admin/consultations/{consultationId}`

Purpose:

- admin retrieves one consultation detail by `consultationId`

Status:

- `Active`
- Code-verified

Auth:

- JWT Bearer token is required
- `Admin` role is required

Route params:

| Field          | Type | Required | Notes                    |
| -------------- | ---- | -------- | ------------------------ |
| consultationId | guid | Yes      | Existing `Consultation.Id` |

Example request:

```http
GET /api/admin/consultations/8ce96758-71b5-4310-bc35-d83525b2c54f
Authorization: Bearer <admin-jwt>
```

Success response:

- `ApiResponse<AdminConsultationResponse>`

Verified field notes:

- there is currently `no customer/member report field`
- admin can see status, room, booking data, and emergency-request data
- admin cannot yet inspect absent-report text because it is not exposed by current code

### 5.3 Missing Admin Contract For This Module

The following contract is `not active yet` and should not be consumed by admin UI yet:

- `Customer Report` field on admin consultation list response
- `Customer Report` field on admin consultation detail response

## 6. Shared Data Models

### 6.1 Current `MyConsultationResponse`

Current verified shape includes:

- `consultationId`
- `type`
- `status`
- `expertId`
- `expertName`
- `roomId`
- `startTime`
- `endTime`
- `price`
- `problemDescription`
- `bookingId`
- `slotStartTime`
- `slotEndTime`
- `emergencyRequestId`

Current verified shape does not include:

- `customerReport`

### 6.2 Current `AdminConsultationResponse`

Current verified shape includes:

- consultation identity
- type and status
- user and expert identifiers/names
- room and timing data
- price
- `problemDescription`
- booking metadata
- emergency-request metadata
- slot timing

Current verified shape does not include:

- `customerReport`

## 7. Verified Endpoint List

Active endpoints relevant to this module:

- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`
- `GET /api/admin/consultations`
- `GET /api/admin/consultations/{consultationId}`

Not implemented yet:

- member absent-report endpoint for consultation
- member/customer report field in consultation DTOs

## 8. Changelog

### 2026-04-21

- Initialized use guide for the consultation expert-absent module
- Recorded the current verified backend state
- Explicitly marked absent-report contracts as not implemented yet
