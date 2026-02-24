---
doc_role: baseline
module: video-call
kind: feature
status: active
last_updated: 2026-02-24
owners: [mobile-team]
backend_reference:
  module: live-kit-cloud
  path: Docs/05-Backend/01-flows/P3-consulting/live-kit-cloud/
  introduction: live-kit-cloud.introduction.md
---

# Video Call — Flutter Integration Baseline

## 1. Overview

This feature enables real-time video calling between Patient↔Expert and Rescuer↔Expert during consultation sessions. The Flutter app connects to LiveKit Cloud using the `livekit_client` SDK after obtaining a JWT access token from the SnakeAid backend.

**Backend is the source of truth** for:

- Consultation session validity
- Token generation (JWT with LiveKit video grants)
- Room lifecycle (create, close)
- Billing and duration tracking

**Flutter owns**:

- Token acquisition from backend
- LiveKit Room connection and media control
- Camera/microphone state management
- UI rendering of video tracks
- Platform-specific permission handling

---

## 2. Backend Endpoints Consumed

| Endpoint                             | Method | Auth         | Purpose                                              | Backend Ref                          |
| ------------------------------------ | ------ | ------------ | ---------------------------------------------------- | ------------------------------------ |
| `/api/consultation/{id}/video-token` | POST   | Bearer (JWT) | Generate LiveKit access token for joining video room | `live-kit-cloud.usageguide.md` (TBD) |

### Token Response DTO

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "wsUrl": "wss://{{PROJECT_ID}}.livekit.cloud",
  "roomName": "consultation-{consultationId}"
}
```

> [!NOTE]
> Backend usageguide is not yet available (code not implemented). This section will be updated with exact response format and status codes once the backend endpoint is built.

---

## 3. DTO Mapping Table

| Backend Field | Dart Field | Type     | Nullable | Notes                                 |
| ------------- | ---------- | -------- | -------- | ------------------------------------- |
| `token`       | `token`    | `String` | No       | LiveKit JWT access token              |
| `wsUrl`       | `wsUrl`    | `String` | No       | WebSocket URL for LiveKit Cloud       |
| `roomName`    | `roomName` | `String` | No       | Room identifier matching consultation |

---

## 4. SDK Dependencies

### Primary Package

```yaml
dependencies:
  livekit_client: ^2.6.3
```

**Package**: [livekit_client on pub.dev](https://pub.dev/packages/livekit_client)
**Repository**: [livekit/client-sdk-flutter](https://github.com/livekit/client-sdk-flutter)
**Supported platforms**: Android, iOS, Web, macOS, Windows, Linux

### Optional Component Library

```yaml
dependencies:
  livekit_components: ^latest # Prebuilt UI widgets (evaluate during implementation)
```

---

## 5. Platform-Specific Setup

### Android — `AndroidManifest.xml`

Required permissions:

```xml
<manifest>
  <uses-feature android:name="android.hardware.camera" />
  <uses-feature android:name="android.hardware.camera.autofocus" />
  <uses-permission android:name="android.permission.CAMERA" />
  <uses-permission android:name="android.permission.RECORD_AUDIO" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
  <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
  <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
  <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
</manifest>
```

Bluetooth permissions require runtime request via `permission_handler`:

```dart
import 'package:permission_handler/permission_handler.dart';

Future<void> checkBluetoothPermissions() async {
  await Permission.bluetooth.request();
  await Permission.bluetoothConnect.request();
}
```

Android audio mode: `communication` (default, optimized for two-way voice).

### iOS — `Info.plist`

Required entries:

```xml
<dict>
  <key>NSCameraUsageDescription</key>
  <string>SnakeAid uses your camera for video consultation</string>
  <key>NSMicrophoneUsageDescription</key>
  <string>SnakeAid uses your microphone for video consultation</string>
  <key>UIBackgroundModes</key>
  <array>
    <string>audio</string>
  </array>
</dict>
```

Minimum deployment target: iOS 12.1

`Podfile`:

```ruby
platform :ios, '12.1'
```

---

## 6. State Flow

```
┌──────────┐     ┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│   UI     │────▶│  Riverpod    │────▶│  VideoCall      │────▶│  Backend     │
│ (Screen) │     │  Provider    │     │  Repository     │     │  (HTTP)      │
└──────────┘     └──────────────┘     └─────────────────┘     └──────────────┘
     │                  │                      │
     │                  │                      │ GET token
     │                  │                      ▼
     │                  │              ┌──────────────┐
     │                  │              │  LiveKit      │
     │                  │◀─────────────│  Room Object  │
     │                  │  room state  │  (SDK)        │
     │◀─────────────────│              └──────────────┘
     │  UI rebuild                          │
     │                              ┌───────┴───────┐
     │                              │ LiveKit Cloud  │
     │                              │ (WebRTC SFU)   │
     │                              └───────────────┘
```

### Detailed Flow

1. **User taps "Join Call"** → UI triggers provider
2. **Provider** calls `VideoCallRepository.getToken(consultationId)`
3. **Repository** makes HTTP request to backend `/api/consultation/{id}/video-token`
4. **Backend** validates ownership, returns `{ token, wsUrl, roomName }`
5. **Provider** creates `Room()` instance, calls `room.connect(wsUrl, token)`
6. **Provider** enables camera + microphone via `room.localParticipant.setCameraEnabled(true)`
7. **Room events** propagate through `EventsListener<RoomEvent>` to provider
8. **Provider** exposes state to UI: connection status, participants, tracks
9. **UI** renders video tracks using `VideoTrackRenderer` widget
10. **On disconnect**: provider cleans up room, notifies backend

---

## 7. Core SDK Usage Patterns

### Connecting to a Room

```dart
final room = Room();
final roomOptions = RoomOptions(
  adaptiveStream: true,
  dynacast: true,
);

await room.prepareConnection(wsUrl, token);
await room.connect(wsUrl, token, roomOptions: roomOptions);

await room.localParticipant?.setCameraEnabled(true);
await room.localParticipant?.setMicrophoneEnabled(true);
```

### Listening to Events

```dart
final listener = room.createListener();

listener
  ..on<RoomDisconnectedEvent>((event) {
    // Handle disconnect — show end screen
  })
  ..on<ParticipantConnectedEvent>((event) {
    // Remote participant joined
  })
  ..on<TrackPublishedEvent>((event) {
    // Remote track available — subscribe to render
  });
```

### Rendering Video

```dart
Widget build(BuildContext context) {
  final videoTrack = participant.videoTracks.values
    .where((pub) => pub.kind == TrackType.VIDEO && !pub.isScreenShare && pub.subscribed)
    .firstOrNull?.track as VideoTrack?;

  if (videoTrack != null) {
    return VideoTrackRenderer(videoTrack);
  }
  return Container(color: Colors.grey); // Placeholder
}
```

### Mute/Unmute

```dart
// Mute microphone
trackPublication.muted = true;

// Unmute
trackPublication.muted = false;
```

---

## 8. Error Handling Strategy

| Error                           | Source        | Handling                                                           |
| ------------------------------- | ------------- | ------------------------------------------------------------------ |
| Token fetch failure (401, 403)  | Backend HTTP  | Show "Session expired" → redirect to login                         |
| Token fetch failure (404)       | Backend HTTP  | Show "Consultation not found"                                      |
| Token fetch failure (409)       | Backend HTTP  | Show "Consultation not ready for video" (not paid, cancelled)      |
| Room connection failure         | LiveKit SDK   | Show retry option; check network connectivity                      |
| Media permission denied         | OS            | Show explanation dialog; guide to settings                         |
| Remote participant disconnected | LiveKit event | Show "Waiting for other participant" with timeout                  |
| Room disconnected unexpectedly  | LiveKit event | Attempt auto-reconnect (SDK built-in); show reconnecting indicator |

Error handling uses the shared error normalization from `networking-core.standards.md`.
No duplicated `_handleError` logic — all HTTP errors flow through centralized interceptors.

---

## 9. Backend Dependencies

| Dependency          | Notes                                                                        |
| ------------------- | ---------------------------------------------------------------------------- |
| Bearer token auth   | Standard JWT auth via existing `HttpService` interceptor                     |
| Token refresh       | Relies on existing `networking-core` token refresh mechanism                 |
| Consultation status | Backend validates consultation is in `paid` / `scheduled` status             |
| API Key isolation   | LiveKit API Key/Secret never sent to mobile; only the generated access token |

---

## 10. Permissions Check Flow (Pre-Call)

Before joining a video call, the app should:

1. **Check camera permission** → request if not granted
2. **Check microphone permission** → request if not granted
3. **Check Bluetooth permissions** (Android) → request if not granted
4. **Show preview screen** — local camera preview before joining
5. **Allow user to mute camera/mic before joining**
6. **Only then → fetch token and connect**

If any critical permission (camera OR microphone) is permanently denied, show a dialog guiding the user to app settings.

---

## 11. Connection Quality Indicator

LiveKit SDK provides connection quality metrics per participant.
The UI should display a signal-strength style indicator:

| Quality   | Display                   |
| --------- | ------------------------- |
| Excellent | 3 green bars              |
| Good      | 2 yellow bars             |
| Poor      | 1 red bar                 |
| Lost      | "Reconnecting..." overlay |

This is available via `Participant.connectionQuality` property and `ParticipantConnectedEvent`.
