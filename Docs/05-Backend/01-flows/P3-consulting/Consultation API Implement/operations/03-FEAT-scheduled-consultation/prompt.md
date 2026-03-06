---
doc_role: operation
operation_id: 03-FEAT-scheduled-consultation
generated_from: plan.md
status: done
created_at: 2026-03-05
---

# Prompt: Implement Scheduled Consultation Flow

## Requirements

Implement the API endpoints defined in the gap analysis of `plan.md`.

Specific tasks:

1. Update EF Core entity `ConsultationBooking` to add `ProblemDescription`. Create and apply an EF Core migration.
2. Implement `POST /api/v1/consultation-bookings`. Fetch the requested `ExpertTimeSlot`, verify it is `Available`, update its status to `Reserved`, and save. **Crucial**: Utilize the `Version` field to enforce Optimistic Concurrency. Return HTTP 409 if a conflict occurs.
3. Implement `GET /api/v1/consultation-bookings/my-bookings` returning filtered history for the current caller.
4. Ensure `POST /api/videocall/livekit-token/{consultationId}` securely returns a LiveKit token using the current user's identity and the established `RoomId`.
5. Implement `POST /api/v1/consultations/{consultationId}/end` to finalize the status and clear related constraints.
6. Implement `POST /api/v1/consultations/{consultationId}/reviews` inserting into `UserFeedback`.
7. Preserve scope boundaries in implementation notes:
   - Do not expand this operation into consultation payment checkout APIs.
   - Do not treat chat/in-room realtime feature set as covered by this operation.
   - Mobile should treat payment confirmation and consultation chat as deferred backend work.

## Constraints

- Ensure proper mapping via DTOs.
- Respect the LiveKit service contract already established in the codebase.
- Maintain tests for concurrency conflict handling.
