---
doc_role: operation
operation_id: FEAT-integrate-video-call
type: FEAT
status: draft
created_at: 2026-02-24
backend_reference:
  module: live-kit-cloud
  operation: FEAT-video-call
  path: Docs/05-Backend/01-flows/P3-consulting/live-kit-cloud/
  usageguide: live-kit-cloud.usageguide.md
---

# FEAT-integrate-video-call — Flutter Plan

## 1. Backend Contract Reference

| Item                   | Value                                             |
| ---------------------- | ------------------------------------------------- |
| **Backend module**     | `P3-consulting/live-kit-cloud`                    |
| **Backend operation**  | `FEAT-video-call`                                 |
| **Backend endpoint**   | `POST /api/livekit/consultation/{id}/video-token` |
| **Backend usageguide** | `live-kit-cloud.usageguide.md` (TBD)              |

### Expected Backend Response

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "wsUrl": "wss://{{PROJECT_ID}}.livekit.cloud",
  "roomName": "consultation-{consultationId}"
}
```

## 2. Code Culture Reference (from actual codebase)

| Pattern            | Convention                                                                     | Reference                                           |
| ------------------ | ------------------------------------------------------------------------------ | --------------------------------------------------- |
| Feature folder     | `lib/features/{feature_name}/models\|providers\|repository\|screens\|widgets/` | `features/emergency/`, `features/consultation/`     |
| Shared services    | `lib/core/services/{name}_service.dart`                                        | `core/services/rescuer_signalr_service.dart`        |
| HTTP provider      | `lib/core/providers/http_provider.dart` → `httpServiceProvider`                | `http_provider.dart`                                |
| File naming        | `snake_case.dart`                                                              | All files                                           |
| Folder naming      | `snake_case`                                                                   | `features/snake_catching/`                          |
| Riverpod providers | `Provider<T>((ref) { ... })` for services                                      | `httpServiceProvider`, `healthCheckServiceProvider` |

## 3. As-Is (Flutter State)

| Component                   | Status                                                                             |
| --------------------------- | ---------------------------------------------------------------------------------- |
| `features/consultation/`    | Exists with `models/providers/repository/screens/widgets/` (stub Class.dart files) |
| `livekit_client` dependency | Not installed                                                                      |
| Video call feature folder   | Does NOT exist                                                                     |
| Core LiveKit service        | Does NOT exist                                                                     |

## 4. Gap Analysis

| Gap                    | Required Action                                | Target Location                                                 |
| ---------------------- | ---------------------------------------------- | --------------------------------------------------------------- |
| **New dependency**     | Add `livekit_client: ^2.6.3` to `pubspec.yaml` | `pubspec.yaml`                                                  |
| **New core service**   | `LiveKitService` — wraps Room lifecycle        | `lib/core/services/livekit_service.dart`                        |
| **New feature folder** | `video_call` with standard subfolders          | `lib/features/video_call/`                                      |
| **New model**          | `VideoTokenResponse` DTO                       | `lib/features/video_call/models/video_token_response.dart`      |
| **New repository**     | `VideoCallRepository` for token fetch          | `lib/features/video_call/repository/video_call_repository.dart` |
| **New providers**      | Video call state providers                     | `lib/features/video_call/providers/video_call_provider.dart`    |
| **New screens**        | Pre-call, In-call, End-call                    | `lib/features/video_call/screens/`                              |
| **New widgets**        | Video tile, controls bar                       | `lib/features/video_call/widgets/`                              |
| **Platform config**    | Android Bluetooth perms, iOS background audio  | `AndroidManifest.xml`, `Info.plist`                             |

## 5. File Structure Plan

```
lib/
├── core/
│   └── services/
│       └── livekit_service.dart          ← NEW (wraps Room lifecycle, like rescuer_signalr_service.dart)
│
└── features/
    └── video_call/                       ← NEW feature folder (snake_case!)
        ├── models/
        │   └── video_token_response.dart
        ├── providers/
        │   └── video_call_provider.dart
        ├── repository/
        │   └── video_call_repository.dart
        ├── screens/
        │   ├── pre_call_screen.dart
        │   ├── in_call_screen.dart
        │   └── end_call_screen.dart
        └── widgets/
            ├── video_tile.dart
            └── call_controls.dart
```

## 6. Mapping Definition

### Token Response DTO

| Backend JSON Key | Dart Field | Type     | Nullable |
| ---------------- | ---------- | -------- | -------- |
| `token`          | `token`    | `String` | No       |
| `wsUrl`          | `wsUrl`    | `String` | No       |
| `roomName`       | `roomName` | `String` | No       |

## 7. Duplication Check

| Check                                             | Status                           |
| ------------------------------------------------- | -------------------------------- |
| Uses `httpServiceProvider` from `core/providers/` | ✅ No duplicate HTTP setup       |
| Error handling via `HttpService` interceptors     | ✅ No `_handleError` duplication |
| BASE_URL from `.env` via `dotenv`                 | ✅ No hardcoded URLs             |

## 8. Validation Plan

### Happy Path

1. User opens consultation → taps "Video Call"
2. Pre-call: permissions granted, camera preview shown
3. "Join" → token fetched → room connected → remote participant appears
4. "End Call" → room disconnected → end screen

### Reconnection (SFU Model)

1. User A loses connection → SDK auto-reconnect
2. If fails → user taps "Rejoin" → fetch new token → `room.connect()` → rejoins immediately
3. **No accept from other user needed** — SFU room persists on server

### Error Scenarios

- 401/403 → token refresh → retry
- 404 → "Consultation not found"
- 409 → "Consultation not ready"
- Permission denied → guide to settings
