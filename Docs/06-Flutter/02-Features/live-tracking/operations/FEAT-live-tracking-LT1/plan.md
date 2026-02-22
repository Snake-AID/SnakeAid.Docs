---
doc_role: operation
operation_id: FEAT-live-tracking-LT1
type: FEAT
status: done
created_at: 2026-02-22
---

# Integration Plan: Rescuer Live Tracking (LT-1 Foundation)

## 1. Backend Contract Reference

- **Backend Repo Path**: `Docs/05-Backend/01-flows/P1-emergency/S1-identified/emergency-rescue/live-tracking/`
- **Backend Usage Guide**: `live-tracking.usageguide.md`
- **Backend Operation**: `FEAT-phase-2-ingestion-foundation` (LT-1 phase)

## 2. As-Is (Flutter State)

Current state:

- We have a flutter project with `signalr_netcore`, `geolocator`, `flutter_secure_storage`.
- The generic rescuer home screen exists `rescuer_home_screen.dart`.
- There is an "Online" status toggle UI, but it probably doesn't start continuous location tracking and emit to the backend yet.
- No `RescuerSignalRService` or `LocationManager` implemented specifically for this hub flow.

## 3. Gap Analysis

- Need a `RescuerSignalRService` to manage the SignalR `HubConnectionBuilder` and handle the `/rescuer-hub` events.
- Need a `LocationManager` to stream GPS data via `Geolocator`, throttle it to 10 seconds intervals, and pass it to the SignalR service.
- State updates: Hook the online/offline state change in the UI to connect/disconnect the tracking logic.

## 4. Mapping Definition

No complex JSON model mappings required for LT-1 output. We just invoke `UpdateLocation(string userId, double lat, double lng)` on the SignalR connection.
The auth token must be passed to the SignalR builder `accessTokenFactory`.

## 5. Duplication Check

- Ensure we do not set up multiple overlapping `Geolocator` streams in the app root, we keep it isolated to when the Rescuer is ONLINE.
- Endpoint url will be defined safely. (Assuming an `Env` or constant file for BASE_URL).

## 6. Validation Plan

- Rescuer toggles to online -> Location permission requested if needed -> Prints show location fetched -> Sent over SignalR.
- Move (simulate location change) -> Console should show location sent every 10 seconds max.
- Server debug logs should acknowledge location persistence.
