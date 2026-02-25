---
doc_role: baseline
module: live-kit-cloud
kind: flow
status: active
last_updated: 2026-02-25
owners: [backend-team]
---

# LiveKit Cloud — Source Code Map

## API Surface

### Controller

| Item         | Value                                             |
| ------------ | ------------------------------------------------- |
| Class        | `VideoCallController`                             |
| Base         | `BaseController<VideoCallController>`             |
| Route prefix | `api/videocall`                                   |
| File         | `SnakeAid.Api/Controllers/VideoCallController.cs` |

### Endpoints

| Method | Route                            | Auth               | Description                                    |
| ------ | -------------------------------- | ------------------ | ---------------------------------------------- |
| POST   | `livekit-token/{consultationId}` | `[Authorize]`      | Generate LiveKit token for consultation        |
| POST   | `livekit-token/demo/{roomname}`  | `[Authorize]`      | **[DEV]** Generate token with custom room name |
| POST   | `livekit-webhook`                | `[AllowAnonymous]` | Receive LiveKit Cloud event callbacks          |

---

## Service Layer

### ILiveKitService (`SnakeAid.Service/Interfaces/ILiveKitService.cs`)

```csharp
string GenerateAccessToken(string identity, string roomName, VideoGrants grants,
    string? metadata = null, TimeSpan? ttl = null);

Task<RoomInfoResponse> CreateRoomAsync(string roomName, int maxParticipants = 2,
    int emptyTimeoutSeconds = 600, CancellationToken cancellationToken = default);

Task DeleteRoomAsync(string roomName, CancellationToken cancellationToken = default);

Task<List<RoomInfoResponse>> ListRoomsAsync(CancellationToken cancellationToken = default);

LiveKitWebhookPayload? ValidateWebhook(string body, string authorizationHeader);
```

### Implementation (`SnakeAid.Service/Implements/LiveKitService.cs`)

Dependencies: `IOptions<LiveKitOptions>`, `ILiveKitApi`, `ILogger<LiveKitService>`

- **GenerateAccessToken**: Local JWT creation with `HmacSha256`, claims: `sub` (identity), `video` (JSON grants), `jti`
- **CreateRoomAsync / DeleteRoomAsync / ListRoomsAsync**: Generate server JWT → delegate to `ILiveKitApi` Refit client
- **ValidateWebhook**: Verify JWT signature from Authorization header → deserialize body

### ILiveKitApi — Refit (`SnakeAid.Service/Interfaces/ILiveKitApi.cs`)

```csharp
[Post("/twirp/livekit.RoomService/CreateRoom")]
Task<RoomInfoResponse> CreateRoomAsync([Body] CreateRoomRequest request,
    [Authorize("Bearer")] string token, CancellationToken cancellationToken = default);

[Post("/twirp/livekit.RoomService/DeleteRoom")]
Task DeleteRoomAsync([Body] DeleteRoomRequest request,
    [Authorize("Bearer")] string token, CancellationToken cancellationToken = default);

[Post("/twirp/livekit.RoomService/ListRooms")]
Task<ListRoomsResponse> ListRoomsAsync([Body] object request,
    [Authorize("Bearer")] string token, CancellationToken cancellationToken = default);
```

---

## DTOs

### Settings — `SnakeAid.Core/Settings/LiveKitOptions.cs`

| Property                | Type   | Default |
| ----------------------- | ------ | ------- |
| ApiKey                  | string | `""`    |
| ApiSecret               | string | `""`    |
| WsUrl                   | string | `""`    |
| TokenTtlMinutes         | int    | `10`    |
| RoomEmptyTimeoutSeconds | int    | `600`   |

### Request DTOs — `SnakeAid.Core/Requests/LiveKit/`

| Class                   | Properties                                                                  |
| ----------------------- | --------------------------------------------------------------------------- |
| `VideoGrants`           | Room, RoomJoin, CanPublish, CanSubscribe, CanPublishData, CanPublishSources |
| `CreateRoomRequest`     | Name, EmptyTimeout, MaxParticipants                                         |
| `DeleteRoomRequest`     | Room                                                                        |
| `LiveKitWebhookPayload` | Id, Event, CreatedAt, Room, Participant, RoomName                           |

### Response DTOs — `SnakeAid.Core/Responses/LiveKit/`

| Class                | Properties                               |
| -------------------- | ---------------------------------------- |
| `VideoTokenResponse` | Token, WsUrl, RoomName                   |
| `RoomInfoResponse`   | Name, Sid, NumParticipants, CreationTime |
| `ListRoomsResponse`  | Rooms (List\<RoomInfoResponse\>)         |

---

## DI Registration (`SnakeAid.Api/DI/DependencyInjection.AddServices()`)

- `Configure<LiveKitOptions>` from `"LiveKit"` config section
- `AddRefitClient<ILiveKitApi>` → base URL derived from `WsUrl` (wss→https) + Polly retry + circuit breaker
- `AddScoped<ILiveKitService, LiveKitService>`

## Configuration (`appsettings.json`)

Section: `"LiveKit"` with keys: `ApiKey`, `ApiSecret`, `WsUrl`, `TokenTtlMinutes`, `RoomEmptyTimeoutSeconds`

## Cross-cutting

- **Auth**: Video token endpoints require Bearer JWT; webhook uses LiveKit's own JWT signature validation
- **Logging**: Structured logging on token generation, room operations, webhook events
- **Error handling**: Try-catch in controller, returns `{ success, message }` envelope
- **Role-based grants**: Expert role gets `screen_share` + `screen_share_audio` publish sources

---

## Operations History

| Operation                 | Type | Status |
| ------------------------- | ---- | ------ |
| `01-FEAT-video-call`      | FEAT | done   |
| `02-FEAT-video-call-demo` | FEAT | done   |
