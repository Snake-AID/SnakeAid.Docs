---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: hallucination
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-investigated
---

# Consultation Instant Booking Cancel Hallucination Risks

## H-001: Should Expert-Rejected Instant Requests Appear In Consultation History?

- status: `Open`
- discovered: `2026-05-04`

### Why This Requires A Decision

Expert-rejected instant/emergency requests currently exist as `ConsultationPingRequest` records, not `Consultation` records.

Current code:

- creates `Consultation` only when the expert accepts the instant request
- sets rejected instant requests to `ConsultationPingStatus.DeclinedByExpert`
- leaves rejected instant requests with `ConsultationId = null`
- returns instant/emergency history only for accepted requests that have a linked `Consultation`

The product decision is whether request-level rejected instant rows should appear inside the existing consultation history endpoints.

### Impact If Guessed Wrong

- Mobile may need to render history rows with no consultation room.
- Clients may currently assume `consultationId` is always present.
- Mapping request status into consultation status can create misleading UI if not explicit.
- Keeping rejected requests hidden may make users think the request disappeared.

### Decision Options

#### Option 1: Keep Current Session-Only History

Behavior:

- rejected instant requests remain invisible in member/expert consultation history
- accepted instant requests continue to appear because they have a linked `Consultation`
- current DTOs remain unchanged:
  - `MyConsultationResponse.ConsultationId: Guid`
  - `ExpertConsultationResponse.ConsultationId: Guid`
  - `AdminConsultationResponse.ConsultationId: Guid`

Pros:

- lowest implementation risk
- no mobile breaking change
- preserves the existing meaning of consultation history as actual sessions only
- no ambiguity around chat room, review, message history, or payment settlement actions

Cons:

- rejected instant requests disappear from the user's visible history
- expert rejection accountability is not visible in the same place as accepted emergency sessions
- mobile may need another source to explain why an instant request ended

Best fit when:

- product defines consultation history as completed/created consultation sessions only
- rejected requests are treated as notification/activity events, not history records

#### Option 2: Add Rejected Requests To Existing Consultation History

Behavior:

- member/expert history endpoints return a mixed list:
  - real `Consultation` session rows
  - request-only `ConsultationPingRequest` rows for `DeclinedByExpert`
- request-only rows must be explicitly distinguishable from consultation rows

Pros:

- rejected instant requests appear where users already look for consultation outcomes
- no extra mobile screen or endpoint is required for the narrow rejected-request case
- accepted and rejected emergency outcomes can sort together in one timeline

Cons:

- current response contract is not compatible with request-only rows because `consultationId` is non-null
- mobile must handle rows with no room and no created consultation
- status filtering becomes ambiguous unless `ConsultationStatus` and `ConsultationPingStatus` are separated
- follow-up actions tied to real consultations must be guarded by `recordKind`

Best fit when:

- product wants one unified history timeline
- mobile is ready to handle a discriminator and nullable consultation fields
- the scope is limited to terminal request outcomes such as `DeclinedByExpert`, not all live pending requests

#### Option 3: Add A Separate Instant/Emergency Request History Endpoint

Behavior:

- existing consultation history remains session-only
- a new request-history endpoint returns request lifecycle rows such as pending, accepted, declined, expired, and cancelled
- request history uses `ConsultationPingRequest.Id` as the primary id and includes optional `ConsultationId`

Pros:

- cleanest domain contract: consultation history stays about sessions; request history stays about ping/request lifecycle
- avoids making `consultationId` nullable in existing history DTOs
- can support pending/expired/cancelled request states without overloading consultation status
- lower risk of breaking existing mobile assumptions

Cons:

- requires a new endpoint, mobile integration path, and API documentation
- accepted emergency sessions may appear in both request history and consultation history unless the contract documents the relationship clearly
- does not satisfy a strict requirement to show rejected requests inside the existing history list

Best fit when:

- product needs a full instant request audit trail, not only expert-rejected rows
- mobile can add a separate request history screen/section
- preserving existing consultation history semantics is more important than a single timeline

### Option Comparison

| Criterion | Option 1: Keep Current | Option 2: Mixed History | Option 3: Separate Request History |
| --- | --- | --- | --- |
| Shows expert-rejected instant requests | No | Yes, in existing history | Yes, in new request history |
| Existing endpoint contract impact | None | High | None or low |
| Requires nullable `consultationId` in existing DTOs | No | Yes, if done correctly | No |
| Mobile rendering complexity | Low | Medium to high | Medium |
| Domain clarity | High for session history | Medium | High |
| Future support for pending/expired requests | Poor | Risky in history list | Strong |
| Fastest implementation | Option 1 | Option 2A only, but unsafe | Moderate |
| Best long-term extensibility | Weak | Medium | Strong |

### Evidence From Current Code

- `EmergencyConsultationService.AcceptEmergencyRequestAsync(...)` creates a `Consultation` and assigns `ConsultationPingRequest.ConsultationId`.
- `EmergencyConsultationService.RejectEmergencyRequestAsync(...)` sets `Status = ConsultationPingStatus.DeclinedByExpert`, sets `RespondedAt`, and does not create a `Consultation`.
- `GetMyConsultationsAsync(...)` only includes emergency requests where:
  - `p.RescuerId == userId`
  - `p.ConsultationId.HasValue`
  - `p.Status == ConsultationPingStatus.AcceptedByExpert`
- `GetExpertConsultationsAsync(...)` only includes emergency requests where:
  - `p.ExpertId == expertId`
  - `p.ConsultationId.HasValue`
  - `p.Status == ConsultationPingStatus.AcceptedByExpert`
- current member/expert DTOs require `ConsultationId` as non-null `Guid`.
- admin history has richer accepted emergency metadata (`EmergencyRequestId`, `EmergencyRequestStatus`, request timestamps), but still uses non-null `ConsultationId`.

### Option 2 Analysis

Option 2 is the user-preferred direction under consideration.

#### Option 2A: Minimal Contract Change

Implementation shape:

- include `ConsultationPingStatus.DeclinedByExpert` in emergency history queries
- map rejected rows directly from `ConsultationPingRequest`
- set `EmergencyRequestId = ping.Id`
- set `Type = "Emergency"`
- set `RoomId = null`
- synthesize `StartTime = ping.RespondedAt ?? ping.RequestedAt`
- set price from request payment transaction when available

Problem:

- current `consultationId` is a non-null `Guid`
- rejected instant rows have no real consultation id
- using `Guid.Empty` would be ambiguous and should be avoided

Additional risk:

- the history list would contain rows that look like consultations but cannot support consultation actions
- `status=Cancelled` would be synthesized from request status, not loaded from `Consultation.Status`
- existing tests that assert history invariants around non-null `consultationId` would need updates or new exceptions
- clients may accidentally call consultation detail, message history, room join, or review APIs with a fake id

#### Option 2B: Contract-Correct Request-Level Row

Preferred shape if Option 2 is selected:

- make `consultationId` nullable for history responses
- add `recordKind = "Consultation" | "EmergencyRequest"`
- add exact `requestStatus`, e.g. `DeclinedByExpert`
- use a unified `status` only for UI grouping if needed
- keep `roomId = null` for request-only rows
- keep `startTime` and `endTime` derived from request timestamps only for timeline display
- document that request-only rows do not support room join, message history, review, or consultation detail APIs

Recommended request-only row:

```json
{
  "recordKind": "EmergencyRequest",
  "consultationId": null,
  "emergencyRequestId": "11111111-1111-1111-1111-111111111111",
  "type": "Emergency",
  "status": "Cancelled",
  "requestStatus": "DeclinedByExpert",
  "roomId": null,
  "startTime": "2026-05-04T03:10:00Z",
  "endTime": "2026-05-04T03:10:00Z"
}
```

Implementation notes:

- member response model and expert response model both need the same discriminator/nullability change.
- admin response can reuse existing emergency request metadata fields, but it still needs either nullable `ConsultationId` or a separate request-history response if rejected requests are added there later.
- sorting should use `RespondedAt ?? RequestedAt` for request-only rows so declined requests appear at the time the expert acted when available.
- pagination should happen after merging consultation rows and request-only rows, matching the current in-memory merge pattern.

### Status Mapping Decision Inside Option 2

Current code has two status enums:

- `ConsultationStatus`
- `ConsultationPingStatus`

Sub-decision required:

1. return raw ping status in `status`
2. map rejected request rows to a unified `status = "Cancelled"`
3. return both `status = "Cancelled"` and `requestStatus = "DeclinedByExpert"`

Recommended shape:

- return both unified display status and exact request status
- document that `requestStatus` is only meaningful for `recordKind = "EmergencyRequest"`

### Filter Behavior Decision Inside Option 2

Current history filters parse `query.Status` as `ConsultationStatus`. That means `DeclinedByExpert` is not currently a valid history status filter.

Sub-decision required:

1. `status=Cancelled` includes request-only rows with `requestStatus=DeclinedByExpert`.
   - Pros: mobile can reuse the existing status filter.
   - Cons: backend is mapping a ping status into a consultation status bucket.

2. add `requestStatus=DeclinedByExpert` as a separate query parameter.
   - Pros: precise and domain-correct.
   - Cons: API surface grows and mobile must understand two filters.

3. include request-only rows only when no `status` filter is provided.
   - Pros: avoids ambiguous status mapping.
   - Cons: users filtering to cancelled history would not see expert-declined instant requests.

Recommended shape if Option 2 is selected:

- support `status=Cancelled` as the unified mobile grouping
- add `requestStatus` to the response for exact meaning
- defer a `requestStatus` query parameter unless product needs direct filtering by ping lifecycle state

### Recommendation

Code evidence does not justify silently choosing between Option 2 and Option 3 because both are valid product contracts.

Recommended decision path:

1. choose Option 2B only if the product requirement is explicitly "show expert-rejected instant requests in the existing member/expert consultation history list."
2. choose Option 3 if the product requirement is "show full instant request lifecycle history" or if mobile cannot safely absorb a nullable `consultationId` change.
3. do not implement Option 2A because it either requires fake consultation ids or leaves the existing DTO contract misleading.

### Required User Decision

Confirm one of:

- `Option 1`: keep current session-only history
- `Option 2B`: mixed consultation/request history with nullable `consultationId`, `recordKind`, unified `status`, exact `requestStatus`, and `status=Cancelled` including `DeclinedByExpert`
- `Option 3`: separate instant/emergency request history endpoint
