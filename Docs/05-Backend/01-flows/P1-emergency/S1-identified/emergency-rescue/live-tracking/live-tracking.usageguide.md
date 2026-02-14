---
doc_role: baseline
module: live-tracking
kind: flow
status: active
last_updated: 2026-02-15
owners: [backend-team]
---

# Live Tracking Module - Usage Guide

This guide documents what clients can use today and what is not available yet.

## 0. Phase Context

Roadmap: `../emergency-rescue.roadmap.md`

Current behavior is pre-LT-1 completion:
- Dispatch exists via rescue-trigger.
- Location ingestion is active (LT-1).
- Full viewer tracking contracts (LT-2) are not active.

Phase intent:
1. `LT-1`: make location ingestion real and reliable.
2. `LT-2`: provide full viewer tracking experience.

## 1. Current Integration Surface

### Available now
1. Start rescue dispatch via `POST /api/incidents/sos`.
2. Rescuer real-time offer handling via SignalR hub `/rescuer-hub`.
3. Incident detail read via `GET /api/incidents/{incidentId}`.
4. Session timeout monitoring endpoints for operations.

### Not available yet
1. Patient/admin map snapshot endpoint.
2. Patient/admin tracking stream endpoint/group contract.
3. Location history endpoint.
4. Fallback push channel for critical dispatch events.

3. Location history endpoint.
4. Fallback push channel for critical dispatch events.

### Now Reliable (LT-1)
1. `UpdateLocation` persists rescuer profile location (PostGIS) with throttling (default 10s).

## 2. SOS Entry for Tracking Lifecycle

### Request
- Method: `POST`
- URL: `/api/incidents/sos`
- Auth: required
- Body:

```json
{
  "lng": 106.660172,
  "lat": 10.762622
}
```

### Purpose
Starts dispatch lifecycle and creates first rescue session.
Use returned `incidentId` and `sessionId` as correlation keys in client state.

## 3. Rescuer Hub Contract

### Connect
- Hub URL: `/rescuer-hub`

### Client -> Server methods
1. `JoinAsRescuer(string userId)`
2. `AcceptRequest(Guid requestId, Guid rescuerId)`
3. `RejectRequest(Guid requestId)`
4. `UpdateLocation(string userId, double latitude, double longitude)`
5. `GetConnectedRescuers()`

### Server -> Client events
1. `Joined`
2. `NewRescueRequest`
3. `RequestAccepted`
4. `RequestTaken`
5. `RequestCancelled`
6. `RequestRejected`
7. `RequestError`
8. `LocationUpdated`
9. `ConnectedRescuers`

### `NewRescueRequest` payload example
```json
{
  "requestId": "request-guid",
  "sessionId": "session-guid",
  "incidentId": "incident-guid",
  "radiusKm": 10,
  "expiredAt": "2026-02-06T12:01:00Z",
  "requestSentAt": "2026-02-06T12:00:00Z"
}
```

## 4. Important Behavior Notes

1. `UpdateLocation(...)` now persists `LastLocation` for matching (LT-1).
2. Manual `raise-range` currently does not trigger dispatch broadcast/schedule.
3. Hub authorization hardening is not complete in current code.
4. `RequestExpired` push is not emitted in production timeout flow.

## 5. Planned Contracts by Phase

### LT-1 expected additions (Delivered 2026-02-13)
1. Persisted location ingestion from rescuer publish path.
2. Throttling and stale-location policy (default 10s interval).
3. PostGIS `geometry(Point, 4326)` integration.

### LT-2 expected additions
1. `GET /api/sessions/{id}/tracking/snapshot`
2. `GET /api/sessions/{id}/tracking/history`
3. Session viewer methods:
   - `JoinSession(sessionId)`
   - `LeaveSession(sessionId)`
4. Viewer event:
   - `LocationUpdated(sessionId, locationPayload)`
5. Redis-backed NOW-state for live viewer updates.

## 6. Client Strategy Until Full LLT Is Delivered

1. Treat live tracking as dispatch notifications only.
2. Do not build patient/admin map stream dependency yet.
3. Use incident detail endpoint for periodic status refresh.
4. Show graceful fallback text when true live map is unavailable.

## 7. Response Envelope Reminder

All REST endpoints use:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {},
  "error": null
}
```

Check `is_success` before using `data`.

## 8. Integration Rule

Do not integrate LT-2 endpoints/events until they exist in backend source and are marked implemented in:
- `live-tracking.sourcecode.md`
- `rescue-trigger.sourcecode.md` (if dispatch dependency changes)

## 9. Flutter Implementation Guide (LT-1)

This section details how to implement the Rescuer Live Tracking feature in the Flutter app.

### 9.1. Dependencies
Add the following to your `pubspec.yaml`:

```yaml
dependencies:
  signalr_netcore: ^1.3.6 # Recommended for SignalR
  geolocator: ^13.0.1     # For GPS location
  flutter_secure_storage: ^9.0.0 # To retrieve JWT token
```

### 9.2. Connection Manager (`RescuerSignalRService`)

Create a singleton service to manage the connection.

```dart
// lib/services/rescuer_signalr_service.dart

import 'package:signalr_netcore/signalr_client.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class RescuerSignalRService {
  HubConnection? _hubConnection;
  final _storage = const FlutterSecureStorage();
  final String _hubUrl = "https://your-backend-url/rescuer-hub"; // Update URL
  
  // Events
  Function(String)? onStatusMessage;

  Future<void> connect() async {
    final token = await _storage.read(key: "access_token"); // Ensure key matches your auth logic
    
    if (token == null) {
      throw Exception("Unauthorized: No access token found");
    }

    _hubConnection = HubConnectionBuilder()
        .withUrl(_hubUrl, options: HttpConnectionOptions(
          accessTokenFactory: () async => token,
          // transport: HttpTransportType.WebSockets, // Optional: Force WebSockets
        ))
        .withAutomaticReconnect()
        .build();

    _hubConnection?.onclose((error) {
      print("Connection Closed: $error");
    });

    _hubConnection?.on("Joined", (arguments) {
      final data = arguments?[0] as Map<String, dynamic>;
      print("Joined: ${data['message']}");
    });

    _hubConnection?.on("LocationUpdated", (arguments) {
        // Ack from server
        print("Location ack received");
    });

    await _hubConnection?.start();
    print("SignalR Connected: ${_hubConnection?.state}");
  }

  // LT-1: Send Location
  Future<void> updateLocation(String userId, double lat, double lng) async {
    if (_hubConnection?.state == HubConnectionState.Connected) {
      print("Sending location: $lat, $lng");
      await _hubConnection?.invoke("UpdateLocation", args: [userId, lat, lng]);
    } else {
      print("Cannot send location: Disconnected");
    }
  }

  Future<void> joinAsRescuer(String userId) async {
      await _hubConnection?.invoke("JoinAsRescuer", args: [userId]);
  }
}
```

### 9.3. Throttling Logic & Background Location

**Requirement:** The server throttles updates to **once every 10 seconds**. The client must respect this to save battery.

```dart
// lib/managers/location_manager.dart

import 'package:geolocator/geolocator.dart';
import 'dart:async';

class LocationManager {
  final RescuerSignalRService _signalRService;
  Timer? _throttleTimer;
  Position? _lastPosition;
  bool _isThrottled = false;
  
  LocationManager(this._signalRService);

  void startTracking(String userId) {
    const locationSettings = LocationSettings(
      accuracy: LocationAccuracy.high,
      distanceFilter: 10, // Only push if moved 10 meters
    );

    Geolocator.getPositionStream(locationSettings: locationSettings)
        .listen((Position position) {
      _handleNewPosition(userId, position);
    });
  }

  void _handleNewPosition(String userId, Position position) {
    _lastPosition = position;
    
    if (_isThrottled) return;

    // Send immediately
    _sendLocation(userId, position);
    
    // Enable throttle
    _isThrottled = true;
    _throttleTimer = Timer(const Duration(seconds: 10), () {
      _isThrottled = false;
      // Optionally: if position changed significantly while throttled, send valid latest immediately
      // But for simple implementation, just wait for next stream event
    });
  }

  void _sendLocation(String userId, Position position) {
    _signalRService.updateLocation(userId, position.latitude, position.longitude);
  }
}
```

### 9.4. Important Notes for Flutter Devs
1. **Background Execution**: Standard `Geolocator` stream may pause when app is backgrounded. For true "On-Shift" tracking (Phase 2), consider using `flutter_background_service` or `background_locator_2`. For Phase 1 (LT-1), foreground/active tracking is acceptable.
2. **Permissions**: Ensure `AndroidManifest.xml` and `Info.plist` have correct location permissions (`ACCESS_FINE_LOCATION`, `NSLocationWhenInUseUsageDescription`).
3. **Reconnect**: `signalr_netcore` has `.withAutomaticReconnect()`. Ensure your app handles re-auth if the token expires during a long shift.
