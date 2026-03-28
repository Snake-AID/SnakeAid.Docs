---
doc_role: operation
operation_id: 05-payment-and-stabilization
type: FEAT
status: done
created_at: 2026-03-07
merged_from: [06-STAB-mobile-readiness-gap-closure, 07-FEAT-consultation-payment-payos-readiness, 08-FEAT-consultation-payos-option]
affects:
  - Api/Controllers/ConsultationPaymentsController.cs
  - Api/Controllers/PayOsController.cs
  - Service/Implements/ConsultationPaymentService.cs
  - Service/Implements/ExpertService.cs
  - Service/Implements/BookingService.cs
  - Service/Implements/EmergencyConsultationService.cs
  - Service/Implements/ConsultationLifecycleBackgroundService.cs
  - Core/Domains/ExpertProfile.cs
  - Core/Domains/ConsultationBooking.cs
  - Core/Domains/ConsultationPingRequest.cs
  - Core/Requests/Consultation/ProcessConsultationPaymentRequest.cs
  - Core/Responses/Consultation/ConsultationPaymentResponse.cs
---

# Operation 05: Payment & Stabilization

## Mục tiêu

Đóng tất cả gap còn lại của Op 01/02/03 về payment, pricing, profile stats, và thêm PayOS như payment option thứ hai bên cạnh WalletBalance.

## A. Stabilization Pass (từ Op 06 gốc)

### Expert Profile Completion (Op 01 gaps)

- Dual pricing: `ScheduledConsultationFee` + `EmergencyConsultationFee`
- Profile stats: `TotalConsultations`, `AverageResponseTimeMinutes`, `SuccessRate`
- `IsVerified` deferred khỏi MVP

### Consultation Payment — WalletBalance

- `POST /api/consultations/scheduled/{bookingId}/payments` — scheduled booking payment
- `POST /api/consultations/instant/{requestId}/payments` — emergency payment
- Payment method: `WalletBalance` hoặc `PayOs`
- Wallet path: debit member → credit system escrow → immediate `Escrowed`

### Escrow Lifecycle

- Scheduled: payment success → `Confirmed` → completion → settle to expert
- Emergency: payment success → `PendingExpertResponse` → accept → completion → settle
- Reject/Expired → refund escrow → member wallet
- Settlement idempotent (check existing payout transaction)

### Background Automation

- `ConsultationLifecycleBackgroundService` (30s polling):
  - Expire timed-out emergency requests + refund
  - Auto-complete elapsed scheduled consultations + settle
- PostgreSQL advisory lock cho multi-replica safety

### Multi-Replica Safety

- Global session advisory lock cho lifecycle worker
- Transaction-scoped advisory lock per aggregate cho payment/refund/settlement

## B. PayOS Integration (từ Op 07 + 08 gốc)

### Framing

PayOS là thêm một lựa chọn thanh toán bên cạnh WalletBalance. Không phải flow business mới. Cùng business outcome: escrow → refund/settlement.

### PayOS Flow

1. User chọn `paymentMethod: "PayOs"`
2. Backend tạo payment intent, trả `checkoutUrl`, `orderCode`, `paymentLinkId`
3. Response `status: "Pending"` (chưa escrow)
4. User thanh toán qua PayOS checkout
5. Confirm qua: webhook (`POST /api/v1/PayOs/webhook`) hoặc return (`GET /api/v1/PayOs/return`) hoặc manual confirm (`POST /api/consultations/payments/confirm`)
6. Sau confirm success → materialize escrow → advance consultation state

### Wallet vs PayOS

| Aspect | WalletBalance | PayOs |
|--------|--------------|-------|
| Escrow timing | Immediate | After external confirm |
| Response status | `Escrowed` | `Pending` → `Escrowed` |
| Refund/Settlement | Identical | Identical |
| Business outcome | Same | Same |

### Architecture Decisions

- `ConsultationPaymentService` vẫn là domain owner
- `IPaymentGateway` / `PayOsGateway` chỉ cho provider-facing actions
- Refund/settlement logic ở consultation domain, không ở gateway
- Wallet path backward compatible hoàn toàn

## C. Hardcoded / Technical Debt

- System wallet account id hardcoded
- Emergency TTL hardcoded 2 phút
- Background polling interval hardcoded 30s
- Consultation payment status query endpoint chưa implement
- Completion/payment summary contract chưa đầy đủ cho mobile
- Expert no-show / dispute handling chưa có trong MVP

## D. Analysis References (từ Op 07 gốc)

Các analysis docs kỹ thuật cho PayOS integration:
- `analysis/01-architecture-decision.md`
- `analysis/02-state-machine.md`
- `analysis/03-sequence-flows.md`

## E. Test Coverage

- `ConsultationPaymentIntegrationTests` — escrow/refund/settlement
- `ConsultationBookingsControllerIntegrationTests` — expert scheduled inbox
- Dual pricing behavior tests
- Expert stats mapping tests
- Emergency expiry broadcast tests
- Idempotent settlement tests
