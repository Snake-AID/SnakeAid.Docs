---
doc_role: planning
module: consultation-message-history
kind: flow
doc_type: introduction
status: planning
last_updated: 2026-04-21
owners: [backend-team]
verification_status: current-state-code-verified-with-planned-implementation-direction
---

# Consultation Message History Introduction

## Goal

This module plans the backend work needed to expose consultation chat history after a consultation has already ended.

Business goal:

- `Consultation` already supports message exchange similar to a messaging app
- message sending currently happens inside `ConsultationHub`
- messages are persisted to `ChatMessages`
- after the video call ends, users can no longer reconnect for active chat usage
- users currently have no HTTP endpoint to read message history for the finished consultation

Target behavior:

- a participant of a terminal consultation can retrieve the persisted chat history
- the endpoint is read-only
- this work does not re-open chat sending after completion

## Resume Summary

If this work must be resumed later with no prior chat memory, the current code-verified situation is:

1. `ConsultationHub.ReceiveMessage(...)` already saves each message to `ChatMessages`.
2. `ChatMessage` already contains `Id`, `ConsultationId`, `SenderId`, `Content`, `AttachmentUrl`, and `SentAt`.
3. `ConsultationHub.OnConnectedAsync(...)` already restricts realtime access to consultation participants only.
4. `ConsultationsController` currently has no endpoint for consultation message history.
5. `IConsultationService` currently has no method for consultation message history.
6. `Consultation.Status` already supports `Completed`.
7. `ConsultationService.EndConsultationAsync(...)` completes the consultation, but does not expose any post-call message-history read API.

## Code-Verified Current Backend State

### Messaging Persistence

Current verified facts:

- SignalR hub route exists at `/hubs/consultation`
- `ReceiveMessage(content, attachmentUrl)` persists a `ChatMessage`
- `ChatMessages` already exists as an EF Core table
- `ChatMessageConfiguration` already indexes:
  - `ConsultationId`
  - `SenderId`
  - `SentAt`

This means the core data needed for history retrieval already exists.

### Access Control

Current verified participant rule:

- realtime hub connection is allowed only when current user is either:
  - `Consultation.CallerId`
  - `Consultation.CalleeId`

Current recommended direction is to reuse the same ownership boundary for the future history endpoint.

### Missing Read Surface

What is missing today:

- no HTTP route to list messages for one consultation
- no response DTO dedicated to consultation messages
- no service-layer method that validates participant ownership and completed status before returning messages
- no integration tests covering post-completion message history retrieval

## Problem Statement

The current implementation stores consultation chat correctly, but the read path is incomplete.

That creates a business gap:

- the chat behaves like consultation-scoped messaging during the call
- the stored messages remain in the database
- the user still cannot review those messages once the active call experience is over

So the problem is not missing persistence.

The problem is missing retrieval contract.

## Recommended Implementation Direction

The current recommended direction for planning is:

- add a participant-facing read endpoint under `ConsultationsController`
- recommended route: `GET /api/consultations/{consultationId}/messages`
- allow retrieval only when `Consultation.Status` is one of:
  - `Completed`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`
- allow retrieval only for `CallerId` or `CalleeId`
- keep this endpoint read-only
- keep message sending inside `ConsultationHub` for now
- paginate results so the endpoint remains safe when long consultations produce many messages
- return each page in ascending order by `SentAt`, then `Id`
- use newest-window-first paging semantics:
  - `pageNumber = 1` returns the newest history batch
  - `pageNumber = 2` returns the next older batch
  - each next page continues backward to older history
- keep the response aligned with stored truth and do not add sender enrichment in v1

Recommended response shape should stay close to persisted truth:

- `id`
- `consultationId`
- `senderId`
- `content`
- `attachmentUrl`
- `sentAt`

Sender enrichment such as display name or role is intentionally excluded from v1.

## Scope Boundary

In scope:

- read persisted messages of one terminal consultation
- participant authorization
- terminal-status validation
- response DTOs and endpoint contract
- tests and docs for the new read path

Out of scope:

- reopening chat after consultation completion
- adding edit or delete message features
- adding message read receipts
- migrating messaging away from SignalR
- redesigning the current realtime send flow

## Expected Impacted Areas

- `SnakeAid.Api/Controllers/ConsultationsController.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Core/Responses/Consultation/*`
- integration tests for consultation APIs
- docs under `SnakeAid.Docs`

## Delivered Planning Artifacts

- `consultation-message-history.introduction.md`
- `consultation-message-history.roadmap.md`
- `consultation-message-history.hallucination.md`
- `consultation-message-history.sourcecode.md`
- `consultation-message-history.useguide.md`
