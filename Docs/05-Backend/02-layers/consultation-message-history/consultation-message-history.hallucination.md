# Consultation Message History Hallucination

## Status

Current status:

- closed for the current planning baseline

## Purpose

This file now serves two purposes:

- preserve ambiguity that should not be silently invented
- record which former risks have already been decided and must be reflected in the baseline docs

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
- allow participant and admin access to the endpoint
- make the endpoint read-only

Most of the client contract and rule boundary are now locked.

Already decided:

- allow terminal consultation states:
  - `Cancelled`
  - `Completed`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`
- skip sender enrichment in v1
- allow admin access even when admin is not a consultation participant
- return ascending by `SentAt`, then `Id` as a deterministic tiebreaker
- preserve stored truth exactly for attachment-only messages
- use business validation failure mapped to `400` for non-eligible consultation states

There is no remaining open risk for the current planning baseline.

## Risk 1. Terminal-State Rule

### Decision

- allow:
  - `Cancelled`
  - `Completed`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`

### Impact

- the endpoint should be described as available for `terminal consultation states`
- `Cancelled` is part of the current allowed set

## Risk 2. Response Enrichment

### Decision

- skip sender enrichment in v1

### Reason

- the UI inside the video call already works correctly on the app
- this feature only exposes the chat UI outside the video call in read-only mode
- v1 should keep the response as close as possible to the stored message truth

### Locked v1 message shape

- `id`
- `consultationId`
- `senderId`
- `content`
- `attachmentUrl`
- `sentAt`

## Risk 3. Sort Direction

### Decision

- return ascending by `SentAt`, then `Id` as a tiebreaker

### Impact

- every returned page can be rendered directly from top to bottom
- duplicate timestamps still produce deterministic ordering

## Risk 4. Pagination Shape

### Decision

- reuse existing `PagingResponse<T>` style with `pageNumber` and `pageSize`
- accepted as the locked baseline direction

### Deep analysis

The mobile user journey is not the same as a normal list page.

Expected mobile UX:

1. user opens the readonly chat history screen
2. app should show the newest batch first
3. inside that batch, messages should still render from old to new
4. when user scrolls upward past the oldest loaded message, app requests the next older batch

This creates two simultaneous requirements:

- page selection must be newest-first
- item order inside each page must remain ascending

If the backend uses classic oldest-first pagination:

- page `1` = oldest items
- page `N` = newest items

then mobile would need extra client logic:

- fetch metadata first to compute the last page
- or fetch descending and reverse locally
- or keep a custom translation layer between UI paging and server paging

That is avoidable.

### Recommended paging contract

Keep `PagingResponse<T>`, but define page semantics as:

- `pageNumber = 1` returns the newest history batch
- `pageNumber = 2` returns the next older batch
- `pageNumber = 3` returns the next older batch after that
- items inside each page are still sorted ascending by `SentAt`, then `Id`

### Concrete example

Assume:

- total messages = `120`
- page size = `50`

Recommended server behavior:

- page `1` -> messages `71..120`, returned ascending
- page `2` -> messages `21..70`, returned ascending
- page `3` -> messages `1..20`, returned ascending

This allows mobile to:

- open with `pageNumber=1`
- prepend older messages by incrementing `pageNumber`
- avoid local reverse sorting

### Why this is stable enough

This contract works well because the endpoint is only for terminal consultation states:

- `Cancelled`
- `Completed`
- `UserAbsent`
- `ExpertAbsent`
- `AllAbsent`

So the underlying history is frozen while paging and page windows do not shift.

### Required documentation note

Main docs must explicitly say both:

- `pageNumber` counts from the newest batch backward
- items inside each page are still ascending

Without both statements, implementers will assume classic oldest-first paging and break the intended UX.

### Final status

- closed

## Risk 5. Attachment-Only Messages

### Decision

- preserve stored truth exactly
- document that `content` may be empty when `attachmentUrl` is present

## Risk 6. Failure Code For Non-Eligible Consultation States

### Decision

- use a business validation failure that maps to `400`

### Clarified trigger

This applies when:

- consultation exists
- caller is a valid participant
- but consultation status is not one of:
  - `Cancelled`
  - `Completed`
  - `UserAbsent`
  - `ExpertAbsent`
  - `AllAbsent`

## Promotion Rule

Only promote an item from this file into the main planning docs when:

- one option is explicitly chosen
- the chosen option is reflected consistently in API contract and roadmap
- the failure behavior and edge cases are clear enough to implement and test

## Closure Summary

All six risks for the current planning scope are now decided and reflected in:

- `consultation-message-history.introduction.md`
- `consultation-message-history.roadmap.md`
- `consultation-message-history.sourcecode.md`
- `consultation-message-history.useguide.md`

This file should stay closed unless a new business ambiguity appears during implementation.
