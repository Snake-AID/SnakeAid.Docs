---
doc_role: operation
operation_id: FEAT-live-tracking-dashboard
generated_from: plan.md
status: done
created_at: 2026-02-23
---

# Execution Prompt: Live Tracking Monitoring Dashboard

## 1. Objective

Implement a read-only ASP.NET Core Razor Pages dashboard (`Index.cshtml`) that visualizes real-time SignalR events from `RescuerHub`. This dashboard acts as a "God Mode/Admin View" for QA and developers to observe Flutter client behavior without manually querying the database.

## 2. Requirements

### 2.1 Backend Changes (`SnakeAid.Api`)

**`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml` (New)**

- Create a UI with a 3-Tab main area and a sidebar:
  1.  **Tab 1 — Connected Rescuers**: Card grid of rescuers currently in SignalR hub. Cards show Avatar/Name/Type/Rating/TotalMissions, a **Ping Timer** circle, SignalR/IsOnline status pills, and live Lat/Lng pills that blink on update.
  2.  **Tab 2 — Ghost Scanner**: Rescuers with `IsOnline=true` in DB but absent from Hub (state drift). Trigger via `?handler=Ghosts`.
  3.  **Tab 3 — All Rescuers**: Full DB list via `?handler=AllRescuers`; offline cards are dimmed.
  4.  **Sidebar — Active Incident Tracker**: Shows `incidentId`, `sessionId`, `radiusKm`, 60s countdown bar on `NewRescueRequest`.
  5.  **Sidebar — Event Console**: Auto-scrolling terminal log for all SignalR events.
- `createRescuerCard(rescuer)` supports both a plain `id` string (placeholder, from early `LocationUpdated`) and a full object (from `AdminLog/RescuerJoined`). When a card already exists and richer data arrives, call `updateRescuerCard(id, rescuer)` to upgrade placeholder content in-place.
- Do NOT require manual input of JWT. Read the auto-generated token from the Model.
- Auto-connect on page load (no manual button required).

**`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs` (New)**

- Generates an Admin-role JWT in `OnGet()` using JWT settings.
- Expose the token as a string property inside the model.

**`SnakeAid.Api/Hubs/RescuerHub.cs` (Modify)**

- Support a mechanism to broadcast admin-level logs to the Monitor without polluting actual mobile clients.
  - _Strategy_: Introduce a `JoinAsMonitor` method that adds the caller to a `"Monitors"` SignalR group.
  - _Instrumentation:_ When `JoinAsRescuer`, `UpdateLocation`, `AcceptRequest`, or `OnDisconnectedAsync` occurs, emit a lightweight `AdminLog` event specifically to the `"Monitors"` group.

### 2.2 Client-side Logic (in `Index.cshtml`)

- Import `@microsoft/signalr` via CDN (do not require npm for Razor pages).
- **SignalR Connect Logic**:
  - Connect to `/rescuer-hub` using the provided JWT.
  - Call `JoinAsMonitor` on connect.
- **Event Listeners**:
  - `ConnectedRescuers`: Sync initial card grid from Hub state on join.
  - `AdminLog` (types: `RescuerJoined`, `RescuerDisconnected`, `MonitorJoined`): Create/remove rescuer cards and log to Console.
  - `LocationUpdated`: Update Lat/Lng pills, restart Ping Timer. If card doesn't exist yet, call `createRescuerCard(data.userId)` to create a placeholder.
  - `NewRescueRequest`: Update Active Incident Tracker, start 60s countdown, mark targeted rescuer card as `PINGED`.
  - `RequestAccepted`: Log winner and mark card as `BUSY (MISSION)`.
  - `RequestTaken`: Mark losing rescuer card back to `IDLE`.
  - `RequestExpired`: Reset countdown, mark card back to `IDLE`.
  - `RequestCancelled`: Hide Incident Tracker, reset all `PINGED` cards to `IDLE`.
- **Reconnect**: Attach `connection.onreconnected(async () => { await connection.invoke("JoinAsMonitor"); })` after `.build()` to restore Monitor group membership after any automatic reconnect.

## 3. Constraints & Rules

- **No Mobile Impact**: The new `AdminLog` event **MUST NOT** be broadcasted via `Clients.All`. Use `Clients.Group("Monitors")` exclusively to ensure Flutter apps only receive what is documented in `live-tracking.integration.md`.
- **CSS Framework**: Use basic internal CSS/inline styles or a lightweight CDN (like Bootstrap) to build a decent looking terminal/dashboard quickly. Do not introduce heavy frontend build pipelines.

## 4. Expected Deliverables

1. `Index.cshtml` with JS/HTML.
2. `Index.cshtml.cs` page model.
3. Updated `RescuerHub.cs` with `JoinAsMonitor` and `"Monitors"` group log broadcasts.
