---
doc_role: operation
operation_id: FEAT-integrate-video-call-consultation
type: FEAT
status: blocked
created_at: 2026-02-25
blocked_by: P3 Consultation module (Flutter)
depends_on: FEAT-integrate-video-call-demo
backend_reference:
  module: live-kit-cloud
  operations:
    - FEAT-video-call
  path: Docs/05-Backend/01-flows/P3-consulting/live-kit-cloud/
---

# FEAT-integrate-video-call-consultation — Flutter Plan

## 1. Objective

Integrate video call into the **consultation flow** using `livekit-token/{consultationId}` endpoint. This is Operation 2 of 2 — requires both the Consultation module (P3) and `FEAT-integrate-video-call-demo` to be completed first.

> [!WARNING]
> This operation is **blocked** until the Flutter Consultation module (P3) is implemented. Do not start this until consultation screens exist and the user can navigate to a consultation detail view.

## 2. Backend Contract Reference

| Item               | Value                                                        |
| ------------------ | ------------------------------------------------------------ |
| **Backend module** | `P3-consulting/live-kit-cloud`                               |
| **Endpoint used**  | `POST api/videocall/livekit-token/{consultationId}`          |
| **Auth**           | `[Authorize]` — requires logged-in user + consultation owner |
| **Response**       | `{ token, wsUrl, roomName }` — `VideoTokenResponse`          |

## 3. Prerequisites from Operation 1 (Demo)

These components already exist from `FEAT-integrate-video-call-demo`:

| Component             | Location                                 |
| --------------------- | ---------------------------------------- |
| `livekit_client` dep  | `pubspec.yaml`                           |
| Platform permissions  | `AndroidManifest.xml`, `Info.plist`      |
| `LiveKitService`      | `lib/core/services/livekit_service.dart` |
| `VideoTokenResponse`  | `lib/features/video_call/models/`        |
| `VideoCallRepository` | `lib/features/video_call/repository/`    |
| `video_tile.dart`     | `lib/features/video_call/widgets/`       |
| `call_controls.dart`  | `lib/features/video_call/widgets/`       |
| Providers (base)      | `lib/features/video_call/providers/`     |

## 4. Gap Analysis (on top of Demo)

| Gap                          | Required Action                                      | Target Location                                                 |
| ---------------------------- | ---------------------------------------------------- | --------------------------------------------------------------- |
| **Repository method**        | Add `getVideoToken(consultationId)` to existing repo | `lib/features/video_call/repository/video_call_repository.dart` |
| **Pre-call screen**          | Permission check + camera preview + "Join" button    | `lib/features/video_call/screens/pre_call_screen.dart`          |
| **In-call screen**           | Full video call UI with PiP + controls               | `lib/features/video_call/screens/in_call_screen.dart`           |
| **End-call screen**          | Call duration + "Back to consultation" button        | `lib/features/video_call/screens/end_call_screen.dart`          |
| **Consultation integration** | "Video Call" button on consultation detail screen    | `lib/features/consultation/screens/` (TBD)                      |
| **Routes**                   | Add pre-call/in-call/end-call routes                 | `lib/app/router.dart`                                           |

## 5. Screen Flow

```
Consultation Detail Screen
        │
        ▼ tap "Video Call"
┌──────────────────┐
│  PreCallScreen   │  ← Camera preview, permission check, mute toggles
│  "Join Call"     │
└────────┬─────────┘
         │ fetch token via getVideoToken(consultationId)
         ▼
┌──────────────────┐
│  InCallScreen    │  ← Full video call (reuses video_tile.dart, call_controls.dart)
│  Remote + Local  │
└────────┬─────────┘
         │ "End Call" or remote disconnect
         ▼
┌──────────────────┐
│  EndCallScreen   │  ← Duration, "Back to consultation"
└──────────────────┘
```

## 6. Validation Plan

### Happy Path

1. User opens consultation detail → taps "Video Call"
2. Pre-call: permissions granted, camera preview shown
3. "Join" → token fetched via `livekit-token/{consultationId}` → room connected
4. Remote participant appears → full video call
5. "End Call" → room disconnected → end screen → "Back to consultation"

### Error Scenarios

- 401/403 → token refresh → retry
- 404 → "Consultation not found"
- 409 → "Consultation not ready" (not paid, cancelled)
- Permission denied → guide to settings

## 7. Notes

- This plan will be refined when the Consultation module design is finalized
- Screen designs should be consistent with consultation UI theme
- The `video_tile.dart` and `call_controls.dart` widgets from Demo operation should be reused without modification
