---
doc_role: planning
module: consultation-message-history
kind: flow
doc_type: roadmap
status: in_progress
last_updated: 2026-04-21
owners: [backend-team]
verification_status: current-state-code-verified-with-planned-api-contract
---

# Consultation Message History Roadmap

## Current Status Snapshot

- module status: `Planning`
- current message persistence: `Implemented`
- current message history HTTP API: `Implemented`
- current participant validation in realtime hub: `Implemented`
- current post-completion message review for app users: `Not available`

## Current Truth To Resume From

This roadmap is written so work can resume from zero memory.

Current verified state:

- `ConsultationHub.ReceiveMessage(...)` inserts `ChatMessage`
- messages are stored in `ChatMessages`
- users connect to the hub with query `consultationId`
- hub access is restricted to consultation participants
- `ConsultationService.EndConsultationAsync(...)` can mark consultation as completed
- controller and service contract now return consultation message history

## Target Outcome

After this work is complete:

1. a consultation participant can request message history through HTTP
2. the endpoint returns persisted messages for one consultation
3. only terminal consultation states are eligible for this read path
4. only `CallerId` or `CalleeId` can read the history
5. the endpoint is read-only and does not extend chat sending after completion
6. docs and tests clearly separate implemented behavior from planned behavior during rollout

## Provisional Decisions

- [x] Reuse `ConsultationsController` instead of creating a separate controller
- [x] Use route `GET /api/consultations/{consultationId}/messages-history`
- [x] Keep message send flow in `ConsultationHub`
- [x] Add a read-only endpoint for terminal consultations
- [x] Reuse participant ownership rule based on `CallerId` and `CalleeId`
- [x] Allow admin to access the endpoint
- [x] Do not add sender enrichment in v1 beyond `senderId`
- [x] Use ascending message order by `SentAt`, then `Id`
- [x] Reuse `PagingResponse<T>` with newest-window-first page semantics
- [x] Lock `pageNumber = 1` as newest batch and each next page as the next older batch

## Implementation Checklist

### Phase 1. Contract Lock

- [ ] Lock route template for message history
- [x] Lock terminal-state rule
- [x] Lock pagination fields
- [x] Lock sort order
- [x] Lock minimal response DTO shape
- [x] Lock newest-window-first paging behavior for mobile UX

### Phase 2. Service Layer

- [x] Add a service method to retrieve consultation message history
- [x] Validate consultation existence
- [x] Validate actor is a participant or admin
- [x] Validate consultation is in an allowed terminal state
- [x] Query `ChatMessages` by `ConsultationId`
- [x] Order ascending by `SentAt`, then `Id`
- [x] Implement newest-window-first paging using `pageNumber` and `pageSize`

### Phase 3. API Layer

- [x] Add `GET /api/consultations/{consultationId}/messages-history`
- [x] Bind paging query parameters
- [x] Return typed `ApiResponse`
- [x] Keep authorization aligned with existing consultation endpoints

### Phase 4. DTOs

- [x] Add query request model for message-history paging
- [x] Add response DTO for one consultation message
- [x] Add paging wrapper usage if the endpoint is paged

### Phase 5. Tests

- [x] Participant of terminal consultation can read history
- [x] Admin can read history without being a participant
- [x] Non-participant gets forbidden
- [x] Participant of non-terminal consultation is rejected with business validation mapped to `400`
- [x] Messages are returned in the locked order
- [x] Attachment-only messages are preserved correctly
- [ ] Empty history returns success with empty items
- [x] `pageNumber = 1` returns newest batch while items stay ascending inside the batch

### Phase 6. Documentation Sync

- [ ] Update `introduction` with implementation status once coding starts
- [ ] Update `sourcecode` diagrams to implemented method names
- [ ] Update `useguide` from planned contract to active contract after code verification
- [ ] Resolve or explicitly defer all open decisions in `hallucination`

## Candidate File Targets

- [ ] `SnakeAid.Api/Controllers/ConsultationsController.cs`
- [ ] `SnakeAid.Service/Interfaces/IConsultationService.cs`
- [ ] `SnakeAid.Service/Implements/ConsultationService.cs`
- [ ] `SnakeAid.Core/Requests/Consultation/*`
- [ ] `SnakeAid.Core/Responses/Consultation/*`
- [ ] `SnakeAid.Tests/Integration/*Consultation*.cs`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-message-history/consultation-message-history.introduction.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-message-history/consultation-message-history.roadmap.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-message-history/consultation-message-history.hallucination.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-message-history/consultation-message-history.sourcecode.md`
- [x] `SnakeAid.Docs/Docs/05-Backend/02-layers/consultation-message-history/consultation-message-history.useguide.md`

## Verification Strategy

Minimum verification before activating the endpoint contract in `useguide`:

1. create or seed a completed consultation with stored `ChatMessages`
2. call `pageNumber = 1` and confirm newest batch is returned
3. call the next page and confirm it returns the next older batch
4. confirm a third-party user cannot read the same consultation
5. confirm non-terminal consultation behavior matches the locked rule
6. confirm ordering and pagination match the documented contract
7. confirm response examples in docs match real payload shape

## Open Questions

1. Should `Cancelled` ever be treated as a terminal readable state for consultation messages?

## Change Log

### 2026-04-21

- initialized planning docs for consultation message history
- documented that persistence already exists in `ChatMessages`
- documented that the missing piece is the post-completion HTTP read contract
- proposed a participant-only terminal-consultation history endpoint
- locked v1 response to stored-truth message fields without sender enrichment
- locked ascending order by `SentAt`, then `Id`
- locked newest-window-first paging semantics using `pageNumber` and `pageSize`

### 2026-04-21 Implementation Update

- implemented `GET /api/consultations/{consultationId}/messages-history`
- added `ConsultationMessageHistoryQueryRequest`
- added `ConsultationMessageHistoryItemResponse`
- allowed admin access in addition to participant access
- implemented newest-window-first paging with ascending ordering inside each page
- added controller tests, route convention tests, and integration tests for the new endpoint logic
