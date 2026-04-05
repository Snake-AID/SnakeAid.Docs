# Money Aspect Research Buckets

## Purpose

File này không dùng để chốt thiết kế.

File này dùng để gom các điểm còn chưa chốt thành các cụm research question, để khi soi codebase có thể đi theo từng mảng thay vì nhảy giữa 20 quyết định rời rạc.

Nguyên tắc:

- research trước
- chốt decision sau
- chỉ update `refactoring.md` và `sourcemap.md` khi đã có kết luận đủ chắc

## Research Order

Đề xuất thứ tự research:

1. current state và ownership thật sự của 4 flow
2. callback routing và prefix dependency
3. wallet movement và transaction semantics
4. duplication của money primitive
5. blast radius lên API contract và client flow
6. mức refactor an toàn cho từng flow
7. structure code mới và placement
8. documentation mode của `sourcemap`

---

## Bucket A. Current Ownership Map

### Mục tiêu

Chốt flow nào hiện đang sở hữu:

- create payment
- confirm payment
- webhook processing
- wallet balance update
- domain side-effect
- payout/refund

### Câu hỏi cần trả lời

- `wallet topup` hiện thật sự dừng ở `WalletTopupService` hay đã đi sâu vào service khác ở pha callback
- `snake catching` đang split ownership ở những điểm nào
- `consultation` và `snakebite incident` có thật sự self-contained hay vẫn đang mượn primitive hoặc status handling ở nơi khác
- `WalletPaymentService` đang phục vụ chính xác flow nào

### Khu vực code cần soi

- `SnakeAid.Api/Controllers/WalletController.cs`
- `SnakeAid.Api/Controllers/PayOsController.cs`
- `SnakeAid.Api/Controllers/SnakeCatchingPaymentsController.cs`
- `SnakeAid.Api/Controllers/ConsultationPaymentsController.cs`
- `SnakeAid.Api/Controllers/SnakebiteIncidentController.cs`
- `SnakeAid.Service/Implements/WalletTopupService.cs`
- `SnakeAid.Service/Implements/WalletPaymentService.cs`
- `SnakeAid.Service/Implements/SnakeCatchingPaymentService.cs`
- `SnakeAid.Service/Implements/ConsultationPaymentService.cs`
- `SnakeAid.Service/Implements/SnakebiteIncidentPaymentService.cs`

### Decision phụ thuộc bucket này

- `4. WalletTopupPaymentService có tạo mới hay không`
- `6. WalletPaymentService sẽ bị xử lý thế nào`
- `7. Mức refactor của snake catching`
- `8. Mức refactor của consultation và snakebite incident`

---

## Bucket B. PayOS Callback Routing

### Mục tiêu

Chốt callback đang đi như thế nào và prefix nào thật sự là key dispatch ổn định.

### Câu hỏi cần trả lời

- `PayOsController` đang route theo `description prefix`, `orderCode`, hay transaction lookup kiểu kết hợp
- callback, return, confirm manual có cùng logic dispatch không
- prefix hiện tại có chỗ nào bị reuse sai semantic không
- snake catching có bắt buộc phải giữ `SNAKEAID-` vì backward compatibility hay không
- topup có thể tách sang prefix riêng mà không phá flow hiện tại không

### Khu vực code cần soi

- `SnakeAid.Api/Controllers/PayOsController.cs`
- các hàm build description / order code trong:
  - `WalletTopupService.cs`
  - `SnakeCatchingPaymentService.cs`
  - `ConsultationPaymentService.cs`
  - `SnakebiteIncidentPaymentService.cs`
- integration test liên quan PayOS routing

### Decision phụ thuộc bucket này

- `9. PayOsController dispatch bằng gì`
- `10. Prefix cuối cùng của 4 flow`
- `11. CATCHING- hay giữ SNAKEAID-`
- `16. transactionId hay orderCode là key resolve chính`

---

## Bucket C. Wallet Movement And Transaction Semantics

### Mục tiêu

Chốt hệ thống hiện đang model tiền như thế nào, đặc biệt là các thao tác tăng/giảm số dư ví và semantic của transaction records.

### Câu hỏi cần trả lời

- chỗ nào đang cộng tiền vào ví
- chỗ nào đang trừ số dư ví để thanh toán
- chỗ nào đang chuyển tiền vào escrow
- chỗ nào đang mark confirmed / pending / failed
- `TransactionType.WalletTopup` có đang bị reuse làm generic credit event không
- payout và refund đang dùng cùng semantic transaction hay có type riêng

### Khu vực code cần soi

- entity / enum / repository liên quan transaction và wallet
- `WalletService.cs`
- `WalletTopupService.cs`
- `WalletPaymentService.cs`
- `ConsultationPaymentService.cs`
- `SnakebiteIncidentPaymentService.cs`
- `SnakeCatchingPaymentService.cs`
- các query/list API transaction

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

### Mục tiêu

Chốt `sourcemap` sẽ đóng vai trò gì sau khi implementation bắt đầu.

### Câu hỏi cần trả lời

- `sourcemap` dùng để mô tả target state hay implementation state
- diagram có cần update theo từng phase hay chỉ update khi kết thúc
- `refactoring.md` có giữ hoàn toàn decision-only hay cho phép progress notes ngắn

### Khu vực code cần soi

- không phụ thuộc code runtime nhiều
- phụ thuộc cách team muốn review và resume

### Decision phụ thuộc bucket này

- `18. money-aspect.sourcemap.md là target-state hay implementation-state`
- `20. Mức độ decision-only của 2 doc`

---

## Bucket To Decision Mapping

| Bucket | Decisions |
|---|---|
| A. Current Ownership Map | 4, 6, 7, 8 |
| B. PayOS Callback Routing | 9, 10, 11, 16 |
| C. Wallet Movement And Transaction Semantics | 1, 2, 3, 12, 15, 17 |
| D. Primitive Duplication And Shared Layer Boundary | 1, 2, 3, 17 |
| E. Public API And Client Impact | 13, 16 |
| F. Safe Refactor Scope | 5, 6, 7, 8 |
| G. Structure Placement And Naming | 4, 5, 19 |
| H. Documentation Mode | 18, 20 |

## Research Checklist

Khi research codebase, nên tick theo bucket thay vì theo decision rời:

- [ ] Bucket A done
- [ ] Bucket B done
- [ ] Bucket C done
- [ ] Bucket D done
- [ ] Bucket E done
- [ ] Bucket F done
- [ ] Bucket G done
- [ ] Bucket H done

## Output Format Sau Mỗi Bucket

Sau khi research xong một bucket, output nên theo format ngắn:

1. fact đã verify từ code
2. ambiguity còn lại
3. decision nào giờ đã đủ dữ kiện để chốt
4. impact lên `refactoring.md` và `sourcemap.md`
