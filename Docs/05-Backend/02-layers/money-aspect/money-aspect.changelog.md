# Money Aspect Front-Facing Changelog

## Purpose

File này dùng để kiểm soát các thay đổi có thể ảnh hưởng frontend/mobile trong money aspect.

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
