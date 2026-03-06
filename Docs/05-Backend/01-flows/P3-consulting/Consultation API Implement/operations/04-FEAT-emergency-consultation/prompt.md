---
doc_role: operation
operation_id: 04-FEAT-emergency-consultation
generated_from: plan.md
status: draft
created_at: 2026-03-05
---

# Prompt: Implement Emergency Consultation Flow

## Requirements

Implement the API endpoints and Hub logic defined in the gap analysis of `plan.md`.

Specific tasks:

1. Implement `Api/Hubs/ExpertHub.cs`. Derive from standard SignalR `Hub`. Track connections in a ConcurrentDictionary for available experts. On connect/disconnect, update the `ExpertProfile.IsOnline` DB flag.
   - Extend hub role scope to allow both `Expert` and `User` connections.
   - Add `JoinAsMember` for users (member app clients) to subscribe expert presence.
   - On member join, send `OnlineExpertsSnapshot` from SignalR in-memory connected experts (SignalR-first snapshot).
   - On expert online/offline change, broadcast `ExpertPresenceChanged` to subscribed members.
   - Add hardcoded switch `EnablePresenceSelfHealing` (default `false`) to control optional DB reconcile from SignalR memory.
2. Implement `POST /api/v1/consultations/emergency` to create a `ConsultationPingRequest` with explicit selected `ExpertId` from user input, and send a directed notify via `IHubContext<ExpertHub>` to that expert only (no broadcast matching).
3. Implement `POST /api/v1/consultations/emergency-requests/{requestId}/accept`. Ensure only the targeted expert can accept, then create the `Consultation` record as `Ongoing`.
4. **Slot Paradox Requirement**: Within the accept transaction, query for overlapping `ExpertTimeSlot` rows. Change them to `Reserved` to block parallel bookings.
5. Implement `POST /api/v1/consultations/emergency-requests/{requestId}/reject`. Ensure only the targeted expert can reject.
6. Fold Operation-1 corrective changes into this operation:
   - Add `ExpertDirectoryQueryRequest` (inherits pagination) for `GET /api/v1/experts` with optional fields:
     - `specialization`
     - `isOnline`
     - `sortBy` (`isOnline`, `rating`, `consultationFee`)
     - `sortOrder` (`asc`, `desc`)
   - Update `ExpertController.GetExperts` and `IExpertService`/`ExpertService` signatures to use the new query request.
   - Implement filter/sort logic in `ExpertService.GetExpertsAsync` with deterministic secondary ordering for stable pagination.
   - Keep response contract `ApiResponse<PagingResponse<ExpertProfileResponse>>` unchanged.
7. Add/extend tests:
   - Integration tests for `GET /api/v1/experts` filtering/sorting behavior.
   - Unit tests for service-level query composition and deterministic ordering.

## Constraints

- Do not mix room chat logic into `ExpertHub`. This hub is strictly for global presence and routing emergency requests to the user-selected expert.
- Do not let users invoke expert-only actions. Validate role at hub method level (`JoinAsExpert` expert-only, `JoinAsMember` user-only).
- Handle state drift on SignalR disconnects gracefully.
- For environments sharing one DB, keep `EnablePresenceSelfHealing = false`.
- This operation supersedes prior draft corrective notes for Op1; implementation should be executed from Operation 4 scope only.
