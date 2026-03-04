---
doc_role: analysis
operation_id: 01-INIT-consulting
type: INIT
status: draft
created_at: 2026-03-05
---

# Decision Log: Multi-Agent Brainstorming for Consultation Flow

## Process Overview

This log captures the simulated structured review (Primary Designer, Skeptic, Constraint Guardian, User Advocate, Integrator/Arbiter) for the Consultation API Endpoints based on the UI Wireframes. It includes the second round of review based on deeper codebase context.

## 1. Primary Designer Proposal

- **Proposal**: Create RESTful endpoints grouped by `/api/v1/consultations`, `/api/v1/experts`, and `/api/v1/consultation-bookings`. Manage presence and chat via SignalR Hubs. Delegate video token generation to existing LiveKit integration.

## 2. Reviewer Objections & Feedback

### Skeptic / Challenger

- **Objection 1 (Slot Paradox)**: "The wireframe mentions the 'Slot Paradox'. How does the DB reflect this?"
  - _Mitigation_: Must use Domain Events to block overlapping slots automatically.
- **Objection 2 (Missing Entity Fields)**: "The 'Tài liệu tư vấn' screen requires users to enter problem descriptions. `ConsultationBooking` entity lacks `ProblemDescription`."
  - _Mitigation_: Require schema update to include this field.
- **Objection 6 (Working Hours vs Time Slots)**: "Is the working hours configuration a fixed weekly schedule or generated week-by-week?"
  - _Mitigation_: The UX requires experts to set up their availability week-by-week with flexible, non-fixed time blocks (e.g., variable number of sessions per day). `POST /api/v1/experts/me/time-slots/bulk` is the correct approach to generate these discrete slots.

### Constraint Guardian

- **Objection 3 (Concurrency on Booking)**: "When two users try to book the same `ExpertTimeSlot` simultaneously, we risk double-booking."
  - _Mitigation_: Leverage EF Core optimistic concurrency using `Version`.
- **Objection 4 (Online Status Strategy)**: "Polling DB for online status is too slow and out of sync with established patterns."
  - _Mitigation_: Implement an `ExpertHub` maintaining connected users in a ConcurrentDictionary, updating the DB `IsOnline` on connect/disconnect (similar to `RescuerHub`).
- **Objection 8 (Hub Separation Violation)**: "Combining chat, emergency pings, and presence into one `ConsultationHub` violates Single Responsibility Principle. `RescuerHub` and `MissionHub` are cleanly separated."
  - _Mitigation_: Split into `ExpertHub` (for presence and emergency pings) and `ConsultationHub` (for active consultation session chat/signaling).

### User Advocate

- **Objection 5 (Media Uploads)**: "Users need to send images during consultation."
  - _Mitigation_: Include media endpoints for chat.
- **Objection 9 (Video Provider)**: "We already have `LiveKit Cloud` integrated. Don't invent new endpoints for it."
  - _Mitigation_: Use the existing `VideoCallController` (`POST /api/videocall/livekit-token/{consultationId}`) instead of defining a new mock provider.

## 3. Integrator / Arbiter Resolutions

### Decision 1: Handling the Slot Paradox (Accepted)

When an `Emergency` Consultation transitions to `Ongoing`, trigger a Domain Event to mark overlapping `ExpertTimeSlot` as `Reserved` or `Cancelled`.

### Decision 2: Working Hours Schedule (Accepted)

- Keep `POST /api/v1/experts/me/time-slots/bulk`. The UI flow dictates that experts will configure their available time blocks week-by-week, and those blocks can be disjointed (e.g., 4 blocks in a day). The API will receive this bulk configuration and directly generate `ExpertTimeSlot` rows for the specifically configured week. We do not need a separate `WorkingHours` schema for infinite recurrence.

### Decision 3: Expert Presence & SignalR Hub Split (Accepted)

- **ExpertHub**: Handles `/hubs/expert`. Tracks `ConnectedExperts`, updates `ExpertProfile.IsOnline` on connect/disconnect, and broadcasts `EmergencyPing` requests to available experts.
- **ConsultationHub**: Handles `/hubs/consultation`. Scoped to a specific `ConsultationId` (similar to `MissionHub`). Handles `ChatMessage` delivery and in-room signaling (Mic/Cam toggles states).

### Decision 4: Video Call Integration (Accepted)

- Drop custom video endpoints from the design. Document that clients must call the existing `POST /api/videocall/livekit-token/{consultationId}` route to join the LiveKit room.

### Decision 5: Concurrency Control (Accepted)

- Enforce EF Core optimistic concurrency on `ExpertTimeSlot.Version`.

---

**Final Status:** APPROVED with required modifications to the API design document.
