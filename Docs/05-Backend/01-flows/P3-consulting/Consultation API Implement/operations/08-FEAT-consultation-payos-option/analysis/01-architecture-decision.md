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

# ADR: PayOS as an Additional Consultation Payment Option

## Decision Summary

Trong flow consultation, PayOS sẽ được thiết kế như:

- **một lựa chọn thanh toán mới cho người dùng**
- song song với **WalletBalance**
- nhưng **không thay đổi business lifecycle của consultation escrow**

Nói ngắn gọn:

- UX layer: thêm option thanh toán
- domain layer: vẫn một consultation payment lifecycle

## User-Facing Framing

Từ góc nhìn người dùng, màn hình payment cho consultation sẽ có 2 lựa chọn:

1. `Ví hệ thống`
2. `PayOS`

Kết quả business mà người dùng kỳ vọng là giống nhau:

- scheduled consultation chỉ được xác nhận khi thanh toán thành công
- emergency consultation chỉ được gửi tới expert khi thanh toán thành công
- nếu flow emergency bị reject/expired thì tiền được hoàn theo đúng policy

## Technical Interpretation

Mặc dù UX chỉ là “thêm 1 option”, backend phải hỗ trợ 2 execution path khác nhau:

### Wallet path

- internal
- synchronous
- debit member wallet ngay
- credit system wallet ngay
- materialize escrow ngay trong transaction hiện tại

### PayOS path

- external gateway
- asynchronous hoặc semi-async
- tạo payment intent / paylink trước
- chờ webhook hoặc manual confirmation
- chỉ materialize escrow sau khi payment success được xác nhận

## Preserved Invariants

Việc thêm PayOS không được làm đổi các invariant hiện tại của consultation domain:

1. `Confirmed` không được set trước payment success.
2. `PendingExpertResponse` không được set trước payment success.
3. refund emergency vẫn là domain behavior, không phải gateway behavior.
4. settlement cho expert vẫn chỉ xảy ra khi consultation completed.
5. idempotency vẫn bám vào transaction markers hiện có.

## Why This Direction Is Correct

Nếu coi PayOS là một “payment provider choice” thay vì một “new consultation flow”, ta giữ được:

- UX đơn giản cho mobile/web
- domain behavior nhất quán
- reuse được PayOS layer hiện có
- không làm vỡ refund/settlement graph đang đúng với consultation lifecycle

## What Must Not Happen

1. Không để PayOS branch tự update booking/request status độc lập với consultation domain service.
2. Không để webhook trực tiếp quyết định business transition mà không đi qua consultation orchestration.
3. Không trộn lẫn pending external payment với escrowed money state.
4. Không thay `WalletBalance` path thành phụ thuộc gateway abstractions không cần thiết.

## Outcome

Documentation và implementation kế tiếp phải luôn dùng câu này làm framing chính:

> PayOS là thêm một payment option cho consultation, không phải thay đổi bản chất của consultation escrow lifecycle.
