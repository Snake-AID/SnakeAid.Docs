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

### Fact đã verify

- `Transaction` hiện có `TransactionType` để phân biệt loại giao dịch theo enum
- `Transaction` ko có lifecycle status riêng cho pending/confirmed/failed/cancelled business status sẽ do domain entity của từng flow quản lý
- `ExternalTransactionId` hiện là external reference bridge, đồng thời trong nhiều flow còn bị dùng như processing marker để suy ra payment đã được gateway confirm hay chưa
- `WalletTopupService` tạo pending transaction type `WalletTopup`, sau đó khi callback thành công thì topup side-effect lại đi nhờ snake catching
- `SnakeCatchingPaymentService` khi xử lý PayOS thành công sẽ:
  - credit `system wallet`
  - tạo thêm transaction type `WalletTopup` cho phía system
- `WalletPaymentService` khi xử lý wallet payment cho snake catching sẽ:
  - debit user wallet
  - tạo transaction phía user bằng type domain thật (`CatchingPayment` / `CatchingDeposit` / ...)
  - credit `system wallet`
  - tạo transaction phía system bằng `TransactionType.WalletTopup`
- `ConsultationPaymentService.MoveMoneyToEscrowAsync` và `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync` có pattern gần như giống nhau:
  - nếu là wallet payment thì debit user wallet
  - luôn credit `system wallet`
  - transaction payment chính giữ type domain thật (`ConsultationPayment` / `SnakebiteIncidentPayment`)
  - transaction credit phía system lại dùng `TransactionType.WalletTopup`
- refund/settlement ở consultation và incident đang dùng pattern khá rõ:
  - transaction nguồn rời system wallet dùng `TransactionType.WalletWithdraw`
  - transaction đích dùng type domain thật như `ConsultationRefund`, `ExpertPayout`, `SnakebiteIncidentRefund`
- vì vậy `TransactionType.WalletTopup` hiện đang bị reuse như generic “money vào system wallet”, không còn mang nghĩa thuần là “user nạp tiền”
- tương tự, `TransactionType.WalletWithdraw` đang bị reuse như generic “money ra khỏi system wallet”, không còn mang nghĩa thuần là “user rút tiền”
- snake catching còn lẫn thêm topup logic trong `HandleWalletTopupAsync`, làm semantic của `WalletTopup` càng mờ hơn

### Ambiguity còn lại

- semantic cleanup của `TransactionType.WalletTopup` và `TransactionType.WalletWithdraw` sẽ làm ngay trong lượt refactor này
- quyết định shared crosscutting (`MoneyEscrowService`, `MoneyLedgerService`, `MoneyTransferService`) được dời về phase cuối, sau khi 4 flow đã được chuẩn hóa ownership và semantics
- không khóa abstraction dựa trên hiện trạng méo của codebase trước khi chuẩn hóa flow

### Decision phụ thuộc bucket này

- `1. Shared money module có tồn tại hay không`
- `2. Bộ class shared nào là bắt buộc`
- `3. MoneyTransferService có tồn tại hay không`
- `12. Transaction semantic cleanup có nằm trong scope hay không`
- `15. payment intent hay chỉ pending transaction`
- `17. refund và payout có được chuẩn hóa chung không`

---

## Bucket D. Primitive Duplication And Shared Layer Boundary

### Mục tiêu

Xác định đâu là duplication thật, đâu là domain-specific logic không được trích nhầm.

### Câu hỏi cần trả lời

- `MoveMoneyToEscrowAsync` ở consultation và incident giống nhau đến mức nào
- snake catching có primitive tương đương hay khác bản chất
- topup có dùng chung được primitive nào với 3 flow domain, và primitive nào tuyệt đối không nên shared
- ledger creation có giống nhau giữa các flow không

### Khu vực code cần soi

- các hàm `MoveMoneyToEscrowAsync`
- các đoạn create transaction pending / mark confirmed
- các đoạn wallet balance mutation
- các đoạn refund / payout / commission logic

### Decision phụ thuộc bucket này

- `1. Shared money module có tồn tại hay không`
- `2. Bộ class shared nào là bắt buộc`
- `3. MoneyTransferService có tồn tại hay không`
- `17. refund và payout có được chuẩn hóa chung không`

---

## Bucket E. Public API And Client Impact

### Mục tiêu

Chốt blast radius ra bên ngoài trước khi refactor.

### Câu hỏi cần trả lời

- mobile/client hiện đang phụ thuộc vào endpoint nào cho topup, wallet payment, transaction refresh
- client có đọc `transactionType`, `description`, `orderCode`, `checkoutUrl`, `status` trực tiếp không
- callback refactor có làm thay đổi response shape hay polling flow không
- có API nào public nhưng thực chất đang phản ánh implementation leak cũ không

### Khu vực code cần soi

- `WalletController.cs`
- `TransactionController.cs`
- `ConsultationPaymentsController.cs`
- `SnakeCatchingPaymentsController.cs`
- `SnakebiteIncidentController.cs`
- docs hiện có trong `SnakeAid.Docs`

### Decision phụ thuộc bucket này

- `13. Mức đổi contract API/client-visible`
- `16. transactionId hay orderCode là key resolve chính`

---

## Bucket F. Safe Refactor Scope

### Mục tiêu

Chốt refactor lượt này sẽ đi xa đến đâu mà vẫn an toàn.

### Câu hỏi cần trả lời

- có cần làm full 4 flow trong một lượt implementation không
- topup + catching có đủ lớn để tách thành trọng tâm chính không
- consultation và incident có đang ổn đủ để chỉ dùng làm baseline
- có test coverage hay integration verification đủ để absorb `WalletPaymentService` ngay không

### Khu vực code cần soi

- test project
- regression tests liên quan payment / webhook / wallet
- controller/service call graph

### Decision phụ thuộc bucket này

- `5. Số lượng class mới tối đa được phép`
- `6. WalletPaymentService sẽ bị xử lý thế nào`
- `7. Mức refactor của snake catching`
- `8. Mức refactor của consultation và snakebite incident`

---

## Bucket G. Structure Placement And Naming

### Mục tiêu

Chốt placement và naming theo convention codebase, tránh sinh abstraction lệch phong cách repo.

### Câu hỏi cần trả lời

- repo hiện có pattern rõ cho service owner, helper service, adapter service hay chưa
- flow owner mới nên là class mới hay mở rộng class cũ
- shared primitive nếu có nên đặt cùng `Implements`/`Interfaces` hay tạo subfolder riêng
- naming nào sát domain nhất: `WalletTopupService` hay `WalletTopupPaymentService`

### Khu vực code cần soi

- `SnakeAid.Service/Interfaces`
- `SnakeAid.Service/Implements`
- convention của các service payment hiện tại

### Decision phụ thuộc bucket này

- `4. WalletTopupPaymentService có tạo mới hay không`
- `5. Số lượng class mới tối đa được phép`
- `19. File placement list là bắt buộc hay tham chiếu`

---

## Bucket H. Documentation Mode

### Decision đã chốt

- `money-aspect.sourcemap.md` giữ các diagram ở `target-state`
- `implementation-state` không thay diagram hiện tại, mà được ghi bằng note cho biết hệ thống đã implement tới đâu so với target
- cả `money-aspect.refactoring.md` và `money-aspect.sourcemap.md` đều là `decision-only`
- không để doc mở ra các nhánh quyết định mới trong lúc implementation
- mọi thay đổi quyết định kiến trúc phải được chốt trước trong doc, không đổi ngầm trong code

---

## Research Checklist

Khi research codebase, nên tick theo bucket thay vì theo decision rời:

- [x] Bucket A done
- [x] Bucket B done
- [ ] Bucket C done
- [ ] Bucket D done
- [ ] Bucket E done
- [ ] Bucket F done
- [ ] Bucket G done
- [x] Bucket H done
