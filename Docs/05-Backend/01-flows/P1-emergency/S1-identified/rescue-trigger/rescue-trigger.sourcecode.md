# 💻 Rescue Trigger Source Code

> **State**: Implemented
> **Last Updated**: 2026-02-06
> **Commit**: `f6aca477`

## File Paths (Quick Reference)
| Component | Path |
|-----------|------|
| API Controller | `SnakeAid.Api/Controllers/SnakebiteIncidentController.cs` |
| Session Service | `SnakeAid.Service/Implements/RescueRequestSessionService.cs` |
| Incident Service | `SnakeAid.Service/Implements/SnakebiteIncidentService.cs` |
| SignalR Hub | `SnakeAid.Api/Hubs/RescuerHub.cs` |
| Timeout Service | `SnakeAid.Service/Implements/SessionTimeoutService.cs` |
| Demo Controller | `SnakeAid.Api/Controllers/RescueDemoController.cs` |

## Constants (Hardcoded)
```csharp
// SnakebiteIncidentService.cs:26 & RescueRequestSessionService.cs:36
private static readonly int[] RADIUS_PROGRESSION = { 10, 20, 30 }; // km

// SessionTimeoutService (derived from usage)
private const int SESSION_TIMEOUT_SECONDS = 60;
private const int MAX_SESSIONS = 3; // = RADIUS_PROGRESSION.Length
```

## 1. API Extensions (`SnakebiteIncidentController`)

### `POST /api/incidents/sos`
Combines incident creation and rescue triggering.
```csharp
[HttpPost("sos")]
public async Task<IActionResult> CreateSnakebiteIncident([FromBody] CreateIncidentRequest request)
{
    var userId = GetCurrentUserId();
    // 1. Create Incident
    var result = await _incidentService.CreateIncidentAsync(request, userId);
    // 2. Start Rescue (Broadcast)
    var rescueResult = await _incidentService.StartRescueAsync(result.Id);
    
    // Combine results
    result.SessionId = rescueResult.SessionId;
    // ... maps other session properties
    return StatusCode(200, ...);
}
```

### `POST /api/incidents/{incidentId}/raise-range`
Expands the search radius manually.
```csharp
[HttpPost("{incidentId}/raise-range")]
public async Task<IActionResult> RaiseSessionRange(Guid incidentId)
{
    var request = new RaiseSessionRangeRequest { IncidentId = incidentId };
    var result = await _incidentService.RaiseSessionRangeAsync(request);
    return Ok(...);
}
```

## 2. Service Interface (`ISnakebiteIncidentService`)

Extensions to the core service to support rescue mechanics.

```csharp
public interface ISnakebiteIncidentService
{
    // ... existing CRUD methods

    // Trigger rescue: Create session initial, broadcast requests
    Task<TriggerRescueResponse> TriggerRescueAsync(Guid incidentId);

    // Start rescue session for existing incident (separated from CreateIncident)
    Task<TriggerRescueResponse> StartRescueAsync(Guid incidentId);
}
```

## 3. Core Logic & Algorithms

### A. Finding Rescuers (Spatial Query)
Located in `RescueRequestSessionService.GetRescuersInRadiusAsync`.
Uses **PostGIS** to calculate distance on a sphere (WGS84).

```csharp
var rescuers = await _unitOfWork.GetRepository<RescuerProfile>()
    .CreateBaseQuery(asNoTracking: true)
    .Where(r => r.IsOnline)
    // Filter by capabilities
    .Where(r => r.Type == RescuerType.Emergency || r.Type == RescuerType.Both)
    // Spatial Filter: ST_Distance(point, point) <= meters
    .Where(r => r.LastLocation!.Distance(incidentLocation) <= radiusMeters)
    .OrderBy(r => r.LastLocation!.Distance(incidentLocation))
    .ToListAsync();

// Final check: Must be connected to SignalR Hub
var connectedRescuers = rescuers
    .Where(r => _notificationService.IsRescuerConnected(r.AccountId.ToString()))
    .ToList();
```

### B. Accepting Request (Race Condition Handling)
Located in `RescueRequestSessionService.AcceptRequestAsync`.
Uses **Serializable Transaction** (implicit via UoW) to prevent double-booking.

1.  **Validate**: Check if Request is Pending and not Expired.
2.  **Check Race**: `if (Session.Status == Completed) throw "Already Taken"`.
3.  **Update Winner**: Mark Request as `Accepted`, Session as `Completed`.
4.  **Update Losers**: Mark all other `Pending` requests in session as `Taken`.
5.  **Create Mission**: Immediately create `RescueMission` record.
6.  **Notify**: Broadcast updates to all involved rescuers.

## 4. Data Transfer Objects (DTOs)

### `CreateIncidentRequest`
Input payload for the SOS signal.

```csharp
public class CreateIncidentRequest
{
    [Required] [Range(-180, 180)]
    public double Lng { get; set; }

    [Required] [Range(-90, 90)]
    public double Lat { get; set; }
}
```

### `CreateIncidentResponse`
Unified response containing incident ID and session status.

```csharp
public class CreateIncidentResponse
{
    public Guid Id { get; set; }
    public Guid UserId { get; set; }
    public SnakebiteIncidentStatus Status { get; set; } // Default: Pending
    
    // Session Info
    public int CurrentSessionNumber { get; set; }
    public int CurrentRadiusKm { get; set; } // Default: 10
    
    // Session Details (Populated on Start)
    public Guid? SessionId { get; set; }
    public int SessionNumber { get; set; }
    public int RadiusKm { get; set; }
    public int RescuersPinged { get; set; }
}
```

## 5. Dependencies
*   **`IRescueRequestSessionService`**: Orchestrates the search logic.
*   **`ISessionTimeoutService`**: Background scheduler for the 60s timeout.
*   **`RescuerHub`**: SignalR Hub for real-time bi-directional communication.
*   **`SignalRRescueNotificationService`**: Manages connection mapping (`UserId` <-> `ConnectionId`).

## 6. Developer Tools (Demo & Monitoring)

### `RescueDemoController`
A full-stack simulation controller for testing the entire rescue flow without mobile devices.
*   **Purpose**: Debugging SignalR events, validating the "Race Condition" logic, and simulating Rescuers.
*   **Path**: `api/rescuedemo`
*   **UI**: `SnakeAid.Api/Pages/Demo/RescueDemo.cshtml` (Self-contained Razor page)

### `MonitoringController`
Health-check endpoints for the background schedulers.
*   **`GET /api/monitoring/session-timeout-status`**: Inspects the active timeout queue.
*   **`GET /api/monitoring/health/session-timeout`**: Liveness probe for the timeout service.
