---
doc_role: operation
operation_id: FEAT-video-call
generated_from: plan.md
status: done
created_at: 2026-02-24
---

# FEAT-video-call — Backend Prompt

## Objective

Implement LiveKit Cloud integration for video call in expert consultation.
Follow existing codebase conventions (PayOs integration as reference pattern).

## Code Culture Rules (MUST FOLLOW)

| Rule                      | Convention                                               | Reference                           |
| ------------------------- | -------------------------------------------------------- | ----------------------------------- |
| **Refit API interface**   | `SnakeAid.Service/Interfaces/I{Name}Api.cs`              | `ISnakeAIApi.cs`                    |
| **Refit DI registration** | `AddRefitClient<T>()` + Polly retry + circuit breaker    | `DependencyInjection.AddServices()` |
| Service interfaces        | `SnakeAid.Service/Interfaces/I{Name}Service.cs`          | `IPayOsClient.cs`                   |
| Service implementations   | `SnakeAid.Service/Implements/{Name}Service.cs`           | `PayOsClient.cs`                    |
| Settings POCOs            | `SnakeAid.Core/Settings/{Name}Options.cs`                | `PayOsOptions.cs`                   |
| Request DTOs              | `SnakeAid.Core/Requests/{Domain}/` subfolder             | `Requests/PayOS/`                   |
| Response DTOs             | `SnakeAid.Core/Responses/{Domain}/` subfolder            | `Responses/PayOS/`                  |
| Controllers               | Inherit `BaseController<T>`                              | `PayOsController`                   |
| Controller DI             | `(ILogger<T>, IHttpContextAccessor, IMapper, +services)` | `PayOsController` ctor              |
| DI registration           | `DI/DependencyInjection.AddServices()` extension         | `DependencyInjection.cs`            |
| Async methods             | Always accept `CancellationToken cancellationToken`      | All service methods                 |
| Webhook                   | Same controller, read raw body                           | `PayOsController.Webhook`           |

## Required Outputs

### 1. Settings — `SnakeAid.Core/Settings/LiveKitOptions.cs`

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

### 2. Request DTOs — `SnakeAid.Core/Requests/LiveKit/`

**`VideoGrants.cs`**:

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

**`CreateRoomRequest.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class CreateRoomRequest
{
    public string Name { get; set; } = string.Empty;
    public int EmptyTimeout { get; set; } = 600;
    public int MaxParticipants { get; set; } = 2;
}
```

**`DeleteRoomRequest.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class DeleteRoomRequest
{
    public string Room { get; set; } = string.Empty;
}
```

**`LiveKitWebhookPayload.cs`**:

```csharp
namespace SnakeAid.Core.Requests.LiveKit;

public class LiveKitWebhookPayload
{
    public string? Id { get; set; }
    public string? Event { get; set; }
    public long CreatedAt { get; set; }
    public LiveKitWebhookRoom? Room { get; set; }
    public LiveKitWebhookParticipant? Participant { get; set; }
    public string RoomName { get; set; } = string.Empty;
}

public class LiveKitWebhookRoom
{
    public string? Name { get; set; }
    public string? Sid { get; set; }
}

public class LiveKitWebhookParticipant
{
    public string? Identity { get; set; }
    public string? Sid { get; set; }
    public string? Name { get; set; }
}
```

### 3. Response DTOs — `SnakeAid.Core/Responses/LiveKit/`

**`VideoTokenResponse.cs`**:

```csharp
namespace SnakeAid.Core.Responses.LiveKit;

public class VideoTokenResponse
{
    public string Token { get; set; } = string.Empty;
    public string WsUrl { get; set; } = string.Empty;
    public string RoomName { get; set; } = string.Empty;
}
```

**`RoomInfoResponse.cs`**:

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

**`ListRoomsResponse.cs`**:

```csharp
namespace SnakeAid.Core.Responses.LiveKit;

public class ListRoomsResponse
{
    public List<RoomInfoResponse> Rooms { get; set; } = new();
}
```

### 4. Refit API Interface — `SnakeAid.Service/Interfaces/ILiveKitApi.cs`

Following `ISnakeAIApi` pattern:

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

### 5. Business Service Interface — `SnakeAid.Service/Interfaces/ILiveKitService.cs`

```csharp
using SnakeAid.Core.Requests.LiveKit;
using SnakeAid.Core.Responses.LiveKit;

namespace SnakeAid.Service.Interfaces;

public interface ILiveKitService
{
    // JWT token generation (local, no HTTP)
    string GenerateAccessToken(
        string identity,
        string roomName,
        VideoGrants grants,
        string? metadata = null,
        TimeSpan? ttl = null);

    // Room management (delegates to ILiveKitApi)
    Task<RoomInfoResponse> CreateRoomAsync(
        string roomName, int maxParticipants = 2,
        int emptyTimeoutSeconds = 600,
        CancellationToken cancellationToken = default);

    Task DeleteRoomAsync(string roomName,
        CancellationToken cancellationToken = default);

    Task<List<RoomInfoResponse>> ListRoomsAsync(
        CancellationToken cancellationToken = default);

    // Webhook validation (local JWT verification)
    LiveKitWebhookPayload? ValidateWebhook(
        string body, string authorizationHeader);
}
```

### 6. Service Implementation — `SnakeAid.Service/Implements/LiveKitService.cs`

```csharp
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
using Microsoft.IdentityModel.Tokens;
using SnakeAid.Core.Settings;
using SnakeAid.Core.Requests.LiveKit;
using SnakeAid.Core.Responses.LiveKit;
using SnakeAid.Service.Interfaces;
using System.IdentityModel.Tokens.Jwt;
using System.Text;

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
    //   SigningKey: SymmetricSecurityKey(Encoding.UTF8.GetBytes(_options.ApiSecret))
    //   Algorithm: HmacSha256
    //   Claims: iss = ApiKey, sub = identity, "video" = VideoGrants JSON

    // CreateRoomAsync() → generate server JWT → _liveKitApi.CreateRoomAsync(request, serverToken)
    // DeleteRoomAsync() → generate server JWT → _liveKitApi.DeleteRoomAsync(request, serverToken)
    // ListRoomsAsync() → generate server JWT → _liveKitApi.ListRoomsAsync(new {}, serverToken)

    // ValidateWebhook() → JWT signature verification (local, no HTTP)
}
```

### 7. Controller — `SnakeAid.Api/Controllers/LiveKitController.cs`

```csharp
using MapsterMapper;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using SnakeAid.Core.Responses.LiveKit;
using SnakeAid.Service.Interfaces;

namespace SnakeAid.Api.Controllers;

[Route("api/livekit")]
public class LiveKitController : BaseController<LiveKitController>
{
    private readonly ILiveKitService _liveKitService;
    // Inject consultation service for ownership validation

    public LiveKitController(
        ILogger<LiveKitController> logger,
        IHttpContextAccessor httpContextAccessor,
        IMapper mapper,
        ILiveKitService liveKitService)
        : base(logger, httpContextAccessor, mapper)
    {
        _liveKitService = liveKitService;
    }

    /// <summary>
    /// Generate LiveKit video token for a consultation
    /// </summary>
    [HttpPost("consultation/{consultationId}/video-token")]
    [Authorize]
    public async Task<IActionResult> GenerateVideoToken(
        Guid consultationId,
        CancellationToken cancellationToken)
    {
        // 1. GetCurrentUserId() from BaseController
        // 2. Load consultation, validate user is participant
        // 3. Validate consultation status (Paid/Scheduled/InProgress)
        // 4. Determine grants by role (Expert gets screen_share)
        // 5. Generate token via _liveKitService.GenerateAccessToken()
        // 6. Return Ok(VideoTokenResponse)
    }

    /// <summary>
    /// LiveKit webhook endpoint
    /// </summary>
    [HttpPost("webhook")]
    [AllowAnonymous]
    public async Task<IActionResult> Webhook(
        CancellationToken cancellationToken)
    {
        // Following PayOsController.Webhook pattern:
        // 1. Read raw body via Request.Body with StreamReader
        // 2. Get Authorization header
        // 3. _liveKitService.ValidateWebhook(body, authHeader)
        // 4. Route by event type
        // 5. Return Ok()
    }
}
```

### 8. DI Registration — Add to `DI/DependencyInjection.AddServices()`

Following `ISnakeAIApi` Refit + Polly pattern:

```csharp
// Inside AddServices() method, add:
var liveKitOptions = configuration.GetSection("LiveKit").Get<LiveKitOptions>();
if (liveKitOptions is null)
    throw new InvalidOperationException("LiveKit settings are not configured properly.");

services.Configure<LiveKitOptions>(configuration.GetSection("LiveKit"));

services
    .AddRefitClient<ILiveKitApi>()
    .ConfigureHttpClient(c =>
    {
        c.BaseAddress = new Uri(liveKitOptions.WsUrl.Replace("wss://", "https://"));
    })
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy());

services.AddScoped<ILiveKitService, LiveKitService>();
```

### 9. Configuration — Add to `appsettings.json`

```json
{
  "LiveKit": {
    "ApiKey": "{{LIVEKIT_API_KEY}}",
    "ApiSecret": "{{LIVEKIT_API_SECRET}}",
    "WsUrl": "wss://{{PROJECT_ID}}.livekit.cloud",
    "TokenTtlMinutes": 10,
    "RoomEmptyTimeoutSeconds": 600
  }
}
```

## Forbidden Changes

- Do NOT use raw `HttpClient` for LiveKit API — use Refit `ILiveKitApi`
- Do NOT create service files in `SnakeAid.Api/Services/` — use `SnakeAid.Service/` project
- Do NOT create settings in `SnakeAid.Api/` — use `SnakeAid.Core/Settings/`
- Do NOT create DTOs outside `SnakeAid.Core/Requests/` or `SnakeAid.Core/Responses/`
- Do NOT create a separate `WebhookController` — put webhook in `LiveKitController`
- Do NOT register DI directly in `Program.cs` — use `DependencyInjection.AddServices()`
- Do NOT skip `CancellationToken` on async methods
- Do NOT skip `BaseController<T>` inheritance
- Do NOT store real secrets in `appsettings.json` — use placeholders

## NuGet Dependencies

```xml
<!-- In SnakeAid.Service.csproj -->
<PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="8.*" />
<PackageReference Include="Microsoft.IdentityModel.Tokens" Version="8.*" />
```

## Test Requirements

- Unit test: `GenerateAccessToken` → valid JWT with correct claims
- Unit test: `ValidateWebhook` → accepts valid, rejects tampered
- Unit test: `GenerateVideoToken` endpoint → 403 for wrong user, 409 for wrong status
