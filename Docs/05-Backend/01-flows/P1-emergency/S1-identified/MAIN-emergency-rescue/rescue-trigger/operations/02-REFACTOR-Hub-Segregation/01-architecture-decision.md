---
doc_role: operation
operation_id: REFACTOR-split-mission-hub
type: REFACTOR
status: draft
created_at: 2026-02-26
---

# ADR 01: Strategy for Separating Rescuer Hub and Mission Hub

## 1. Context & Problem Statement

Currently, `RescuerHub` handles both the pairing phase (broadcasting emergency requests to rescuers) and the active mission phase (coordinating the rescuer and member during the rescue).

**New Direction:**

- **Rescuer Hub:** Exclusively used for broadcasting and receiving new requests.
- **Mission Hub:** A dedicated hub created per mission, exclusively for the assigned Rescuer and the Member.

**The Problem:**
Determining the exact moment to transition the Member and Rescuer into the `MissionHub`.
If we transition them _immediately_ when a rescuer accepts the request:

- **User Cancels:** The `MissionHub` must notify the rescuer (for app redirect), but the system must also notify the `RescuerHub` to clear the request for other rescuers (if it was somehow still pending or for audit).
- **Rescuer Cancels (Before EnRoute):** The system must notify both hubs, remove the rescuer from the `MissionHub`, and crucially, **re-register the user back into the `RescuerHub`** to broadcast to a new batch of rescuers (Phase 2/3 of finding a rescuer).

This introduces heavy state churn, complex rollback logic, and racing conditions across two separate SignalR connections.

## 2. Options Considered

### Option A: Transition Immediately on `Accept` (The Initial Problematic Flow)

- **Flow:** Member request -> Broadcast on RescuerHub -> Rescuer A accepts -> Both Member and Rescuer A disconnect from RescuerHub and connect to MissionHub.
- **Pros:** Conceptually isolates the mission immediately.
- **Cons:** High overhead for rollbacks. If Rescuer A cancels early, the Member must drop MissionHub, reconnect to RescuerHub, and trigger a new broadcast. This is prone to connection drops and missed events.

### Option B: Transition on `EnRoute`

- **Flow:** Rescuer accepts on RescuerHub -> Stays on RescuerHub (in a private group). Once Rescuer taps "En Route" -> Both transition to MissionHub.
- **Pros:** Early cancellations (most common) happen before the hub transition, keeping rollback simple.
- **Cons:** Transitioning hubs while the rescuer is physically starting to move might cause connectivity drops precisely when live tracking needs to start.

### Option C: Asymmetric Connection (Member in Mission Hub, Rescuers in Rescuer Hub)

- **Flow:**
  1. Member creates an incident and **immediately connects to the `MissionHub`** (sitting alone initially).
  2. The system broadcasts the incident via `RescuerHub` to all idle rescuers.
  3. Rescuer A accepts the request via API.
  4. Rescuer A connects to the `MissionHub` to join the Member.
- **Pros:**
  - The Member **never** changes hubs. Zero risk of the victim losing connection during a panic state.
  - If Rescuer A cancels, Rescuer A simply disconnects from `MissionHub`. The system broadcasts the request again on `RescuerHub`. The Member stays put in `MissionHub`.
  - Perfect separation of concerns: `RescuerHub` is strictly a "job board". `MissionHub` is the "execution room".
- **Cons:**
  - Requires Member to connect to MissionHub before a rescuer is even found.
  - Mission Hub needs logic to handle "waiting for rescuer" state.

## 3. Proposed Solution (Phase 1 Design)

**Option C (Asymmetric Connection)** is proposed as the primary design.

By keeping the Member stationary in the `MissionHub` from the moment the incident is created, we completely eliminate the complex rollback of "re-registering the user into the Rescuer Hub". The Rescuer Hub simply becomes a one-way notification channel for idle rescuers.

## 4. Final Decision & Mitigations (Post Multi-Agent Review)

**Decision: APPROVED Option C (Asymmetric Connection).**
This design acts as the foundation for the breaking change.

**Key Mitigations Identified:**

1. **Timeout Handling:** Current logic spans 3 phases (each 60s). If no rescuer accepts after all 3 phases (total 180s), a background worker marks the mission as `Expired`. `MissionHub` pushes a `Session_Expired` event to the Member to close the UI gracefully.
2. **Race Conditions & Unauthorized Access:** Two rescuers might accept simultaneously, but the API will only commit one to the DB. `MissionHub.OnConnectedAsync` must strictly validate that `Context.UserIdentifier` equals the mission's `AssignedRescuerId` (or the `MemberId`). Unauthorized connections will be immediately aborted.
3. **Rescuer Re-entry Prevention:** When the system (re)broadcasts requests to `RescuerHub`, the query must filter out any Rescuer who previously cancelled this specific mission.
4. **Accidental Disconnects:** Dropping the SignalR connection MUST NOT cancel the mission. `OnDisconnectedAsync` will only update a "Presence" state (e.g., `Member_Offline`). The mission state remains unchanged until explicit API cancellation.
5. **Seamless Transition:** The mobile client will automatically initiate the `MissionHub` connection the moment the `POST /api/mission/accept` API returns a success response. No manual "join room" interaction is required from the Rescuer.
