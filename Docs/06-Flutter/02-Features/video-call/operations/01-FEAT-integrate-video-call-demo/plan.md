---
doc_role: operation
operation_id: FEAT-integrate-video-call-demo
type: FEAT
status: draft
created_at: 2026-02-25
supersedes_partial: FEAT-integrate-video-call
backend_reference:
  module: live-kit-cloud
  operations:
    - FEAT-video-call
    - FEAT-video-call-demo
  path: Docs/05-Backend/01-flows/P3-consulting/live-kit-cloud/
---

# FEAT-integrate-video-call-demo — Flutter Plan

## 1. Objective

Implement a **standalone demo video call screen** that uses the `livekit-token/demo/{roomname}` endpoint. This validates the entire LiveKit integration (SDK, permissions, core service, UI) without requiring the Consultation module (P3).

> [!NOTE]
> This is Operation 1 of 2. Operation 2 (`FEAT-integrate-video-call-consultation`) will integrate video call into the consultation flow when P3 is ready.

## 2. Backend Contract Reference

| Item               | Value                                               |
| ------------------ | --------------------------------------------------- |
| **Backend module** | `P3-consulting/live-kit-cloud`                      |
| **Endpoint used**  | `POST api/videocall/livekit-token/demo/{roomname}`  |
| **Auth**           | `[Authorize]` — requires logged-in user             |
| **Response**       | `{ token, wsUrl, roomName }` — `VideoTokenResponse` |

## 3. Code Culture Reference

| Pattern            | Convention                                                                     | Reference                                           |
| ------------------ | ------------------------------------------------------------------------------ | --------------------------------------------------- |
| Feature folder     | `lib/features/{feature_name}/models\|providers\|repository\|screens\|widgets/` | `features/emergency/`, `features/consultation/`     |
| Shared services    | `lib/core/services/{name}_service.dart`                                        | `core/services/rescuer_signalr_service.dart`        |
| HTTP provider      | `lib/core/providers/http_provider.dart` → `httpServiceProvider`                | `http_provider.dart`                                |
| Routing            | `go_router` — routes in `lib/app/router.dart`                                  | `router.dart`                                       |
| File naming        | `snake_case.dart`                                                              | All files                                           |
| Riverpod providers | `Provider<T>((ref) { ... })` for services                                      | `httpServiceProvider`, `healthCheckServiceProvider` |

## 4. As-Is (Flutter State)

| Component                   | Status                                                                             |
| --------------------------- | ---------------------------------------------------------------------------------- |
| `features/consultation/`    | Exists with `models/providers/repository/screens/widgets/` (stub Class.dart files) |
| `livekit_client` dependency | Not installed                                                                      |
| Video call feature folder   | Does NOT exist                                                                     |
| Core LiveKit service        | Does NOT exist                                                                     |
| Demo screen route           | Does NOT exist in `router.dart`                                                    |

## 5. Gap Analysis

| Gap                    | Required Action                                     | Target Location                                                 |
| ---------------------- | --------------------------------------------------- | --------------------------------------------------------------- |
| **New dependency**     | Add `livekit_client: ^2.5.4` to `pubspec.yaml`      | `pubspec.yaml`                                                  |
| **New core service**   | `LiveKitService` — wraps Room lifecycle             | `lib/core/services/livekit_service.dart`                        |
| **New feature folder** | `video_call` with standard subfolders               | `lib/features/video_call/`                                      |
| **New model**          | `VideoTokenResponse` DTO                            | `lib/features/video_call/models/video_token_response.dart`      |
| **New repository**     | `VideoCallRepository` — `getDemoVideoToken()` only  | `lib/features/video_call/repository/video_call_repository.dart` |
| **New providers**      | Video call state providers for demo                 | `lib/features/video_call/providers/video_call_provider.dart`    |
| **New screen**         | `DemoVideoCallScreen` — join by room name           | `lib/features/video_call/screens/demo_video_call_screen.dart`   |
| **New widgets**        | Video tile, controls bar (reusable for Operation 2) | `lib/features/video_call/widgets/`                              |
| **New route**          | `/demo-video-call` in `router.dart`                 | `lib/app/router.dart`                                           |
| **Platform config**    | Android Bluetooth perms, iOS background audio       | `AndroidManifest.xml`, `Info.plist`                             |

## 6. File Structure Plan

```
lib/
├── core/
│   └── services/
│       └── livekit_service.dart          ← NEW (wraps Room lifecycle)
│
├── app/
│   └── router.dart                       ← MODIFY (add demo-video-call route)
│
└── features/
    └── video_call/                       ← NEW feature folder
        ├── models/
        │   └── video_token_response.dart
        ├── providers/
        │   └── video_call_provider.dart
        ├── repository/
        │   └── video_call_repository.dart  ← getDemoVideoToken() only
        ├── screens/
        │   └── demo_video_call_screen.dart ← Standalone demo screen
        └── widgets/
            ├── video_tile.dart            ← Reusable for Operation 2
            └── call_controls.dart         ← Reusable for Operation 2
```

## 7. Screen Design — `DemoVideoCallScreen`

### States

| State            | UI                                                           |
| ---------------- | ------------------------------------------------------------ |
| **Idle**         | Text field for room name + "Join" button                     |
| **Connecting**   | Loading indicator + "Connecting..."                          |
| **Connected**    | Local video (small PiP) + Remote video (full) + controls bar |
| **Waiting**      | Local video + "Waiting for other participant..." placeholder |
| **Disconnected** | "Call ended" + "Rejoin" button                               |

### Controls Bar

- Toggle microphone (mute/unmute)
- Toggle camera (on/off)
- Flip camera (front/rear)
- End call (disconnect + return to idle)

### Entry Point

Add a temporary "Video Call Demo" button accessible from dev/debug context (e.g. a dev menu, or directly navigable via `/demo-video-call` route).

## 8. Mapping Definition

### Token Response DTO

| Backend JSON Key | Dart Field | Type     | Nullable |
| ---------------- | ---------- | -------- | -------- |
| `token`          | `token`    | `String` | No       |
| `wsUrl`          | `wsUrl`    | `String` | No       |
| `roomName`       | `roomName` | `String` | No       |

## 9. Duplication Check

| Check                                             | Status                           |
| ------------------------------------------------- | -------------------------------- |
| Uses `httpServiceProvider` from `core/providers/` | ✅ No duplicate HTTP setup       |
| Error handling via `HttpService` interceptors     | ✅ No `_handleError` duplication |
| BASE_URL from `.env` via `dotenv`                 | ✅ No hardcoded URLs             |

## 10. Validation Plan

### Happy Path (Demo)

1. Navigate to `/demo-video-call`
2. Enter room name, e.g. `test-room`
3. Grant camera/mic permissions
4. Tap "Join" → token fetched → room connected
5. Open same room from another device → see each other's video
6. Test mute/camera toggle/flip
7. Tap "End Call" → return to idle state

### Error Scenarios

- 401/403 → token refresh → retry
- Permission denied → guide to settings
- Network failure → retry option

## 11. Reusability for Operation 2

These components will be reused as-is in `FEAT-integrate-video-call-consultation`:

- `LiveKitService` (core service)
- `VideoTokenResponse` (model)
- `video_tile.dart`, `call_controls.dart` (widgets)
- Platform configuration (permissions)

Operation 2 will ADD:

- `getVideoToken(consultationId)` to repository
- `PreCallScreen`, `InCallScreen`, `EndCallScreen` for consultation flow
- Navigation from consultation detail screen
