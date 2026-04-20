# Consultation Message History Hallucination

## Purpose

This file does not lock design.

This file collects ambiguity, unresolved product decisions, and implementation choices that should not be silently invented while planning or coding.

Only move an item from here into the main docs after one option is explicitly chosen and its impact is understood.

Main docs affected by these decisions:

- `consultation-message-history.introduction.md`
- `consultation-message-history.roadmap.md`
- `consultation-message-history.sourcecode.md`
- `consultation-message-history.useguide.md`

## Current Direction Summary

Current direction is already fairly clear:

- keep message sending in `ConsultationHub`
- expose message history through HTTP
- scope the new endpoint to consultation participants
- make the endpoint read-only

What is not fully locked yet is the exact client contract and rule boundary.

## Risk 1. Completed-Only Rule

### Current ambiguity

The business request says:

- users should be able to get history of a completed consultation
- they should not be able to continue chatting

But it does not fully lock whether history retrieval must be blocked before completion.

Possible choices:

- allow only `Completed`
- allow `Completed`, `UserAbsent`, `ExpertAbsent`, `AllAbsent`, and maybe `Cancelled`
- allow participants to read persisted history at any time

### Why this matters

This changes:

- service validation logic
- frontend screen timing
- expected error behavior for ongoing consultations

### Recommended default

- v1 should allow only `Completed`

## Risk 2. Response Enrichment

### Current ambiguity

Minimal data already exists in `ChatMessage`:

- `id`
- `consultationId`
- `senderId`
- `content`
- `attachmentUrl`
- `sentAt`

But mobile may want convenience fields such as:

- `senderDisplayName`
- `senderRole`
- avatar URL

### Why this matters

Adding enrichment increases:

- query joins
- mapper complexity
- response-lock surface

### Recommended default

- keep v1 minimal with `senderId`
- only add sender convenience fields if mobile proves they are required

## Risk 3. Sort Direction

### Current ambiguity

History screens often want:

- ascending order for chat replay

But list APIs often want:

- descending order for newest-first browsing

### Why this matters

This impacts:

- database order
- paging behavior
- mobile scroll experience
- documentation examples

### Recommended default

- return ascending by `SentAt`, then `Id` as a tiebreaker

## Risk 4. Pagination Shape

### Current ambiguity

Possible response styles:

- return a simple array
- return `PagingResponse<T>`
- return cursor-based pagination

### Why this matters

History can grow long enough that returning all messages may become unsafe.

### Recommended default

- reuse existing `PagingResponse<T>` style with `pageNumber` and `pageSize`

## Risk 5. Attachment-Only Messages

### Current ambiguity

`ConsultationHub.ReceiveMessage(...)` currently allows:

- content only
- attachment only
- content plus attachment

But the mobile history contract must clearly define whether `content` can be empty or null in the returned payload.

### Why this matters

Frontend rendering can break if it assumes message text always exists.

### Recommended default

- preserve stored truth exactly
- document that `content` may be empty when `attachmentUrl` is present

## Risk 6. Failure Code For Non-Completed Consultations

### Current ambiguity

If caller is a valid participant but consultation is not completed yet, the backend could return:

- `400`
- `403`
- `404`

### Why this matters

This affects frontend error handling and business semantics.

### Recommended default

- use a business validation failure that maps to `400`

## Promotion Rule

Only promote an item from this file into the main planning docs when:

- one option is explicitly chosen
- the chosen option is reflected consistently in API contract and roadmap
- the failure behavior and edge cases are clear enough to implement and test
