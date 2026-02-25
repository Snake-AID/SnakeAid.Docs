---
doc_role: operation
operation_id: FEAT-video-call-demo
generated_from: plan.md
status: done
created_at: 2026-02-25
---

# FEAT-video-call-demo — Backend Prompt

## Objective

Rename `LiveKitController` to `VideoCallController`, change route prefix from `api/livekit` to `api/videocall`, update endpoint routes, and add a demo token endpoint for development testing.

## Code Culture Rules (MUST FOLLOW)

Same conventions as FEAT-video-call. No changes to service layer or DTOs.

## Required Changes

### 1. Rename controller file

`SnakeAid.Api/Controllers/LiveKitController.cs` → `SnakeAid.Api/Controllers/VideoCallController.cs`

### 2. Update controller class

```csharp
[ApiController]
[Route("api/videocall")]
public class VideoCallController : BaseController<VideoCallController>
{
    // Same DI: ILiveKitService, IOptions<LiveKitOptions>
    // Constructor signature unchanged except class name
}
```

### 3. Update existing endpoint routes

**Token endpoint** — route change only:

```csharp
[HttpPost("livekit-token/{consultationId}")]
[Authorize]
[SwaggerOperation(
    Summary = "Generate LiveKit video token for consultation",
    Description = "Generates a LiveKit access token for the authenticated user to join a consultation video call.",
    Tags = new[] { "VideoCall" })]
public Task<IActionResult> GenerateVideoToken(
    Guid consultationId,
    CancellationToken cancellationToken)
{
    // Logic unchanged
}
```

**Webhook endpoint** — route change only:

```csharp
[HttpPost("livekit-webhook")]
[AllowAnonymous]
[SwaggerOperation(
    Summary = "LiveKit webhook endpoint",
    Description = "Receives LiveKit event notifications (room_started, room_finished, participant_joined, etc.).",
    Tags = new[] { "VideoCall" })]
public async Task<IActionResult> Webhook(CancellationToken cancellationToken)
{
    // Logic unchanged
}
```

### 4. Add demo token endpoint (NEW)

```csharp
/// <summary>
/// Generate LiveKit video token for testing (dev only)
/// </summary>
[HttpPost("livekit-token/demo/{roomname}")]
[Authorize]
[SwaggerOperation(
    Summary = "[DEV] Generate LiveKit video token for testing",
    Description = "Development-only endpoint. generates a LiveKit token using a custom room name, bypassing consultation validation.",
    Tags = new[] { "VideoCall" })]
[ProducesResponseType(typeof(VideoTokenResponse), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status401Unauthorized)]
public Task<IActionResult> GenerateDemoVideoToken(
    string roomname,
    CancellationToken cancellationToken)
{
    // 1. GetCurrentUserId() + GetCurrentUserRole() from BaseController
    // 2. Use roomname directly (no consultation lookup)
    // 3. Role-based grants (same logic as consultation endpoint)
    // 4. Generate token via _liveKitService.GenerateAccessToken()
    // 5. Return Ok(VideoTokenResponse { Token, WsUrl, RoomName })
}
```

## Forbidden Changes

- Do NOT modify `ILiveKitService`, `LiveKitService`, `ILiveKitApi`, or any DTO
- Do NOT modify `DependencyInjection.cs`
- Do NOT create a separate controller — all endpoints stay in `VideoCallController`
- Do NOT remove the consultation endpoint — it must remain for future use
- Do NOT skip `CancellationToken` on async methods

## Test Requirements

- `dotnet build` — 0 errors
- Swagger: 3 endpoints visible under `VideoCall` tag
- Demo endpoint: Login → Authorize → `POST /api/videocall/livekit-token/demo/test-room` → valid `{ token, wsUrl, roomName }`
