# Money Aspect Front-Facing Changelog

## Purpose

File này dùng để kiểm soát các thay đổi có thể ảnh hưởng frontend/mobile trong money aspect.

File này là handoff artifact cho frontend/mobile dev. Bất kỳ subphase nào của money refactor làm đổi endpoint, request DTO, response DTO, response field semantic, payment prefix, hoặc transaction type mà client có thể đọc/filter thì phải cập nhật file này ngay trong cùng lượt code.

Nếu không có client-visible change, vẫn nên ghi rõ `NO CLIENT-VISIBLE CONTRACT CHANGE` cho phase đó để reviewer không phải suy luận từ backend diff.

Chỉ ghi thay đổi client-visible:

- route
- HTTP method
- auth/role expectation
- request DTO
- response DTO
- response field semantic
- payment prefix nếu client có parse/hardcode
- transaction type nếu client hiển thị hoặc filter theo transaction list

## 2026-04-08 - Phase 6/7 planning watchlist

Trạng thái: `PLANNED`, chưa đổi production endpoint/DTO trong code ở entry này.

Các field có rủi ro đổi semantic khi Phase 6 bãi bỏ system wallet:

- `ConsultationPaymentResponse.SystemWalletBalanceAfter`
- `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter`
- `RefundTransactionResponse.SystemWalletBalanceBefore`
- `RefundTransactionResponse.SystemWalletBalanceAfter`
- `TransferToRescuerResponse.SystemWalletBalanceBefore`
- `TransferToRescuerResponse.SystemWalletBalanceAfter`

Quy tắc khi implement:

- nếu field được giữ nhưng trả `null`, `0`, hoặc đổi nghĩa sang transaction-sourced value thì phải thêm entry changelog mới
- nếu field bị remove/rename thì phải ghi rõ old field -> new field
- nếu thêm fee breakdown cho consultation settlement response thì phải ghi rõ field mới và endpoint liên quan
- nếu transaction list bắt đầu expose transaction type mới hoặc bỏ `EscrowHold` / `EscrowRelease`, phải ghi rõ impact với `GET /api/transactions`

Decision đã chốt cho Phase 7:

- consultation platform fee default là `20%` nếu system setting chưa tồn tại
- rounding ưu tiên expert: làm tròn lên `expertNetAmount` theo đơn vị VND, rồi tính `platformFeeAmount = grossAmount - expertNetAmount`
- client cần fee breakdown khi có response/contract liên quan consultation payout hoặc transaction detail
- nếu thêm fee breakdown, field dự kiến cần document gồm:
  - `grossAmount`
  - `platformFeePercent`
  - `platformFeeAmount`
  - `expertNetAmount`

## 2026-04-08 - Phase 6A regression coverage

Trạng thái: `NO CLIENT-VISIBLE CONTRACT CHANGE`.

Phase 6A chỉ thêm regression/characterization tests cho consultation escrow. Production endpoint, HTTP method, request DTO, response DTO, payment prefix, và transaction enum chưa đổi trong entry này.

Client-facing watchlist giữ nguyên cho Phase 6B+:

- nếu `ConsultationPaymentResponse.SystemWalletBalanceAfter` bắt đầu trả `null`, `0`, hoặc đổi nghĩa theo transaction-sourced ledger thì phải ghi entry changelog mới
- nếu `GET /api/transactions` bỏ hoặc đổi nghĩa `EscrowHold` / `EscrowRelease`, phải ghi rõ impact cho client đang filter theo transaction type

## 2026-04-08 - Phase 6B consultation transaction-sourced escrow

Trạng thái: `CLIENT-VISIBLE RESPONSE FIELD SEMANTIC CHANGE`.

Áp dụng cho consultation payment responses:

- `POST /api/consultations/scheduled/{bookingId:guid}/payments`
- `POST /api/consultations/instant/{requestId:guid}/payments`
- `POST /api/consultations/payments/confirm`
- consultation PayOS webhook/confirm path trả cùng response model nội bộ trước khi map webhook response

Thay đổi:

- `ConsultationPaymentResponse.SystemWalletBalanceAfter` vẫn tồn tại và vẫn nullable
- từ Phase 6B, field này trả `null` cho consultation escrow responses
- lý do: consultation escrow không còn tạo/update system wallet; availability được suy ra từ `Transaction`

Transaction exposure:

- consultation hold không còn tạo `EscrowHold`
- consultation refund/settlement không còn tạo `EscrowRelease`
- `ConsultationPayment`, `ConsultationRefund`, và `ExpertPayout` vẫn là các transaction domain chính cho consultation
- `EscrowHold` / `EscrowRelease` vẫn có thể xuất hiện ở flow khác cho tới khi Phase 6C/6D/6E hoàn tất

## 2026-04-08 - Phase 6C snakebite incident transaction-sourced escrow

Trạng thái: `CLIENT-VISIBLE RESPONSE FIELD SEMANTIC CHANGE`.

Áp dụng cho snakebite incident payment/refund responses:

- `POST /api/incidents/{incidentId}/payment/wallet`
- `POST /api/incidents/{incidentId}/payment/payos` sau khi PayOS được confirm/webhook xử lý
- `POST /api/incidents/{incidentId}/payment/refund`

Thay đổi:

- `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter` vẫn tồn tại và vẫn nullable
- từ Phase 6C, field này trả `null` cho incident escrow responses
- `RefundTransactionResponse.SystemWalletBalanceBefore` đổi từ `decimal` sang `decimal?`
- `RefundTransactionResponse.SystemWalletBalanceAfter` đổi từ `decimal` sang `decimal?`
- đây là shared response model; nếu flow khác còn dùng model này và vẫn trả số, client vẫn phải treat hai field này là nullable từ Phase 6C
- với incident refunds, hai field `RefundTransactionResponse.SystemWalletBalanceBefore/After` trả `null`
- lý do: snakebite incident escrow không còn tạo/update system wallet; availability được suy ra từ `Transaction`

Transaction exposure:

- incident hold không còn tạo `EscrowHold`
- incident refund không còn tạo `EscrowRelease`
- `SnakebiteIncidentPayment` và `SnakebiteIncidentRefund` là các transaction domain chính cho incident escrow
- `EscrowHold` / `EscrowRelease` vẫn có thể xuất hiện ở snake catching cho tới khi Phase 6D/6E hoàn tất
