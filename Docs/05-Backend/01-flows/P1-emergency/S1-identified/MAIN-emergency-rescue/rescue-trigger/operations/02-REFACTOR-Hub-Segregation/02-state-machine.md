---
doc_role: operation
operation_id: REFACTOR-split-mission-hub
type: REFACTOR
status: draft
created_at: 2026-02-26
---

# State Machine: Hub Separation

Based on the proposed "Asymmetric Connection" architecture (Option C), here is the state machine for the Member and Rescuer.

## 1. Connection States

| Actor            | State                 | Description                                                                                                                  |
| ---------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Member           | `Emergency_Initiated` | Member creates request, API returns `MissionId`. Member connects to `MissionHub(MissionId)`.                                 |
| Idle Rescuer     | `Polling_For_Jobs`    | Rescuer is online and connected to `RescuerHub`. Receiving broadcasts.                                                       |
| Assigned Rescuer | `In_Mission`          | Rescuer has accepted a job, disconnected from `RescuerHub` (or stopped listening), and connected to `MissionHub(MissionId)`. |

## 2. Transition Matrix & Edge Cases

### Scenario 1: Normal Flow

- **Event:** Member creates incident.
- **Action:** System creates DB record -> Member connects to `MissionHub` -> System broadcasts to `RescuerHub`.
- **Event:** Rescuer A accepts incident via API.
- **Action:** API validates -> Updates DB to `Accepted` by Rescuer A -> Rescuer A connects to `MissionHub` -> System broadcasts `Request_Taken(MissionId)` to `RescuerHub` so other rescuers drop it from UI.

### Scenario 2: Rescuer Cancels (Before EnRoute)

- **Initial State:** Member in `MissionHub`, Rescuer A in `MissionHub`.
- **Event:** Rescuer A calls `POST /api/mission/cancel`.
- **Action:**
  1. API updates mission state.
  2. System pushes `Rescuer_Cancelled` event to `MissionHub`. Member UI shows "Finding new rescuer...".
  3. Rescuer A disconnects from `MissionHub` and reconnects to `RescuerHub` (if they want to stay active).
  4. System broadcasts the incident again to `RescuerHub` (Phase 2 expansion). Member _stays_ in `MissionHub`.

### Scenario 3: Member Cancels (While Waiting for Rescuer)

- **Initial State:** Member in `MissionHub`, no rescuer assigned.
- **Event:** Member calls `POST /api/mission/cancel`.
- **Action:**
  1. API marks incident as `Cancelled_By_User`.
  2. System pushes `Mission_Cancelled` to `MissionHub` (Member UI closes).
  3. System broadcasts `Request_Cancelled(MissionId)` to `RescuerHub` so idle rescuers remove it from their UI.

### Scenario 4: Member Cancels (After Rescuer Assigned)

- **Initial State:** Member and Rescuer A in `MissionHub`.
- **Event:** Member calls `POST /api/mission/cancel`.
- **Action:**
  1. API marks incident as `Cancelled_By_User`.
  2. System pushes `Mission_Cancelled` to `MissionHub`.
  3. Rescuer A's UI receives event -> Redirects to Home Screen -> Rescuer A reconnects to `RescuerHub`.
  4. Member UI closes.

### Scenario 5: Timeout (No Rescuer Accepts)

- **Initial State:** Member in `MissionHub`, no rescuer assigned.
- **Event:** 3 phases pass (each 60s, total 180s) without an accept. Background job triggers timeout.
- **Action:**
  1. API/Job marks incident as `Expired`.
  2. System pushes `Session_Expired` to `MissionHub` (Member UI shows apology/fallback).
  3. System broadcasts `Request_Expired(MissionId)` to `RescuerHub` to clear it from all idle rescuers.

## 3. SignalR Network Resilience (Presence vs. State)

**RULE: Connection drops do NOT change mission state.**

- **Member Disconnects from MissionHub:**
  - `OnDisconnectedAsync` updates presence flag `Member_Offline` in Redis.
  - Rescuer receives `Member_Offline` event (UI might gray out the member).
  - If Member re-opens the app, they fetch active missions from API and reconnect. Presence becomes `Member_Online`.
- **Rescuer Disconnects from MissionHub:**
  - `OnDisconnectedAsync` updates presence flag `Rescuer_Offline`.
  - Member receives `Rescuer_Offline` event. Live tracking stops.
  - Rescuer app must auto-reconnect.
  - If timeout exceeds `Y` minutes while Offline, system may auto-cancel the assignment and fallback to Scenario 2.

## 4. Authorization Constraints

- **MissionHub Connection:** `OnConnectedAsync` MUST validate `UserId == Mission.AssignedRescuerId` OR `UserId == Mission.MemberId`. All others are `Context.Abort()`.
