---
doc_role: operation
operation_id: 05-FEAT-in-room-features
generated_from: plan.md
status: completed
created_at: 2026-03-05
implemented_at: 2026-03-18
---

# Prompt: Implement In-Room Features

## Requirements

Implement the `ConsultationHub` and auxiliary endpoints as defined in `plan.md`.

Specific tasks:

1. Implement `Api/Hubs/ConsultationHub.cs` derived from `Hub`. In `OnConnectedAsync`, extract `consultationId` from the query string. Look up the `Consultation` in the DB. If the current JWT user is not `CallerId` nor `CalleeId`, terminate the connection. Otherwise, assign the connection to a SignalR Group using the `consultationId`.
2. Implement robust `ReceiveMessage(string content, string? attachmentUrl)` logic in the Hub. Save to `ChatMessage` table with `Content` and `AttachmentUrl` fields, then push to the grouped clients. Clients pre-upload images via `POST /api/media/upload-image` (domain: "chat-media") and pass the returned `secureUrl` as `attachmentUrl`.
3. Implement `Signal(string eventType, string payload)` to bounce real-time UI states (like typing indicators or camera toggles). Do not persist these events in the DB.
4. Add `GET /api/v1/snakes/search` to `SnakesController`. Map standard text search against the `SnakeSpecies` table, including `SpeciesVenom` and `SpeciesAntivenom` details needed for quick reference by the expert.

## Constraints

- Ensure strict privacy isolation. Messages must never be broadcast outside the immediate `ConsultationId` group.
- Keep the `ConsultationHub` strictly out of global presence management (which belongs to `ExpertHub`).
- Media support: Images only (JPG/PNG/GIF), max 5MB file size. Implement client-side file header validation before upload.
- Moderation: Implement rate limiting at 10 messages per minute per user per consultation.
- Data persistence: Chat messages stored forever with no retention policy.
- Integration: Reuse existing JWT validation patterns, LiveKit integration from VideoCallController.
- Testing: Include unit tests for authorization, integration tests for multi-client chat, privacy tests for cross-consultation isolation, and mobile SignalR connection testing.
