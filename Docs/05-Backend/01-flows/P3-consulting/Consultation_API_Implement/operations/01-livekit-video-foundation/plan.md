---
doc_role: operation
operation_id: 01-INIT-livekit-video-foundation
type: INIT
status: done
created_at: 2026-02-24
merged_from: [live-kit-cloud, 01-FEAT-video-call, 02-FEAT-video-call-demo]
affects:
  - SnakeAid.Service/Interfaces/ILiveKitApi.cs
  - SnakeAid.Service/Interfaces/ILiveKitService.cs
  - SnakeAid.Service/Implements/LiveKitService.cs
  - SnakeAid.Core/Settings/LiveKitOptions.cs
  - SnakeAid.Core/Requests/LiveKit/VideoGrants.cs
  - SnakeAid.Core/Requests/LiveKit/CreateRoomRequest.cs
  - SnakeAid.Core/Requests/LiveKit/DeleteRoomRequest.cs
  - SnakeAid.Core/Requests/LiveKit/LiveKitWebhookPayload.cs
  - SnakeAid.Core/Responses/LiveKit/VideoTokenResponse.cs
  - SnakeAid.Core/Responses/LiveKit/RoomInfoResponse.cs
  - SnakeAid.Core/Responses/LiveKit/ListRoomsResponse.cs
  - SnakeAid.Api/Controllers/VideoCallController.cs
  - SnakeAid.Api/DI/DependencyInjection.cs
  - SnakeAid.Api/appsettings.json
---

# Operation 01: LiveKit Video Foundation

## 1. As-Is

Before the consultation flow was implemented end-to-end, the backend had no video-call provider integration.

- no LiveKit service abstraction
- no consultation video-token endpoint
- no LiveKit webhook receiver
- no room-management integration
- no dev-only endpoint to test Flutter connectivity before the full consultation business flow was ready

This operation established the technical foundation first, so later consultation operations could reuse an existing provider boundary instead of designing video transport in parallel with booking, payment, and room logic.

## 2. Gap Analysis

| Gap | Description |
|---|---|
| Provider integration missing | Consultation needed a managed video-call provider with backend-issued join tokens |
| .NET server SDK missing | LiveKit has no official .NET server SDK, so JWT + HTTP integration had to be implemented manually |
| Room lifecycle missing | Backend needed create/list/delete room capability and webhook validation |
| Early mobile testing blocked | Flutter/mobile needed a demo token endpoint before consultation ownership validation was fully wired |
| Provider naming vs domain naming | Initial provider-specific routes had to be normalized into the consultation/video-call route layout used by the flow |

## 3. To-Be Design

### Provider boundary

- Introduce `ILiveKitApi` as the Refit client for Twirp room APIs
- Introduce `ILiveKitService` / `LiveKitService` for token generation, room operations, and webhook validation
- Configure `LiveKitOptions` from application settings

### Endpoint surface

- `POST /api/consultations/{consultationId}/video-token`
  - authenticated participant only
  - validates consultation ownership/state
  - returns `{ token, wsUrl, roomName }`
- `POST /api/videocall/livekit-token/demo/{roomname}`
  - authenticated user
  - bypasses consultation validation
  - exists only for development/testing of LiveKit connectivity
- `POST /api/videocall/livekit-webhook`
  - anonymous endpoint
  - validates LiveKit-signed webhook payload

### Runtime rules introduced

- room naming convention: `consultation-{consultationId}`
- token TTL: 10 minutes
- expert publish grants include screen-share sources
- non-expert participants publish camera/microphone only
- webhook handling recognizes `room_started`, `participant_joined`, `participant_left`, and `room_finished`

### Analysis references

- `analysis/01-livekit-domain-context.md`
- `analysis/02-livekit-source-map.md`
- `analysis/03-livekit-client-contract.md`

## 4. Impacted Components

| Component | Change |
|---|---|
| `ILiveKitApi` | new Refit integration boundary |
| `ILiveKitService` / `LiveKitService` | new service abstraction for JWT, room APIs, webhook validation |
| `LiveKitOptions` | new settings object for API key/secret, wsUrl, TTL, empty timeout |
| `Core/Requests/LiveKit/*` | new request DTOs and webhook payload models |
| `Core/Responses/LiveKit/*` | new token/room response DTOs |
| `VideoCallController` | provider-backed token + webhook endpoints under consultation/video-call routes |
| `DependencyInjection` | new Refit client registration and service registration |
| `appsettings.json` | new `LiveKit` section |

## 5. Risks & Constraints

| Risk | Mitigation |
|---|---|
| No official .NET SDK | isolate provider logic behind `ILiveKitService` and Refit client |
| JWT claim shape errors | validate generated token claims against LiveKit expectations |
| Webhook tampering | verify signature with `ApiSecret` before processing |
| Route churn during consultation build-out | keep provider logic in `VideoCallController` while final participant route remains under `api/consultations/{id}/video-token` |
| Early testing without full consultation flow | expose explicit dev-only demo token endpoint |

## 6. Validation Plan

- unit test token generation and claim payload
- unit test webhook signature validation (accept valid, reject tampered)
- verify consultation token endpoint rejects non-participants / invalid states
- verify demo endpoint returns a valid token for development
- verify webhook endpoint accepts supported LiveKit events without exposing provider secrets
