---
doc_role: baseline
module: consultation
kind: flow
doc_type: sourcecode
status: active
last_updated: 2026-04-17
owners: [backend-team]
---

# Consultation Module Source Code Map

## API Controllers

- `SnakeAid.Api/Controllers/ExpertController.cs`
  - Expert settings
  - Bulk time-slot generation
  - Expert directory, profile, reviews, time slots
  - Expert consultation history
- `SnakeAid.Api/Controllers/ConsultationScheduledController.cs`
  - Scheduled booking create
  - Member scheduled bookings
  - Expert scheduled bookings
- `SnakeAid.Api/Controllers/ConsultationInstantController.cs`
  - Emergency request create
  - Expert accept and reject
- `SnakeAid.Api/Controllers/ConsultationPaymentsController.cs`
  - Scheduled booking payment
  - Emergency request payment
  - Manual PayOS confirm
- `SnakeAid.Api/Controllers/ConsultationsController.cs`
  - Unified member consultation history
  - End consultation
  - Create review
  - Get review
- `SnakeAid.Api/Controllers/VideoCallController.cs`
  - Consultation video token
  - Demo token endpoint
  - LiveKit webhook
- `SnakeAid.Api/Controllers/AdminConsultationsController.cs`
  - Admin consultation list
  - Admin consultation detail
- `SnakeAid.Api/Controllers/MediaController.cs`
  - Consultation image upload uses `POST /api/media/upload-image`
- `SnakeAid.Api/Controllers/SnakeSpeciesController.cs`
  - Consultation support search uses `GET /api/snake-species/search`

## Services

- `SnakeAid.Service/Implements/BookingService.cs`
  - Creates scheduled bookings
  - Reads member and expert scheduled booking lists
  - Auto-completes elapsed consultations
- `SnakeAid.Service/Implements/EmergencyConsultationService.cs`
  - Creates emergency requests
  - Accepts and rejects emergency requests
  - Reserves overlapping slots on expert accept
- `SnakeAid.Service/Implements/ConsultationPaymentService.cs`
  - Wallet and PayOS payment orchestration
  - PayOS confirm and webhook processing
  - Emergency refund
  - Escrow settlement
- `SnakeAid.Service/Implements/ConsultationService.cs`
  - End consultation
  - Review create and get
  - Member, expert, and admin consultation history
- `SnakeAid.Service/Implements/ConsultationLifecycleBackgroundService.cs`
  - Expires emergency requests
  - Auto-completes elapsed consultations
  - Uses PostgreSQL advisory lock for multi-replica safety

## Hubs

- `SnakeAid.Api/Hubs/ExpertHub.cs`
  - `JoinAsExpert`
  - `JoinAsMember`
  - `JoinEmergencyRequestRoom`
- `SnakeAid.Service/Hubs/ConsultationHub.cs`
  - Query-string gated consultation connection
  - Auto-joins group `consultation:{consultationId}`
  - `ReceiveMessage`
  - `Signal`

## Core DTOs Used By Public APIs

- `SnakeAid.Core/Requests/Consultation/CreateConsultationBookingRequest.cs`
- `SnakeAid.Core/Requests/Consultation/CreateEmergencyConsultationRequest.cs`
- `SnakeAid.Core/Requests/Consultation/ProcessConsultationPaymentRequest.cs`
- `SnakeAid.Core/Requests/Consultation/CreateConsultationReviewRequest.cs`
- `SnakeAid.Core/Requests/Consultation/MyConsultationsQueryRequest.cs`
- `SnakeAid.Core/Requests/Consultation/AdminConsultationsQueryRequest.cs`
- `SnakeAid.Core/Responses/Consultation/ConsultationBookingResponse.cs`
- `SnakeAid.Core/Responses/Consultation/EmergencyConsultationRequestResponse.cs`
- `SnakeAid.Core/Responses/Consultation/ConsultationPaymentResponse.cs`
- `SnakeAid.Core/Responses/Consultation/MyConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Core/Responses/Consultation/AdminConsultationResponse.cs`
- `SnakeAid.Core/Responses/LiveKit/VideoTokenResponse.cs`

## Mapping And Realtime Support Types

- `SnakeAid.Core/Mappings/AdminConsultationMapper.cs`
  - base and enrichment mappings for admin consultation responses
- `SnakeAid.Core/Constants/ConsultationRealtimeEvents.cs`
  - realtime event names and end-call reason values

## Domain Enums Worth Reading Before Editing Contracts

- `SnakeAid.Core/Domains/Consultation.cs`
  - `ConsultationStatus`
  - `ConsultationType`
- `SnakeAid.Core/Domains/ConsultationBooking.cs`
  - `BookingStatus`
- `SnakeAid.Core/Domains/ConsultationPingRequest.cs`
  - `ConsultationPingStatus`

## Notes

- The usage guide is the canonical contract document.
- This file exists to show where contract behavior is implemented in code.
- Admin list and admin detail both use `AdminConsultationResponse`.
- Emergency admin price is resolved from `Transaction`, not from `ConsultationPingRequest`.
