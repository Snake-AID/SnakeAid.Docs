---
doc_role: operation
operation_id: FEAT-integrate-video-call
generated_from: plan.md
status: draft
created_at: 2026-02-24
---

# FEAT-integrate-video-call — Flutter Prompt

## Objective

Integrate LiveKit Cloud video call into SnakeAid Flutter app.
Follow existing codebase conventions strictly.

## Code Culture Rules (MUST FOLLOW)

| Rule                | Convention                                                     | Reference                                           |
| ------------------- | -------------------------------------------------------------- | --------------------------------------------------- |
| Feature folder      | `lib/features/video_call/` (snake_case)                        | `features/emergency/`, `features/consultation/`     |
| Subfolder structure | `models/`, `providers/`, `repository/`, `screens/`, `widgets/` | All feature folders                                 |
| Shared services     | `lib/core/services/{name}_service.dart`                        | `rescuer_signalr_service.dart`, `http_service.dart` |
| HTTP provider       | Import from `core/providers/http_provider.dart`                | `httpServiceProvider`                               |
| File naming         | `snake_case.dart`                                              | All .dart files                                     |
| Riverpod            | `Provider<T>((ref) { ... })` for services                      | `http_provider.dart`                                |

## Required Outputs

### 1. Add Dependency

```yaml
# pubspec.yaml
dependencies:
  livekit_client: ^2.6.3
```

Run `flutter pub get`.

### 2. Platform Configuration

**Android** — Add to `android/app/src/main/AndroidManifest.xml`:

```xml
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
```

**iOS** — Ensure `ios/Runner/Info.plist` contains:

```xml
<key>NSCameraUsageDescription</key>
<string>SnakeAid uses your camera for video consultation</string>
<key>NSMicrophoneUsageDescription</key>
<string>SnakeAid uses your microphone for video consultation</string>
<key>UIBackgroundModes</key>
<array>
  <string>audio</string>
</array>
```

### 3. Core Service — `lib/core/services/livekit_service.dart`

> Place here because it manages a long-lived connection (like `rescuer_signalr_service.dart`).

```dart
import 'package:livekit_client/livekit_client.dart';

class LiveKitService {
  Room? _room;
  EventsListener<RoomEvent>? _listener;

  /// Connect to a LiveKit room
  Future<Room> connect(String wsUrl, String token) async {
    _room = Room();
    final options = RoomOptions(
      adaptiveStream: true,
      dynacast: true,
    );
    await _room!.prepareConnection(wsUrl, token);
    await _room!.connect(wsUrl, token, roomOptions: options);
    _listener = _room!.createListener();
    return _room!;
  }

  Future<void> setCameraEnabled(bool enabled) async {
    await _room?.localParticipant?.setCameraEnabled(enabled);
  }

  Future<void> setMicrophoneEnabled(bool enabled) async {
    await _room?.localParticipant?.setMicrophoneEnabled(enabled);
  }

  Future<void> disconnect() async {
    _listener?.dispose();
    await _room?.disconnect();
    _room = null;
    _listener = null;
  }

  Room? get room => _room;
  EventsListener<RoomEvent>? get listener => _listener;
}
```

### 4. Model — `lib/features/video_call/models/video_token_response.dart`

```dart
class VideoTokenResponse {
  final String token;
  final String wsUrl;
  final String roomName;

  VideoTokenResponse({
    required this.token,
    required this.wsUrl,
    required this.roomName,
  });

  factory VideoTokenResponse.fromJson(Map<String, dynamic> json) {
    return VideoTokenResponse(
      token: json['token'] as String,
      wsUrl: json['wsUrl'] as String,
      roomName: json['roomName'] as String,
    );
  }
}
```

### 5. Repository — `lib/features/video_call/repository/video_call_repository.dart`

```dart
import '../../../core/services/http_service.dart';
import '../models/video_token_response.dart';

class VideoCallRepository {
  final HttpService _httpService;

  VideoCallRepository(this._httpService);

  Future<VideoTokenResponse> getVideoToken(String consultationId) async {
    final response = await _httpService.post(
      '/api/livekit/consultation/$consultationId/video-token',
    );
    return VideoTokenResponse.fromJson(response.data);
  }
}
```

### 6. Providers — `lib/features/video_call/providers/video_call_provider.dart`

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../core/providers/http_provider.dart';
import '../../../core/services/livekit_service.dart';
import '../repository/video_call_repository.dart';

// Service provider (singleton — manages room connection)
final liveKitServiceProvider = Provider<LiveKitService>((ref) {
  return LiveKitService();
});

// Repository provider
final videoCallRepositoryProvider = Provider<VideoCallRepository>((ref) {
  return VideoCallRepository(ref.read(httpServiceProvider));
});

// State management for video call UI
// Follow existing provider patterns in the codebase
```

### 7. Screens — `lib/features/video_call/screens/`

**`pre_call_screen.dart`**:

- Request camera/mic permissions
- Show local camera preview
- Mute toggles before joining
- "Join Call" button → fetch token + connect

**`in_call_screen.dart`**:

- Local video (small overlay)
- Remote video (full screen)
- Controls: mute mic, toggle camera, end call, flip camera
- Connection quality indicator
- "Reconnecting..." overlay on disconnect
- "Waiting for participant..." when alone

**`end_call_screen.dart`**:

- Call duration
- "Back to consultation" button

### 8. Widgets — `lib/features/video_call/widgets/`

**`video_tile.dart`**: Wraps `VideoTrackRenderer` with participant info overlay
**`call_controls.dart`**: Bottom bar with mute/camera/end buttons

### 9. Permission Helper

```dart
import 'package:permission_handler/permission_handler.dart';

Future<bool> requestVideoCallPermissions() async {
  final camera = await Permission.camera.request();
  final mic = await Permission.microphone.request();

  if (camera.isPermanentlyDenied || mic.isPermanentlyDenied) {
    return false; // Show dialog to open app settings
  }
  return camera.isGranted && mic.isGranted;
}
```

## Forbidden Changes

- Do NOT put LiveKit service in feature folder — put in `core/services/`
- Do NOT use `video-call` (kebab-case) — use `video_call` (snake_case)
- Do NOT create flat files without subfolder structure
- Do NOT create new HTTP service — use existing `httpServiceProvider`
- Do NOT duplicate error handling — use `HttpService` interceptors
- Do NOT hardcode WebSocket URLs — use backend-provided `wsUrl`
- Do NOT store LiveKit API keys in Flutter
- Do NOT create custom endpoint constants if codebase uses inline strings

## Test Requirements

- Widget test: `PreCallScreen` permission flow
- Widget test: `InCallScreen` video tile rendering
- Unit test: `VideoCallRepository.getVideoToken()` DTO parsing
- Unit test: `LiveKitService.connect()` Room creation
