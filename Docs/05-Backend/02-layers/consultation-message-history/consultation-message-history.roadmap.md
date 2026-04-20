---
doc_role: planning
module: consultation-message-history
kind: flow
doc_type: roadmap
status: planning
last_updated: 2026-04-21
owners: [backend-team]
verification_status: current-state-code-verified-with-planned-api-contract
---

# Consultation Message History Roadmap

## Current Status Snapshot

- module status: `Planning`
- current message persistence: `Implemented`
- current message history HTTP API: `Missing`
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
- no controller or service contract currently returns consultation messages

## Target Outcome

After this work is complete:

1. a consultation participant can request message history through HTTP
2. the endpoint returns persisted messages for one consultation
3. only completed consultations are eligible for this read path
4. only `CallerId` or `CalleeId` can read the history
5. the endpoint is read-only and does not extend chat sending after completion
6. docs and tests clearly separate implemented behavior from planned behavior during rollout

## Provisional Decisions

- [x] Reuse `ConsultationsController` instead of creating a separate controller
- [x] Keep message send flow in `ConsultationHub`
- [x] Add a read-only endpoint for completed consultations
- [x] Reuse participant ownership rule based on `CallerId` and `CalleeId`
- [ ] Lock final paging and ordering contract
- [ ] Lock whether sender metadata beyond `senderId` is part of v1 response

## Implementation Checklist

### Phase 1. Contract Lock

- [ ] Lock route template for message history
- [ ] Lock completed-only rule
- [ ] Lock pagination fields
- [ ] Lock sort order
- [ ] Lock minimal response DTO shape
- [ ] Move unresolved API shape questions to `hallucination`

### Phase 2. Service Layer

- [ ] Add a service method to retrieve consultation message history
- [ ] Validate consultation existence
- [ ] Validate actor is a participant
- [ ] Validate consultation is `Completed`
- [ ] Query `ChatMessages` by `ConsultationId`
- [ ] Order and paginate safely

### Phase 3. API Layer

- [ ] Add `GET /api/consultations/{consultationId}/messages`
- [ ] Bind paging query parameters
- [ ] Return typed `ApiResponse`
- [ ] Keep authorization aligned with existing consultation endpoints

### Phase 4. DTOs

- [ ] Add query request model for message-history paging
- [ ] Add response DTO for one consultation message
- [ ] Add paging wrapper usage if the endpoint is paged

### Phase 5. Tests

- [ ] Participant of completed consultation can read history
- [ ] Non-participant gets forbidden
- [ ] Participant of non-completed consultation is rejected according to locked rule
- [ ] Messages are returned in the locked order
- [ ] Attachment-only messages are preserved correctly
- [ ] Empty history returns success with empty items

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
2. call the history endpoint as caller
3. call the history endpoint as callee
4. confirm a third-party user cannot read the same consultation
5. confirm non-completed consultation behavior matches the locked rule
6. confirm ordering and pagination match the documented contract
7. confirm response examples in docs match real payload shape

## Open Questions

1. Should the endpoint be completed-only, or should ongoing consultations also be allowed to read already persisted history?
2. Should the response include sender enrichment such as display name or role?
3. Should v1 default ordering be ascending by `SentAt` for chat replay, or descending for recent-first history screens?
4. Should mobile receive all messages in one list for completed consultations, or a paged response with infinite scroll?

## Change Log

### 2026-04-21

- initialized planning docs for consultation message history
- documented that persistence already exists in `ChatMessages`
- documented that the missing piece is the post-completion HTTP read contract
- proposed a participant-only completed-consultation history endpoint
