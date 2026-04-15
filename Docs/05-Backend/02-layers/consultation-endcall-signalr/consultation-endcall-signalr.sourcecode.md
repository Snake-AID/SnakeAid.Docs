---
doc_role: planning
module: consultation-endcall-signalr
kind: flow
doc_type: sourcecode
status: proposed
last_updated: 2026-04-15
owners: [backend-team]
verification_status: mixed-current-code-and-target-design
---

# Consultation EndCall SignalR Sourcecode

## 1. Relevant Classes

Current code-verified classes:

- `ConsultationHub`
- `ConsultationLifecycleBackgroundService`
- `BookingService`
- `ConsultationService`
- `ConsultationsController`
- `VideoCallController`
- `LiveKitService`
- `RoomCleanupTests`
- `ScheduledConsultationIntegrationTests`

Planned classes and contracts:

- `IConsultationRealtimeNotifier`
- `ConsultationRealtimeNotifier`

## 2. Current Backend Surface

### HTTP

- `POST /api/consultations/{consultationId}/end`
- `POST /api/consultations/{consultationId}/video-token`

### SignalR Hub

- hub route: `/hubs/consultation`
- group format: `consultation:{consultationId}`

### Current Server-To-Client Events

- `ReceiveMessage`
- `Signal`
- `RoomExpiring`

`RoomExpiring` is currently the only termination-like event already implemented in Flutter.

## 3. Planned Backend Surface

### Shared Realtime Contract

- `RoomExpiring`

Proposed payload:

- `consultationId`
- `reason`

## 4. Class Diagram

```mermaid
classDiagram
    class ConsultationsController {
        +EndConsultation(consultationId)
    }

    class ConsultationService {
        +EndConsultationAsync(consultationId, actorId)
    }

    class ConsultationLifecycleBackgroundService {
        +ExecuteAsync(stoppingToken)
    }

    class BookingService {
        +AutoCompleteElapsedScheduledConsultationsAsync(cancellationToken)
        +AutoCompleteElapsedEmergencyConsultationsAsync(cancellationToken)
    }

    class ConsultationHub {
        +OnConnectedAsync()
        +OnDisconnectedAsync(exception)
        +ReceiveMessage(content, attachmentUrl)
        +Signal(eventType, payload)
    }

    class LiveKitService {
        +GenerateAccessToken(identity, roomName, grants, metadata)
        +DeleteRoomAsync(roomName, cancellationToken)
    }

    class IConsultationRealtimeNotifier {
        +NotifyCallTerminatedAsync(signal, cancellationToken)
    }

    class ConsultationRealtimeNotifier {
        -IHubContext~ConsultationHub~ _hubContext
        +NotifyCallTerminatedAsync(signal, cancellationToken)
    }

    class ConsultationRoomExpiringSignal {
        +Guid ConsultationId
        +string Reason
    }

    ConsultationsController --> ConsultationService
    ConsultationLifecycleBackgroundService --> BookingService
    ConsultationService --> IConsultationRealtimeNotifier
    BookingService --> IConsultationRealtimeNotifier
    ConsultationService --> LiveKitService
    BookingService --> LiveKitService
    ConsultationRealtimeNotifier ..|> IConsultationRealtimeNotifier
    ConsultationRealtimeNotifier --> ConsultationHub
    ConsultationRealtimeNotifier --> ConsultationRoomExpiringSignal
```

## 5. Sequence Diagram

### 5.1 Timeout Flow

```mermaid
sequenceDiagram
    participant BG as ConsultationLifecycleBackgroundService
    participant Booking as BookingService
    participant Notify as ConsultationRealtimeNotifier
    participant Hub as ConsultationHub Group
    participant Flutter as Flutter Client
    participant LiveKit as LiveKitService
    participant DB as Database

    BG->>Booking: AutoCompleteElapsed...()
    Booking->>Notify: NotifyRoomExpiringAsync(reason="timeout")
    Notify->>Hub: RoomExpiring(payload)
    Hub-->>Flutter: RoomExpiring
    Flutter->>Flutter: endcall + leave room
    Booking->>LiveKit: DeleteRoomAsync(roomName)
    Booking->>DB: update consultation/booking/slot status
    Booking->>DB: CommitAsync()
```

### 5.2 Manual End Flow

```mermaid
sequenceDiagram
    participant App as Member/Expert App
    participant API as ConsultationsController
    participant Service as ConsultationService
    participant Notify as ConsultationRealtimeNotifier
    participant Hub as ConsultationHub Group
    participant Flutter as Flutter Client
    participant LiveKit as LiveKitService
    participant DB as Database

    App->>API: POST /api/consultations/{consultationId}/end
    API->>Service: EndConsultationAsync(consultationId, actorId)
    Service->>Notify: NotifyRoomExpiringAsync(reason="participant_ended")
    Notify->>Hub: RoomExpiring(payload)
    Hub-->>Flutter: RoomExpiring
    Flutter->>Flutter: endcall + leave room
    Service->>LiveKit: DeleteRoomAsync(roomName)
    Service->>DB: update consultation/booking/slot status
    Service->>DB: CommitAsync()
    API-->>App: 200 success
```

## 6. Current Code-Verified Observations

### Timeout Path

- the timeout path already depends on `IHubContext<ConsultationHub>` inside `BookingService`
- the current event name is `RoomExpiring`
- the current payload is:
  - `ConsultationId`
  - `Reason = "slot_elapsed"`
- room deletion is already tested with the format `consultation-{consultationId}`
- Flutter already treats `RoomExpiring` as a forced termination flow in the live consultation screen

### Manual End Path

- `ConsultationService` does not currently inject `IHubContext<ConsultationHub>`
- no SignalR event is currently emitted when calling `POST /api/consultations/{consultationId}/end`
- `DeleteRoomAsync(...)` is not currently part of the manual-end path
- observed integration finding: when manual end triggers `RoomExpiring`, the expert still does not automatically leave the room in practice

## 7. Design Notes

1. `BookingService` and `ConsultationService` should not each build separate anonymous `RoomExpiring` payloads because both pushes are meant to trigger the same Flutter endcall behavior.
2. A typed payload is still preferable because it makes payload-shape testing easier.
3. The existing hub route, group format, and event name should remain unchanged so Flutter does not need a parallel termination contract.
4. The main remaining problem is not naming; it is manual-end delivery and expert-side handling consistency.

## 8. Test Focus

- ordering:
  - signal before `DeleteRoomAsync`
  - `DeleteRoomAsync` before `CommitAsync`
- payload:
  - `consultationId`
  - `reason`
- resiliency:
  - SignalR failure must not block DB completion
  - room-deletion failure must not roll back business completion if the flow remains best-effort
- integration finding:
  - manual-end-triggered `RoomExpiring` must be verified on both member and expert active-call clients
