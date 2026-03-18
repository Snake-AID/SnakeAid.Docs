---
doc_role: operation-specific
module: consultation
operation: 05-FEAT-in-room-features
kind: flow
status: active
last_updated: 2026-03-18
owners: [backend-team, mobile-team]
---

# Operation 5: In-Room Consultation Features Usage Guide

## Mục tiêu tài liệu

Tài liệu này là integration reference dành riêng cho mobile dev về operation 5 (In-Room Features).

**Operation 5 bao gồm:**
- Real-time text chat trong phòng tư vấn
- Gửi hình ảnh trong chat
- Tìm kiếm thông tin loài rắn cho chuyên gia
- UI signaling (typing indicators, mic/camera status)

**Tài liệu chính:**
- `Consultation Screen API.md`: Màn hình nào gọi gì, khi nào gọi
- `consultation.usageguide.md`: Chi tiết tất cả operations (toàn diện nhưng dài)
- Tài liệu này: Chi tiết chỉ operation 5 (tối ưu context window)

## Response Envelope

Tất cả REST success response đều dùng `ApiResponse<T>`:

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": {}
}
```

## Public API Summary (Operation 5)

### In-Room Consultation Features

- **Search Snake Species**: `GET /api/v1/snakes/search?q={query}`
- **Upload Chat Media**: `POST /api/media/upload-image` (domain: "chat-media")

### SignalR Consultation Chat

- **Hub Endpoint**: `/hubs/consultation`
- **Methods** (both Caller and Callee):
  - `ReceiveMessage(string content, string? attachmentUrl)`
  - `Signal(string eventType, string payload)`
- **Server Events**:
  - `MessageReceived`
  - `SignalReceived`

## Detailed Endpoint Contracts

### Endpoint: Search Snake Species

`GET /api/v1/snakes/search?q={query}`

**Query Parameters**

- `q` (string, required): Search query for snake species by scientific name, common name, or alternative names

**Response DTO**

- `ApiResponse<List<SearchSnakeSpeciesResponse>>`

**Response Sample**

```json
{
  "status_code": 200,
  "message": "Operation successful",
  "is_success": true,
  "data": [
    {
      "id": 1,
      "scientificName": "Naja naja",
      "commonName": "Indian Cobra",
      "imageUrl": "https://cloudinary.com/snakeaid/...",
      "isVenomous": true,
      "primaryVenomType": "Neurotoxic",
      "venoms": [
        {
          "venomType": "Neurotoxic",
          "description": "Affects nervous system"
        }
      ],
      "antivenoms": [
        {
          "antivenomName": "Anti-Cobra Venom",
          "manufacturer": "Serum Institute",
          "effectiveness": "Highly effective"
        }
      ]
    }
  ]
}
```

**Usage Context**
- Used by expert app during consultation
- Helps expert quickly reference snake species information
- Returns venom types and available antivenoms

### Endpoint: Upload Chat Media

`POST /api/media/upload-image`

**Request Form Data**
- `file`: Image file (JPG, PNG, GIF only, max 5MB)
- `domain`: "chat-media"

**Response DTO**
- `ApiResponse<CloudinaryUploadResult>`

**Response Sample**
```json
{
  "status_code": 200,
  "message": "Image uploaded successfully",
  "is_success": true,
  "data": {
    "secureUrl": "https://res.cloudinary.com/snakeaid/image/upload/...",
    "publicId": "chat-media-1234567890",
    "resourceType": "image",
    "format": "jpg",
    "bytes": 2048576,
    "width": 1200,
    "height": 800,
    "folder": "snakeaid/development/chat-media/user-id",
    "tags": ["snakeaid", "development", "chat-media"]
  }
}
```

**Usage Context**
- Pre-upload images before sending in chat
- Use the returned `secureUrl` as `attachmentUrl` in `ReceiveMessage`

## SignalR Consultation Chat

### Hub Endpoint
`/hubs/consultation?consultationId={guid}`

### Connection Requirements

- **Query Parameter**: `consultationId` (GUID) - ID of the active consultation
- **Authorization**: User must be either Caller or Callee of the consultation
- **Automatic Check**: `OnConnectedAsync` validates permissions on connect

### Client Methods

#### ReceiveMessage
```csharp
ReceiveMessage(string content, string? attachmentUrl)
```

**Parameters**
- `content`: Text message content
- `attachmentUrl`: Optional pre-uploaded image URL from media upload

**Behavior**
- Saves message to database (ChatMessage entity)
- Broadcasts to all consultation participants
- Rate limit: 10 messages per minute per user

#### Signal
```csharp
Signal(string eventType, string payload)
```

**Parameters**
- `eventType`: String identifier for signal type (e.g., "typing", "mic-on")
- `payload`: String data for the signal

**Behavior**
- Volatile signals (not saved to database)
- Broadcasts immediately to other participants
- Used for UI state synchronization

### Server Events

#### MessageReceived
**Payload**: `ChatMessage` object

**Sample**
```json
{
  "id": 123,
  "consultationId": "bfa25be0-10ae-4b9c-b923-2180703eeb7e",
  "senderId": "2a6d6f8f-6468-4d6c-b628-9b0fa326f7a8",
  "content": "Hello, can you help?",
  "attachmentUrl": null,
  "sentAt": "2026-03-18T10:00:00Z",
  "senderName": "Alice Smith"
}
```

#### SignalReceived
**Payload**: `eventType` (string), `payload` (string)

**Sample**
```json
{
  "eventType": "typing",
  "payload": "true"
}
```

## Integration Flow

### For Mobile Chat Implementation

1. **Connect to Hub**
   ```javascript
   const connection = new signalR.HubConnectionBuilder()
     .withUrl(`/hubs/consultation?consultationId=${consultationId}`)
     .build();
   ```

2. **Listen to Events**
   ```javascript
   connection.on('MessageReceived', (message) => {
     // Display message in chat UI
   });

   connection.on('SignalReceived', (eventType, payload) => {
     // Handle UI signals (typing, mic status, etc.)
   });
   ```

3. **Send Messages**
   ```javascript
   // For text messages
   await connection.invoke('ReceiveMessage', content, null);

   // For image messages
   const uploadResult = await uploadImage(file);
   await connection.invoke('ReceiveMessage', content, uploadResult.secureUrl);
   ```

4. **Send Signals**
   ```javascript
   await connection.invoke('Signal', 'typing', 'true');
   ```

## Error Notes

- **403 Forbidden**: When user is not a participant of the consultation
- **429 Too Many Requests**: When exceeding 10 messages per minute
- **400 Bad Request**: Invalid file format or size for media upload
- **413 Payload Too Large**: File exceeds 5MB limit

## Current MVP Limits (Operation 5)

- Chat history stored forever (no retention policy)
- Rate limiting: 10 messages/minute per user per consultation
- Media: Images only (JPG, PNG, GIF), max 5MB
- No file attachments beyond images
- Signals are volatile (not persisted)
- Moderation: No content filtering implemented

## Related Documentation

- `Consultation Screen API.md`: Screen flow and timing
- `consultation.usageguide.md`: Full consultation API reference
- `docs/image-upload-cloudinary-handling.md`: Media upload technical details