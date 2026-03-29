---
doc_role: baseline
module: consultation.in-room
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-30
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

#### `RoomExpiring`

Gửi từ backend khi consultation hết giờ (scheduled: slot hết giờ, emergency: hết 30 phút). Gửi TRƯỚC KHI xóa phòng LiveKit.

```json
{
  "ConsultationId": "bfa25be0-...",
  "Reason": "slot_elapsed"
}
```

Mobile nên hiển thị thông báo "Phòng sắp đóng" rồi navigate về màn hình kết thúc. Chi tiết: xem `consultation.usageguide.expert-history.md` Mục 2.

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

### `GET /api/snake-species/search?q={query}`

Expert side-panel: search theo scientific name, common name, alternative names. Dùng PostgreSQL `ILIKE` cho case-insensitive matching.

**Response**: `ApiResponse<List<SearchSnakeSpeciesResponse>>`

```json
{
  "data": [{
    "id": 3,
    "scientificName": "Ophiophagus hannah",
    "commonName": "Rắn Hổ Mang Chúa",
    "imageUrl": "https://vietnamsnakes.com/storage/snakes/species/55/...",
    "galleryUrls": ["https://cloudinary.com/snakeaid/gallery1.jpg"],
    "isVenomous": true,
    "primaryVenomType": "Neurotoxic",
    "riskLevel": 10.0,
    "identification": {
      "physicalTraits": ["Kích thước khổng lồ (4-6m)", "Phình mang hẹp dài"],
      "behaviors": ["Chủ động tấn công nếu bị kích động"],
      "habitat": "Rừng rậm, nương rẫy, gần nguồn nước"
    },
    "venoms": [
      { "venomType": "Độc thần kinh", "description": "Nọc độc ảnh hưởng đến hệ thần kinh, gây tê liệt và suy hô hấp." }
    ],
    "antivenoms": [
      { "antivenomName": "Anti-Cobra Venom", "manufacturer": "Serum Institute", "effectiveness": "Highly effective" }
    ],
    "firstAid": {
      "mode": "Append",
      "doItems": ["Băng ép ngay vết cắn", "Gọi cấp cứu 115"],
      "dontItems": ["Không rạch vết thương", "Không hút nọc"]
    },
    "tags": ["Miền Nam", "Rừng rậm", "Cực độc"]
  }]
}
```

**Giải thích các trường mới**:

| Trường | Kiểu | Ghi chú |
|--------|------|---------|
| galleryUrls | string[] | Ảnh bổ sung từ LibraryMedia (có thể rỗng) |
| riskLevel | float | 0-10, mức độ nguy hiểm |
| identification | object? | Đặc điểm nhận dạng (physicalTraits, behaviors, habitat) |
| firstAid | object? | Hướng dẫn sơ cứu (doItems, dontItems) |
| tags | string[] | Filter tags từ FilterSnakeMapping (vùng miền, đặc điểm) |

## Mobile Integration Pattern

```javascript
// Connect
const conn = new signalR.HubConnectionBuilder()
  .withUrl(`/hubs/consultation?consultationId=${id}`)
  .build();

// Listen
conn.on('MessageReceived', (msg) => { /* display */ });
conn.on('SignalReceived', (type, payload) => { /* handle */ });
conn.on('RoomExpiring', (data) => {
  // data.ConsultationId, data.Reason
  // Show "Room closing" notification, then navigate to end screen
});

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
