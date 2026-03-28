---
doc_role: operation
operation_id: 04-in-room-features
type: FEAT
status: completed
created_at: 2026-03-05
implemented_at: 2026-03-18
merged_from: [05-FEAT-in-room-features]
affects:
  - Api/Hubs/ConsultationHub.cs
  - Api/Controllers/SnakeSpeciesController.cs
  - Core/Domains/ChatMessage.cs
---

# Operation 04: In-Room Consultation Features

## Mục tiêu

Implement tính năng trong phòng tư vấn: real-time chat, image attachments, UI signaling, và expert snake species lookup.

## Scope đã implement

### ConsultationHub (`/hubs/consultation`)

- Connect với `?consultationId={id}`, auto-check Caller/Callee authorization
- `ReceiveMessage(content, attachmentUrl?)` — lưu DB + broadcast cho group
- `Signal(eventType, payload)` — volatile UI state (typing, mic/cam), không lưu DB
- Events: `MessageReceived`, `SignalReceived`

### Image Message Flow

- Client upload trước qua `POST /api/media/upload-image` (domain: "chat-media")
- Gửi `secureUrl` làm `attachmentUrl` trong `ReceiveMessage`
- Giới hạn: JPG/PNG/GIF, max 5MB

### Snake Species Search

- `GET /api/snake-species/search?q={query}` — search theo scientific name, common name, alternative names
- Trả venom types + antivenom data cho expert side-panel

### Constraints

- Rate limit: 10 messages/phút/user/consultation
- Chat history lưu vĩnh viễn (no retention policy)
- Strict privacy: messages không leak qua consultation khác
- ConsultationHub tách biệt hoàn toàn với ExpertHub
