---
doc_role: operation
operation_id: 06-STAB-mobile-readiness-gap-closure
type: STAB
status: implemented
created_at: 2026-03-08
last_updated: 2026-03-08
---

# Source Code: Mobile Readiness Gap Closure for Operations 01, 03, 04

## 1. Current Intent

Operation 06 is a stabilization pass over existing consultation flows. It does not introduce a new user journey. It closes backend gaps left by:

- `Operation 01`: expert pricing/settings and profile completeness
- `Operation 03`: scheduled consultation pre-payment and settlement
- `Operation 04`: emergency consultation pre-payment, refund, expiry, and settlement

Operation 05 remains the owner of in-room chat and consultation-scoped signaling beyond the emergency request room.

## 2. Code Areas Touched

### Domain / DTO

- `SnakeAid.Core/Domains/ExpertProfile.cs`
  - adds `EmergencyConsultationFee`
- `SnakeAid.Core/Domains/ConsultationPingRequest.cs`
  - emergency state machine now includes `PendingPayment`
- `SnakeAid.Core/Requests/Expert/ExpertSettingsRequest.cs`
  - supports `ScheduledConsultationFee` and `EmergencyConsultationFee`
- `SnakeAid.Core/Responses/Expert/ExpertProfileResponse.cs`
  - exposes:
    - `ScheduledConsultationFee`
    - `EmergencyConsultationFee`
    - `TotalConsultations`
    - `AverageResponseTimeMinutes`
    - `SuccessRate`
- `SnakeAid.Core/Requests/Consultation/ProcessConsultationPaymentRequest.cs`
  - payment request contract for consultation payment APIs
- `SnakeAid.Core/Responses/Consultation/ConsultationPaymentResponse.cs`
  - payment result contract for escrowed consultation flows
- `SnakeAid.Core/Responses/Consultation/ConsultationBookingResponse.cs`
  - now exposes `UserName` for expert-side scheduled booking cards

### Services

- `SnakeAid.Service/Implements/ExpertService.cs`
  - updates expert settings with dual pricing
  - maps profile statistics for mobile
- `SnakeAid.Service/Implements/BookingService.cs`
  - adds scheduled auto-completion by slot end
  - adds expert-side scheduled booking listing for confirmed/completed bookings
- `SnakeAid.Service/Implements/ConsultationService.cs`
  - triggers escrow settlement on explicit consultation end
- `SnakeAid.Service/Implements/EmergencyConsultationService.cs`
  - emergency request creation now starts at `PendingPayment`
  - reject triggers refund through consultation payment service
- `SnakeAid.Service/Implements/ConsultationPaymentService.cs`
  - new service for:
    - scheduled payment
    - emergency payment
    - escrow movement
    - refund on reject / expiry
    - settlement to expert on completion
- `SnakeAid.Service/Implements/ConsultationLifecycleBackgroundService.cs`
  - background sweep for:
    - expiring pending emergency requests
    - auto-completing elapsed scheduled consultations

### Interfaces

- `SnakeAid.Service/Interfaces/IConsultationPaymentService.cs`
- `SnakeAid.Service/Interfaces/IBookingService.cs`
  - adds `AutoCompleteElapsedScheduledConsultationsAsync`

### Controllers

- `SnakeAid.Api/Controllers/ConsultationPaymentsController.cs`
  - new payment endpoints for consultation flows
- `SnakeAid.Api/Controllers/ConsultationBookingsController.cs`
  - adds `GET /api/experts/me/consultation-bookings`
- `SnakeAid.Api/Program.cs`
  - registers `ConsultationLifecycleBackgroundService`

### Persistence

- `SnakeAid.Repository/Migrations/20260306203521_AddEmergencyConsultationFeeToExpertProfile.cs`
- `SnakeAid.Repository/Migrations/SnakeAidDbContextModelSnapshot.cs`

### Tests

- `SnakeAid.Tests/Integration/ConsultationPaymentIntegrationTests.cs`
  - new integration tests for escrow/refund/settlement
- existing consultation integration/unit tests were updated to reflect the new constructors and state flow
- `SnakeAid.Tests/Integration/ConsultationBookingsControllerIntegrationTests.cs`
  - verifies expert can load scheduled bookings with `consultationId`, `roomId`, and `UserName`

## 3. Current State Machine

### Scheduled Consultation Money Flow

1. `POST /api/consultation-bookings`
   - creates booking with `BookingStatus.PendingPayment`
2. `POST /api/consultation-bookings/{bookingId}/payments`
   - accepts `WalletBalance`
   - debits member wallet
   - credits system wallet (`SApay escrow`)
   - creates `TransactionType.ConsultationPayment`
   - moves booking to `BookingStatus.Confirmed`
3. Expert inbox visibility:
   - expert loads scheduled consultations through `GET /api/experts/me/consultation-bookings`
   - endpoint currently returns only `Confirmed` and `Completed` bookings
4. Completion trigger:
   - slot end via `ConsultationLifecycleBackgroundService`
   - or explicit `POST /api/consultations/{consultationId}/end`
5. Settlement:
   - escrow transferred to expert
   - creates `TransactionType.ExpertPayout`
   - settlement is idempotent by checking existing payout transaction

### Emergency Consultation Money Flow

1. `POST /api/consultations/emergency-requests`
   - creates `ConsultationPingRequest`
   - initial state = `PendingPayment`
   - no realtime push to expert yet
2. `POST /api/consultations/emergency-requests/{requestId}/payments`
   - accepts `WalletBalance`
   - debits member wallet
   - credits system wallet (`SApay escrow`)
   - creates `TransactionType.ConsultationPayment`
   - transitions request to `PendingExpertResponse`
   - sets `ExpiresAt`
   - pushes `EmergencyConsultationRequest` to selected expert
3. Expert acts:
   - `accept` -> consultation created, escrow remains locked
   - `reject` -> refund immediately to member wallet, creates `TransactionType.ConsultationRefund`
4. Backend expiry:
   - lifecycle background service marks timed-out request as `Expired`
   - refund immediately to member wallet
   - pushes `EmergencyRequestStatusChanged`
5. Completion:
   - explicit end consultation triggers settlement to expert
   - payout is idempotent

## 4. Public API Surface Added by Operation 06

### Consultation Payment Endpoints

- `POST /api/consultation-bookings/{bookingId}/payments`
- `POST /api/consultations/emergency-requests/{requestId}/payments`

### Scheduled Expert Inbox Endpoint

- `GET /api/experts/me/consultation-bookings`
  - returns scheduled bookings for the current expert
  - current filter scope: `Confirmed` and `Completed`
  - response now includes `UserName`, `consultationId`, `roomId`, and slot timing
  - current implementation is REST pull only; no scheduled-consultation SignalR inbox was introduced in Operation 06

Current request body:

```json
{
  "paymentMethod": "WalletBalance"
}
```

Current response shape:

- `referenceId`
- `referenceType`
- `transactionId`
- `amount`
- `currency`
- `paymentMethod`
- `status`
- `userWalletBalanceAfter`
- `systemWalletBalanceAfter`
- `paidAtUtc`

## 5. Expert Profile Contract After Operation 06

`GET /api/experts` and `GET /api/experts/{id}` now surface enough mobile-facing pricing/stat fields for Operation 01 closure:

- `ConsultationFee`
  - still kept for backward compatibility
- `ScheduledConsultationFee`
- `EmergencyConsultationFee`
- `TotalConsultations`
- `AverageResponseTimeMinutes`
- `SuccessRate`

Current semantics:

- `IsVerified` remains deferred for MVP and is not fully implemented
- `SuccessRate` is calculated from completed consultations over all consultations for that expert
- `AverageResponseTimeMinutes` is calculated from responded emergency requests

## 6. Background Automation Introduced

`ConsultationLifecycleBackgroundService` runs every 30 seconds and performs two jobs:

- expires emergency requests that are still `PendingExpertResponse` and passed `ExpiresAt`
- auto-completes scheduled consultations whose slot end time has already passed

This is the server-side enforcement layer added in Operation 06 to reduce dependence on mobile-local countdown logic.

## 7. Transaction Semantics in Current Code

Consultation money movement currently uses existing wallet + transaction primitives:

- `TransactionType.ConsultationPayment`
  - member -> system escrow
- `TransactionType.ConsultationRefund`
  - system escrow -> member
- `TransactionType.ExpertPayout`
  - system escrow -> expert
- `TransactionType.WalletTopup`
  - system-side incoming mirror record
- `TransactionType.WalletWithdraw`
  - system-side outgoing mirror record

The system wallet account is still hardcoded by ID in the current implementation, same style as existing wallet/payment code.

## 8. Limits / Deferred Items

These points are intentionally not solved by Operation 06:

- `IsVerified` real business semantics
- consultation PayOS / external gateway orchestration
- multi-method payment beyond `WalletBalance`
- in-room chat
- in-room consultation signaling beyond emergency request room
- expert no-show / attendance dispute handling

## 9. Hardcoded Runtime Values

The current implementation still contains several runtime values hardcoded in source code instead of `appsettings`, `IOptions`, or database-backed system settings:

- `ConsultationPaymentService`
  - system wallet account id is hardcoded
  - emergency request TTL is hardcoded to `2 minutes`
- `ConsultationLifecycleBackgroundService`
  - background polling interval is hardcoded to `30 seconds`

These values are part of the current implementation state and should be treated as technical debt, not as finalized configuration strategy.

## 10. Current Wiring

The code path after Operation 06 is wired as follows:

- `ConsultationPaymentsController`
  - calls `IConsultationPaymentService`
- `ConsultationPaymentService`
  - owns consultation payment, escrow, refund, and settlement logic
  - pushes emergency request to expert only after emergency payment succeeds
- `EmergencyConsultationService`
  - creates request in `PendingPayment`
  - delegates refund on reject to `IConsultationPaymentService`
- `ConsultationService`
  - delegates settlement on explicit end to `IConsultationPaymentService`
- `BookingService`
  - delegates slot-end settlement path through auto-complete flow
- `ConsultationLifecycleBackgroundService`
  - calls:
    - `ExpireEmergencyRequestsAsync`
    - `AutoCompleteElapsedScheduledConsultationsAsync`
- `Program.cs`
  - registers `ConsultationLifecycleBackgroundService` as hosted service

## 11. Known Technical Limits

There are several implementation limits that are important for reviewers and future maintainers:

- expiry and scheduled auto-completion are polling-based, not scheduler-precise
  - effective trigger delay can drift by up to one polling interval
- only `WalletBalance` is supported as a working consultation payment method
- consultation-specific payment status query endpoint is still missing
- consultation-specific completion/payment summary contract is still missing
- system wallet ownership is tied to a hardcoded account id
- timing values are not externally configurable yet
- settlement/refund logic currently assumes a single system escrow wallet model
- `IsVerified` remains intentionally unresolved in MVP

## 12. Coverage Status

### Covered by Current Code

- `Operation 01`
  - dual pricing for expert settings/profile
  - profile statistics contract for mobile:
    - `TotalConsultations`
    - `AverageResponseTimeMinutes`
    - `SuccessRate`
- `Operation 03`
  - scheduled consultation pre-payment with `WalletBalance`
  - escrow creation after payment
  - booking transition from `PendingPayment` to `Confirmed`
  - expert-side scheduled booking list so expert app can discover room-entry candidates
  - consultation settlement to expert on completion
  - auto-completion by slot end
- `Operation 04`
  - emergency consultation pre-payment with `WalletBalance`
  - request activation only after payment success
  - backend-driven expiry
  - refund on `Rejected`
  - refund on `Expired`
  - settlement to expert only after completion

### Partially Covered / Still Open

- payment method selection is only implemented for `WalletBalance`
- consultation-specific payment status query endpoint is not implemented yet
- completion/payment summary payload for mobile completion screens is not implemented as a separate contract yet
- `IsVerified` remains deferred from MVP by design
- external gateway orchestration for consultation payment is not implemented
- Operation 05 scope remains untouched:
  - chat
  - in-room signaling
  - expert side room tools

## 13. Verification Snapshot

Current verification status after implementation:

- `dotnet build SnakeAid.Backend.sln` succeeds
- targeted consultation test suite passes:
  - `ConsultationPaymentIntegrationTests`
  - `ScheduledConsultationIntegrationTests`
  - `EmergencyConsultationIntegrationTests`
  - `ExpertServiceTests`
  - `BookingServiceConcurrencyTests`

## 14. Review Focus

If reviewing Operation 06 code, the highest-risk files are:

1. `ConsultationPaymentService.cs`
2. `EmergencyConsultationService.cs`
3. `ConsultationLifecycleBackgroundService.cs`
4. `BookingService.cs`
5. `ConsultationService.cs`

These files own the core escrow, refund, expiry, and settlement behavior.


