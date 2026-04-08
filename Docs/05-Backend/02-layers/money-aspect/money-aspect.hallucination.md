# Money Aspect Research Buckets

## Purpose

File này không dùng để chốt thiết kế.

File này dùng để gom các điểm còn chưa chốt thành các cụm research question, để khi soi codebase có thể đi theo từng mảng thay vì nhảy giữa 20 quyết định rời rạc.

Nguyên tắc:

- research trước
- chốt decision sau
- chỉ update `refactoring.md` và `sourcemap.md` khi đã có kết luận đủ chắc

---

File này chỉ giữ:

- bucket chưa khóa để phục vụ research tiếp
- decision đã chốt của bucket đã khóa

Bucket nào đã khóa thì xóa phần research scaffolding để tránh doc phình to.

## Current Direction Summary

| Flow | Before | After |
|---|---|---|
| Consultation | ambiguity cũ xoay quanh việc có còn dùng `system wallet` để hold/release hay không | đã khóa: consultation là escrow thật, nhưng ledger-driven |
| Snakebite Incident | ambiguity lớn nhất là có phải escrow flow giống consultation không | đã khóa: không phải escrow; là ledger-only system revenue |
| Snake Catching | ambiguity lớn nhất là còn `transfer-to-rescuer` và commission split hay không | đã khóa và implement hoàn tất trong Phase 6D: không phải escrow; customer money path là ledger-only system revenue |
| Snake Catching Payment Path | hallucination cũ là wallet payment / PayOS confirm phải tạo `EscrowHold` và credit `system.wallet` | đã resolve trong 6D1: payment path là ledger-only system/platform revenue; phần pending còn lại nằm ở settlement/refund |

## Bucket A. Current Ownership Map

### Decision đã chốt

- `WalletPaymentService` không phải service của `WalletTopup`
- logic trong `WalletPaymentService` sẽ được absorb vào `SnakeCatchingPaymentService`
- `WalletTopup` sẽ giữ owner service riêng, không absorb vào `SnakeCatchingPaymentService`
- `consultation` được dùng làm baseline pattern chính
- `snakebite incident` được xem là đã follow baseline này ở tầng ownership, không phải primary ownership defect

---

## Bucket B. PayOS Callback Routing

### Fact đã verify

- cả `confirm manual`, `return`, `webhook` đều được dispatch trực tiếp trong `PayOsController`
- 3 đường vào khác nhau ở input lookup, nhưng cùng hội tụ về `description prefix`
- `confirm manual` đi vào bằng `transactionId`, rồi lookup ra `description`
- `return` đi vào bằng `orderCode`, rồi lookup ra `description`
- `webhook` đi vào bằng payload đã verify, rồi dùng luôn `description`
- `PayOsDescriptionLookup` hiện chỉ recognize 3 prefix:
  - `SNAKEAID-`
  - `INCIDENT-`
  - `CONSULTPAY-`
- `wallet topup` hiện đang build `SNAKEAID-`, nên callback bị route nhầm vào owner của snake catching
- `consultation` đang dùng `CONSULTPAY-` nhất quán
- `snakebite incident` đang dùng `INCIDENT-` nhất quán
- `snake catching` đang dùng `SNAKEAID-` nhất quán
- `SNAKEAID-` đang xuất hiện ở service logic, lookup logic, test, và helper script; đổi prefix này có blast radius cao hơn tách prefix riêng cho topup

### Decision đã chốt từ bucket này

- không tạo router abstraction riêng
- callback dispatch tiếp tục nằm trong `PayOsController`
- dispatch rule cốt lõi tiếp tục dựa trên `description prefix`
- prefix final của `wallet topup` là `TOPUP-`
- prefix final của `snake catching` là `CATCHING-`
- chấp nhận migration cost để đổi từ `SNAKEAID-` sang `CATCHING-` nhằm lấy lại trật tự prefix theo flow
- `manual confirm` tiếp tục nhận `transactionId`
- `return` tiếp tục nhận `orderCode`
- `manual confirm` và `return` phải hội tụ về cùng processing path sau bước lookup ban đầu
- owner flow luôn được resolve cuối cùng bằng `description prefix`

---

## Bucket C. Wallet Movement And Transaction Semantics

### Decision đã chốt

- `TransactionType` tiếp tục là enum phân biệt loại giao dịch theo business type
- `Transaction` không có lifecycle status riêng; business status tiếp tục thuộc domain entity của từng flow
- `ExternalTransactionId` được hiểu là external reference bridge; việc nó đang bị dùng như processing marker là hiện trạng cần được kiểm soát, không phải semantic đích
- semantic cleanup của `TransactionType.WalletTopup` và `TransactionType.WalletWithdraw` sẽ làm ngay trong lượt refactor này
- không được tiếp tục sinh thêm code mới dùng `WalletTopup` như generic system credit hoặc `WalletWithdraw` như generic system payout/refund source
- quyết định shared crosscutting (`MoneyEscrowService`, `MoneyLedgerService`, `MoneyTransferService`) được dời về phase cuối, sau khi 4 flow đã được chuẩn hóa ownership và semantics
- không khóa abstraction shared dựa trên hiện trạng méo của codebase trước khi chuẩn hóa flow

---

## Bucket D. Primitive Duplication And Shared Layer Boundary

### Phase 3 re-verify từ code hiện tại

- `ConsultationPaymentService.MoveMoneyToEscrowAsync` và `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync` vẫn là duplication thật mạnh nhất:
  - cùng debit user wallet khi `PaymentMethod == "Wallet"`
  - cùng credit system wallet như escrow
  - cùng reuse pending payment transaction khi `skipExistingPaymentInsert == true`
  - cùng insert system-side credit transaction bằng `TransactionType.WalletTopup`
  - cùng return tuple balance/id/timestamp/external reference
- khác biệt giữa 2 method này hiện nằm ở domain transaction type, reference semantic, description wording, và exception message.
- `WalletTopupService.ProcessConfirmedPaymentAsync` không thuộc escrow duplication vì chỉ credit user wallet sau PayOS topup.
- `SnakeCatchingPaymentService.CreateWalletPaymentAsync` có shape gần escrow nhưng vẫn trộn domain state update `IsPrePaid`, `PrePaidAt`, `Status`, nên không đủ sạch để extract shared primitive từ đây trong phase 3.
- `SnakeCatchingPaymentService.ProcessWebhookCoreAsync` vẫn chứa credit system wallet, paired system transaction, catching status update, commission branch, và legacy `WalletTopup` branch; đây là coupling residue cần giữ trong owner flow, không kéo vào shared layer.
- `WalletPaymentService` / `IWalletPaymentService` không còn xuất hiện trong repo backend hiện tại; wallet payment của snake catching đã thuộc `SnakeCatchingPaymentService`.
- route leak `POST /api/wallet/payment` không còn xuất hiện trong repo backend hiện tại; route wallet payment hiện là `POST /api/snakecatching/payment/wallet`.

### Fact đã verify

- `ConsultationPaymentService.MoveMoneyToEscrowAsync` và `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync` gần như là cùng một primitive, khác chủ yếu ở:
  - transaction type domain thật
  - description wording
  - reference naming
- consultation có thêm 2 primitive riêng sau escrow:
  - `RefundFromEscrowAsync`
  - `TransferEscrowToExpertAsync`
- snakebite incident hiện có refund pattern gần consultation, nhưng chưa có settlement/payout tương ứng
- snake catching chưa có `MoveMoneyToEscrowAsync` clean primitive tương đương; money movement đang bị trộn trong:
  - PayOS webhook confirm path
  - `HandleCatcherCommissionAsync`
  - `HandleWalletTopupAsync`
  - `TransferSnakeCatchingFundsToRescuerAsync`
  - `RefundSnakeCatchingTransactionAsync`
- `TransferSnakeCatchingFundsToRescuerAsync` không chỉ là payout primitive:
  - nó aggregate từ nhiều paid transaction của cùng catching request
  - tự trừ `commissionFee`
  - tự tạo `PlatformFee`
  - tự update `SnakeCatchingRequest.Status = Completed`
- `RefundSnakeCatchingTransactionAsync` của snake catching gần refund pattern của consultation/incident ở tầng wallet movement, nhưng vẫn nằm trong domain-specific flow API và semantics
- `HandleCatcherCommissionAsync` là logic domain-specific của snake catching, không nên shared ra money primitive layer
- `HandleWalletTopupAsync` là logic tuyệt đối không được shared với 3 flow domain vì semantic của topup khác escrow/payment
- `wallet topup` không nên dùng chung primitive escrow với 3 flow domain, vì semantic của nó là nạp tiền vào ví user chứ không phải đưa tiền vào domain escrow

### Decision đã chốt

- `wallet topup` không dùng chung primitive escrow với 3 flow domain
- commission của `snake catching` là domain-specific logic, không đưa vào shared primitive layer
- payout của `snake catching` giữ trong flow owner; không shared trong lượt refactor hiện tại
- refund của `snake catching` dù có wallet movement gần consultation/incident thì vẫn giữ trong flow owner ở lượt refactor hiện tại
- `consultation` và `snakebite incident` tiếp tục giữ refund/payout/settlement logic trong owner service hiện tại; lượt refactor này chỉ align semantic cần thiết, không trích shared primitive mới từ các nhánh đó
- candidate shared mạnh nhất về mặt lý thuyết hiện tại là `escrow primitive`, nhưng lượt refactor này không tạo shared escrow abstraction mới
- lượt refactor hiện tại không tạo shared refund abstraction
- lượt refactor hiện tại không tạo shared payout abstraction
- cleanup logic tiền ở phase hiện tại phải diễn ra bên trong flow owner, không mở thêm crosscutting layer mới
- quyết định shared boundary cuối cùng vẫn defer tới phase cuối sau khi flow đã được chuẩn hóa và route migration đã ổn định

### Kết luận bucket này

- mọi quyết định actionable của bucket này cho lượt refactor hiện tại đã được khóa
- các câu hỏi còn lại về `escrow only` hay `ledger pair`, hoặc refund shared tới đâu, được chuyển hẳn sang phase cuối và không còn là ambiguity cản tiến độ của implementation hiện tại
---

## Bucket E. Public API And Client Impact

### Fact đã verify

- client-visible payment entrypoints hiện tại gồm:
  - `POST /api/wallet/topup`
  - `POST /api/wallet/payment`
  - `POST /api/snakecatching/payment/create-link`
  - `POST /api/snakecatching/payment/cancel-link/{orderCode}`
  - `POST /api/snakecatching/payment/transfer-to-rescuer`
  - `POST /api/consultations/scheduled/{bookingId}/payments`
  - `POST /api/consultations/instant/{requestId}/payments`
  - `POST /api/consultations/payments/confirm`
  - `POST /api/incidents/{incidentId}/payment/payos`
  - `POST /api/incidents/{incidentId}/payment/wallet`
  - `DELETE /api/incidents/{incidentId}/payment/payos/{orderCode}`
  - `POST /api/incidents/{incidentId}/payment/refund`
- `TransactionController` public response đang expose trực tiếp các field nhạy cảm với refactor:
  - `TransactionType`
  - `Description`
  - `PaymentMethod`
  - `ExternalTransactionId`
  - `CreatedAt`
- topup response hiện expose:
  - `TransactionId`
  - `Amount`
  - `Status`
  - `CheckoutUrl`
  - `OrderCode`
  - `PaymentLinkId`
  - `Provider`
  - `GatewayRawResponse`
- snake catching payment response hiện expose:
  - `TransactionId`
  - `SnakeCatchingRequestId`
  - `Amount`
  - `Status`
  - `CheckoutUrl`
  - `OrderCode`
  - `PaymentLinkId`
  - `Provider`
  - `GatewayRawResponse`
- consultation payment response expose thêm:
  - `PaymentMethod`
  - `UserWalletBalanceAfter`
  - `SystemWalletBalanceAfter`
  - `PaidAtUtc`
  - `ExternalTransactionId`
- incident payment response expose thêm:
  - `ExternalTransactionId`
  - `UserWalletBalanceAfter`
  - `SystemWalletBalanceAfter`
  - `PaidAt`
- shape response giữa 4 flow hiện chưa thống nhất; client đang thấy implementation differences thật sự
- `WalletController.POST /api/wallet/payment` là API snake catching wallet payment nhưng đang nằm dưới wallet namespace; đây là implementation leak cũ ra public API
- `ConsultationPaymentsController` có manual confirm endpoint public dưới role `User`; đây là client-visible contract thật, không thể xem như internal-only

### Ambiguity còn lại

### Decision đã chốt

- public API contract được phép đổi route/response ngay trong lượt refactor này
- route leak `POST /api/wallet/payment` sẽ bị xóa; flow snake catching phải chuyển hẳn sang route đúng domain
- `TransactionController` giữ nguyên response contract trong suốt lượt refactor; semantic cleanup nội bộ không được làm breaking API ngay tại controller này
- `manual confirm` và `return` giữ contract ngoài như hiện tại; chỉ unify processing path bên trong
- chưa cố đồng nhất response shape của 4 flow trong lượt này; chỉ giữ backward compatibility ở những chỗ đã public
- nếu có thay đổi lớn ở phía frontend-facing route/DTO, bắt buộc phải có documentation hướng dẫn frontend migration
- doc migration phải chỉ rõ:
  - route cũ -> route mới
  - request DTO cũ -> request DTO mới
  - response DTO cũ -> response DTO mới
  - thay đổi calling flow nếu có

### Decision phụ thuộc bucket này

- `13. Mức đổi contract API/client-visible`
- `16. transactionId hay orderCode là key resolve chính`

---

## Bucket F. Safe Refactor Scope

### Fact đã verify

- test coverage hiện lệch mạnh giữa các flow, không đồng đều cho một big-bang refactor:
  - `consultation` có integration coverage thật cho wallet payment, PayOS pending, manual confirm, refund, và idempotent settlement
  - `snakebite incident` có unit test + property test khá dày, nhưng thiếu integration coverage tương đương consultation
  - `snake catching` chủ yếu có preservation/property tests và route contract tests; gần như không có service-level integration coverage tương đương consultation
  - `wallet topup` hiện gần như không có unit/integration coverage riêng; chủ yếu mới có bash endpoint scripts
- `SnakeCatchingPaymentsController` đã tồn tại dưới route đúng domain `api/snakecatching/payment`, nhưng `WalletController.POST /api/wallet/payment` vẫn còn sống và vẫn inject `IWalletPaymentService`
- điều này cho thấy route migration của snake catching đang ở trạng thái dual-path, chưa clean cut
- coverage hiện có đủ để xem `consultation` là executable baseline cho pattern target
- coverage hiện có không đủ mạnh để vừa absorb `WalletPaymentService`, vừa cleanup semantic ledger, vừa extract shared primitive cho cả 4 flow trong cùng một pass mà vẫn an toàn
- rủi ro cao nhất của lượt refactor này nằm ở `wallet topup` và `snake catching`, vì đây là nơi vừa có ownership defect vừa có public route migration

### Decision đã chốt

- lượt refactor này không làm full structural rewrite cho cả 4 flow trong cùng một pass
- trọng tâm implementation full sẽ là `wallet topup` và `snake catching`, vì đây là 2 flow có ownership defect thật sự
- `consultation` được giữ làm baseline/reference implementation; chỉ align những điểm semantic cần thiết, không làm deep structural rewrite trong lượt này
- `snakebite incident` chỉ align theo baseline và semantic cleanup cần thiết; không mở rộng scope sang deep rewrite hoặc primitive extraction trong lượt này
- `WalletPaymentService` sẽ bị absorb vào `SnakeCatchingPaymentService` trong lượt refactor này, nhưng theo kiểu move-and-stabilize, không redesign thêm abstraction mới xung quanh nó
- quyết định shared primitive vẫn defer tới phase cuối; lượt này không được đồng thời vừa chuẩn hóa flow vừa tạo shared crosscutting layer
- số lượng class production mới phải giữ ở mức tối thiểu; chỉ tạo class mới khi nó trực tiếp giải quyết ownership/route defect, không tạo thêm service trung gian vì tiện tay

### Decision phụ thuộc bucket này

- `5. Số lượng class mới tối đa được phép`
- `6. WalletPaymentService sẽ bị xử lý thế nào`
- `7. Mức refactor của snake catching`
- `8. Mức refactor của consultation và snakebite incident`
---

## Bucket G. Structure Placement And Naming

### Fact đã verify

- convention chính của repo hiện tại là:
  - interface đặt trong `SnakeAid.Service/Interfaces`
  - implementation đặt trong `SnakeAid.Service/Implements`
  - naming theo mẫu `I{Name}Service` / `{Name}Service`
- các payment owner service hiện tại đều đang follow convention này:
  - `IConsultationPaymentService` / `ConsultationPaymentService`
  - `ISnakebiteIncidentPaymentService` / `SnakebiteIncidentPaymentService`
  - `ISnakeCatchingPaymentService` / `SnakeCatchingPaymentService`
  - `IWalletTopupService` / `WalletTopupService`
  - `IWalletPaymentService` / `WalletPaymentService`
- repo hiện không có pattern payment subfolder riêng trong `Interfaces` hoặc `Implements`; payment service vẫn đứng ngang hàng với service khác
- `WalletTopupService` theo naming hiện tại khớp convention repo hơn `WalletTopupPaymentService`
- `IWalletPaymentService` / `WalletPaymentService` đang là tên gây nhiễu semantic:
  - method duy nhất là `CreateWalletPaymentAsync`
  - request/response lại là snake catching DTO
  - controller gọi nó từ `WalletController.POST /api/wallet/payment`
  - nhưng business thật của nó là snake catching wallet payment, không phải generic wallet payment
- `Program.cs` hiện đăng ký rõ `ISnakeCatchingPaymentService` và `IWalletTopupService`
- kết quả grep hiện không thấy đăng ký DI cho `IWalletPaymentService` trong `Program.cs`, trong khi `WalletController` vẫn inject interface này; đây là dấu hiệu cấu trúc cũ chưa được clean up triệt để hoặc registration đang nằm ngoài vùng payment registration hiện tại
- target docs hiện còn chứa một số giả định naming/placement chưa khớp với scope đã chốt ở các bucket trước:
  - `WalletTopupPaymentService`
  - `MoneyEscrowService`
  - `MoneyLedgerService`
  - `MoneyTransferService`
- trong khi Bucket C, D, F đã chốt rằng shared crosscutting chưa được tạo ở lượt refactor hiện tại và số class mới phải giữ tối thiểu

### Decision đã chốt

- giữ naming `WalletTopupService`; không tạo hay rename sang `WalletTopupPaymentService`
- `WalletPaymentService` và `IWalletPaymentService` sẽ bị xóa hẳn; logic được chuyển hết về `SnakeCatchingPaymentService`
- nếu cần compatibility trong quá trình code move, đó chỉ là trạng thái tạm implementation, không phải target-state để ghi vào doc kiến trúc
- file placement list trong doc chỉ là tham chiếu ưu tiên, không phải danh sách bắt buộc cứng
- placement thực tế được phép linh hoạt miễn vẫn bám convention repo và không tạo structure rối hơn hiện trạng

### Kết luận bucket này

- naming target-state bám theo convention repo hiện có, không cố phát minh tên mới cho cùng một vai trò
- owner service của `wallet topup` giữ tên hiện tại
- owner service của snake catching hấp thụ toàn bộ wallet payment logic và xóa lớp naming gây nhiễu cũ
---

## Bucket H. Documentation Mode

### Decision đã chốt

- `money-aspect.sourcemap.md` giữ các diagram ở `target-state`
- `implementation-state` không thay diagram hiện tại, mà được ghi bằng note cho biết hệ thống đã implement tới đâu so với target
- cả `money-aspect.refactoring.md` và `money-aspect.sourcemap.md` đều là `decision-only`
- không để doc mở ra các nhánh quyết định mới trong lúc implementation
- mọi thay đổi quyết định kiến trúc phải được chốt trước trong doc, không đổi ngầm trong code

---

## Bucket I. Phase 6/7 Transaction-Sourced Escrow And Consultation Platform Fee

### Fact đã verify

- `ConsultationPaymentService` còn dùng `SystemWalletUserId` và update system wallet balance trong:
  - `MoveMoneyToEscrowAsync`
  - `RefundFromEscrowAsync`
  - `TransferEscrowToExpertAsync`
- Phase 6B update: `ConsultationPaymentService` đã bỏ `SystemWalletUserId` và không còn update/validate system wallet trong consultation hold/refund/settlement path
- Phase 6B update: `ConsultationPaymentResponse.SystemWalletBalanceAfter` vẫn tồn tại nhưng trả `null` cho consultation escrow responses
- Phase 6C corrective update: `SnakebiteIncidentPaymentService` đã bỏ `SystemWalletUserId` và không còn update/validate system wallet trong incident payment/refund path
- Phase 6C corrective update: `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter` vẫn tồn tại nhưng trả `null` cho incident payment responses
- Phase 6C update: `RefundTransactionResponse.SystemWalletBalanceBefore/After` chuyển sang nullable và trả `null` cho incident refunds
- Business correction 2026-04-08: chỉ consultation dùng escrow + net payout + platform fee
- Business correction 2026-04-08: incident và catching là payment một chiều vào system/platform; không release qua rescuer vì rescuer là nhân viên system
- Business correction 2026-04-08: implementation dùng ledger-only system revenue transaction cho incident/catching; admin analytics đọc từ `Transaction`, không từ `system.wallet`
- Business correction 2026-04-08: Phase 6C corrective review done; incident now uses ledger-only system revenue wording/validation
- Phase 6E update: `SnakeCatchingPaymentService` không còn production usage của system wallet; phần còn lại chỉ là response compatibility fields nullable/null cho deprecated payout và refund responses
- DTO public còn expose `SystemWalletBalance*`:
  - `ConsultationPaymentResponse.SystemWalletBalanceAfter`
  - `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter`
  - `RefundTransactionResponse.SystemWalletBalanceBefore`
  - `RefundTransactionResponse.SystemWalletBalanceAfter`
  - `TransferToRescuerResponse.SystemWalletBalanceBefore`
  - `TransferToRescuerResponse.SystemWalletBalanceAfter`
- consultation settlement hiện cộng 100% amount cho expert wallet và chỉ tạo `ExpertPayout`; chưa tạo `PlatformFee`
- Phase 6E update: `commissionFee` hardcoded của snake catching đã bị xóa cùng deprecated settlement semantics
- repo đã có `SystemSettingKeys` và `ISystemSettingService`, phù hợp để thêm platform fee percent cho consultation

### Decision đã chốt từ bucket này

- Phase 6 chỉ xử lý transaction-sourced escrow cho consultation; incident/catching phải được đưa về system/platform revenue semantics
- Phase 6 không implement consultation platform fee
- Phase 7 không nên dùng `EscrowRelease` generic cho consultation settlement nếu release được biểu diễn bằng `PlatformFee` + `ExpertPayout`
- consultation platform fee default là `20%` nếu system setting chưa tồn tại
- rounding rule ưu tiên expert: tính `expertNetAmount` rồi làm tròn lên theo đơn vị VND; `platformFeeAmount = grossAmount - expertNetAmount`
- client cần thấy fee breakdown khi settlement/response có contract liên quan consultation payout hoặc transaction detail
- Phase 6E done: `EscrowHold` / `EscrowRelease` đã bị xóa sau khi production logic không còn sử dụng
- Phase 6 target-state phải được sửa: transaction-sourced escrow chỉ áp dụng cho consultation; incident/catching cần ledger-only system/platform revenue semantics riêng
- Phase 6D không còn bị block bởi revenue representation: catching phải dùng ledger-only system/platform revenue và không được mô tả như `transfer-to-rescuer` hoặc marketplace payout flow
- `money-aspect.changelog.md` là nơi tracking mọi thay đổi front-facing liên quan `SystemWalletBalance*`, fee breakdown, hoặc transaction type exposure

### Ambiguity còn lại

- chưa có ambiguity đã biết sau input 2026-04-08

---

## Research Checklist

Khi research codebase, nên tick theo bucket thay vì theo decision rời:

- [x] Bucket A done
- [x] Bucket B done
- [x] Bucket C done
- [x] Bucket D done
- [x] Bucket E done
- [x] Bucket F done
- [x] Bucket G done
- [x] Bucket H done
- [x] Bucket I done
