---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: useguide
status: active
last_updated: 2026-04-15
api_version: v1
owners: [backend-team]
verification_status: mixed-current-code-and-reported-runtime-behavior
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

This document records the consultation termination contract as it exists today and the exact target state that should be implemented next.

Current direction:

- keep `RoomExpiring`
- reuse it for both timeout and manual end
- do not introduce a second termination event name

Current truth:

- timeout flow already emits `RoomExpiring`
- manual-end flow in the current backend workspace does not yet emit `RoomExpiring`
- Flutter already treats `RoomExpiring` as a forced endcall trigger

Reported runtime finding to preserve:

- when manual end triggers `RoomExpiring`, expert may still fail to auto-leave the room

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

- completes consultation state
- completes booking/slot state when applicable
- settles escrow

Current code-verified backend limitation:

- does not currently emit `RoomExpiring`
- does not currently close the LiveKit room in this workspace

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

Target behavior after implementation:

- emit `RoomExpiring`
- close the active room
- keep business completion behavior

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

### 4.5 Current Server-To-Client Termination Event

#### 4.5.1 `RoomExpiring`

Purpose:

- force the active consultation call to terminate on Flutter

Current backend source:

- scheduled timeout cleanup
- emergency timeout cleanup

Target backend source:

- scheduled timeout cleanup
- emergency timeout cleanup
- manual end via `POST /api/consultations/{consultationId}/end`

Current example payload:

```json
{
  "consultationId": "550e8400-e29b-41d4-a716-446655440000",
  "reason": "slot_elapsed"
}
```

Current Flutter behavior on receipt:

- shows termination feedback
- disconnects room
- calls `endConsultation(...)`
- navigates away from active call

Known reported issue:

- expert may still fail to auto-leave after manual-end-triggered `RoomExpiring`

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

### Current `RoomExpiring` Payload

| Field | Type | Description |
|------|------|-------------|
| consultationId | Guid | Consultation ID |
| reason | string | Current verified value: `slot_elapsed` |

### Target `RoomExpiring` Payload

| Field | Type | Description |
|------|------|-------------|
| consultationId | Guid | Consultation ID |
| reason | string | Should support both timeout and manual-end semantics |

## 7. Verified Endpoint List

- `POST /api/consultations/{consultationId}/video-token`
- `POST /api/consultations/{consultationId}/end`
- `GET /hubs/consultation?consultationId={consultationId}`

## 8. Changelog

### 2026-04-15

- Rewrote useguide to distinguish current verified behavior from target behavior
- Preserved that timeout already emits `RoomExpiring`
- Preserved that manual-end backend emission is still missing in the current workspace
- Preserved the reported expert-not-auto-leaving runtime issue for later verification
