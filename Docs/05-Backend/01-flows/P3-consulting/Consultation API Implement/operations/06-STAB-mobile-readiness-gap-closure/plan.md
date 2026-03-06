---
doc_role: operation
operation_id: 06-STAB-mobile-readiness-gap-closure
type: STAB
status: draft
created_at: 2026-03-07
affects:
  - Api/Controllers/ExpertController.cs
  - Api/Controllers/ConsultationBookingsController.cs
  - Api/Controllers/ConsultationsController.cs
  - Api/Controllers/PaymentsController.cs
  - Api/Hubs/ExpertHub.cs
  - Service/Implements/ExpertService.cs
  - Service/Implements/BookingService.cs
  - Service/Implements/EmergencyConsultationService.cs
  - Service/Implements/ConsultationPaymentService.cs
  - Service/Interfaces/IConsultationPaymentService.cs
  - Core/Domains/ExpertProfile.cs
  - Core/Domains/ConsultationBooking.cs
  - Core/Domains/ConsultationPingRequest.cs
  - Core/Responses/Expert/ExpertProfileResponse.cs
  - Core/Responses/Consultation/ConsultationBookingResponse.cs
  - Tests/Integration/*
  - Tests/Unit/*
---

# Plan: Mobile Readiness Gap Closure for Operations 01, 03, 04

## 1. As-Is

Operations 01, 03, and 04 already provide the backbone for expert directory, scheduled consultation, and emergency consultation. However, multiple wireframe components are still marked `Build With Placeholder` or `Wait Backend` in mobile handoff because the backend contract is incomplete for production-grade mobile delivery.

Operation 06 is not a new business flow. It is a stabilization pass whose sole purpose is to close remaining backend gaps inside Operations 01, 03, and 04.

Operation 05 already owns in-room features (chat, consultation-scoped signaling, expert side tools). Operation 06 must not absorb that room-scope work.

## 2. Gap Analysis

- **Operation 01 gaps**
  - Expert profile response still lacks wireframe statistics: `Ca tư vấn`, `Thời gian phản hồi`, `Tỉ lệ thành công`.
  - Expert settings still expose only one `ConsultationFee`, while wireframe requires separate pricing for `tư vấn ngay` and `đặt lịch`.
- **Operation 03 gaps**
  - Scheduled consultation pre-payment flow is not implemented end-to-end for mobile.
  - Booking lifecycle surfaces `PendingPayment`, but there is no consultation-specific payment API contract for checkout, payment method selection, wallet balance usage, escrow creation, or payment status refresh.
  - Expert-side completion/payment breakdown remains incomplete.
- **Operation 04 gaps**
  - Emergency consultation flow now has request-room realtime for `Accepted/Rejected`, but expiry handling still depends on client-side countdown instead of a backend-driven terminal event.
  - Immediate consultation pre-payment flow is not implemented end-to-end.
  - Refund behavior for `Rejected` / `Expired` and escrow settlement after completion are not formalized in backend yet.
- **Cross-operation mobile gap**
  - Wireframe payment screens for both scheduled and immediate consultation are still `Wait Backend`.

## 3. To-Be Design

Implement Operation 06 as a stabilization pass with the following scope:

- **Operation 01 completion**
  - Extend expert settings to support two prices:
    - `scheduledConsultationFee`
    - `emergencyConsultationFee`
  - Extend expert profile/list response as needed so mobile can render the correct price per consultation type.
  - Add profile statistics fields required by wireframe:
    - `totalConsultations`
    - `averageResponseTime`
    - `successRate`
  - Keep `IsVerified` out of MVP scope for now; do not treat it as a blocker in Operation 06.

- **Operation 03 completion**
  - Add consultation-specific payment orchestration for scheduled consultation:
    - create checkout/payment session
    - support payment method selection
    - support `SApay` balance usage
    - query payment status
    - move funds into `SApay` escrow after payment success
  - Ensure scheduled bookings transition cleanly from `PendingPayment` to escrowed/confirmed states.
  - Extend consultation completion data so user/expert completion screens can render payment summary consistently.
  - Support escrow settlement to expert when consultation completes, where completion can be triggered by slot end or explicit finish API.

- **Operation 04 completion**
  - Add consultation-specific payment orchestration for emergency consultation before expert action:
    - create checkout/payment session
    - support payment method selection
    - support `SApay` balance usage
    - query payment status
    - move funds into `SApay` escrow after payment success
  - Add backend-driven expiry transition for pending emergency requests.
  - Push terminal `Expired` state to the request room via `EmergencyRequestStatusChanged`.
  - Refund escrow immediately back to member `SApay` balance when request is `Rejected` or `Expired`.
  - Keep accepted emergency consultations in escrow until consultation completes.
  - Settle escrow to expert when consultation completes, where completion can be triggered by slot end or explicit finish API.

## 4. Explicit Non-Scope

- Do not move consultation chat or in-room UI signaling into Operation 06.
- Do not absorb expert side-panel snake lookup into Operation 06.
- Those remain in **Operation 05: In-Room Features**.
- Do not rewrite `Consultation Screen API.md` to assign ownership of those gaps to Operation 06; mobile handoff should still point to Operations 01, 03, and 04 as the business owners.

## 5. Impacted Components

- **Controllers**:
  - `ExpertController`
  - `ConsultationBookingsController`
  - `ConsultationsController`
  - New consultation payment controller if needed
- **Services**:
  - `ExpertService`
  - `BookingService`
  - `EmergencyConsultationService`
  - New `ConsultationPaymentService`
- **Entities / DTOs**:
  - `ExpertProfile`
  - `ConsultationBooking`
  - `ConsultationPingRequest`
  - expert/profile/payment response DTOs

## 6. Risks & Constraints

- Payment semantics must be designed explicitly before implementation. Do not bolt consultation payments onto unrelated payment flows.
- Both scheduled and emergency consultations are `pre-payment` flows in MVP.
- Funds are not transferred directly to expert on payment success; they must first enter `SApay` escrow.
- `Reject` and `Expired` for emergency must refund escrow immediately to member `SApay` balance.
- Settlement to expert happens only on consultation completion.
- Completion may be triggered by slot end or explicit finish API, so settlement logic must be idempotent.
- Statistics fields must have stable definitions; avoid shipping placeholders disguised as real metrics.
- `IsVerified` is deferred out of MVP and should not receive fake completion work in Operation 06.
- Expiry broadcasting must avoid race conditions with simultaneous accept/reject actions.
- Operation 06 documentation must read as a closure pass for Operations 01, 03, and 04, not as a new product flow.

## 7. Validation Plan

- Integration tests for consultation payment lifecycle:
  - scheduled payment create -> payment success -> escrow created
  - emergency payment create -> payment success -> escrow created -> expert can act
- Integration tests for emergency expiry push:
  - pending request transitions to `Expired`
  - requester receives `EmergencyRequestStatusChanged`
- Integration tests for emergency refund flow:
  - `Rejected` -> escrow refunded to member `SApay`
  - `Expired` -> escrow refunded to member `SApay`
- Integration tests for settlement flow:
  - consultation completed by slot end -> escrow settled to expert
  - consultation completed by explicit finish API -> escrow settled to expert
  - repeated completion trigger does not duplicate settlement
- Unit/integration tests for expert profile statistics.
- Regression tests to ensure Operation 05 room features remain decoupled from Operation 06.
