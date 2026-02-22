---
doc_role: operation
operation_id: FEAT-live-tracking-LT1
generated_from: plan.md
status: done
created_at: 2026-02-22
---

# Instruction/Prompt for AI: Implement LT-1 in Flutter

## Goal

Implement the `RescuerSignalRService` and `LocationManager` to send real-time rescuer locations (throttled to 10s) to the backend via SignalR, as defined in the LT-1 contract. Integrate this with `RescuerHomeScreen.dart`.

## Instructions

1. **Create `lib/core/services/rescuer_signalr_service.dart`** (Moved from `features/rescuer` for shared access)
   - Import `signalr_netcore`.
   - Setup a Singleton or Provider class.
   - Read token from storage. Build `HubConnection`.
   - Expose `updateLocation(String userId, double lat, double lng)`.
   - Implement `joinAsRescuer(String userId)`.

2. **Create `lib/features/rescuer/managers/location_manager.dart`**
   - Accept the signalR service as a dependency.
   - Use `Geolocator.getPositionStream` (with `distanceFilter: 10`).
   - Add a 10s throttling mechanism so we don't spam the server.

3. **Modify `lib/features/rescuer/screens/rescuer_home_screen.dart`** (or its related provider)
   - Read user online status state.
   - When becoming 'online', start the signalR service, call connect, get userId, and start the `LocationManager`.
   - Note: Handle location permissions (`geolocator.checkPermission` / `requestPermission()`), request if not granted before starting stream.

## Strict Flutter Restrictions

- **DO NOT** invent new endpoint paths. Use `/rescuer-hub` for the relative hub path.
- **DO NOT** duplicate error handling. If SignalR disconnects, let `automaticReconnect` handle it and log cleanly.
- Keep the DTO assumptions primitive. `UpdateLocation` takes standard `double` for lat/lng.
