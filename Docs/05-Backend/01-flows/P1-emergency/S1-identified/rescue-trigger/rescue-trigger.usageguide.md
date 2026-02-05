# 📖 Rescue Trigger Usage Guide

> **Audience**: Mobile App (Flutter) Developers
> **Purpose**: Trigger an emergency rescue and manage search range.

## 1. Trigger SOS (Create Incident)

Call this endpoint immediately when the user confirms an emergency or requests help.

### Endpoint
`POST api/incidents/sos`

### Request Header
`Authorization: Bearer <token>`

### Request Body
```json
{
  "lng": 106.660172,
  "lat": 10.762622
}
```

### Response (200 OK)
```json
{
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "locationCoordinates": {
      "latitude": 10.762622,
      "longitude": 106.660172
    },
    "status": 0,
    "currentSessionNumber": 1,
    "currentRadiusKm": 10,
    "incidentOccurredAt": "2026-02-06T12:00:00Z",
    "sessionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "sessionNumber": 1,
    "radiusKm": 10,
    "rescuersPinged": 12
  },
  "success": true,
  "message": "Snakebite Incident created and rescue session started! Broadcasting to nearby rescuers."
}
```

---

## 2. Expand Search Range

Call this if the user manually requests to widen the search or if no rescuers accept within the timeout.

### Endpoint
`POST api/incidents/{incidentId}/raise-range`

### Request Params
*   `incidentId` (Path): The GUID of the current incident.

### Response (200 OK)
```json
{
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "currentRadiusKm": 10,
    "currentSessionNumber": 2
    // ... other fields similar to Create Response
  },
  "success": true,
  "message": "Session range expanded successfully. New radius: 10km"
}
```


## 3. Rescuer App Integration (SignalR)

Rescuers must connect to the `RescuerHub` to receive requests.

### Connection
`Hub URL`: `/rescuer-hub`
`Auth`: JWT Token required.

### Client-to-Server Methods
*   **`JoinAsRescuer(string userId)`**: Registers the socket connection for receiving alerts.
*   **`AcceptRequest(Guid requestId, Guid rescuerId)`**: Attempt to accept a rescue. Setting `rescuerId` is redundant if using Token, but required by API.
*   **`RejectRequest(Guid requestId)`**: Decline the request.
*   **`UpdateLocation(string userId, double lat, double lng)`**: Keep location fresh for spatial queries.

### Server-to-Client Events
*   **`Joined`**: Confirmation of connection.
*   **`NewRescueRequest`**: A new rescue request is available for you.
    ```json
    {
      "RequestId": "...",
      "IncidentId": "...",
      "Lat": 10.762622,
      "Lng": 106.660172,
      "RadiusKm": 10,
      "TimeoutSeconds": 60
    }
    ```
*   **`RequestAccepted`**: You successfully won the mission race.
    ```json
    { "RequestId": "...", "Message": "..." }
    ```
*   **`RequestTaken`**: Another rescuer claimed this mission first.
*   **`RequestExpired`**: The session timed out before you responded.
*   **`RequestError`**: Operation failed (e.g., "Already accepted by another rescuer").
    ```json
    { "RequestId": "...", "Error": "Request has expired." }
    ```
*   **`RequestRejected`**: Confirmation of rejection.

---

## 4. Testing & Development
For testing without real mobile devices, use the built-in **Rescue Demo Dashboard**:
*   **URL**: `/demo/rescuedemo` (Requires Backend to be running)
*   **Features**:
    *   Simulate User (Connect, Create Incident, SOS).
    *   Simulate 4 Mock Rescuers (Auto-connect, Accept/Reject).
    *   Visual map of "Race Condition" logic.

## 5. Status Codes
| Code | Meaning |
|---|---|
| 200 | Success. The SOS is active. |
| 400 | Invalid request (e.g. invalid coordinates or missing profile). |
| 404 | Incident not found (for raise-range). |
| 401 | Unauthorized (User not logged in). |
