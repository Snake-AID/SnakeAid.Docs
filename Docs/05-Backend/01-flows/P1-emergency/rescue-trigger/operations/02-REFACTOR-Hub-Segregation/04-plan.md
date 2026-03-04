---
doc_role: operation
operation_id: REFACTOR-split-mission-hub
type: REFACTOR
status: done
created_at: 2026-02-26
---

# 04-plan.md - Execution Plan for Hub Separation

## 1. As-Is

- Currently, `RescuerHub` operates as a dual-purpose monolith. It acts as both a "Job Board" broadcasting `RescueRequest` ping events to online rescuers, and as an active execution room (e.g., handling location updates and tracking).
- The `RescueRequestSessionService` generates pending rescue requests and broadcasts them using `SignalRRescueNotificationService` targeting `RescuerHub`.
- When a rescuer accepts, `RescuerHub.AcceptRequest` modifies the database but the rescuer remains tracked on `RescuerHub`.
- If a mission is canceled or aborted, there's complexity involving state rollbacks across all connected participants inside this single hub structure.

## 2. Gap Analysis

- The monolithic `RescuerHub` causes race conditions and rollback nightmares. If a rescuer cancels an accepted mission early, the system must juggle connections and status updates over the same channel used for job hunting.
- **Goal:** Break the application into an "Asymmetric Connection" model.
  - `RescuerHub`: Dedicated purely to idling rescuers looking for requests.
  - `MissionHub`: A dedicated instance per active mission for the `Member` and the assigned `Rescuer` only.
- The `To-Be` design explicitly relies on `01-architecture-decision.md`, `02-state-machine.md`, and `03-sequence-flows.md` for logical rules and state matrices.

## 3. To-Be Design

- **Create `MissionHub`:** A new SignalR hub mapped to `/hubs/mission`.
- **Authorization Check in `MissionHub`:** Overwrite `OnConnectedAsync` to fetch `missionId` from the context (e.g., query string). Query the DB (`RescueMission` or `SnakebiteIncident`) to verify `Context.UserIdentifier` equals either `mission.MemberId` or `mission.AssignedRescuerId`. Abort connection if invalid.
- **Presence Tracking:** Separate presence for Mission participants. If a connection drops, update a cache flag (e.g., `Member_Offline`, `Rescuer_Offline`) without altering the mission DB state.
- **Background Worker Update:** Modify the timeout handler in `RescueRequestSessionService.HandleSessionTimeoutAsync` (which expires a session after 3 phases x 60s) to push a `Session_Expired` command to the Member via `MissionHub`, and `Request_Expired` to rescuers via `RescuerHub`.
- **API Event Bindings:** Update `RescueMissionService.cs` (`UserCancelMissionAsync`, `RescuerAbortMissionAsync`, `CompleteMissionAsync`) to use the new `MissionHub` to emit events like `Rescuer_Cancelled` and `Mission_Cancelled`.

## 4. Impacted Components

- `SnakeAid.Api/Hubs/RescuerHub.cs` (Remove mission execution logic to be purely broadcast)
- `SnakeAid.Api/Hubs/MissionHub.cs` (NEW FILE)
- `SnakeAid.Api/Services/SignalRMissionNotificationService.cs` (NEW FILE)
- `SnakeAid.Service/Interfaces/IMissionNotificationService.cs` (NEW FILE)
- `SnakeAid.Service/Implements/RescueMissionService.cs` (Dispatch events to `MissionHub`)
- `SnakeAid.Service/Implements/RescueRequestSessionService.cs` (Update timeout to notify `MissionHub`)

## 5. Risks & Constraints

- **Risk:** Rescuer might accept a mission but fail to transition to the `MissionHub` due to network drops.
  - **Constraint Mitigation:** The flutter client must strictly auto-connect right after HTTP 200 OK from `POST /api/mission/accept`.
- **Risk:** Double-join.
  - **Constraint Mitigation:** Strictly enforce 1-rescuer max via DB transaction in `AcceptRequestAsync` (already implemented) and validate in `MissionHub.OnConnectedAsync`.
- **Risk:** Stale database queries during SignalR `OnConnectedAsync`.
  - **Constraint Mitigation:** Add Redis or in-memory caching for active missions to quickly validate `OnConnectedAsync` without hammering PostgreSQL.

## 6. Validation Plan

- **Verification 1:** Manual mobile client simulation or postman sockets simulating Member creating an incident and attaching to `/hubs/mission?incidentId=xxx`. Verify successful connection.
- **Verification 2:** Connect fake rescuer to `/hubs/mission?incidentId=xxx`. Verify `Context.Abort()` is triggered because the rescuer hasn't been assigned yet.
- **Verification 3:** Call `POST /api/mission/accept`. The assigned rescuer attaches to `/hubs/mission?incidentId=xxx`. Verify connection successful.
- **Verification 4:** Call `POST /api/mission/cancel` as rescuer. Verify the Member living inside `MissionHub` receives the `Rescuer_Cancelled` socket event immediately. Ensure incident goes back to `RescuerHub` queue for re-broadcast.
