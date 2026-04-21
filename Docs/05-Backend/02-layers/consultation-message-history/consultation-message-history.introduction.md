---
doc_role: implementation
module: consultation-message-history
kind: flow
doc_type: introduction
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: code-verified
---

# Consultation Message History Introduction

## Goal

This module documents the implemented backend support for exposing consultation chat history after a consultation has already ended.

Business goal:

- `Consultation` already supports message exchange similar to a messaging app
- message sending currently happens inside `ConsultationHub`
- messages are persisted to `ChatMessages`
- after the video call ends, users can no longer reconnect for active chat usage
- users and admin now have an HTTP endpoint to read message history for terminal consultations

Target behavior:

- a participant of a terminal consultation can retrieve the persisted chat history
- the endpoint is read-only
- this work does not re-open chat sending after completion

## Resume Summary

If this module must be resumed later with no prior chat memory, the current code-verified situation is:

1. `ConsultationHub.ReceiveMessage(...)` already saves each message to `ChatMessages`.
2. `ChatMessage` already contains `Id`, `ConsultationId`, `SenderId`, `Content`, `AttachmentUrl`, and `SentAt`.
3. `ConsultationHub.OnConnectedAsync(...)` already restricts realtime access to consultation participants only.
4. `ConsultationsController` now exposes `GET /api/consultations/{consultationId}/messages-history`.
5. `IConsultationService` now has a message-history read contract.
6. `Consultation.Status` already supports `Completed`.
7. `ConsultationService.GetConsultationMessageHistoryAsync(...)` exposes the post-call message-history read API.

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

Current verified access rule:

- realtime hub connection is allowed only when current user is either:
  - `Consultation.CallerId`
  - `Consultation.CalleeId`
- message-history HTTP read is allowed for:
  - consultation participants
  - admin

### Implemented Read Surface

What is now implemented:

- HTTP route to list messages for one consultation
- response DTO dedicated to consultation messages
- service-layer method that validates participant or admin access plus terminal status before returning messages
- integration tests covering post-completion message history retrieval

## Problem Statement

The earlier implementation stored consultation chat correctly, but the read path was incomplete.

That creates a business gap:

- the chat behaves like consultation-scoped messaging during the call
- the stored messages remain in the database
- the user still cannot review those messages once the active call experience is over

So the problem is not missing persistence.

That missing retrieval contract is now implemented.

## Implemented Direction

The current implemented direction is:

- active route: `GET /api/consultations/{consultationId}/messages-history`
- allow retrieval only when `Consultation.Status` is one of:
  - `Cancelled`
  - `Completed`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`
- allow retrieval for:
  - consultation participants
  - admin
- keep this endpoint read-only
- keep message sending inside `ConsultationHub` for now
- paginate results so the endpoint remains safe when long consultations produce many messages
- return each page in ascending order by `SentAt`, then `Id`
- use newest-window-first paging semantics:
  - `pageNumber = 1` returns the newest history batch
  - `pageNumber = 2` returns the next older batch
  - each next page continues backward to older history
- keep the response aligned with stored truth and do not add sender enrichment in v1

Response shape stays close to persisted truth:

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
- `SnakeAid.Core/Requests/Consultation/ConsultationMessageHistoryQueryRequest.cs`
- `SnakeAid.Core/Responses/Consultation/*`
- integration tests for consultation APIs
- docs under `SnakeAid.Docs`

## Delivered Artifacts

- `consultation-message-history.introduction.md`
- `consultation-message-history.roadmap.md`
- `consultation-message-history.hallucination.md`
- `consultation-message-history.sourcecode.md`
- `consultation-message-history.useguide.md`
