---
doc_role: operation
operation_id: 03-FEAT-scheduled-consultation
type: FEAT
status: done
created_at: 2026-03-05
affects:
  - Api/Controllers/ConsultationBookingsController.cs
  - Api/Controllers/ConsultationsController.cs
  - Api/Controllers/VideoCallController.cs
  - Service/Implements/BookingService.cs
  - Service/Implements/ConsultationService.cs
  - Service/Interfaces/IBookingService.cs
  - Service/Interfaces/IConsultationService.cs
  - Core/Domains/ConsultationBooking.cs
  - Core/Domains/Consultation.cs
  - Core/Requests/Consultation/CreateConsultationBookingRequest.cs
  - Core/Requests/Consultation/CreateConsultationReviewRequest.cs
  - Core/Responses/Consultation/ConsultationBookingResponse.cs
  - Repository/Migrations/*AddProblemDescriptionToConsultationBooking*
  - Tests/Integration/ScheduledConsultationIntegrationTests.cs
  - Tests/Unit/BookingServiceConcurrencyTests.cs
---

# Plan: Scheduled Consultation Flow

## 1. As-Is

The Expert Directory and availability slots are functional (via `01-INIT`). However, the ability for a user to book an available slot, transition into the consultation room, and submit a review does not exist.

## 2. Gap Analysis

- Users need an endpoint to book an `ExpertTimeSlot` and create a pending `ConsultationBooking`.
- Users need to retrieve their booking history.
- The `ConsultationBooking` schema lacks a `ProblemDescription` field for user notes (wireframe requirement).
- Video token generation relies on LiveKit Cloud but must be explicitly routed.
- Endpoints to finish a consultation and submit reviews are missing.

## 3. To-Be Design

Implement the following:

- Schema update: Add `ProblemDescription` to `ConsultationBooking`.
- `POST /api/v1/consultation-bookings`: Initiate a booking with optimistic concurrency on the time slot.
- `GET /api/v1/consultation-bookings/my-bookings`: Retrieve user bookings.
- Ensure `POST /api/videocall/livekit-token/{consultationId}` validates access and provides a WebRTC token.
- `POST /api/v1/consultations/{consultationId}/end`: Mark consultation complete.
- `POST /api/v1/consultations/{consultationId}/reviews`: Allow users to rate the consultation.

## 4. Impacted Components

- **Controllers**: `ConsultationBookingsController`, `ConsultationsController`
- **Services**: `BookingService`, `ConsultationService`
- **Entities**: Update `ConsultationBooking` to include `ProblemDescription`.

## 5. Risks & Constraints

- Potential race conditions when booking: EF Core `Version` must be enforced to prevent double-booking of an `ExpertTimeSlot`.
- Video call tokens should only be granted to authorized participants of the specific `Consultation`.

## 6. Validation Plan

- Unit test to ensure `DbUpdateConcurrencyException` is handled returning an HTTP 409 Conflict.
- Verify JWT tokens returned by LiveKit integration only contain the correct Room ID.
