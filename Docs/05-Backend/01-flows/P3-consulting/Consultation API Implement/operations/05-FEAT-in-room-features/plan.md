---
doc_role: operation
operation_id: 05-FEAT-in-room-features
type: FEAT
status: completed
created_at: 2026-03-05
implemented_at: 2026-03-18
affects:
  - Api/Hubs/ConsultationHub.cs
  - Api/Controllers/SnakesController.cs
  - Core/Domains/ChatMessage.cs
---

# Plan: In-Room Consultation Features

## 1. As-Is

Users can book consultations (Operation 03) and start emergency rooms (Operation 04). However, inside the active consultation session room, there is no real-time text chat, no UI signaling (e.g., typing, mic toggled), and no auxiliary tools for the expert.

## 2. Gap Analysis

- Missing `ConsultationHub` for scoped socket connections tied strictly to a `ConsultationId`.
- Missing database persistence for `ChatMessage` exchanged inside the room.
- Missing the expert's auxiliary "Tra cứu rắn" lookup endpoint for quick species matching during the call.

## 3. To-Be Design

Implement the following:

- `ConsultationHub` at `/hubs/consultation`. Connections must provide a valid `?consultationId={id}` and pass authorization checks.
- **Image Message Flow**: Clients pre-upload images via `POST /api/media/upload-image` (domain: "chat-media"), then send chat messages with `attachmentUrl` containing the Cloudinary secure URL.
- SignalR `ReceiveMessage` method to save chat messages to the DB and broadcast to the consultation group, including support for `attachmentUrl` field.
- SignalR `Signal` method to forward volatile UI states (mic/cam UI layout hints) without saving to the DB.
- `GET /api/v1/snakes/search`: A lightweight endpoint returning snake species and antivenom data for the expert side-panel.

## 4. Impacted Components

- **Hubs**: `ConsultationHub`
- **Controllers**: `SnakesController`
- **Entities**: Add/Verify `ChatMessage` entity links to `Consultation`.

## 5. Risks & Constraints

- Privacy: The `ConsultationHub` must strictly validate that the connecting user is either the `Caller` or `Callee` of the specified `Consultation`.
- DB Load: Every chat message requires an Insert. Use standard async Repository patterns.
- Media Constraints: Image uploads limited to 5MB max, JPG/PNG/GIF formats only. No file attachments beyond images. Client-side file header validation required.
- Moderation: Rate limiting at 10 messages per minute per user per consultation.
- Data Persistence: Chat history stored forever (no retention policy).
- Integration: Maintain separation from `ExpertHub` and `MissionHub`. Reuse existing JWT validation and LiveKit integration patterns.

## 6. Validation Plan

- Multi-client simulation testing for `ConsultationHub` ensuring messages do not leak across different `ConsultationId` namespaces.
- Unit testing authorization logic in the Hub connect phase.
- Integration testing with existing VideoCallController and LiveKit setup.
- Load testing with 100+ concurrent consultations.
- Privacy testing to prevent cross-consultation message leakage.
- Mobile SignalR connection testing (mobile app ready for testing).
