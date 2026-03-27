---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
affects:
  - SnakeAid.Core/Domains/ConsultationBooking.cs
  - SnakeAid.Core/Domains/ConsultationPingRequest.cs
  - SnakeAid.Core/Domains/Consultation.cs
  - SnakeAid.Core/Domains/Wallet.cs
  - SnakeAid.Core/Domains/Transaction.cs
---

# Consultation Payment State Machine

## 1. Scheduled Booking Payment State

Entity chính:

- `ConsultationBooking`
- `Consultation`
- `Wallet`
- `Transaction`

### Booking state transitions

`PendingPayment`
- created by `BookingService.CreateScheduledBookingAsync`
- slot bị chuyển sang `Reserved`
- consultation record đã được tạo ở trạng thái `Scheduled`

`Confirmed`
- set bởi `ConsultationPaymentService.PayScheduledBookingAsync`
- condition:
  - requester đúng owner booking
  - status vẫn là `PendingPayment`
  - chưa có `TransactionType.ConsultationPayment`
  - wallet đủ số dư

`Completed`
- set bởi:
  - `ConsultationService.EndConsultationAsync`
  - `BookingService.AutoCompleteElapsedScheduledConsultationsAsync`

### Money state transitions

`NoPayment`
- chưa có `ConsultationPayment` transaction

`Escrowed`
- tạo bởi `MoveMoneyToEscrowAsync`
- invariant:
  - member wallet giảm đúng `booking.Price`
  - system wallet tăng đúng `booking.Price`
  - có 1 member transaction `ConsultationPayment`
  - có 1 system mirror transaction `WalletTopup`

`SettledToExpert`
- tạo bởi `SettleConsultationEscrowAsync`
- invariant:
  - chỉ settle một lần, check qua `TransactionType.ExpertPayout`
  - system wallet giảm đúng amount payment
  - expert wallet tăng đúng amount payment

## 2. Emergency Request Payment State

Entity chính:

- `ConsultationPingRequest`
- `Consultation`
- `Wallet`
- `Transaction`

### Request state transitions

`PendingPayment`
- created by emergency request create flow
- chưa notify expert

`PendingExpertResponse`
- set bởi `ConsultationPaymentService.PayEmergencyRequestAsync`
- condition:
  - requester đúng owner request
  - request vẫn `PendingPayment`
  - expert đang online theo `IsExpertConnected`
  - chưa có duplicate `ConsultationPayment`
  - wallet đủ tiền
- side effect:
  - set `RequestedAt`
  - set `ExpiresAt`
  - gửi `SendEmergencyRequestAsync` sau commit

`DeclinedByExpert`
- set bởi `EmergencyConsultationService.RejectEmergencyRequestAsync`
- side effect:
  - gọi `RefundEmergencyEscrowAsync`

`Expired`
- set bởi `ConsultationPaymentService.ExpireEmergencyRequestsAsync`
- side effect:
  - gọi `RefundEmergencyEscrowAsync`
  - push `NotifyEmergencyRequestStatusChangedAsync`

`AcceptedByExpert`
- request đã bind vào consultation
- escrow vẫn giữ nguyên trong system wallet

### Money state transitions

`NoPayment`
- chưa có `ConsultationPayment` transaction

`Escrowed`
- tạo bởi `MoveMoneyToEscrowAsync`
- invariant giống scheduled flow

`RefundedToRequester`
- tạo bởi `RefundEmergencyEscrowAsync`
- invariant:
  - chỉ refund một lần, check qua `TransactionType.ConsultationRefund`
  - system wallet giảm đúng amount payment
  - rescuer wallet tăng đúng amount payment

`SettledToExpert`
- chỉ xảy ra sau khi consultation hoàn thành
- không settle ở bước accept

## 3. Consultation Completion State

`Consultation.Status`

`Scheduled`
- consultation scheduled chưa complete

`Completed`
- set bởi:
  - `ConsultationService.EndConsultationAsync`
  - `BookingService.AutoCompleteElapsedScheduledConsultationsAsync`

Invariant:

- completion có thể bị trigger nhiều nguồn
- settlement phải idempotent
- idempotency hiện tại dựa vào existence của `ExpertPayout`

## 4. Wallet / Transaction Invariants

### Wallet invariants

1. Member wallet không được âm.
2. System wallet phải đủ balance trước refund hoặc settlement.
3. Expert wallet được auto-create nếu chưa tồn tại.

### Transaction invariants

1. `ConsultationPayment` là business anchor cho escrow amount.
2. `ConsultationRefund` là refund marker cho emergency request.
3. `ExpertPayout` là settlement marker cho consultation.
4. `WalletTopup` và `WalletWithdraw` ở system wallet đang được dùng như mirror ledger records.

## 5. PayOS Readiness Constraints From State Machine

Nếu thêm PayOS:

1. Phải có state phân biệt `payment intent created` với `money already escrowed`.
2. Không được set `Confirmed` hoặc `PendingExpertResponse` chỉ vì paylink được tạo.
3. Escrow chỉ được materialize sau verified payment.
4. Refund và settlement vẫn bám vào escrow materialized state, không bám vào paylink state.
