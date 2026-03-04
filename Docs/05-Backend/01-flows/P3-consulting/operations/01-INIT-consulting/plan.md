---
doc_role: operation
operation_id: 01-INIT-consulting
type: INIT
status: draft
created_at: 2026-03-05
affects:
  - Core/Domains/Consultation*
  - Core/Domains/Expert*
  - Api/Hubs/ExpertHub.cs
  - Api/Hubs/ConsultationHub.cs
---

# Plan: Consultation Flow Base Initialization

## 1. As-Is

The Consultation module currently lacks API endpoints. The domain entities exist (`Consultation`, `ConsultationBooking`, `ExpertProfile`, `ExpertTimeSlot`, etc.) but are not exposed or orchestrated to serve the functional wireframes. Video call functionality via LiveKit Cloud is already integrated.

## 2. Gap Analysis

- No REST endpoints map to the UI wireframes for both User and Expert apps.
- Missing field in DB for "Tài liệu tư vấn" (Problem description / User Notes) on the `ConsultationBooking` entity.
- Need endpoint to generate bulk `ExpertTimeSlot` for a week based on dynamic time-block configurations.
- The "Slot Paradox" requires domain logic to automatically block slots when an unpredictable emergency call starts.

## 3. To-Be Design

Implement the API endpoints documented in `analysis/04-endpoints-design.md`.
Set up two distinct SignalR Hubs:

- `ExpertHub`: For presence tracking and routing incoming emergency pings to available experts.
- `ConsultationHub`: For scoped in-room chat message delivery and UI signaling.
  Utilize the existing `VideoCallController` for generating WebRTC tokens for the `Consultation` room.

## 4. Impacted Components

- **Controllers**: `ConsultationsController`, `ConsultationBookingsController`, `ExpertsController`.
- **Services**: `ConsultationService`, `ExpertService`.
- **Hubs**: `ExpertHub`, `ConsultationHub`.
- **Entities**: Minor updates needed (e.g., adding `ProblemDescription` to `ConsultationBooking`).

## 5. Risks & Constraints

- **Concurrency**: Double booking of `ExpertTimeSlot` must be prevented using optimistic concurrency (`Version`).
- **Slot Paradox**: Emergency calls must logically lock out regular bookings that happen to overlap during the unpredicted 30-minute block.
- **State Drift**: Real-time presence drops must be handled gracefully. Hub disconnects must sync to `ExpertProfile.IsOnline` to ensure accurate UI status. Note decisions in `analysis/decision-log.md`.

## 6. Validation Plan

- Unit test booking logic ensuring 409 Conflict if `Version` mismatch occurs.
- Unit test "Slot Paradox" domain event to verify overlapping slots switch to `Reserved`.
- Swagger manual test for expert schedule operations, list retrival, and booking creation.
- SignalR connection tests to verify `IsOnline` toggle in DB correctly reflects socket lifecycle.
