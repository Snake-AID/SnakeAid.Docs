---
doc_role: implementation
module: consultation-message-history
kind: flow
doc_type: sourcecode
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: code-verified
---

# Consultation Message History Sourcecode

## 1. Relevant Classes

### Backend

- `ConsultationsController`
- `IConsultationService`
- `ConsultationService`
- `ConsultationHub`
- `Consultation`
- `ChatMessage`
- `ChatMessageConfiguration`
- `SnakeAidDbContext`

## 2. Code-Verified Current Backend Surface

### HTTP

Current related consultation routes:

- `POST /api/consultations/{consultationId}/end`
- `GET /api/users/me/consultations`
- `POST /api/consultations/{consultationId}/expert-absent-report`
- `POST /api/consultations/{consultationId}/reviews`
- `GET /api/consultations/{consultationId}/reviews`

- `GET /api/consultations/{consultationId}/messages-history`

### SignalR

Current messaging surface:

- hub route: `/hubs/consultation`
- connection query: `consultationId=<guid>`
- client method to server: `ReceiveMessage(content, attachmentUrl)`
- server event to group: `ReceiveMessage`

### Persistence

Current message persistence:

- entity: `ChatMessage`
- table: `ChatMessages`
- foreign key to `Consultation`
- foreign key to `Account` as sender
- indexes on `ConsultationId`, `SenderId`, `SentAt`

## 3. Code-Verified Current Flow

### Current Realtime Send Flow

`ConsultationHub.ReceiveMessage(...)` currently:

1. extracts `consultationId` from query string
2. extracts current user id from JWT
3. checks in-memory rate limit
4. validates that message has content or attachment
5. inserts `ChatMessage`
6. commits to database
7. broadcasts `ReceiveMessage` to SignalR group `consultation:{consultationId}`

### Current Participation Check

`ConsultationHub.OnConnectedAsync(...)` currently:

1. loads the consultation
2. rejects missing consultation
3. rejects users who are neither caller nor callee
4. adds valid participant to SignalR group

This is the cleanest existing authorization rule to reuse for history reads.

## 4. Implemented Backend Surface

Implemented surface:

- controller route: `GET /api/consultations/{consultationId}/messages-history`
- controller action: `ConsultationsController.GetMessageHistory(...)`
- service method: `IConsultationService.GetConsultationMessageHistoryAsync(...)`
- service implementation: `ConsultationService.GetConsultationMessageHistoryAsync(...)`
- response DTO: `ConsultationMessageHistoryItemResponse`
- query DTO: `ConsultationMessageHistoryQueryRequest`
- terminal-state validation for:
  - `Cancelled`
  - `Completed`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`
- access rule:
  - participant
  - or admin
- DB-side paging:
  - `CountAsync()` on the newest-first query
  - `Skip(...)` and `Take(...)` on the same query
- service-level paging normalization:
  - `pageNumber < 1` becomes `1`
  - `pageSize < 1` becomes `10`
- newest-window-first page selection with ascending item order inside each page
- `pageNumber = 1` means newest history batch

## 5. Class Diagram

```mermaid
classDiagram
    class ConsultationsController {
        +GetMessageHistory(Guid consultationId, ConsultationMessageHistoryQueryRequest query)
    }

    class IConsultationService {
        +GetConsultationMessageHistoryAsync(Guid consultationId, Guid actorId, bool isAdmin, ConsultationMessageHistoryQueryRequest query)
    }

    class ConsultationService {
        +GetConsultationMessageHistoryAsync(Guid consultationId, Guid actorId, bool isAdmin, ConsultationMessageHistoryQueryRequest query)
    }

    class Consultation {
        +Guid Id
        +Guid CallerId
        +Guid CalleeId
        +ConsultationStatus Status
    }

    class ChatMessage {
        +Guid Id
        +Guid ConsultationId
        +Guid SenderId
        +string Content
        +string AttachmentUrl
        +DateTime SentAt
    }

    class ConsultationMessageHistoryItemResponse {
        +Guid Id
        +Guid ConsultationId
        +Guid SenderId
        +string Content
        +string AttachmentUrl
        +DateTime SentAt
    }

    class ConsultationMessageHistoryQueryRequest {
        +int PageNumber
        +int PageSize
    }

    ConsultationsController --> IConsultationService
    ConsultationService ..|> IConsultationService
    ConsultationService --> Consultation
    ConsultationService --> ChatMessage
    ChatMessage --> Consultation
    ConsultationsController --> ConsultationMessageHistoryQueryRequest
```

## 6. Sequence Diagram

### 6.1 Current Send Flow

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant Hub as ConsultationHub
    participant DB as Database
    participant Group as SignalR Group

    App->>Hub: ReceiveMessage(content, attachmentUrl)
    Hub->>Hub: validate participant identity
    Hub->>Hub: validate rate limit
    Hub->>DB: insert ChatMessage
    Hub->>DB: CommitAsync()
    Hub-->>Group: ReceiveMessage(messagePayload)
```

### 6.2 Implemented History Read Flow

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant API as ConsultationsController
    participant Service as ConsultationService
    participant DB as Database

    App->>API: GET /api/consultations/{consultationId}/messages-history?pageNumber=1&pageSize=10
    API->>Service: GetConsultationMessageHistoryAsync(consultationId, actorId, isAdmin, query)
    Service->>DB: load Consultation
    Service->>Service: validate participant ownership or admin access
    Service->>Service: validate terminal consultation status
    Service->>Service: normalize paging request defensively
    Service->>DB: count ChatMessages by ConsultationId
    Service->>DB: query newest-first page window with Skip/Take
    Service->>Service: return items ascending by SentAt, then Id
    Service-->>API: PagingResponse<ConsultationMessageHistoryItemResponse>
    API-->>App: ApiResponse(...)
```

## 7. Test Focus

- consultation participants can read history for `Cancelled`, `Completed`, `UserAbsent`, `ExpertAbsent`, and `AllAbsent`
- admin can read history even when not a participant
- non-participants cannot read them
- ordering matches the locked contract
- attachment-only messages remain renderable
- page `1` returns the newest message window while preserving ascending order inside the page
- page `2` returns the next older message window without overlap or reversal
- paging is executed at DB level rather than materializing the whole consultation history
- the endpoint stays read-only and does not alter realtime messaging behavior
