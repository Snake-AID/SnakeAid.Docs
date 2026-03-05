---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-05
owners: [backend-team]
---

# Consultation Module Source Code

## Entities and Schema

- `Consultation` ([Consultation.cs](SnakeAid.Backend/SnakeAid.Core/Domains/Consultation.cs)): The core session record (`Id`, `CallerId`, `CalleeId`, `StartTime`, `EndTime`, `Status`).
- `ConsultationBooking` ([ConsultationBooking.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ConsultationBooking.cs)): Connects a scheduled booking to a user and time slot.
- `ExpertProfile` ([ExpertProfile.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ExpertProfile.cs)): Expert metadata (`IsOnline`, `ConsultationFee`, `Rating`).
- `ExpertTimeSlot` ([ExpertTimeSlot.cs](SnakeAid.Backend/SnakeAid.Core/Domains/ExpertTimeSlot.cs)): Discrete time block for booking (`StartTime`, `EndTime`, `Status`, `Version` for optimistic concurrency).

## Public API Surface (Endpoints)

### Expert Directory & Availability

_Implemented in [ExpertController.cs](SnakeAid.Backend/SnakeAid.Api/Controllers/ExpertController.cs) and [ExpertService.cs](SnakeAid.Backend/SnakeAid.Service/Implements/ExpertService.cs)_

- `PUT /api/v1/experts/me/settings`: Update expert settings (Biography, ConsultationFee).
- `POST /api/v1/experts/me/time-slots/bulk`: Setup weekly working hours (deduplicates overlapping/generated duplicates inside payload and DB-existing slots).
- `GET /api/v1/experts`: Get list of active experts with pagination.
- `GET /api/v1/experts/{expertId}`: Get expert profile details.
- `GET /api/v1/experts/{expertId}/reviews`: Get paginated **consultation** reviews for an expert.
- `GET /api/v1/experts/{expertId}/time-slots`: Get available future time slots for an expert.

## Hubs

_None currently implemented._

## Cross-Cutting Concerns

- **Concurrency**: Optimistic concurrency via `Version` field on `ExpertTimeSlot` is used by booking flows, and a unique DB index on (`ExpertId`, `StartTime`, `EndTime`) prevents duplicate slot rows under concurrent bulk setup requests.
- **Time Standard**: `POST /api/v1/experts/me/time-slots/bulk` requires `weekStartDate` in UTC (`...Z`). Generated `ExpertTimeSlot` timestamps are persisted in UTC.
- **Authentication**: Custom roles required for Expert-facing vs User-facing endpoints. User contexts resolved via Claims.
- **Error Semantics**:
  - Non-UTC `weekStartDate` is rejected with `ValidationException` (HTTP `422`).
  - Concurrent duplicate slot insertions are rejected with `ConflictException` (HTTP `409`).
