---
doc_role: baseline
module: emergency-consultation
kind: flow
doc_type: usageguide
status: active
last_updated: 2026-03-18
api_version: v1
owners: [backend-team]
---

# Emergency Consultation API Usage Guide

## Overview

The Emergency Consultation feature enables users to request immediate consultations with selected experts. This real-time service allows users to bypass scheduled appointments and connect directly with available experts for urgent medical advice.

### Key Capabilities
- Real-time expert availability tracking
- Direct emergency consultation requests to specific experts
- Automatic slot conflict resolution
- Real-time status updates via WebSocket
- Secure payment escrow for emergency consultations

### Prerequisites
- Valid JWT authentication token
- User role: `User` for requesting consultations, `Expert` for accepting/rejecting
- Active expert account for consultation targets

---

## Authentication & Authorization

All endpoints require JWT Bearer token authentication:

```
Authorization: Bearer {{TOKEN}}
```

### Role Requirements
- **Users** (`User` role): Can request emergency consultations and join presence rooms
- **Experts** (`Expert` role): Can accept/reject emergency requests and manage presence

---

## Endpoints

### `POST /api/consultations/emergency-requests`

Create a new emergency consultation request targeting a specific expert.

**Authentication**: Required (User role)

**Required Permissions**: User role

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |
| Content-Type | string | Yes | application/json |

**Request Body**:
```json
{
  "expertId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Request Body Schema**:
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| expertId | string (UUID) | Yes | Valid expert ID | ID of the expert to request consultation from |

**Success Response** (200 OK):
```json
{
  "data": {
    "requestId": "550e8400-e29b-41d4-a716-446655440001",
    "requesterId": "550e8400-e29b-41d4-a716-446655440002",
    "expertId": "550e8400-e29b-41d4-a716-446655440000",
    "status": "PendingPayment",
    "requestedAt": "2026-03-18T09:30:00Z",
    "expiresAt": null,
    "respondedAt": null,
    "consultationId": null,
    "roomId": null
  }
}
```

**Error Responses**: See Error Catalog section.

---

### `POST /api/consultations/emergency-requests/{requestId}/accept`

Expert accepts an emergency consultation request. Automatically reserves conflicting time slots and creates an ongoing consultation.

**Authentication**: Required (Expert role)

**Required Permissions**: Expert role (must be the targeted expert)

**Path Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| requestId | string (UUID) | Yes | Emergency request identifier |

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Request Body**: None

**Success Response** (200 OK):
```json
{
  "data": {
    "requestId": "550e8400-e29b-41d4-a716-446655440001",
    "requesterId": "550e8400-e29b-41d4-a716-446655440002",
    "expertId": "550e8400-e29b-41d4-a716-446655440000",
    "status": "AcceptedByExpert",
    "requestedAt": "2026-03-18T09:30:00Z",
    "expiresAt": null,
    "respondedAt": "2026-03-18T09:31:00Z",
    "consultationId": "550e8400-e29b-41d4-a716-446655440003",
    "roomId": "consultation-550e8400-e29b-41d4-a716-446655440003"
  }
}
```

**Error Responses**: See Error Catalog section.

---

### `POST /api/consultations/emergency-requests/{requestId}/reject`

Expert rejects an emergency consultation request. Automatically refunds escrow payment.

**Authentication**: Required (Expert role)

**Required Permissions**: Expert role (must be the targeted expert)

**Path Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| requestId | string (UUID) | Yes | Emergency request identifier |

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Request Body**: None

**Success Response** (200 OK):
```json
{
  "data": {
    "requestId": "550e8400-e29b-41d4-a716-446655440001",
    "requesterId": "550e8400-e29b-41d4-a716-446655440002",
    "expertId": "550e8400-e29b-41d4-a716-446655440000",
    "status": "DeclinedByExpert",
    "requestedAt": "2026-03-18T09:30:00Z",
    "expiresAt": null,
    "respondedAt": "2026-03-18T09:31:00Z",
    "consultationId": null,
    "roomId": null
  }
}
```

**Error Responses**: See Error Catalog section.

---

### `GET /api/experts`

Get paginated list of experts with filtering and sorting options for emergency consultation selection.

**Authentication**: Optional

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | No | Bearer {token} (optional for public access) |

**Query Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| specialization | string | No | null | Filter by specialization (partial match) |
| isOnline | boolean | No | null | Filter by online status |
| sortBy | string | No | rating | Sort field: isOnline, rating, consultationFee |
| sortOrder | string | No | desc | Sort order: asc, desc |
| pageNumber | int | No | 1 | Page number (1-based) |
| pageSize | int | No | 10 | Items per page (max 100) |

**Success Response** (200 OK):
```json
{
  "data": {
    "items": [
      {
        "accountId": "550e8400-e29b-41d4-a716-446655440000",
        "fullName": "Dr. Nguyen Van A",
        "avatarUrl": "https://example.com/avatar.jpg",
        "isOnline": true,
        "rating": 4.8,
        "ratingCount": 150,
        "consultationFee": 150000,
        "emergencyConsultationFee": 200000,
        "specializations": [
          {
            "name": "Rắn Độc",
            "description": "Venomous snake bites treatment"
          }
        ],
        "bio": "Experienced toxicologist specializing in snake bite emergencies"
      }
    ],
    "meta": {
      "pageNumber": 1,
      "pageSize": 10,
      "totalCount": 25,
      "totalPages": 3,
      "hasNextPage": true,
      "hasPreviousPage": false
    }
  }
}
```

---

## Real-time Communication (SignalR)

### Hub Connection
Connect to the Expert Hub for real-time updates:

**Hub URL**: `/hubs/expert`

**Authentication**: Required (JWT token in query string or headers)

### Client Methods

#### `JoinAsMember()`
Subscribe to expert presence updates. Call after connecting.

**Parameters**: None

**Response Events**:
- `OnlineExpertsSnapshot`: Initial list of online experts
- `ExpertPresenceChanged`: Delta updates when experts connect/disconnect

#### `JoinEmergencyRequestRoom(requestId)`
Subscribe to status updates for a specific emergency request. Only the requester can join.

**Parameters**:
- `requestId`: string (UUID) - The emergency request ID

**Response Events**:
- `EmergencyRequestStatusChanged`: Status updates for the request

### Server Events

#### `OnlineExpertsSnapshot`
Initial snapshot of currently online experts.

```json
{
  "onlineExpertIds": [
    "550e8400-e29b-41d4-a716-446655440000",
    "550e8400-e29b-41d4-a716-446655440004"
  ],
  "serverTimeUtc": "2026-03-18T09:30:00Z"
}
```

#### `ExpertPresenceChanged`
Real-time presence updates.

```json
{
  "expertId": "550e8400-e29b-41d4-a716-446655440000",
  "isOnline": true,
  "changedAtUtc": "2026-03-18T09:30:15Z"
}
```

#### `EmergencyRequestStatusChanged`
Emergency request status updates.

```json
{
  "requestId": "550e8400-e29b-41d4-a716-446655440001",
  "status": "AcceptedByExpert",
  "consultationId": "550e8400-e29b-41d4-a716-446655440003",
  "roomId": "consultation-550e8400-e29b-41d4-a716-446655440003",
  "updatedAtUtc": "2026-03-18T09:31:00Z"
}
```

---

## Data Models

### EmergencyConsultationRequestResponse
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| requestId | string (UUID) | Yes | Unique request identifier |
| requesterId | string (UUID) | Yes | User who made the request |
| expertId | string (UUID) | Yes | Targeted expert |
| status | string (enum) | Yes | Request status (see below) |
| requestedAt | string (datetime) | Yes | ISO 8601 UTC timestamp |
| expiresAt | string (datetime) | No | ISO 8601 UTC timestamp |
| respondedAt | string (datetime) | No | ISO 8601 UTC timestamp |
| consultationId | string (UUID) | No | Created consultation ID (if accepted) |
| roomId | string | No | Video call room ID (if accepted) |

### Request Status Enum
- `PendingPayment`: Initial state, waiting for payment
- `PendingExpertResponse`: Payment confirmed, waiting for expert
- `AcceptedByExpert`: Expert accepted, consultation created
- `DeclinedByExpert`: Expert declined
- `RescuerCancelled`: User cancelled
- `Expired`: Request expired

### ExpertProfileResponse
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| accountId | string (UUID) | Yes | Expert's account ID |
| fullName | string | Yes | Expert's full name |
| avatarUrl | string | No | Profile picture URL |
| isOnline | boolean | Yes | Current online status |
| rating | decimal | Yes | Average rating (0-5) |
| ratingCount | int | Yes | Number of ratings |
| consultationFee | int | Yes | Regular consultation fee (VND) |
| emergencyConsultationFee | int | No | Emergency consultation fee (VND) |
| specializations | array | Yes | List of specializations |
| bio | string | No | Professional biography |

---

## Error Catalog

| Status Code | Error Code | Message | Description | Resolution |
|-------------|------------|---------|-------------|------------|
| 400 | INVALID_INPUT | Invalid request format | Request body validation failed | Check request schema and required fields |
| 401 | UNAUTHORIZED | Authentication required | Missing or invalid JWT token | Provide valid Bearer token |
| 403 | FORBIDDEN | Insufficient permissions | User lacks required role or ownership | Check user role and request ownership |
| 404 | NOT_FOUND | Resource not found | Emergency request or expert not found | Verify request/expert ID exists |
| 409 | CONFLICT | Resource conflict | Duplicate request or invalid state transition | Check current request status |
| 500 | INTERNAL_ERROR | Internal server error | Unexpected server error | Contact support |

---

## Rate Limiting

- **Authenticated requests**: 100 requests per minute per user
- **Anonymous expert directory**: 60 requests per minute per IP
- **SignalR connections**: 10 concurrent connections per user

Rate limit headers returned:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1640995200
```

---

## Examples

### Example: Request Emergency Consultation

**Request**:
```bash
curl -X POST https://api.snakeaid.com/api/consultations/emergency-requests \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "expertId": "550e8400-e29b-41d4-a716-446655440000"
  }'
```

**Response**:
```json
{
  "data": {
    "requestId": "550e8400-e29b-41d4-a716-446655440001",
    "requesterId": "550e8400-e29b-41d4-a716-446655440002",
    "expertId": "550e8400-e29b-41d4-a716-446655440000",
    "status": "PendingPayment",
    "requestedAt": "2026-03-18T09:30:00Z"
  }
}
```

### Example: Browse Online Experts

**Request**:
```bash
curl -X GET "https://api.snakeaid.com/api/experts?isOnline=true&sortBy=rating&sortOrder=desc&pageSize=5" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response**:
```json
{
  "data": {
    "items": [
      {
        "accountId": "550e8400-e29b-41d4-a716-446655440000",
        "fullName": "Dr. Nguyen Van A",
        "isOnline": true,
        "rating": 4.9,
        "consultationFee": 150000,
        "emergencyConsultationFee": 200000,
        "specializations": [{"name": "Rắn Độc"}]
      }
    ],
    "meta": {
      "pageNumber": 1,
      "pageSize": 5,
      "totalCount": 12,
      "totalPages": 3
    }
  }
}
```

---

## Integration Notes

1. **Real-time Updates**: Always establish SignalR connection before requesting consultations to receive status updates
2. **Payment Flow**: Emergency requests require payment confirmation before expert notification
3. **Slot Conflicts**: Accepted requests automatically reserve overlapping time slots for 30 minutes
4. **Presence Tracking**: Expert online status updates in real-time via SignalR events
5. **Room Management**: Use `JoinEmergencyRequestRoom` to track specific request status
6. **Error Handling**: Implement exponential backoff for failed SignalR reconnections

---

## Changelog

### v1.0.0 (2026-03-18)
- Initial release of Emergency Consultation API
- Real-time presence tracking via SignalR
- Automatic slot conflict resolution
- Enhanced expert directory with filtering/sorting