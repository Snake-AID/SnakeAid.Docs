# Multi-Agent Brainstorming: Decision Log

**Topic:** Separating Rescuer Hub and Mission Hub
**Lead Designer:** AI Assistant

## Phase 1: Single-Agent Design Proposal

- **Proposed Architecture:** Option C (Asymmetric Connection). Member connects immediately to MissionHub upon incident creation. RescuerHub is used strictly for broadcasting to idle rescuers. When a rescuer accepts, they join the MissionHub.
- **Rationale:** This drastically simplifies the most painful edge case (Early Cancellation). The Member never has to switch hubs or re-register to the RescuerHub. The RescuerHub remains a pure "job board".

---

## Phase 2: Structured Review Loop

### 1️⃣ Skeptic / Challenger Agent Review

**Objection 1: Timeout & Infinite Waiting**
Assume this design fails in production because a Member creates an incident, connects to `MissionHub`, but NO rescuer ever accepts. How does the `MissionHub` handle timeout? If the Member loses connection and reconnects, do they just sit in the `MissionHub` indefinitely?

**Objection 2: Race Condition on Accept & Unauthorized Hub Access**
Assume Rescuer A and Rescuer B accept at the same time. The API must reject one. But what if the mobile app preemptively tries to connect to `MissionHub`? The `MissionHub` must strictly validate that ONLY the currently assigned rescuer can join the group.

**Objection 3: Rescuer Re-entry**
Assume a Rescuer cancels _after_ joining the `MissionHub`. The ADR says they reconnect to `RescuerHub`. But the request is broadcasted _again_. What prevents the _same_ rescuer from receiving the broadcast and accepting it again? Or what if they ignore the redirect and stay connected to `MissionHub`?

**_Primary Designer Response:_**

- **Re: Objection 1**: We will implement a background job (or use Redis key expiration) that marks the mission as `Expired` after `X` minutes if no rescuer accepts. When this happens, a `Session_Expired` event is sent to the `MissionHub` and the Member is safely disconnected.
- **Re: Objection 2**: We will add an Authorization filter/check in `MissionHub.OnConnectedAsync`. It will verify that the connecting `UserId` is exactly the `AssignedRescuerId` for that mission, or the `MemberId`. Anyone else gets `Context.Abort()`.
- **Re: Objection 3**: The query that finds eligible rescuers to broadcast to via `RescuerHub` will explicitly filter out any `RescuerId` that has a `Cancelled` status for this specific mission. Furthermore, `MissionHub` will kick them upon cancellation.

### 2️⃣ Constraint Guardian Agent Review

**Objection 4: Connection Drain**
Assume the mission takes 2 hours. Keeping a continuous SignalR connection alive drains mobile battery and server resources.
**Objection 5: Missed Critical Events**
If a rescuer is in `MissionHub`, they are disconnected from `RescuerHub`. If a new, higher-priority emergency happens 50m away from them, they will blind to it.

**_Primary Designer Response:_**

- **Re: Objection 4**: SignalR group creation is virtual and cheap on memory. For battery drain, we rely on the standard Flutter backgrounding behavior. If the socket closes, we rely on REST API fallbacks until it reconnects.
- **Re: Objection 5**: This is a known product constraint. Currently, a rescuer can only handle 1 mission at a time. This is out-of-scope for the Hub split.

### 3️⃣ User Advocate Agent Review

**Objection 6: Accidental Disconnects = Cancel?**
If the Member closes the app in a panic to call their family, does the connection drop cancel the mission? It MUST NOT.
**Objection 7: Seamless Transition**
The Rescuer must not have to click "Join Room" after accepting. It must be automatic.

**_Primary Designer Response:_**

- **Re: Objection 6**: Agreed. SignalR `OnDisconnectedAsync` will ONLY update a "Presence" flag in Redis (e.g., `Member_Offline`). It will **never** alter the Mission State in the DB. Only explicit API calls can cancel the mission.
- **Re: Objection 7**: Agreed. The Flutter client will be instructed to automatically initialize the `MissionHub` connection the moment the `POST /api/mission/accept` returns a `200 OK`.

---

## Phase 3: Integration & Arbitration

**Arbiter Decision:** **APPROVED**

- The Asymmetric Connection (Option C) perfectly solves the "rollback" nightmare of Option A.
- The constraints and edge cases brought up by the Skeptic and User Advocate have solid mitigations (Auth checks on `OnConnected`, Presence vs State separation).
- The design is locked. Proceed to update the ADR and State Machine with these mitigations.
