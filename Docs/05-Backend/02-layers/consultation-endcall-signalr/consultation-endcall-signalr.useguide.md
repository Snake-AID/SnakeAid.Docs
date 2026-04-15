---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: useguide
status: active
last_updated: 2026-04-15
api_version: v1
owners: [backend-team]
verification_status: code-verified
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

This document only describes consultation call contracts that are verified in the current codebase.

Current client-visible behavior:

- a participant can get a LiveKit token through `POST /api/consultations/{consultationId}/video-token`
- a participant can connect to `ConsultationHub` to join the consultation SignalR group
- a participant can end the consultation through `POST /api/consultations/{consultationId}/end`
- the timeout flow can currently emit the server-to-client event `RoomExpiring`

Important notes:

- `RoomExpiring` is currently emitted only by the auto-complete timeout flow
- `POST /api/consultations/{consultationId}/end` does not currently emit a SignalR call-termination event
- the planned SignalR trigger for Flutter is tracked in:
  - `consultation-endcall-signalr.introduction.md`
  - `consultation-endcall-signalr.roadmap.md`
  - `consultation-endcall-signalr.sourcecode.md`

Planned direction:

- both timeout-triggered and manual-end-triggered pushes are intended to be consumed by Flutter for the same action: `endcall` and leave the LiveKit room

## 3. Authentication & Authorization

### Expert/Member Operations

- JWT Bearer token is required
- the user must be a participant of the consultation to connect to the hub or get a video token

### Admin Operations

- an admin can get a video token when the role is `Admin`
- this module currently has no admin-specific end-consultation endpoint

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Business Scope

- join the consultation room through a LiveKit token
- join the consultation realtime group through the SignalR hub
- exchange chat messages through the hub
- send volatile UI signals through the hub
- end a consultation through the HTTP endpoint
- receive a timeout event from the backend when the room has elapsed

### 4.2 `POST /api/consultations/{consultationId}/video-token`

Purpose:

- generate a LiveKit access token for the authenticated user to join the consultation room

Auth:

- JWT Bearer token is required
- the user must be a consultation participant or `Admin`

Request:

```http
POST /api/consultations/550e8400-e29b-41d4-a716-446655440000/video-token
Authorization: Bearer <jwt>
```

Success response:

- `ApiResponse<VideoTokenResponse>`

Example response:

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

Field notes:

- `roomName` comes from `Consultation.RoomId`
- if the consultation does not have `RoomId`, the backend returns a validation error
- Flutter should use `token + wsUrl + roomName` to join LiveKit

### 4.3 `POST /api/consultations/{consultationId}/end`

Purpose:

- allow a participant to end a consultation

Auth:

- JWT Bearer token is required
- the actor must be either `Caller` or `Callee` of the consultation

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

Business notes:

- if the consultation is already `Completed`, the service returns idempotently
- for a scheduled consultation, the service updates:
  - `Consultation.Status = Completed`
  - `Consultation.EndTime = UtcNow`
  - `ConsultationBooking.Status = Completed`
  - `ExpertTimeSlot.Status = Booked` when it is currently `Reserved`
- after commit, the backend settles escrow
- this endpoint does not currently close the LiveKit room or emit a SignalR termination event

### 4.4 `GET /hubs/consultation?consultationId={consultationId}`

Purpose:

- establish a SignalR connection to the consultation realtime group

Auth:

- the hub has `[Authorize]`
- the user must be a participant of the consultation

Connection notes:

- hub route: `/hubs/consultation`
- required query string: `consultationId`
- the backend adds the connection to group `consultation:{consultationId}`

Current hub methods the client can call:

- `ReceiveMessage(content, attachmentUrl?)`
- `Signal(eventType, payload)`

### 4.5 Current Server-To-Client Events

#### 4.5.1 `ReceiveMessage`

Purpose:

- broadcast a chat message after it has been saved to the database

Example payload:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440111",
  "content": "Hello",
  "attachmentUrl": null,
  "senderId": "550e8400-e29b-41d4-a716-446655440001",
  "sentAt": "2026-04-14T08:15:00Z"
}
```

#### 4.5.2 `Signal`

Purpose:

- broadcast volatile UI state to the consultation group

Example payload:

```json
{
  "eventType": "camera_toggled",
  "payload": "{\"enabled\":false}",
  "senderId": "550e8400-e29b-41d4-a716-446655440001",
  "timestamp": "2026-04-14T08:16:00Z"
}
```

#### 4.5.3 `RoomExpiring`

Purpose:

- notify participants that the consultation has elapsed in the backend timeout flow

Current source:

- `BookingService.AutoCompleteElapsedScheduledConsultationsAsync(...)`
- `BookingService.AutoCompleteElapsedEmergencyConsultationsAsync(...)`

Example payload:

```json
{
  "consultationId": "550e8400-e29b-41d4-a716-446655440000",
  "reason": "slot_elapsed"
}
```

Field notes:

- this event is currently best-effort
- after this event, the backend calls `DeleteRoomAsync("consultation-{consultationId}")`
- this event does not currently include `roomName`, `endedByRole`, or `shouldLeaveCall`
- this event is not yet documented by the backend as the final Flutter endcall contract

## 5. Admin Business + Admin APIs

There is currently no admin-specific API contract within the scope of the consultation endcall module.

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

## 7. Verified Endpoint List

- `POST /api/consultations/{consultationId}/video-token`
- `POST /api/consultations/{consultationId}/end`
- `GET /hubs/consultation?consultationId={consultationId}`

## 8. Changelog

### 2026-04-15

- Initialized the current-state useguide for consultation endcall-related contracts
- Documented the verified `video-token`, `end`, and `ConsultationHub` connection contract
- Documented that the current timeout flow emits `RoomExpiring`
- Documented that the manual-end flow does not yet emit a consultation termination SignalR event
