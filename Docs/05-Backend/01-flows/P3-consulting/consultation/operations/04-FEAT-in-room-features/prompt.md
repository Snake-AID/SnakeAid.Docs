---
doc_role: operation
operation_id: 04-FEAT-in-room-features
generated_from: plan.md
status: draft
created_at: 2026-03-05
---

# Prompt: Implement In-Room Features

## Requirements

Implement the `ConsultationHub` and auxiliary endpoints as defined in `plan.md`.

Specific tasks:

1. Implement `Api/Hubs/ConsultationHub.cs` derived from `Hub`. In `OnConnectedAsync`, extract `consultationId` from the query string. Look up the `Consultation` in the DB. If the current JWT user is not `CallerId` nor `CalleeId`, terminate the connection. Otherwise, assign the connection to a SignalR Group using the `consultationId`.
2. Implement robust `ReceiveMessage(string content, ...)` logic in the Hub. Save to `ChatMessage` table, then push to the grouped clients. Include support for basic media/image links if present in the wireframes.
3. Implement `Signal(string eventType, string payload)` to bounce real-time UI states (like typing indicators or camera toggles). Do not persist these events in the DB.
4. Add `GET /api/v1/snakes/search` to `SnakesController`. Map standard text search against the `SnakeSpecies` table, including `SpeciesVenom` and `SpeciesAntivenom` details needed for quick reference by the expert.

## Constraints

- Ensure strict privacy isolation. Messages must never be broadcast outside the immediate `ConsultationId` group.
- Keep the `ConsultationHub` strictly out of global presence management (which belongs to `ExpertHub`).
