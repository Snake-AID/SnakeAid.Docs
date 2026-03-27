---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
affects:
  - SnakeAid.Api/Controllers/ConsultationPaymentsController.cs
  - SnakeAid.Service/Implements/ConsultationPaymentService.cs
  - SnakeAid.Service/Implements/EmergencyConsultationService.cs
  - SnakeAid.Service/Implements/ConsultationService.cs
  - SnakeAid.Service/Implements/BookingService.cs
  - SnakeAid.Service/Implements/ConsultationLifecycleBackgroundService.cs
---

# Consultation Payment Sequence Flows

## 1. Scheduled Consultation Wallet Payment

```mermaid
sequenceDiagram
    participant User
    participant API as ConsultationPaymentsController
    participant Pay as ConsultationPaymentService
    participant DB as PostgreSQL

    User->>API: POST /api/consultation-bookings/{bookingId}/payments
    API->>Pay: PayScheduledBookingAsync(userId, bookingId, request)
    Pay->>Pay: EnsureWalletPaymentMethod()
    Pay->>DB: Load ConsultationBooking
    Pay->>DB: Check duplicate ConsultationPayment
    Pay->>DB: Load member wallet
    Pay->>DB: Load/create system wallet
    Pay->>DB: Debit member wallet
    Pay->>DB: Credit system wallet
    Pay->>DB: Insert ConsultationPayment transaction
    Pay->>DB: Insert system WalletTopup mirror transaction
    Pay->>DB: Update booking status = Confirmed
    Pay->>DB: Commit
    Pay-->>API: ConsultationPaymentResponse(Status=Escrowed)
    API-->>User: 200 OK
```

## 2. Emergency Consultation Wallet Payment

```mermaid
sequenceDiagram
    participant User
    participant API as ConsultationPaymentsController
    participant Pay as ConsultationPaymentService
    participant Notify as IExpertEmergencyNotificationService
    participant DB as PostgreSQL

    User->>API: POST /api/consultations/emergency-requests/{requestId}/payments
    API->>Pay: PayEmergencyRequestAsync(userId, requestId, request)
    Pay->>Pay: EnsureWalletPaymentMethod()
    Pay->>DB: Load ConsultationPingRequest
    Pay->>Notify: IsExpertConnected(expertId)
    Pay->>DB: Check duplicate ConsultationPayment
    Pay->>DB: Load ExpertProfile
    Pay->>DB: Debit member wallet
    Pay->>DB: Credit system wallet
    Pay->>DB: Insert ConsultationPayment transaction
    Pay->>DB: Insert system WalletTopup mirror transaction
    Pay->>DB: Set RequestedAt / ExpiresAt
    Pay->>DB: Update request status = PendingExpertResponse
    Pay->>DB: Commit
    Pay->>Notify: SendEmergencyRequestAsync(...)
    Pay-->>API: ConsultationPaymentResponse(Status=Escrowed)
    API-->>User: 200 OK
```

## 3. Emergency Reject -> Refund

```mermaid
sequenceDiagram
    participant Expert
    participant Svc as EmergencyConsultationService
    participant Pay as ConsultationPaymentService
    participant Notify as IExpertEmergencyNotificationService
    participant DB as PostgreSQL

    Expert->>Svc: RejectEmergencyRequestAsync(requestId, expertId)
    Svc->>DB: Load request
    Svc->>DB: Update status = DeclinedByExpert
    Svc->>DB: Commit
    Svc->>Pay: RefundEmergencyEscrowAsync(requestId, reason)
    Pay->>DB: Check duplicate ConsultationRefund
    Pay->>DB: Require ConsultationPayment transaction
    Pay->>DB: Debit system wallet
    Pay->>DB: Credit rescuer wallet
    Pay->>DB: Insert system WalletWithdraw mirror transaction
    Pay->>DB: Insert ConsultationRefund transaction
    Pay->>DB: Commit
    Svc->>Notify: NotifyEmergencyRequestStatusChangedAsync(...)
```

## 4. Emergency Expire -> Refund

```mermaid
sequenceDiagram
    participant Worker as ConsultationLifecycleBackgroundService
    participant Pay as ConsultationPaymentService
    participant Notify as IExpertEmergencyNotificationService
    participant DB as PostgreSQL

    Worker->>Pay: ExpireEmergencyRequestsAsync()
    Pay->>DB: Query requests PendingExpertResponse where ExpiresAt <= now
    loop each expired request
        Pay->>DB: Update status = Expired
        Pay->>DB: Commit
        Pay->>Pay: RefundEmergencyEscrowAsync(requestId, "expired")
        Pay->>Notify: NotifyEmergencyRequestStatusChangedAsync(...)
    end
```

## 5. Explicit End Consultation -> Settle Escrow

```mermaid
sequenceDiagram
    participant Actor
    participant Svc as ConsultationService
    participant Pay as ConsultationPaymentService
    participant DB as PostgreSQL

    Actor->>Svc: EndConsultationAsync(consultationId, actorId)
    Svc->>DB: Load consultation
    Svc->>DB: Mark consultation Completed
    Svc->>DB: Update booking Completed if scheduled
    Svc->>DB: Update slot Reserved -> Booked if needed
    Svc->>DB: Commit
    Svc->>Pay: SettleConsultationEscrowAsync(consultationId)
    Pay->>DB: Check duplicate ExpertPayout
    Pay->>DB: Resolve scheduled booking or emergency request by consultationId
    Pay->>DB: Require ConsultationPayment transaction
    Pay->>DB: Debit system wallet
    Pay->>DB: Credit expert wallet
    Pay->>DB: Insert system WalletWithdraw mirror transaction
    Pay->>DB: Insert ExpertPayout transaction
    Pay->>DB: Commit
```

## 6. Auto Complete Scheduled Consultation -> Settle Escrow

```mermaid
sequenceDiagram
    participant Worker as ConsultationLifecycleBackgroundService
    participant Booking as BookingService
    participant Pay as ConsultationPaymentService
    participant DB as PostgreSQL

    Worker->>Booking: AutoCompleteElapsedScheduledConsultationsAsync()
    Booking->>DB: Query confirmed bookings where slot end <= now
    loop each elapsed booking
        Booking->>DB: Mark booking Completed
        Booking->>DB: Mark consultation Completed
        Booking->>DB: Mark slot Reserved -> Booked
        Booking->>DB: Commit
        Booking->>Pay: SettleConsultationEscrowAsync(consultationId)
    end
```

## 7. PayOS Readiness Observation

Chỗ duy nhất hợp lý để cắm PayOS cho consultation là thay đoạn:

- `EnsureWalletPaymentMethod`
- `MoveMoneyToEscrowAsync`

bằng nhánh external intent + verified settlement.

Không nên chạm vào:

- `RefundEmergencyEscrowAsync`
- `SettleConsultationEscrowAsync`

trừ khi muốn đổi toàn bộ escrow materialization model.
