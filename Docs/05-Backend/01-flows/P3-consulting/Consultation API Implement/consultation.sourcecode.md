---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-06
owners: [backend-team]
---

# Consultation Module Source Code

## Entities and Schema

- `Consultation` ([Consultation.cs](SnakeAid.Backend/SnakeAid.Core/Domains/Consultation.cs)): The core session record (`Id`, `CallerId`, `CalleeId`, `StartTime`, `EndTime`, `Status`).
- `ConsultationBooking` ([ConsultationBooking.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ConsultationBooking.cs)): Connects a scheduled booking to a user and time slot.
- `ConsultationPingRequest` ([ConsultationPingRequest.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ConsultationPingRequest.cs)): Emergency consultation request lifecycle (`PendingExpertResponse`, `AcceptedByExpert`, `DeclinedByExpert`, `Expired`).
- `ExpertProfile` ([ExpertProfile.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ExpertProfile.cs)): Expert metadata (`IsOnline`, `ConsultationFee`, `Rating`).
- `ExpertTimeSlot` ([ExpertTimeSlot.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ExpertTimeSlot.cs)): Discrete time block for booking (`StartTime`, `EndTime`, `Status`, `Version` for optimistic concurrency).

## Public API Surface (Endpoints)

### Expert Directory & Availability

_Implemented in [ExpertController.cs](SnakeAid.Backend/SnakeAid.Api/Controllers/ExpertController.cs) and [ExpertService.cs](SnakeAid.Backend/SnakeAid.Service/Implements/ExpertService.cs)_

- `PUT /api/v1/experts/me/settings`: Update expert settings (Biography, ConsultationFee).
- `POST /api/v1/experts/me/time-slots/bulk`: Setup weekly working hours (deduplicates overlapping/generated duplicates inside payload and DB-existing slots).
- `GET /api/v1/experts`: Get list of active experts with pagination + optional filters/sorts (`specialization`, `isOnline`, `sortBy`, `sortOrder`).
- `GET /api/v1/experts/{expertId}`: Get expert profile details.
- `GET /api/v1/experts/{expertId}/reviews`: Get paginated **consultation** reviews for an expert.
- `GET /api/v1/experts/{expertId}/time-slots`: Get available future time slots for an expert.

### Scheduled Consultation

_Implemented in `ConsultationBookingsController`, `ConsultationsController`, `BookingService`, `ConsultationService`_

- `POST /api/v1/consultation-bookings`: Reserve an available expert slot and create a pending booking + scheduled consultation room.
- `GET /api/v1/consultation-bookings/my-bookings`: Get booking history of current user.
- `POST /api/v1/consultations/{consultationId}/end`: End consultation and update booking/slot status.
- `POST /api/v1/consultations/{consultationId}/reviews`: Submit consultation review (user -> expert).
- `POST /api/videocall/livekit-token/{consultationId}`: Generate LiveKit token only if caller is consultation participant (or admin), using persisted `RoomId`.

### Emergency Consultation

_Implemented in `ConsultationsController`, `EmergencyConsultationService`, `ExpertHub`, `SignalRExpertEmergencyNotificationService`_

- `POST /api/v1/consultations/emergency`: User creates an immediate consultation request for a selected expert (`ExpertId` required).
- `POST /api/v1/consultations/emergency-requests/{requestId}/accept`: Targeted expert accepts request, creates `Ongoing` emergency consultation.
- `POST /api/v1/consultations/emergency-requests/{requestId}/reject`: Targeted expert rejects request.

## Hubs

- `ExpertHub` (`/hubs/expert`): Presence + emergency realtime hub for consultation domain.
  - `JoinAsExpert` (role `Expert`): marks expert online and binds connection.
  - `JoinAsMember` (role `User`): subscribes member to presence updates and sends `OnlineExpertsSnapshot`.
  - `JoinEmergencyRequestRoom(requestId)` (role `User`, owner-only): subscribes requester to request-specific room.
  - `OnDisconnectedAsync`: marks expert offline and unbinds connection.
- Event contracts:
  - `OnlineExpertsSnapshot`: initial expert online snapshot for member.
  - `ExpertPresenceChanged`: online/offline delta event.
  - `EmergencyConsultationRequest`: directed push to selected expert.
  - `EmergencyRequestStatusChanged`: request-room status update for requester after expert accept/reject.

## Cross-Cutting Concerns

- **Concurrency**: Optimistic concurrency via `Version` field on `ExpertTimeSlot` is used by booking flows, and a unique DB index on (`ExpertId`, `StartTime`, `EndTime`) prevents duplicate slot rows under concurrent bulk setup requests.
- **Slot Paradox Guard**: On emergency accept, overlapping `ExpertTimeSlot` rows in the next 30-minute window are transitioned from `Available` to `Reserved` in the same transaction as consultation creation.
- **Time Standard**: `POST /api/v1/experts/me/time-slots/bulk` requires `weekStartDate` in UTC (`...Z`). Generated `ExpertTimeSlot` timestamps are persisted in UTC.
- **Authentication**: Custom roles required for Expert-facing vs User-facing endpoints. User contexts resolved via Claims.
- **Presence-backed Availability**: realtime presence decisions are SignalR-first (`ConnectedExperts`), while `ExpertProfile.IsOnline` is maintained as eventual-consistency state.
- **Emergency Request Room Naming**: request room uses deterministic key `consultation:emergency:request:{requestId:N}`.
- **Safety Switch (Hardcoded)**: `EnablePresenceSelfHealing = false` by default to avoid accidental DB healing when environments share the same database.
- **Response Contract (Culture)**: Controllers return success payloads via `ApiResponseBuilder` (`ApiResponse<T>` envelope). Error responses are produced by throwing typed `ApiException` and handled centrally by `ApiExceptionHandlerMiddleware`.
- **Error Semantics**:
  - Non-UTC `weekStartDate` is rejected with `ValidationException` (HTTP `422`).
  - Concurrent duplicate slot insertions are rejected with `ConflictException` (HTTP `409`).
  - Concurrent slot reservation in `POST /api/v1/consultation-bookings` is translated to `ConflictException` (HTTP `409`).
  - Emergency accept/reject on non-pending request is rejected with `ConflictException` (HTTP `409`).
