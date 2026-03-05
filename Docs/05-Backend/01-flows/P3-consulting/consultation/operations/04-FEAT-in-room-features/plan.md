---
doc_role: operation
operation_id: 04-FEAT-in-room-features
type: FEAT
status: draft
created_at: 2026-03-05
affects:
  - Api/Hubs/ConsultationHub.cs
  - Api/Controllers/SnakesController.cs
  - Core/Domains/ChatMessage.cs
---

# Plan: In-Room Consultation Features

## 1. As-Is

Users can book consultations (Operation 02) and start emergency rooms (Operation 03). However, inside the active consultation session room, there is no real-time text chat, no UI signaling (e.g., typing, mic toggled), and no auxiliary tools for the expert.

## 2. Gap Analysis

- Missing `ConsultationHub` for scoped socket connections tied strictly to a `ConsultationId`.
- Missing database persistence for `ChatMessage` exchanged inside the room.
- Missing the expert's auxiliary "Tra cứu rắn" lookup endpoint for quick species matching during the call.

## 3. To-Be Design

Implement the following:

- `ConsultationHub` at `/hubs/consultation`. Connections must provide a valid `?consultationId={id}` and pass authorization checks.
- SignalR `ReceiveMessage` method to save chat messages to the DB and broadcast to the consultation group.
- SignalR `Signal` method to forward volatile UI states (mic/cam UI layout hints) without saving to the DB.
- `GET /api/v1/snakes/search`: A lightweight endpoint returning snake species and antivenom data for the expert side-panel.

## 4. Impacted Components

- **Hubs**: `ConsultationHub`
- **Controllers**: `SnakesController`
- **Entities**: Add/Verify `ChatMessage` entity links to `Consultation`.

## 5. Risks & Constraints

- Privacy: The `ConsultationHub` must strictly validate that the connecting user is either the `Caller` or `Callee` of the specified `Consultation`.
- DB Load: Every chat message requires an Insert. Use standard async Repository patterns.

## 6. Validation Plan

- Multi-client simulation testing for `ConsultationHub` ensuring messages do not leak across different `ConsultationId` namespaces.
- Unit testing authorization logic in the Hub connect phase.
