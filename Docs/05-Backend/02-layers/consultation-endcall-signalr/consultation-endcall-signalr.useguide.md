---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: useguide
status: active
last_updated: 2026-04-15
api_version: v1
owners: [backend-team]
verification_status: mixed-current-code-implemented-and-runtime-verification-pending
---

# Consultation EndCall SignalR Useguide

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

This document records the current verified consultation termination contract after the full naming migration.

Current verified contract:

- timeout flow emits `ConsultationCallEnded`
- manual-end flow emits `ConsultationCallEnded`
- Flutter treats `ConsultationCallEnded` as the single forced endcall trigger
- the canonical termination contract is the direct hub event `ConsultationCallEnded`

Reported runtime finding to preserve:

- expert may still fail to auto-leave after manual-end-triggered termination signaling

## 3. Authentication & Authorization

### Expert/Member Operations

- JWT Bearer token is required
- the user must be a consultation participant to connect to the hub
- the user must be a consultation participant or `Admin` to request a video token

### Admin Operations

- admin can request a video token
- this module does not currently define an admin-specific termination API

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Business Scope

- join LiveKit consultation room
- join consultation SignalR hub
- receive forced termination pushes
- end consultation through HTTP API

### 4.2 `POST /api/consultations/{consultationId}/video-token`

Purpose:

- generate a LiveKit access token for a consultation room

Auth:

- JWT Bearer token is required
- caller must be consultation participant or `Admin`

Request:

```http
POST /api/consultations/550e8400-e29b-41d4-a716-446655440000/video-token
Authorization: Bearer <jwt>
```

Success response:

```json
{
  "status_code": 200,
  "message": "Video token generated successfully",
  "is_success": true,
  "data": {
    "token": "<livekit-jwt>",
    "wsUrl": "wss://livekit.example.com",
    "roomName": "consultation-550e8400-e29b-41d4-a716-446655440000"
  },
  "error": null
}
```

### 4.3 `POST /api/consultations/{consultationId}/end`

Purpose:

- end a consultation from the backend business perspective

Auth:

- JWT Bearer token is required
- actor must be consultation participant

Current code-verified backend behavior:

- emits `ConsultationCallEnded` to the consultation SignalR group
- attempts to close the LiveKit room
- completes consultation state
- completes booking/slot state when applicable
- settles escrow

Current code-verified backend event payload:

- `consultationId`
- `reason = "participant_ended"`

Current active behavior:

- emits `ConsultationCallEnded` to the consultation SignalR group
- keeps room shutdown and business completion behavior
- keeps `reason = "participant_ended"` for manual end

Request:

```http
POST /api/consultations/550e8400-e29b-41d4-a716-446655440000/end
Authorization: Bearer <jwt>
```

Success response:

```json
{
  "status_code": 200,
  "message": "Consultation ended successfully.",
  "is_success": true,
  "data": "Consultation ended successfully.",
  "error": null
}
```

Important runtime note:

- expert auto-leave after manual-end-triggered termination signaling is still not trusted until it is re-verified end-to-end

### 4.4 `GET /hubs/consultation?consultationId={consultationId}`

Purpose:

- establish consultation realtime connection

Auth:

- hub requires authorization
- user must be consultation participant

Connection notes:

- hub route: `/hubs/consultation`
- required query string: `consultationId`
- group format: `consultation:{consultationId}`
- current backend auto-adds the connection to group `consultation:{consultationId}` during hub connection
- current backend does not require an explicit join hub method for consultation membership

### 4.5 Server-To-Client Termination Event

#### 4.5.1 Active Event: `ConsultationCallEnded`

Purpose:

- canonical server push that terminates the consultation call on Flutter after the migration
- this is the primary realtime contract for consultation call termination
- clients should not depend on generic `Signal` as the primary termination path when integrating with the current backend

Current backend source:

- scheduled timeout cleanup
- emergency timeout cleanup
- manual end via `POST /api/consultations/{consultationId}/end`

Current example payload:

```json
{
  "consultationId": "550e8400-e29b-41d4-a716-446655440000",
  "reason": "timeout"
}
```

Manual-end example payload:

```json
{
  "consultationId": "550e8400-e29b-41d4-a716-446655440000",
  "reason": "participant_ended"
}
```

Current Flutter behavior on receipt of the termination event:

- shows termination feedback
- disconnects room
- calls `endConsultation(...)`
- navigates away from active call

Expected mobile behavior on current backend:

- when `reason = "timeout"`, client should immediately endcall and leave the LiveKit room
- when `reason = "participant_ended"`, client should immediately endcall and leave the LiveKit room
- client should treat both reasons as final termination states, not reminder states

Known reported issue:

- expert may still fail to auto-leave after manual-end-triggered termination signaling

## 5. Admin Business + Admin APIs

There is no admin-specific consultation termination contract in scope here.

## 6. Shared Data Models

### VideoTokenResponse

| Field | Type | Description |
|------|------|-------------|
| token | string | LiveKit access token |
| wsUrl | string | LiveKit websocket URL |
| roomName | string | Consultation room name |

### ApiResponse<T>

| Field | Type | Description |
|------|------|-------------|
| status_code | int | HTTP-like status code in body |
| message | string | Human-readable message |
| is_success | bool | Success flag |
| data | T | Payload |
| error | object? | Error detail when the request fails |

### Active `ConsultationCallEnded` Payload

| Field | Type | Description |
|------|------|-------------|
| consultationId | Guid | Consultation ID |
| reason | string | Current verified values: `timeout`, `participant_ended` |

## 7. Verified Endpoint List

- `POST /api/consultations/{consultationId}/video-token`
- `POST /api/consultations/{consultationId}/end`
- `GET /hubs/consultation?consultationId={consultationId}`

## 8. Changelog

### 2026-04-15

- Recorded `ConsultationCallEnded` as the active consultation termination event
- Recorded `timeout` and `participant_ended` as the active reason values
- Preserved the reported expert-not-auto-leaving runtime issue for post-migration verification
