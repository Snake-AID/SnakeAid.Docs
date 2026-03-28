---
doc_role: baseline
module: live-kit-cloud
kind: flow
status: active
last_updated: 2026-02-25
owners: [backend-team]
---

# LiveKit Cloud — Usage Guide

External contract for consumers (Flutter mobile, admin portal, integration tests).

---

## Endpoints

### 1. Generate Video Token (Consultation)

```
POST /api/consultations/{consultationId}/video-token
Authorization: Bearer <user-jwt>
```

**Path Parameters:**

| Name           | Type | Description                                              |
| -------------- | ---- | -------------------------------------------------------- |
| consultationId | Guid | Consultation ID (used as room name: `consultation-{id}`) |

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Video token generated successfully",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "wsUrl": "wss://project-id.livekit.cloud",
    "roomName": "consultation-3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
}
```

> **Note**: This endpoint requires an existing consultation. Currently blocked until Consultation module is ready.

---

### 2. Generate Video Token (Demo / Dev)

```
POST /api/videocall/livekit-token/demo/{roomname}
Authorization: Bearer <user-jwt>
```

**Path Parameters:**

| Name     | Type   | Description                                     |
| -------- | ------ | ----------------------------------------------- |
| roomname | string | Custom room name (any string, e.g. `test-room`) |

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Demo video token generated successfully",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "wsUrl": "wss://project-id.livekit.cloud",
    "roomName": "test-room"
  }
}
```

> **Development-only endpoint.** Bypasses consultation validation. Use for testing LiveKit connectivity.

---

### 3. LiveKit Webhook

```
POST /api/videocall/livekit-webhook
Authorization: Bearer <livekit-server-jwt>
Content-Type: application/json
```

**Request Body** (sent by LiveKit Cloud):

```json
{
  "id": "EV_xxx",
  "event": "participant_joined",
  "createdAt": 1709123456,
  "room": {
    "name": "test-room",
    "sid": "RM_xxx"
  },
  "participant": {
    "identity": "user-id",
    "sid": "PA_xxx",
    "name": "User Name"
  }
}
```

**Supported Events:**

| Event                | Description                                        |
| -------------------- | -------------------------------------------------- |
| `room_started`       | Room was created                                   |
| `room_finished`      | Room was closed (all participants left or timeout) |
| `participant_joined` | User joined the room                               |
| `participant_left`   | User left the room                                 |

**Response (200 OK):**

```json
{
  "success": true,
  "message": "Webhook processed"
}
```

---

## Status Codes

| Code | Meaning                                                |
| ---- | ------------------------------------------------------ |
| 200  | Success                                                |
| 400  | Invalid webhook payload / missing Authorization header |
| 401  | Missing or invalid Bearer token (user endpoints)       |
| 403  | User not authorized for this consultation              |
| 500  | Internal server error                                  |

---

## Auth Requirements

| Endpoint                         | Auth Type   | Details                                                                     |
| -------------------------------- | ----------- | --------------------------------------------------------------------------- |
| `livekit-token/{consultationId}` | User JWT    | Must be a valid logged-in user                                              |
| `livekit-token/demo/{roomname}`  | User JWT    | Must be a valid logged-in user                                              |
| `livekit-webhook`                | LiveKit JWT | Signature verified with `ApiSecret` — configured in LiveKit Cloud dashboard |

---

## Role-Based Grants

The generated LiveKit token includes publish permissions based on the user's role:

| Role        | Publish Sources                                              |
| ----------- | ------------------------------------------------------------ |
| Expert      | `camera`, `microphone`, `screen_share`, `screen_share_audio` |
| Other roles | `camera`, `microphone`                                       |

---

## Configuration

LiveKit keys must be configured in `appsettings.json` or Doppler:

```json
{
  "LiveKit": {
    "ApiKey": "<from LiveKit Cloud dashboard>",
    "ApiSecret": "<from LiveKit Cloud dashboard>",
    "WsUrl": "wss://<project-id>.livekit.cloud",
    "TokenTtlMinutes": 10,
    "RoomEmptyTimeoutSeconds": 600
  }
}
```

---

## Flutter Integration

To connect from Flutter:

1. Call `POST /api/videocall/livekit-token/demo/{roomname}` with Bearer token (demo) or `POST /api/consultations/{consultationId}/video-token` (production)
2. Extract `token` and `wsUrl` from response
3. Use `livekit_client` package: `Room.connect(wsUrl, token)`
