---
doc_role: operation
operation_id: 04-FEAT-emergency-consultation
type: FEAT
status: draft
created_at: 2026-03-05
affects:
  - Api/Hubs/ExpertHub.cs
  - Api/Controllers/ConsultationsController.cs
  - Core/Domains/ExpertProfile.cs
  - Core/Domains/ConsultationPingRequest.cs
  - Core/Domains/Consultation.cs
---

# Plan: Emergency Consultation Flow

## 1. As-Is

Users can view the expert directory and experts can set availability, but the real-time "Emergency Ping" flow is not implemented. There is no `ExpertHub` to track presence, nor logic to broadcast a ping and handle the ensuing "Slot Paradox".

## 2. Gap Analysis

- Missing `ExpertHub` SignalR implementation to manage online/offline status in real-time.
- Missing an endpoint for a user to broadcast an emergency ping `POST /api/v1/consultations/emergency`.
- Missing expert endpoints to Accept/Reject a ping.
- Missing Domain Event logic to handle the "Slot Paradox", where accepting an unpredictable 30-minute block must lock out any overlapping regular `ExpertTimeSlot`s.

## 3. To-Be Design

Implement the following:

- `ExpertHub` at `/hubs/expert` handling `JoinAsExpert`, `OnDisconnectedAsync`, and broadcasting `EmergencyPing`.
- `POST /api/v1/consultations/emergency` to create a `ConsultationPingRequest` and trigger the broadcast via `IHubContext`.
- `POST /api/v1/consultations/emergency-requests/{requestId}/accept` to transition the request, create an `Ongoing` consultation, and fire a Domain Event.
- `POST /api/v1/consultations/emergency-requests/{requestId}/reject`.
- **Domain Logic**: Within the acceptance transaction, find any `ExpertTimeSlot` for that expert overlapping the next 30 minutes. If `Available`, update to `Reserved`.

## 4. Impacted Components

- **Hubs**: `ExpertHub`
- **Controllers**: `ConsultationsController`
- **Services**: `EmergencyConsultationService`
- **Entities**: New schema/entity required for `ConsultationPingRequest` if not fully modeled. Updates to `ExpertProfile` online status logic.

## 5. Risks & Constraints

- Socket lifecycle and Database syncing: `OnDisconnectedAsync` might fire unexpectedly. Real-time drops must sync correctly to `ExpertProfile.IsOnline`.
- Racing conditions on concurrent emergency pings to the same expert.

## 6. Validation Plan

- Unit test Domain Event firing to ensure overlapping `ExpertTimeSlot`s switch to `Reserved`.
- Integration test or manual Postman testing involving SignalR clients to verify robust `IsOnline` toggle behavior.
