# Money Aspect API Breaking Change Note

## Purpose

File này là note migration cho frontend/mobile sau Phase 6.

Mục tiêu:

- chỉ ra các breaking change hoặc semantic break của money flows
- giúp mobile dev biết code cũ sẽ sai ở đâu
- chỉ rõ cách sửa code để follow hành vi backend mới

File này không phải roadmap nội bộ.

## Release Scope

Phase 6 là một breaking change lớn cho toàn bộ money semantics:

- consultation vẫn là escrow, nhưng không còn biểu diễn bằng `system wallet`
- snakebite incident không còn là escrow flow; chuyển thành ledger-only system revenue
- snake catching không còn là escrow-to-rescuer flow; chuyển thành ledger-only system revenue
- escrow transitional transaction types đã bị xóa khỏi backend:
  - `EscrowHold`
  - `EscrowRelease`

## What Mobile Must Stop Assuming

Sau Phase 6, mobile/frontend không được tiếp tục assume:

- `system wallet balance` là source of truth cho escrow hoặc revenue
- incident payment có trạng thái escrow
- snake catching payment/refund là escrow flow
- `transfer-to-rescuer` là payout trigger thật
- `EscrowHold` / `EscrowRelease` còn xuất hiện trong transaction list
- `transType=system` còn chứa movement của escrow

## Current Contract Snapshot

| Flow | New contract |
|---|---|
| Consultation | vẫn là escrow; escrow phải được suy ra từ domain transactions; `SystemWalletBalanceAfter` vẫn tồn tại nhưng trả `null` |
| Snakebite Incident | payment/refund là ledger-only system revenue; wallet payment status là `Paid`; `SystemWalletBalance*` là nullable semantic field và trả `null` |
| Snake Catching | payment/refund là ledger-only system revenue; `transfer-to-rescuer` chỉ còn compatibility no-op; không còn rescuer transfer thật |

## Breaking Changes Summary

### 1. Transaction exposure changed

**What changed**

- backend đã xóa `EscrowHold` và `EscrowRelease` khỏi `TransactionType`
- `GET /api/transactions` không còn trả 2 transaction type này
- `TransactionService` group `system` không còn include escrow transitional types

**What breaks in mobile**

- code filter transaction theo `EscrowHold` / `EscrowRelease`
- UI render escrow state từ 2 transaction type này
- code assume `transType=system` có chứa escrow movement

**What mobile must do**

- xóa mapping/filter dựa trên `EscrowHold` / `EscrowRelease`
- không dùng `transType=system` để hiển thị escrow nữa
- render money state bằng domain transaction types thật của từng flow

### 2. Consultation changed from system-wallet-style escrow to ledger-driven escrow

**Scope**

- `POST /api/consultations/scheduled/{bookingId:guid}/payments`
- `POST /api/consultations/instant/{requestId:guid}/payments`
- `POST /api/consultations/payments/confirm`
- consultation PayOS confirm/webhook path
- `ConsultationPaymentResponse`

**What changed**

- consultation vẫn là escrow flow
- nhưng backend không còn tạo/update `system wallet` để biểu diễn escrow
- consultation không còn tạo:
  - `EscrowHold`
  - `EscrowRelease`

**Response behavior**

- `ConsultationPaymentResponse.SystemWalletBalanceAfter` vẫn còn field
- field này trả `null`

**What breaks in mobile**

- code dùng `SystemWalletBalanceAfter` để hiển thị escrow amount
- UI dùng `EscrowHold` / `EscrowRelease` để mô tả consultation escrow state

**What mobile must do**

- treat `SystemWalletBalanceAfter` là nullable compatibility field
- không dùng field này làm source of truth
- nếu cần hiển thị consultation money state, phải bám vào domain semantics của consultation thay vì system wallet

### 3. Snakebite Incident is no longer an escrow flow

**Scope**

- `POST /api/incidents/{incidentId}/payment/wallet`
- incident PayOS confirm/webhook path
- `POST /api/incidents/{incidentId}/payment/refund`
- `SnakebiteIncidentPaymentResponse`
- `RefundTransactionResponse`

**What changed**

- incident payment không còn là escrow
- incident payment được ghi nhận như system/platform revenue
- wallet payment status đổi:
  - `Escrowed` -> `Paid`
- incident không còn tạo:
  - `EscrowHold`
  - `EscrowRelease`

**Response behavior**

- `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter` trả `null`
- `RefundTransactionResponse.SystemWalletBalanceBefore/After` là nullable và trả `null`
- incident refundability được backend tính từ:
  - `SnakebiteIncidentPayment - SnakebiteIncidentRefund`

**What breaks in mobile**

- UI/state machine đang chờ incident wallet payment status = `Escrowed`
- UI hiển thị incident payment/refund như escrow state
- code dùng `SystemWalletBalance*` như field business thật

**What mobile must do**

- đổi logic status của incident wallet payment sang `Paid`
- treat toàn bộ `SystemWalletBalance*` của incident flow là nullable compatibility field
- bỏ toàn bộ wording/logic escrow cho incident

### 4. Snake Catching is no longer an escrow-to-rescuer flow

**Scope**

- snake catching wallet payment
- snake catching PayOS confirm/webhook path
- snake catching refund path
- `SnakeCatchingPaymentResponse`
- `PayOsWebhookResponse`
- `RefundTransactionResponse`

**What changed**

- snake catching payment không còn credit `system.wallet`
- snake catching payment không còn tạo `EscrowHold`
- snake catching refund không còn tạo `EscrowRelease`
- snake catching refundability được backend tính từ:
  - `CatchingPayment + CatchingDeposit - CatchingRefund`
- wallet payment `GatewayRawResponse` không còn expose `SystemWalletBalance`
- `PayOsWebhookResponse.Status` của snake catching confirm/webhook giờ trả explicit:
  - `Paid`
  - `Failed`

**Response behavior**

- `RefundTransactionResponse.SystemWalletBalanceBefore/After` trả `null` cho snake catching refund
- `GatewayRawResponse.SystemWalletBalance` không còn tồn tại

**What breaks in mobile**

- code parse `GatewayRawResponse.SystemWalletBalance`
- code suy luận catching payment state qua escrow semantics
- UI hiển thị snake catching refund như escrow release

**What mobile must do**

- bỏ parse `GatewayRawResponse.SystemWalletBalance`
- dùng `PayOsWebhookResponse.Status` mới để render confirm/webhook state
- bỏ toàn bộ wording/logic escrow cho snake catching payment/refund

### 5. `transfer-to-rescuer` is deprecated and no longer performs payout

**Scope**

- `POST /api/snakecatching/payment/transfer-to-rescuer`
- `TransferToRescuerResponse`

**What changed**

- endpoint vẫn còn để giữ compatibility
- endpoint không còn trigger payout
- endpoint không còn tạo:
  - `CatcherPayout`
  - `PlatformFee`
  - `EscrowRelease`
- endpoint không còn đụng `system.wallet` hoặc rescuer wallet

**Response DTO changes**

- `TransferTransactionId`: `Guid` -> `Guid?`
- `SystemWalletBalanceBefore`: `decimal` -> `decimal?`
- `SystemWalletBalanceAfter`: `decimal` -> `decimal?`
- `RescuerWalletBalanceBefore`: `decimal` -> `decimal?`
- `RescuerWalletBalanceAfter`: `decimal` -> `decimal?`

**Response behavior**

- `Success` vẫn có thể là `true`
- `TransferTransactionId` có thể là `null`
- các balance field có thể là `null`
- `NetAmountToRescuer = 0`
- `Message` nêu rõ endpoint đã deprecated và không còn thực hiện transfer

**What breaks in mobile**

- code xem endpoint này là payout trigger
- UI assume luôn có `TransferTransactionId`
- UI assume luôn có system/rescuer wallet balances

**What mobile must do**

- không dùng endpoint này như business action payout nữa
- handle nullable cho toàn bộ transfer/balance fields
- nếu UI đang hiển thị transfer receipt, phải handle trường hợp không còn transfer transaction

## Mobile Migration Checklist

- [ ] bỏ toàn bộ mapping/filter dùng `EscrowHold`
- [ ] bỏ toàn bộ mapping/filter dùng `EscrowRelease`
- [ ] bỏ assumption `transType=system` chứa escrow movement
- [ ] consultation: không dùng `SystemWalletBalanceAfter` để suy luận escrow amount
- [ ] incident: đổi wallet payment status expectation từ `Escrowed` sang `Paid`
- [ ] incident: bỏ wording/logic escrow
- [ ] snake catching: bỏ parse `GatewayRawResponse.SystemWalletBalance`
- [ ] snake catching: bỏ wording/logic escrow
- [ ] snake catching: không dùng `transfer-to-rescuer` như payout trigger
- [ ] handle nullable cho:
  - `TransferTransactionId`
  - `SystemWalletBalanceBefore`
  - `SystemWalletBalanceAfter`
  - `RescuerWalletBalanceBefore`
  - `RescuerWalletBalanceAfter`
  - incident/snake catching `SystemWalletBalance*`

## Priority For Mobile Fix

Ưu tiên sửa theo thứ tự:

1. transaction filtering / transaction UI
2. incident payment status handling
3. snake catching payment/refund screens
4. snake catching transfer-to-rescuer screens
5. consultation screens còn dùng `SystemWalletBalanceAfter`

## Final Note

Nếu mobile code vẫn follow semantics cũ của Phase 5 trở về trước, app có nguy cơ:

- hiển thị sai payment state
- hiển thị sai escrow state
- chờ status không còn tồn tại
- parse field đã bị null hóa hoặc removed
- trigger sai business flow ở snake catching

Phase 6 phải được xem là một **contract migration bắt buộc** cho toàn bộ money-related client code.
