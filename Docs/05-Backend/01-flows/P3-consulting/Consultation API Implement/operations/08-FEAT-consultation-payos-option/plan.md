---
doc_role: operation
operation_id: 08-FEAT-consultation-payos-option
type: FEAT
status: draft
created_at: 2026-03-21
affects:
  - SnakeAid.Api/Controllers/ConsultationPaymentsController.cs
  - SnakeAid.Service/Implements/ConsultationPaymentService.cs
  - SnakeAid.Service/Interfaces/IConsultationPaymentService.cs
  - SnakeAid.Core/Requests/Consultation/ProcessConsultationPaymentRequest.cs
  - SnakeAid.Core/Responses/Consultation/ConsultationPaymentResponse.cs
  - 05-Backend/01-flows/P3-consulting/Consultation API Implement/consultation.sourcecode.md
  - 05-Backend/02-layers/payos/payos.sourcecode.md
---

# Plan: Frame PayOS as an Additional Consultation Payment Option

## Analysis References

This plan depends on:

- `../07-FEAT-consultation-payment-payos-readiness/analysis/01-architecture-decision.md`
- `../07-FEAT-consultation-payment-payos-readiness/analysis/02-state-machine.md`
- `../07-FEAT-consultation-payment-payos-readiness/analysis/03-sequence-flows.md`
- `analysis/01-architecture-decision.md`

## 1. As-Is

Consultation payment hiện tại:

- public API chỉ có 2 payment endpoints trong `ConsultationPaymentsController`
- request enum chỉ có `WalletBalance`
- response shape assume payment đã escrow xong
- PayOS layer tồn tại trong hệ thống, nhưng consultation flow chưa dùng

User-facing interpretation hiện tại:

- consultation chỉ thanh toán bằng ví hệ thống

## 2. Gap Analysis

Nếu bắt đầu tích hợp PayOS mà không chốt framing, tài liệu rất dễ trượt sang một trong 2 lỗi:

1. mô tả PayOS như một flow business mới
2. hoặc mô tả PayOS như detail kỹ thuật thuần túy, làm mất góc nhìn user-facing

Cần một operation docs để chốt đúng ngôn ngữ:

- về UX: PayOS là thêm một lựa chọn thanh toán
- về backend: PayOS là execution branch mới trước bước materialize escrow

## 3. To-Be Design

Documentation tiếp theo phải mô tả consultation payment theo 2 tầng:

### A. User-facing layer

Người dùng chọn một trong hai cách thanh toán:

- `WalletBalance`
- `PayOS`

### B. Domain lifecycle layer

Bất kể user chọn gì, hệ thống vẫn phải đi qua cùng một business outcome:

- payment success
- escrow materialized
- scheduled booking confirmed hoặc emergency request activated
- refund / settlement theo consultation lifecycle

### C. Provider integration layer

PayOS chỉ là thêm một provider-backed execution path để đi tới business outcome ở trên.

## 4. Impacted Components

Docs cần nhắc đến đúng các component sau:

- `ConsultationPaymentsController`
- `ConsultationPaymentService`
- `ProcessConsultationPaymentRequest`
- `ConsultationPaymentResponse`
- `IPaymentGateway`
- `PayOsGateway`

## 5. Risks & Constraints

1. Không mô tả PayOS như thay thế wallet path.
2. Không mô tả PayOS như bypass consultation domain service.
3. Không mô tả payment success và escrow success là cùng một thời điểm cho mọi payment method.
4. Không quên rằng current response contract đang bias theo wallet path.

## 6. Validation Plan

Docs được coi là đúng framing nếu:

1. người đọc không nhầm PayOS là business flow mới
2. người đọc hiểu PayOS là payment option mới bên cạnh wallet
3. người đọc vẫn thấy refund/settlement thuộc consultation lifecycle
4. người đọc hiểu wallet path và PayOS path có execution model khác nhau nhưng cùng business outcome
