---
doc_role: operation
operation_id: FEAT-video-call
type: FEAT
status: draft
created_at: 2026-02-24
affects:
  - SnakeAid.Service/Interfaces/ILiveKitApi.cs
  - SnakeAid.Service/Interfaces/ILiveKitService.cs
  - SnakeAid.Service/Implements/LiveKitService.cs
  - SnakeAid.Core/Settings/LiveKitOptions.cs
  - SnakeAid.Core/Requests/LiveKit/VideoGrants.cs
  - SnakeAid.Core/Requests/LiveKit/CreateRoomRequest.cs
  - SnakeAid.Core/Requests/LiveKit/DeleteRoomRequest.cs
  - SnakeAid.Core/Requests/LiveKit/LiveKitWebhookPayload.cs
  - SnakeAid.Core/Responses/LiveKit/VideoTokenResponse.cs
  - SnakeAid.Core/Responses/LiveKit/RoomInfoResponse.cs
  - SnakeAid.Core/Responses/LiveKit/ListRoomsResponse.cs
  - SnakeAid.Api/Controllers/LiveKitController.cs
  - SnakeAid.Api/DI/DependencyInjection.cs
  - SnakeAid.Api/appsettings.json
---

# FEAT-video-call — Backend Plan

## 1. As-Is (from `live-kit-cloud.sourcecode.md`)

No video call implementation exists in the backend.
The consultation module (P3) has basic CRUD and scheduling.
No LiveKit integration, no video token generation, no webhook handling.

### Existing Patterns (Code Culture Reference)

| Pattern                 | Example                                                     | Convention                               |
| ----------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| **Refit API interface** | `ISnakeAIApi` with `[Post]`/`[Get]` attributes              | `I{Name}Api.cs` in `Service/Interfaces/` |
| **Refit registration**  | `AddRefitClient<ISnakeAIApi>()` + Polly                     | In `DependencyInjection.AddServices()`   |
| Service interface       | `IPayOsClient.cs`                                           | `I{Name}Service.cs` in `Interfaces/`     |
| Service implementation  | `PayOsClient.cs`                                            | `{Name}Service.cs` in `Implements/`      |
| Settings POCO           | `PayOsOptions.cs`                                           | `{Name}Options.cs` in `Core/Settings/`   |
| Request DTOs            | `Core/Requests/PayOS/`                                      | Domain subfolder under `Requests/`       |
| Response DTOs           | `Core/Responses/PayOS/`                                     | Domain subfolder under `Responses/`      |
| Controller base         | `PayOsController : BaseController<PayOsController>`         | Inherit `BaseController<T>`              |
| Controller DI           | `(ILogger<T>, IHttpContextAccessor, IMapper, IService)`     | Base + domain service                    |
| DI registration         | `DI/DependencyInjection.AddServices()`                      | Extension method on `IServiceCollection` |
| Async methods           | `Task<T> Method(args, CancellationToken cancellationToken)` | Always pass `CancellationToken`          |
| Webhook                 | `PayOsController.Webhook`                                   | Same controller, reads raw body          |

## 2. Gap Analysis

| Gap                         | Description                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------- |
| **No LiveKit service**      | Need `ILiveKitService` in `Service/Interfaces/` and `LiveKitService` in `Service/Implements/` |
| **No video token endpoint** | Need `POST /api/livekit/consultation/{id}/video-token` in `LiveKitController`                 |
| **No webhook receiver**     | Need `POST /api/livekit/webhook` in same `LiveKitController`                                  |
| **No configuration**        | Need `LiveKitOptions` in `Core/Settings/`                                                     |
| **No DTOs**                 | Need request/response models in `Core/Requests/LiveKit/` and `Core/Responses/LiveKit/`        |

## 3. To-Be Design

### 3.1 Configuration — `SnakeAid.Core/Settings/LiveKitOptions.cs`

```csharp
namespace SnakeAid.Core.Settings;

public class LiveKitOptions
{
    public string ApiKey { get; set; } = string.Empty;
    public string ApiSecret { get; set; } = string.Empty;
    public string WsUrl { get; set; } = string.Empty;
    public int TokenTtlMinutes { get; set; } = 10;
    public int RoomEmptyTimeoutSeconds { get; set; } = 600;
}
```

### 3.2 Refit API Interface — `SnakeAid.Service/Interfaces/ILiveKitApi.cs`

Following `ISnakeAIApi` pattern — declarative HTTP API via Refit attributes:

```csharp
using Refit;
using SnakeAid.Core.Requests.LiveKit;
using SnakeAid.Core.Responses.LiveKit;

namespace SnakeAid.Service.Interfaces;

/// <summary>
/// Refit interface for LiveKit Twirp RoomService API
/// </summary>
public interface ILiveKitApi
{
    [Post("/twirp/livekit.RoomService/CreateRoom")]
    Task<RoomInfoResponse> CreateRoomAsync(
        [Body] CreateRoomRequest request,
        [Authorize("Bearer")] string token,
        CancellationToken cancellationToken = default);

    [Post("/twirp/livekit.RoomService/DeleteRoom")]
    Task DeleteRoomAsync(
        [Body] DeleteRoomRequest request,
        [Authorize("Bearer")] string token,
        CancellationToken cancellationToken = default);

    [Post("/twirp/livekit.RoomService/ListRooms")]
    Task<ListRoomsResponse> ListRoomsAsync(
        [Body] object request,
        [Authorize("Bearer")] string token,
        CancellationToken cancellationToken = default);
}
```

### 3.3 Business Service Interface — `SnakeAid.Service/Interfaces/ILiveKitService.cs`

High-level service that uses `ILiveKitApi` for HTTP + handles JWT token gen & webhook:

```csharp
namespace SnakeAid.Service.Interfaces;

public interface ILiveKitService
{
    string GenerateAccessToken(
        string identity,
        string roomName,
        VideoGrants grants,
        string? metadata = null,
        TimeSpan? ttl = null);

    Task<RoomInfoResponse> CreateRoomAsync(
        string roomName,
        int maxParticipants = 2,
        int emptyTimeoutSeconds = 600,
        CancellationToken cancellationToken = default);

    Task DeleteRoomAsync(
        string roomName,
        CancellationToken cancellationToken = default);

    Task<List<RoomInfoResponse>> ListRoomsAsync(
        CancellationToken cancellationToken = default);

    LiveKitWebhookPayload? ValidateWebhook(
        string body,
        string authorizationHeader);
}
```

### 3.4 Service Implementation — `SnakeAid.Service/Implements/LiveKitService.cs`

```csharp
namespace SnakeAid.Service.Implements;

public class LiveKitService : ILiveKitService
{
    private readonly LiveKitOptions _options;
    private readonly ILiveKitApi _liveKitApi;  // Refit client
    private readonly ILogger<LiveKitService> _logger;

    public LiveKitService(
        IOptions<LiveKitOptions> options,
        ILiveKitApi liveKitApi,
        ILogger<LiveKitService> logger)
    {
        _options = options.Value;
        _liveKitApi = liveKitApi;
        _logger = logger;
    }

    // GenerateAccessToken() → JWT creation (local, no HTTP)
    // CreateRoomAsync() → generate server token → _liveKitApi.CreateRoomAsync()
    // DeleteRoomAsync() → generate server token → _liveKitApi.DeleteRoomAsync()
    // ListRoomsAsync() → generate server token → _liveKitApi.ListRoomsAsync()
    // ValidateWebhook() → JWT signature verification (local, no HTTP)
}
```

### 3.5 Request/Response DTOs

**`SnakeAid.Core/Requests/LiveKit/VideoGrants.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class VideoGrants
{
    public string Room { get; set; } = string.Empty;
    public bool RoomJoin { get; set; } = true;
    public bool CanPublish { get; set; } = true;
    public bool CanSubscribe { get; set; } = true;
    public bool CanPublishData { get; set; } = true;
    public List<string>? CanPublishSources { get; set; }
}
```

**`SnakeAid.Core/Requests/LiveKit/CreateRoomRequest.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class CreateRoomRequest
{
    public string Name { get; set; } = string.Empty;
    public int EmptyTimeout { get; set; } = 600;
    public int MaxParticipants { get; set; } = 2;
}
```

**`SnakeAid.Core/Requests/LiveKit/DeleteRoomRequest.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class DeleteRoomRequest
{
    public string Room { get; set; } = string.Empty;
}
```

**`SnakeAid.Core/Responses/LiveKit/VideoTokenResponse.cs`**:

```csharp
namespace SnakeAid.Core.Responses.LiveKit;

public class VideoTokenResponse
{
    public string Token { get; set; } = string.Empty;
    public string WsUrl { get; set; } = string.Empty;
    public string RoomName { get; set; } = string.Empty;
}
```

**`SnakeAid.Core/Responses/LiveKit/RoomInfoResponse.cs`**:

```csharp
namespace SnakeAid.Core.Responses.LiveKit;

public class RoomInfoResponse
{
    public string Name { get; set; } = string.Empty;
    public string Sid { get; set; } = string.Empty;
    public int NumParticipants { get; set; }
    public long CreationTime { get; set; }
}
```

**`SnakeAid.Core/Responses/LiveKit/ListRoomsResponse.cs`**:

```csharp
namespace SnakeAid.Core.Responses.LiveKit;

public class ListRoomsResponse
{
    public List<RoomInfoResponse> Rooms { get; set; } = new();
}
```

**`SnakeAid.Core/Requests/LiveKit/LiveKitWebhookPayload.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class LiveKitWebhookPayload
{
    public string? Id { get; set; }
    public string? Event { get; set; }
    public long CreatedAt { get; set; }
    public LiveKitWebhookRoom? Room { get; set; }
    public LiveKitWebhookParticipant? Participant { get; set; }
}

public class LiveKitWebhookRoom { /* name, sid, etc. */ }
public class LiveKitWebhookParticipant { /* identity, sid, etc. */ }
```

### 3.6 Controller — `SnakeAid.Api/Controllers/LiveKitController.cs`

```csharp
namespace SnakeAid.Api.Controllers;

[Route("api/livekit")]
public class LiveKitController : BaseController<LiveKitController>
{
    private readonly ILiveKitService _liveKitService;

    public LiveKitController(
        ILogger<LiveKitController> logger,
        IHttpContextAccessor httpContextAccessor,
        IMapper mapper,
        ILiveKitService liveKitService)
        : base(logger, httpContextAccessor, mapper)
    {
        _liveKitService = liveKitService;
    }

    [HttpPost("consultation/{consultationId}/video-token")]
    [Authorize]
    public async Task<IActionResult> GenerateVideoToken(
        Guid consultationId,
        CancellationToken cancellationToken) { /* ... */ }

    [HttpPost("webhook")]
    [AllowAnonymous]
    public async Task<IActionResult> Webhook(
        CancellationToken cancellationToken) { /* ... */ }
}
```

### 3.7 DI Registration — Add to `DI/DependencyInjection.AddServices()`

Following `ISnakeAIApi` Refit + Polly registration pattern:

```csharp
// In AddServices method:
var liveKitOptions = configuration.GetSection("LiveKit").Get<LiveKitOptions>();
if (liveKitOptions is null)
    throw new InvalidOperationException("LiveKit settings are not configured properly.");

services.Configure<LiveKitOptions>(configuration.GetSection("LiveKit"));

// Register Refit client with Polly resilience
services
    .AddRefitClient<ILiveKitApi>()
    .ConfigureHttpClient(c =>
    {
        c.BaseAddress = new Uri(liveKitOptions.WsUrl.Replace("wss://", "https://"));
    })
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy());

// Register business service
services.AddScoped<ILiveKitService, LiveKitService>();
```

### 3.8 Token Generation (JWT)

LiveKit access token = standard JWT signed with HMAC-SHA256:

```
Header:  { "alg": "HS256", "typ": "JWT" }
Payload: {
  "iss": "{ApiKey}",
  "sub": "{identity}",
  "exp": {unix_timestamp},
  "nbf": {unix_timestamp},
  "video": { "room": "{roomName}", "roomJoin": true, ... },
  "metadata": "{json_string}"
}
Sign with: HMAC-SHA256(ApiSecret)
```

### 3.9 LiveKit Server API (via Refit)

Room management is handled by `ILiveKitApi` Refit interface.
Each call requires a server JWT (generated internally by `LiveKitService`) passed via `[Authorize("Bearer")]` parameter.
Base URL is derived from `WsUrl` (replace `wss://` → `https://` for HTTP API).

### 3.10 Webhook Handling (in `LiveKitController.Webhook`)

Following `PayOsController.Webhook` pattern:

1. Read raw body via `Request.Body`
2. Get `Authorization` header
3. Call `_liveKitService.ValidateWebhook(body, authHeader)`
4. Route by event type (`room_started`, `participant_joined`, `participant_left`, `room_finished`)
5. Return `Ok()`

## 4. Impacted Components

| Component                  | Project                            | Change                             |
| -------------------------- | ---------------------------------- | ---------------------------------- |
| `ILiveKitApi.cs`           | `SnakeAid.Service/Interfaces/`     | **NEW** — Refit interface          |
| `ILiveKitService.cs`       | `SnakeAid.Service/Interfaces/`     | **NEW** — Business service         |
| `LiveKitService.cs`        | `SnakeAid.Service/Implements/`     | **NEW**                            |
| `LiveKitOptions.cs`        | `SnakeAid.Core/Settings/`          | **NEW**                            |
| `VideoGrants.cs`           | `SnakeAid.Core/Requests/LiveKit/`  | **NEW**                            |
| `CreateRoomRequest.cs`     | `SnakeAid.Core/Requests/LiveKit/`  | **NEW**                            |
| `DeleteRoomRequest.cs`     | `SnakeAid.Core/Requests/LiveKit/`  | **NEW**                            |
| `LiveKitWebhookPayload.cs` | `SnakeAid.Core/Requests/LiveKit/`  | **NEW**                            |
| `VideoTokenResponse.cs`    | `SnakeAid.Core/Responses/LiveKit/` | **NEW**                            |
| `RoomInfoResponse.cs`      | `SnakeAid.Core/Responses/LiveKit/` | **NEW**                            |
| `ListRoomsResponse.cs`     | `SnakeAid.Core/Responses/LiveKit/` | **NEW**                            |
| `LiveKitController.cs`     | `SnakeAid.Api/Controllers/`        | **NEW**                            |
| `DependencyInjection.cs`   | `SnakeAid.Api/DI/`                 | **MODIFY** — add Refit + Polly     |
| `appsettings.json`         | `SnakeAid.Api/`                    | **MODIFY** — add `LiveKit` section |

## 5. Risks & Constraints

| Risk                                         | Mitigation                                                                 |
| -------------------------------------------- | -------------------------------------------------------------------------- |
| JWT `video` claim must be nested JSON object | Test with LiveKit CLI `lk token verify` or Flutter connect                 |
| Refit JSON serialization for Twirp           | Refit uses `System.Text.Json` by default — compatible with Twirp JSON mode |
| Server token for Refit `[Authorize]` param   | `LiveKitService` generates a short-lived server JWT per API call           |
| Webhook body must be read as raw string      | Use `Request.Body` with `StreamReader` (same as PayOs pattern)             |
| ApiSecret is raw string, not base64          | Pass directly to `SymmetricSecurityKey(Encoding.UTF8.GetBytes())`          |

## 6. Validation Plan

| Test               | Method                                           |
| ------------------ | ------------------------------------------------ |
| Token generation   | Unit test: generate → decode → verify claims     |
| Token accepted     | Integration: generate → Flutter `room.connect()` |
| Webhook validation | Unit test: mock signed payload → returns event   |
| Webhook rejected   | Unit test: tampered payload → returns null       |
| Auth: wrong user   | Unit test: non-participant → 403                 |
| Auth: wrong status | Unit test: unpaid consultation → 409             |
