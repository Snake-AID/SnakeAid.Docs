---
doc_role: operation
operation_id: FEAT-live-tracking-dashboard
generated_from: plan.md
status: done
created_at: 2026-02-23
---

# Execution Prompt: Live Tracking Monitoring Dashboard

## 1. Objective

Implement a read-only ASP.NET Core Razor Pages dashboard (`LiveTrackingMonitor.cshtml`) that visualizes real-time SignalR events from `RescuerHub`. This dashboard acts as a "God Mode/Admin View" for QA and developers to observe Flutter client behavior without manually querying the database.

## 2. Requirements

### 2.1 Backend Changes (`SnakeAid.Api`)

**`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml` (New)**

- Create a UI with 4 main components:
  1.  **Connection Table**: Shows currently connected Rescuers.
  2.  **Event Console**: Auto-scrolling div showing SignalR traffic (throttle logs, `LocationUpdated`).
  3.  **Active Session Tracker**: Shows `incidentId`, `sessionId`, `radiusKm`, and a 60s countdown timer for `RequestExpired`.
  4.  **Race Condition Log**: Highlights the winner of `AcceptRequest`.
- Do NOT require manual input of JWT. Read the auto-generated token from the Model.
- Include a "Connect as Admin" button to initialize `@microsoft/signalr` (or auto-connect on page load).

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
  - `AdminLog`: Print to the Event Console.
  - `NewRescueRequest`: Update "Active Session Tracker" and start the 60s countdown progress bar.
  - `RequestAccepted`: Log the winner to the Console and Race Condition panel.
  - `RequestExpired`: Reset countdown and log failure to the Console.
  - `ConnectedRescuers`: Refresh the Connection Table.
  - `LocationUpdated`: Log to Console (to verify 10s throttling).

## 3. Constraints & Rules

- **No Mobile Impact**: The new `AdminLog` event **MUST NOT** be broadcasted via `Clients.All`. Use `Clients.Group("Monitors")` exclusively to ensure Flutter apps only receive what is documented in `live-tracking.integration.md`.
- **CSS Framework**: Use basic internal CSS/inline styles or a lightweight CDN (like Bootstrap) to build a decent looking terminal/dashboard quickly. Do not introduce heavy frontend build pipelines.

## 4. Expected Deliverables

1. `Index.cshtml` with JS/HTML.
2. `Index.cshtml.cs` page model.
3. Updated `RescuerHub.cs` with `JoinAsMonitor` and `"Monitors"` group log broadcasts.
