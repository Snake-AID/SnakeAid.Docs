---
doc_role: operation
operation_id: 04-FEAT-emergency-consultation
type: FEAT
status: draft
created_at: 2026-03-05
affects:
  - Api/Hubs/ExpertHub.cs
  - Api/Controllers/ConsultationsController.cs
  - Api/Controllers/ExpertController.cs
  - Service/Interfaces/IExpertService.cs
  - Service/Implements/ExpertService.cs
  - Core/Requests/Expert/ExpertDirectoryQueryRequest.cs
  - Tests/Integration/ExpertControllerIntegrationTests.cs
  - Tests/Unit/ExpertServiceTests.cs
  - Core/Domains/ExpertProfile.cs
  - Core/Domains/ConsultationPingRequest.cs
  - Core/Domains/Consultation.cs
---

# Plan: Emergency Consultation Flow

## 1. As-Is

Users can view the expert directory and experts can set availability, but the real-time "Emergency Consultation" flow is not implemented. There is no `ExpertHub` to track presence, nor logic for user-selected expert immediate consultation with "Slot Paradox" handling.

Codebase verification (2026-03-06):
- `GET /api/v1/experts` currently accepts only `PaginationRequest` and uses fixed ordering (`Rating desc`, `RatingCount desc`) in `ExpertService.GetExpertsAsync`.
- Directory contract does not yet support explicit UX filters/sorts (`specialization`, `isOnline`, `sortBy`, `sortOrder`).
- `ExpertProfile.IsOnline` is exposed in list/profile responses, but there is no Expert SignalR presence lifecycle yet to keep it near-realtime.

## 2. Gap Analysis

- Missing `ExpertHub` SignalR implementation to manage online/offline status in real-time.
- Missing member-facing realtime consumption path for expert presence updates (users currently depend on REST polling).
- Missing an endpoint for a user to request immediate consultation with a selected expert `POST /api/v1/consultations/emergency`.
- Missing expert endpoints to Accept/Reject a request sent to that selected expert.
- Missing Domain Event logic to handle the "Slot Paradox", where accepting an unpredictable 30-minute block must lock out any overlapping regular `ExpertTimeSlot`s.
- Missing Op1 contract alignment for expert directory selection UX:
  - Add filter/sort contract for `GET /api/v1/experts` (`specialization`, `isOnline`, `sortBy`, `sortOrder` + pagination).
  - Make `isOnline` semantics explicitly presence-backed by ExpertHub lifecycle.

## 3. To-Be Design

Implement the following:

- `ExpertHub` at `/hubs/expert` handling:
  - `JoinAsExpert`, `OnDisconnectedAsync`, and expert presence updates (`IsOnline`).
  - `JoinAsMember` (role `User`) so member clients can subscribe to consultation presence updates.
  - `JoinEmergencyRequestRoom(requestId)` (role `User`) to subscribe status updates of the request they created.
  - Presence events for member clients:
    - initial snapshot event `OnlineExpertsSnapshot`
    - delta event `ExpertPresenceChanged` when expert online status changes.
  - Emergency status event for request room:
    - `EmergencyRequestStatusChanged` (`Pending`/`Accepted`/`Rejected`/`Expired` with metadata)
  - Add hardcoded safety switch `EnablePresenceSelfHealing` (default `false`) to control whether Hub reconciles DB `ExpertProfile.IsOnline` from in-memory SignalR presence.
- `POST /api/v1/consultations/emergency` to create a `ConsultationPingRequest` with explicit `ExpertId` chosen by user and push a directed realtime notification to that expert via `IHubContext`.
- `POST /api/v1/consultations/emergency-requests/{requestId}/accept` to transition the request, create an `Ongoing` consultation, and fire a Domain Event.
- `POST /api/v1/consultations/emergency-requests/{requestId}/reject`.
- Push `EmergencyRequestStatusChanged` to the request room when state changes.
- **Domain Logic**: Within the acceptance transaction, find any `ExpertTimeSlot` for that expert overlapping the next 30 minutes. If `Available`, update to `Reserved`.
- **Operation-1 corrective scope folded into Operation 4**:
  - Extend `GET /api/v1/experts` query contract with optional fields `specialization`, `isOnline`, `sortBy`, `sortOrder`.
  - Replace controller/service input from bare `PaginationRequest` to a directory query request inheriting pagination.
  - Implement deterministic ordering with secondary keys to keep stable pagination.
  - Use presence-backed `ExpertProfile.IsOnline` as directory availability signal.

## 4. Impacted Components

- **Hubs**: `ExpertHub`
- **Controllers**: `ConsultationsController`
- **Services**: `EmergencyConsultationService`
- **Entities**: New schema/entity required for `ConsultationPingRequest` if not fully modeled. Updates to `ExpertProfile` online status logic.
- **Directory components (Op1 corrective)**: `ExpertController`, `IExpertService`, `ExpertService`, `Core/Requests/Expert/*DirectoryQuery*`, `ExpertControllerIntegrationTests`, `ExpertServiceTests`.

## 5. Risks & Constraints

- Socket lifecycle and Database syncing: `OnDisconnectedAsync` might fire unexpectedly. Real-time drops must sync correctly to `ExpertProfile.IsOnline`.
- Hub authorization must avoid privilege leakage: only experts can call `JoinAsExpert`; users can only subscribe via `JoinAsMember`.
- Request room join must enforce ownership: only requester of that `requestId` can join `JoinEmergencyRequestRoom`.
- Shared DB across environments can cause accidental cross-environment healing; `EnablePresenceSelfHealing` must stay `false` unless explicitly approved.
- Racing conditions on concurrent emergency requests targeting the same expert.
- Accept/Reject authorization must ensure only the targeted expert can respond to the request.
- Directory sort/filter expansion must preserve backward compatibility for existing clients using pagination-only query.

## 6. Validation Plan

- Unit test Domain Event firing to ensure overlapping `ExpertTimeSlot`s switch to `Reserved`.
- Integration test or manual Postman testing involving SignalR clients to verify robust `IsOnline` toggle behavior and directed notify to selected expert only.
- Manual SignalR validation:
  - User joins `JoinAsMember` and receives `OnlineExpertsSnapshot`.
  - When an expert joins/disconnects, user receives `ExpertPresenceChanged` with matching `expertId` and `isOnline`.
  - User joins `JoinEmergencyRequestRoom(requestId)` and receives `EmergencyRequestStatusChanged` after expert accept/reject.
- Add/extend directory tests verifying:
  - `GET /api/v1/experts` filters by `isOnline` and `specialization`.
  - Sorting by `isOnline`, `rating`, `consultationFee` with deterministic tie-breakers.
  - Pagination remains stable under the new sort/filter contract.
