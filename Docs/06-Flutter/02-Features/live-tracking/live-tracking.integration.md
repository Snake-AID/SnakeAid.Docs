# Live Tracking (LT-1 & LT-2) - Flutter Integration

## 1. Backend Contract Reference

- **Backend Module Name**: `live-tracking` (Flow)
- **Backend Usage Guide**: `Docs/05-Backend/01-flows/P1-emergency/S1-identified/emergency-rescue/live-tracking/live-tracking.usageguide.md`
- **Current Phase**: LT-1 (Ingestion Foundation)

## 2. Shared Networking/Infrastructure

- **Networking Core**: Uses SignalR connection via `signalr_netcore`
- **Authentication**: Requires valid `access_token`

## 3. Backend Endpoints Consumed (SignalR Hub)

### Rescuer Hub (`/rescuer-hub`)

- **Transport**: WebSockets / SignalR
- **Auth**: Required (Bearer Token)

### Server -> Client Events (Currently Available)

| Event Name         | Purpose                  | Nullable | Notes                                  |
| ------------------ | ------------------------ | -------- | -------------------------------------- |
| `Joined`           | Initial connection ack   | No       | Backend sends `{ message }` on connect |
| `NewRescueRequest` | Realtime dispatch notify | No       | Emits SOS to connected rescuers        |
| `RequestAccepted`  | Acknowledgment of accept | No       |                                        |
| `RequestTaken`     | Another rescuer won      | No       |                                        |
| `RequestCancelled` | Patient cancelled        | No       |                                        |
| `RequestRejected`  | Acknowledgment of reject | No       |                                        |
| `RequestError`     | Operation failure        | No       |                                        |
| `LocationUpdated`  | Server ack of location   | No       | Useful for debug tracking              |

### Client -> Server Methods (Currently Available)

| Method Name      | Arguments                                            | Notes                               |
| ---------------- | ---------------------------------------------------- | ----------------------------------- |
| `JoinAsRescuer`  | `(string userId)`                                    | Must be called after SignalR starts |
| `UpdateLocation` | `(string userId, double latitude, double longitude)` | Requires 10s client throttling      |

## 4. DTO Mapping Table

_(No complex REST DTOs for LT-1 ingestion route. Only primitive types sent to Hub)._

### NewRescueRequest Event Payload Example (from UsageGuide)

| Backend Field   | Dart Field      | Nullable | Notes                     |
| --------------- | --------------- | -------- | ------------------------- |
| `requestId`     | `requestId`     | No       | Guid String               |
| `sessionId`     | `sessionId`     | No       | Guid String               |
| `incidentId`    | `incidentId`    | No       | Guid String               |
| `radiusKm`      | `radiusKm`      | No       | Integer                   |
| `expiredAt`     | `expiredAt`     | No       | DateTime string (ISO8601) |
| `requestSentAt` | `requestSentAt` | No       | DateTime string (ISO8601) |

## 5. Repository / Service Surface

### `RescuerSignalRService` (Singleton)

- Located in `lib/core/services/rescuer_signalr_service.dart` (moved from `features/rescuer` for shared access).
- `connect()`
- `joinAsRescuer(String userId)`
- `updateLocation(String userId, double lat, double lng)`

### `LocationManager`

- `startTracking(String userId)`
- Throttles tracking to 10s intervals
- Calls `updateLocation`

## 6. State Flow

`RescuerHomeScreen` UI (Online Toggle) → `LocationManager.startTracking` → `Geolocator` stream → `LocationManager` (10s throttle filter) → `RescuerSignalRService.updateLocation` → Backend `RescuerHub` -> `RescuerLocationService` (PostGIS)

## 7. Error Handling Strategy

- Catch unauthorized on SignalR connect. SignalR lib supports `withAutomaticReconnect()`.
- Unsent locations during disconnect are dropped (we do not need to queue old locations as the server wants current position).

## 8. Backend Dependencies Assumptions

- The server will throttle updates to 10s internally and persist to PostGIS (`RescuerProfile.LastLocation`).
- Rescuers without recent locations might be excluded from SOS matching (RT-1 guardrail).
