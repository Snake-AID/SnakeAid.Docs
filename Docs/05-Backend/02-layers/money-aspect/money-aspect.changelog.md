# Money Aspect Front-Facing Changelog

## Purpose

File này chỉ dùng để handoff cho frontend/mobile dev.

Chỉ ghi:

- endpoint/route change
- request/response DTO change
- response field semantic change
- deprecation
- transaction exposure change nếu client có đọc/filter transaction list

Không ghi:

- roadmap phase
- reasoning nội bộ
- correction history
- verification log

Các nội dung đó đã thuộc `money-aspect.refactoring.md` và `money-aspect.hallucination.md`.

## Current Contract Snapshot

| Flow | Client-facing contract state |
|---|---|
| Consultation | escrow vẫn tồn tại; `SystemWalletBalanceAfter` được giữ nhưng trả `null` |
| Snakebite Incident | payment là ledger-only system revenue; wallet payment status là `Paid`; `SystemWalletBalance*` nullable và trả `null` |
| Snake Catching | payment là ledger-only system revenue; `transfer-to-rescuer` đã bị deprecate thành compatibility no-op |

## 2026-04-09 - Snake Catching `transfer-to-rescuer` deprecated

**Status:** `deprecation + semantic change`

**Scope**

- `POST /api/snakecatching/payment/transfer-to-rescuer`
- `TransferToRescuerResponse`

**Change**

- endpoint vẫn còn để giữ compatibility
- endpoint không còn trigger payout nữa
- endpoint/service không còn tạo:
  - `CatcherPayout`
  - `PlatformFee`
  - `EscrowRelease`
- endpoint/service không còn đụng `system.wallet` hoặc rescuer wallet

**Response DTO**

- `TransferTransactionId`: `Guid` -> `Guid?`
- `SystemWalletBalanceBefore`: `decimal` -> `decimal?`
- `SystemWalletBalanceAfter`: `decimal` -> `decimal?`
- `RescuerWalletBalanceBefore`: `decimal` -> `decimal?`
- `RescuerWalletBalanceAfter`: `decimal` -> `decimal?`

**Response semantic**

- `Success` vẫn có thể là `true`
- `TransferTransactionId` có thể là `null`
- các balance field có thể là `null`
- `NetAmountToRescuer = 0`
- `Message` nêu rõ endpoint đã deprecated và không còn thực hiện rescuer transfer

**Client action**

- không xem endpoint này là payout trigger nữa
- xử lý nullable cho `TransferTransactionId` và các balance field
- nếu UI đang hiển thị transfer receipt, phải handle trường hợp không còn transfer transaction

## 2026-04-09 - Snake Catching payment path switched to ledger-only system revenue

**Status:** `semantic change`

**Scope**

- snake catching wallet payment
- snake catching PayOS confirm/webhook path
- transaction list views nếu client filter theo `TransactionType`

**Change**

- snake catching payment không còn tạo `EscrowHold`
- snake catching payment không còn credit `system.wallet`
- payment được ghi nhận như ledger-only system/platform revenue

**Response semantic**

- `PayOsWebhookResponse.Status` của snake catching confirm/webhook giờ trả explicit:
  - `Paid` khi thành công
  - `Failed` khi thất bại

**Client action**

- không suy luận snake catching payment state qua `EscrowHold`
- không parse `GatewayRawResponse` của wallet payment như source of truth cho system-wallet revenue/escrow
- nếu UI đang hiển thị webhook/confirm status, dùng `PayOsWebhookResponse.Status` mới thay vì dựa vào default cũ

## 2026-04-09 - Snake Catching refund switched to revenue-ledger semantics

**Status:** `response semantic change`

**Scope**

- snake catching refund path
- `RefundTransactionResponse`
- transaction list views nếu client filter theo `TransactionType`

**Change**

- snake catching refund không còn tạo `EscrowRelease`
- snake catching refund không còn check `system.wallet` balance
- refundability được backend tính từ:
  - `CatchingPayment + CatchingDeposit - CatchingRefund`

**Response semantic**

- `RefundTransactionResponse.SystemWalletBalanceBefore/After` vẫn tồn tại nhưng trả `null` cho snake catching refund path

**Client action**

- treat `SystemWalletBalanceBefore/After` của snake catching refund là nullable semantic field
- không hiển thị snake catching refund như escrow release nữa
- nếu transaction UI đang filter theo `EscrowRelease` để biểu diễn catching refund, logic đó phải bỏ

## 2026-04-08 - Snakebite Incident moved to ledger-only system revenue

**Status:** `response semantic change`

**Scope**

- `POST /api/incidents/{incidentId}/payment/wallet`
- incident PayOS confirm/webhook path
- `POST /api/incidents/{incidentId}/payment/refund`
- `SnakebiteIncidentPaymentResponse`
- `RefundTransactionResponse`

**Change**

- incident không còn là escrow flow
- wallet payment status đổi:
  - `Escrowed` -> `Paid`
- incident payment không còn tạo `EscrowHold`
- incident refund không còn tạo `EscrowRelease`

**Response DTO**

- `RefundTransactionResponse.SystemWalletBalanceBefore`: `decimal` -> `decimal?`
- `RefundTransactionResponse.SystemWalletBalanceAfter`: `decimal` -> `decimal?`

**Response semantic**

- `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter` vẫn tồn tại nhưng trả `null`
- `RefundTransactionResponse.SystemWalletBalanceBefore/After` vẫn tồn tại nhưng có thể trả `null`
- incident refundability được backend tính từ:
  - `SnakebiteIncidentPayment - SnakebiteIncidentRefund`

**Client action**

- treat toàn bộ `SystemWalletBalance*` field của incident flow là nullable
- không hiển thị incident payment/refund như escrow state nữa
- nếu transaction UI đang filter theo `EscrowHold` / `EscrowRelease` để biểu diễn incident money movement, logic đó phải bỏ

## 2026-04-08 - Consultation escrow switched to transaction-sourced ledger

**Status:** `response semantic change`

**Scope**

- `POST /api/consultations/scheduled/{bookingId:guid}/payments`
- `POST /api/consultations/instant/{requestId:guid}/payments`
- `POST /api/consultations/payments/confirm`
- consultation PayOS confirm/webhook path
- `ConsultationPaymentResponse`

**Change**

- consultation vẫn là escrow flow
- nhưng consultation không còn tạo/update system wallet để biểu diễn escrow
- consultation hold không còn tạo `EscrowHold`
- consultation refund/settlement không còn tạo `EscrowRelease`

**Response semantic**

- `ConsultationPaymentResponse.SystemWalletBalanceAfter` vẫn tồn tại và vẫn nullable
- field này trả `null`

**Client action**

- không dùng `SystemWalletBalanceAfter` để suy luận escrow amount nữa
- nếu transaction UI đang filter theo `EscrowHold` / `EscrowRelease` để biểu diễn consultation escrow, logic đó phải đổi sang transaction domain types của consultation
