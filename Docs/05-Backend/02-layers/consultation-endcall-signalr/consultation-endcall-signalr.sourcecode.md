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
- `ConsultationCallTerminationSignal`

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

`RoomExpiring` is currently emitted only by the timeout flow inside `BookingService`.

## 3. Planned Backend Surface

### Shared Realtime Contract

- `ConsultationCallTerminated`

Proposed payload:

- `consultationId`
- `roomName`
- `reason`
- `endedByUserId`
- `endedByRole`
- `triggeredAtUtc`
- `shouldLeaveCall`

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

    class ConsultationCallTerminationSignal {
        +Guid ConsultationId
        +string RoomName
        +string Reason
        +Guid? EndedByUserId
        +string? EndedByRole
        +DateTime TriggeredAtUtc
        +bool ShouldLeaveCall
    }

    ConsultationsController --> ConsultationService
    ConsultationLifecycleBackgroundService --> BookingService
    ConsultationService --> IConsultationRealtimeNotifier
    BookingService --> IConsultationRealtimeNotifier
    ConsultationService --> LiveKitService
    BookingService --> LiveKitService
    ConsultationRealtimeNotifier ..|> IConsultationRealtimeNotifier
    ConsultationRealtimeNotifier --> ConsultationHub
    ConsultationRealtimeNotifier --> ConsultationCallTerminationSignal
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
    Booking->>Notify: NotifyCallTerminatedAsync(reason="timeout")
    Notify->>Hub: ConsultationCallTerminated(payload)
    Hub-->>Flutter: ConsultationCallTerminated
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
    Service->>Notify: NotifyCallTerminatedAsync(reason="participant_ended")
    Notify->>Hub: ConsultationCallTerminated(payload)
    Hub-->>Flutter: ConsultationCallTerminated
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

### Manual End Path

- `ConsultationService` does not currently inject `IHubContext<ConsultationHub>`
- no SignalR event is currently emitted when calling `POST /api/consultations/{consultationId}/end`
- `DeleteRoomAsync(...)` is not currently part of the manual-end path

## 7. Design Notes

1. `BookingService` and `ConsultationService` should not each build separate anonymous termination payloads because both pushes are meant to trigger the same Flutter endcall behavior.
2. A typed payload is preferable because it makes payload-shape testing easier.
3. The existing hub route and group format should remain unchanged so Flutter does not need a new connection strategy.
4. If rollout safety is required, the notifier can support:
   - only the new event
   - or the new event plus `RoomExpiring` for a short transition period

## 8. Test Focus

- ordering:
  - signal before `DeleteRoomAsync`
  - `DeleteRoomAsync` before `CommitAsync`
- payload:
  - `consultationId`
  - `roomName`
  - `reason`
  - `endedByRole`
- resiliency:
  - SignalR failure must not block DB completion
  - room-deletion failure must not roll back business completion if the flow remains best-effort
