---
doc_role: operation
operation_id: REFACTOR-split-mission-hub
type: REFACTOR
status: draft
created_at: 2026-02-26
---

# Sequence Flows: Asymmetric Hub Connection

This document visualizes the exact API and SignalR interactions between the Mobile Apps (Member & Rescuer), the REST API, the `RescuerHub`, and the `MissionHub`.

## 1. Happy Path: Broadcast & Accept

The member creates an incident and waits in `MissionHub`. A rescuer receives the broadcast on `RescuerHub`, accepts it, and joins the `MissionHub`.

```mermaid
sequenceDiagram
    autonumber
    participant M as Member App
    participant API as REST API
    participant MH as MissionHub
    participant RH as RescuerHub
    participant R as Idle Rescuer App

    R->>RH: Connect & Listen for requests

    M->>API: POST /api/mission/create
    API-->>M: 200 OK (MissionId)

    M->>MH: Connect to MissionHub(MissionId)
    API->>RH: Broadcast Request_Created(MissionId)
    RH-->>R: Receive broadcast

    R->>API: POST /api/mission/{MissionId}/accept
    API->>API: Validate & Lock Mission to Rescuer
    API-->>R: 200 OK
    API->>RH: Broadcast Request_Taken(MissionId)

    R->>MH: Connect to MissionHub(MissionId)
    MH->>MH: Validate R == AssignedRescuerId
    MH-->>M: Push Rescuer_Joined event
    MH-->>R: Push Joined_Successfully event

    note over M,R: Both parties now communicate exclusively via MissionHub
```

## 2. Edge Case: Rescuer Cancels Before EnRoute

The rescuer accepts, joins the `MissionHub`, but then cancels. The member remains safely in `MissionHub` while the system re-broadcasts.

```mermaid
sequenceDiagram
    autonumber
    participant M as Member App
    participant API as REST API
    participant MH as MissionHub
    participant RH as RescuerHub
    participant R as Assigned Rescuer App

    note over M,R: Both are connected to MissionHub

    R->>API: POST /api/mission/{MissionId}/cancel
    API->>API: Update Mission State (Rescuer Unassigned/Banned from this mission)
    API-->>R: 200 OK

    API->>MH: Push Rescuer_Cancelled event
    MH-->>M: Receive event (UI updates: "Finding new rescuer")
    MH-->>R: Receive event (Auto disconnect)

    R->>RH: Reconnect to RescuerHub

    API->>RH: Broadcast Request_Created(MissionId) (Phase 2)
    note right of RH: Query excludes the rescuer who just cancelled
```

## 3. Edge Case: Member Cancels While Waiting

The member creates the incident but cancels before anyone accepts.

```mermaid
sequenceDiagram
    autonumber
    participant M as Member App
    participant API as REST API
    participant MH as MissionHub
    participant RH as RescuerHub
    participant R as Idle Rescuer App

    note over M,MH: Member is connected to MissionHub

    M->>API: POST /api/mission/{MissionId}/cancel
    API->>API: Mark Mission Cancelled_By_User
    API-->>M: 200 OK

    API->>MH: Push Mission_Cancelled event
    MH-->>M: Receive event (Auto disconnect)

    API->>RH: Broadcast Request_Cancelled(MissionId)
    RH-->>R: Receive event (Remove from UI map)
```

## 4. Edge Case: Timeout (No Rescuer Accepts)

After 3 phases (180 seconds), no rescuer accepts. The mission expires.

```mermaid
sequenceDiagram
    autonumber
    participant M as Member App
    participant Worker as Background Worker (Redis/Hangfire)
    participant MH as MissionHub
    participant RH as RescuerHub
    participant R as Idle Rescuer App

    note over M,MH: Member is connected to MissionHub
    note right of Worker: 3 Phases x 60s (180s total) elapse

    Worker->>Worker: Mark Mission as Expired

    Worker->>MH: Push Session_Expired event
    MH-->>M: Receive event (Show fallback UI, Auto disconnect)

    Worker->>RH: Broadcast Request_Expired(MissionId)
    RH-->>R: Receive event (Remove from UI map)
```
