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

### Options

1. Keep current behavior.
   - Rejected instant requests stay out of member/expert consultation history.
   - Existing response shape remains unchanged.

2. Add rejected instant requests to existing consultation history as request-level rows.
   - Existing endpoints return both real consultation sessions and rejected instant request rows.
   - Response contract must explicitly support rows with no `Consultation`.

3. Add a separate instant/emergency request history endpoint.
   - Existing consultation history remains session-only.
   - A new endpoint returns pending, accepted, rejected, expired, and cancelled instant requests.

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

#### Option 2B: Contract-Correct Request-Level Row

Preferred shape if Option 2 is selected:

- make `consultationId` nullable for history responses
- add `recordKind = "Consultation" | "EmergencyRequest"`
- add exact `requestStatus`, e.g. `DeclinedByExpert`
- use a unified `status` only for UI grouping if needed
- keep `roomId = null` for request-only rows

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

### Required User Decision

Confirm the Option 2B contract shape and filter behavior before implementation.
