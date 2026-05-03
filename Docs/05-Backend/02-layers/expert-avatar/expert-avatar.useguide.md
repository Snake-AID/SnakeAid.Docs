---
doc_role: integration
module: expert-avatar
kind: response-contract-amendment
doc_type: useguide
status: amendment-planning
last_updated: 2026-05-03
api_version: v1
owners: [backend-team]
verification_status: code-inspected
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

This guide is written as a contract amendment over the already implemented backend response.

Implemented today:

- `GET /api/users/me/consultations` returns `expertAvatarUrl`.
- `GET /api/experts/me/consultations` returns `expertAvatarUrl` for the authenticated expert.

Frontend amendment:

- `GET /api/users/me/consultations` remains correct.
- `GET /api/experts/me/consultations` must replace `expertAvatarUrl` with `userAvatarUrl` for the member/rescuer shown on the expert screen.

Client null rule:

- `expertAvatarUrl = null` means expert avatar is unavailable.
- `userAvatarUrl = null` means member/rescuer avatar is unavailable.

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
- mobile renders consulted expert info

Status:

- implemented and correct for this avatar requirement

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
- mobile renders the member/rescuer participant on the expert screen

Current implemented response item:

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

Amended response item:

```json
{
  "consultationId": "1b1d73f4-a32d-4a3c-95f5-bb0c3b7d8c01",
  "type": "Scheduled",
  "status": "Completed",
  "userId": "bd4f9968-53f8-4757-a3a0-53f2049a8c2f",
  "userName": "Nguyen Van A",
  "userAvatarUrl": "https://cdn.example.com/avatars/member-a.png",
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

- `userId`, `userName`, and `userAvatarUrl` describe the other participant shown to the expert.
- for scheduled rows, `userAvatarUrl` is the member avatar.
- for emergency rows, `userAvatarUrl` is the rescuer avatar.
- `expertAvatarUrl` must be removed from this endpoint.

## 5. Shared Data Models

### Member History Expert Display Fields

```json
{
  "expertId": "4c86152d-2476-4f91-b4fc-4b46726b81f5",
  "expertName": "Dr. Expert A",
  "expertAvatarUrl": "https://cdn.example.com/avatars/expert-a.png"
}
```

### Expert History Participant Display Fields

```json
{
  "userId": "bd4f9968-53f8-4757-a3a0-53f2049a8c2f",
  "userName": "Nguyen Van A",
  "userAvatarUrl": "https://cdn.example.com/avatars/member-a.png"
}
```

## 6. Verified Endpoint List

| Endpoint | Role | Avatar contract |
| --- | --- | --- |
| `GET /api/users/me/consultations` | User | implemented `expertAvatarUrl` |
| `GET /api/experts/me/consultations` | Expert | replace `expertAvatarUrl` with `userAvatarUrl` |

## 7. Changelog

### 2026-05-03

- Reframed docs as an amendment over already implemented backend work.
- Kept member endpoint as implemented: `expertAvatarUrl`.
- Changed expert endpoint target: remove `expertAvatarUrl`, add participant `userAvatarUrl` for member/rescuer avatar.
