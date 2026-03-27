---
doc_role: baseline
module: consultation.in-room
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-28
owners: [backend-team, mobile-team]
---

# In-Room Consultation Features — Usage Guide

## Scope

Real-time chat, image attachments, UI signaling, và snake species search trong phòng tư vấn. Áp dụng cho cả scheduled và emergency consultation.

## ConsultationHub (`/hubs/consultation`)

### Connection

```
/hubs/consultation?consultationId={guid}
```

- User phải là Caller hoặc Callee của consultation
- `OnConnectedAsync` tự động check authorization
- Unauthorized → `HubException`

### Client Methods

#### `ReceiveMessage(string content, string? attachmentUrl)`

Gửi tin nhắn text hoặc kèm hình ảnh. Lưu DB + broadcast cho group.

- `attachmentUrl`: URL từ `POST /api/media/upload-image` (domain: "chat-media")
- Rate limit: 10 messages/phút/user

#### `Signal(string eventType, string payload)`

Gửi UI state signals (typing, mic/cam status). Volatile — không lưu DB.

### Server Events

#### `MessageReceived`

```json
{
  "id": 123,
  "consultationId": "bfa25be0-...",
  "senderId": "2a6d6f8f-...",
  "content": "Hello, can you help?",
  "attachmentUrl": null,
  "sentAt": "2026-03-18T10:00:00Z",
  "senderName": "Alice Smith"
}
```

#### `SignalReceived`

```json
{ "eventType": "typing", "payload": "true" }
```

## Image Upload

### `POST /api/media/upload-image`

Upload trước khi gửi chat. Form data: `file` (JPG/PNG/GIF, max 5MB) + `domain: "chat-media"`.

**Response**:
```json
{
  "data": {
    "secureUrl": "https://res.cloudinary.com/snakeaid/image/upload/...",
    "publicId": "chat-media-1234567890",
    "resourceType": "image",
    "format": "jpg",
    "bytes": 2048576
  }
}
```

Dùng `secureUrl` làm `attachmentUrl` trong `ReceiveMessage`.

## Snake Species Search

### `GET /api/v1/snakes/search?q={query}`

Expert side-panel: search theo scientific name, common name, alternative names.

**Response**: `ApiResponse<List<SearchSnakeSpeciesResponse>>`

```json
{
  "data": [{
    "id": 1,
    "scientificName": "Naja naja",
    "commonName": "Indian Cobra",
    "imageUrl": "https://cloudinary.com/snakeaid/...",
    "isVenomous": true,
    "primaryVenomType": "Neurotoxic",
    "venoms": [{ "venomType": "Neurotoxic", "description": "Affects nervous system" }],
    "antivenoms": [{ "antivenomName": "Anti-Cobra Venom", "manufacturer": "Serum Institute", "effectiveness": "Highly effective" }]
  }]
}
```

## Mobile Integration Pattern

```javascript
// Connect
const conn = new signalR.HubConnectionBuilder()
  .withUrl(`/hubs/consultation?consultationId=${id}`)
  .build();

// Listen
conn.on('MessageReceived', (msg) => { /* display */ });
conn.on('SignalReceived', (type, payload) => { /* handle */ });

// Send text
await conn.invoke('ReceiveMessage', content, null);

// Send image
const upload = await uploadImage(file);
await conn.invoke('ReceiveMessage', caption, upload.secureUrl);

// Send signal
await conn.invoke('Signal', 'typing', 'true');
```

## Screen Delivery Status

- Chat (User): `Build Now`
- Chat (Expert): `Build Now`

## Error Notes

- `403`: user không phải participant
- `429`: vượt 10 messages/phút
- `400`: file format không hợp lệ
- `413`: file > 5MB

## Constraints

- Chat history lưu vĩnh viễn (no retention policy)
- Media: images only (JPG/PNG/GIF), max 5MB
- Signals volatile (không persist)
- Không có content filtering/moderation
