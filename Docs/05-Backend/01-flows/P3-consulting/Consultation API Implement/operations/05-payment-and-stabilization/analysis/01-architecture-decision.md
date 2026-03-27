---
doc_role: operation
operation_id: 07-FEAT-consultation-payment-payos-readiness
type: FEAT
status: approved
created_at: 2026-03-21
affects:
  - SnakeAid.Api/Controllers/ConsultationPaymentsController.cs
  - SnakeAid.Service/Implements/ConsultationPaymentService.cs
  - SnakeAid.Service/Implements/ConsultationService.cs
  - SnakeAid.Service/Implements/BookingService.cs
  - SnakeAid.Service/Implements/EmergencyConsultationService.cs
  - SnakeAid.Service/Implements/ConsultationLifecycleBackgroundService.cs
  - SnakeAid.Service/Interfaces/IConsultationPaymentService.cs
  - SnakeAid.Service/Interfaces/IPaymentGateway.cs
  - SnakeAid.Service/Services/PayOs/PayOsGateway.cs
  - SnakeAid.Api/Program.cs
---

# ADR: Consultation Payment Current Graph and PayOS Readiness

## Problem

Flow 3 consultation đã có payment orchestration chạy được cho `WalletBalance`, nhưng chưa có external gateway path cho consultation. Trước khi nối PayOS, cần chốt rõ:

1. method graph hiện tại của scheduled và emergency payment
2. escrow/refund/settlement đang bám vào aggregate nào
3. seam nào có thể reuse từ PayOS layer hiện có
4. điểm nào trong docs hiện tại đang dễ gây hiểu nhầm

## Current Truth From Code

### 1. Consultation payment hiện là wallet-ledger flow, không phải gateway flow

Code path hiện tại:

- `ConsultationPaymentsController.PayScheduledBooking`
- `ConsultationPaymentsController.PayEmergencyRequest`
- `ConsultationPaymentService.PayScheduledBookingAsync`
- `ConsultationPaymentService.PayEmergencyRequestAsync`

Hai public method này đều gọi `EnsureWalletPaymentMethod`, và enum `ConsultationPaymentMethod` hiện chỉ có:

- `WalletBalance = 0`

Nghĩa là consultation payment hiện chưa có nhánh `PayOs`, chưa có pending external intent, chưa có webhook/manual confirm/status query.

### 2. Escrow được mô hình hóa bằng wallet nội bộ + transaction mirror

`ConsultationPaymentService.MoveMoneyToEscrowAsync`:

- lấy ví member bằng `GetRequiredWalletAsync`
- lấy hoặc tạo ví hệ thống bằng `GetOrCreateWalletAsync(SystemWalletUserId)`
- trừ tiền member wallet
- cộng tiền system wallet
- ghi 2 transaction:
  - member-side `TransactionType.ConsultationPayment`
  - system-side `TransactionType.WalletTopup`

Điều này có nghĩa:

- escrow không phải entity riêng
- escrow là balance đang nằm trong system wallet
- ledger semantics phụ thuộc vào cặp transaction mirror

### 3. Refund và settlement cũng là wallet-internal transfer

`RefundEmergencyEscrowAsync` -> `RefundFromEscrowAsync`

- giảm system wallet
- tăng member wallet
- ghi:
  - system-side `TransactionType.WalletWithdraw`
  - member-side `TransactionType.ConsultationRefund`

`SettleConsultationEscrowAsync` -> `TransferEscrowToExpertAsync`

- giảm system wallet
- tăng expert wallet
- ghi:
  - system-side `TransactionType.WalletWithdraw`
  - expert-side `TransactionType.ExpertPayout`

### 4. Consultation payment graph có 4 trigger point quan trọng

`PayScheduledBookingAsync`
- trigger qua API
- đổi `ConsultationBooking.Status: PendingPayment -> Confirmed`

`PayEmergencyRequestAsync`
- trigger qua API
- đổi `ConsultationPingRequest.Status: PendingPayment -> PendingExpertResponse`
- set `RequestedAt`, `ExpiresAt`
- push `SendEmergencyRequestAsync` sau khi commit

`RefundEmergencyEscrowAsync`
- trigger từ:
  - `EmergencyConsultationService.RejectEmergencyRequestAsync`
  - `ConsultationPaymentService.ExpireEmergencyRequestsAsync`

`SettleConsultationEscrowAsync`
- trigger từ:
  - `ConsultationService.EndConsultationAsync`
  - `BookingService.AutoCompleteElapsedScheduledConsultationsAsync`

### 5. Background lifecycle là một phần của payment graph

`ConsultationLifecycleBackgroundService.ExecuteAsync` mỗi 30 giây gọi:

- `IConsultationPaymentService.ExpireEmergencyRequestsAsync`
- `IBookingService.AutoCompleteElapsedScheduledConsultationsAsync`

Tức là refund và settlement không chỉ đi qua request/response path mà còn đi qua background sweep path.

### 6. PayOS layer hiện có chưa được consultation flow reuse

Trong repo hiện có:

- `IPaymentGateway`
- `PayOsGateway`

Chúng đang được dùng bởi:

- `SnakeCatchingPaymentService`
- `WalletTopupService`

Nhưng `ConsultationPaymentService` không inject `IPaymentGateway`.

Kết luận:

- payos layer đã tồn tại ở horizontal layer
- consultation flow chưa nối vào layer đó
- hiện tại flow 3 payment graph là domain-local wallet orchestration

### 7. DI thực tế của consultation payment service

`Program.cs` không có dòng explicit register `IConsultationPaymentService`, nhưng service vẫn được wire qua Scrutor:

- scan tất cả class tên kết thúc bằng `Service`
- `ConsultationPaymentService` được đăng ký qua `AsImplementedInterfaces()`

Đây là current truth quan trọng khi onboarding service graph runtime.

## Decision

Ghi nhận current architecture của consultation payment như sau:

1. Consultation payment hiện là `wallet-first escrow domain service`.
2. External gateway chưa phải branch hiện hữu trong method graph của flow 3.
3. Điểm nối PayOS đúng nhất trong tương lai là tách payment intent boundary ngay tại `ConsultationPaymentService`, không nhét PayOS vào refund/settlement layer.
4. Refund và settlement phải tiếp tục giữ ở domain service vì chúng phụ thuộc consultation lifecycle, không phụ thuộc provider.

## Rejected Alternatives

### A. Xem consultation payment như đã dùng chung payment gateway

Rejected vì code hiện tại không inject `IPaymentGateway` và không tạo external payment intent cho consultation.

### B. Đưa toàn bộ consultation payment sang reuse `WalletPaymentService`

Rejected vì `WalletPaymentService` hiện đang shape theo snake-catching semantics:

- request DTO là `CreateSnakeCatchingPaymentRequest`
- validation bám `SnakeCatchingRequest`
- transaction meaning bám catching transaction types

### C. Gắn PayOS trực tiếp vào controller và để domain service chỉ xác nhận kết quả

Rejected vì sẽ split business invariant ra khỏi aggregate transition:

- `PendingPayment -> Confirmed`
- `PendingPayment -> PendingExpertResponse`
- refund/settlement idempotency

## PayOS Readiness Direction

Nếu tiếp nhận PayOS cho consultation, hướng ít phá nhất là:

1. giữ `ConsultationPaymentService` là orchestration owner
2. thêm payment method branch mới ở consultation domain
3. reuse `IPaymentGateway` chỉ cho:
   - create external paylink
   - query status
   - verify webhook
4. chỉ move money vào escrow sau khi external payment đã được xác nhận
5. không đổi refund/settlement semantics vì đó là post-payment domain logic

## Risks

1. `SystemWalletUserId` đang hardcode trong `ConsultationPaymentService`.
2. `ExpireEmergencyRequestsAsync` commit status rồi mới refund; nếu crash giữa hai bước thì state business và money state có thể tạm lệch.
3. `FindTransactionAsync` đang là main idempotency check, nên future PayOS integration phải tránh tạo duplicated `ConsultationPayment` records.
4. `payos.sourcecode.md` mô tả layer đúng với refactor direction, nhưng không được hiểu lầm rằng consultation flow đã dùng gateway abstraction.

## Summary

Current method graph cho flow 3 payment có thể tóm gọn:

- API payment vào `ConsultationPaymentService`
- wallet debit/credit được thực hiện nội bộ trong service
- refund được kích hoạt bởi reject hoặc expire
- settlement được kích hoạt bởi end consultation hoặc auto-complete
- PayOS hiện chưa nằm trên execution graph của consultation payment

Đây là baseline nhận thức cần giữ trước khi thiết kế branch PayOS cho flow 3.
