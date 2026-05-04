---
doc_role: implementation
module: scheduled-booking-cancel
kind: flow
doc_type: hallucination
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-investigated
---

# Scheduled Booking Cancel Hallucination Risks

## H-001: Should Expert-Rejected Instant Requests Appear In Consultation History?

- status: `Open`
- discovered: `2026-05-04`

### Why This Requires A Decision

Code currently treats scheduled cancellation and instant/emergency expert rejection differently.

- scheduled cancel has a `ConsultationBooking` and linked `Consultation` before cancellation
- scheduled cancel sets linked `Consultation.Status = Cancelled`
- instant/emergency expert reject happens while the `ConsultationPingRequest` is still pending expert response
- instant/emergency expert reject sets `ConsultationPingRequest.Status = DeclinedByExpert`
- no `Consultation` row is created for the rejected instant/emergency request
- member/expert consultation history currently includes emergency items only when the ping request is `AcceptedByExpert` and linked to a consultation

The code supports multiple valid product interpretations, so backend should not guess the contract.

### Impact If Guessed Wrong

- If rejected instant requests are added to consultation history, mobile may show request-level events as consultations even though no room/session existed.
- If rejected instant requests stay excluded, users may perceive expert cancellation/rejection as missing history.
- If backend creates cancelled `Consultation` rows for rejected instant requests, reporting, admin history, payment, and room semantics may need follow-up changes.

### Options

1. Keep current behavior.
   - `GET /api/users/me/consultations` and `GET /api/experts/me/consultations` remain consultation-session history only.
   - Expert-rejected instant requests are not returned because no consultation session existed.

2. Add rejected instant requests to consultation history as request-level rows.
   - History endpoints would need to include `ConsultationPingStatus.DeclinedByExpert`.
   - Response model must make clear that `consultationId` and `roomId` may be null for rejected instant requests.

3. Expose a separate instant/emergency request history endpoint.
   - Consultation history remains session-based.
   - Mobile gets a dedicated list for pending, rejected, expired, accepted, and cancelled instant requests.

### Recommendation From Current Code Evidence

Option 3 is the cleanest fit with current code boundaries because expert rejection is stored on `ConsultationPingRequest`, not `Consultation`.

### Required User Decision

Choose whether rejected/cancelled instant emergency requests are part of consultation history, request history, or should remain hidden from history.
