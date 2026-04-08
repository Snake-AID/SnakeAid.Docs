# Money Aspect Refactoring

## Background

### Document mode

File này là `decision-only`.

Nó chỉ chứa:

- decision đã chốt
- roadmap thực hiện
- tracking tiến độ bám theo decision đã chốt

Nó không dùng để:

- brainstorm
- mở thêm phương án giữa chừng
- ghi tạm các cân nhắc chưa verify

Mọi ambiguity mới phát sinh trong quá trình research hoặc implementation phải được đưa sang `money-aspect.hallucination.md` trước khi được nâng cấp thành decision trong file này.

### Vấn đề

Hiện tại logic tiền tệ của backend đang phân tán theo nhiều service và chưa thống nhất ownership theo flow.

Các vấn đề chính:

- `wallet topup` không có callback/webhook handler riêng.
- `wallet topup` đang dùng prefix `SNAKEAID-` nên bị PayOS router điều hướng sang `SnakeCatchingPaymentService`.
- `snake catching` đang chia logic tiền sang nhiều nơi:
  - `SnakeCatchingPaymentService`
  - `WalletPaymentService`
- `consultation` và `snakebite incident` đã gần với pattern tốt hơn, nhưng phần primitive tiền vẫn đang lặp lại.
- Một số transaction `WalletTopup` hiện đang được dùng như ledger event chung cho nhiều ngữ cảnh khác nhau, làm mờ ranh giới domain.

### Phân biệt 2 loại logic wallet

`wallet` trong codebase hiện có 2 ngữ cảnh khác nhau và không được trộn lẫn:

- `wallet topup`: nạp tiền vào ví người dùng. Đây là inflow vào user wallet.
- `wallet payment` trong `consultation`, `snakebite incident`, `snake catching`: dùng số dư ví như một payment method để trả cho domain flow. Đây là outflow từ user wallet sang escrow hoặc settlement path của domain.

Nói ngắn gọn:

- `topup` làm tăng số dư ví
- `incident/catching/consultation` wallet flow làm giảm số dư ví để thanh toán

Vì vậy:

- `topup` không được dùng chung callback owner với 3 flow domain
- logic payment bằng wallet của 3 flow domain không được gộp semantic với topup

### Hiện trạng theo flow

#### 1. Wallet topup

- Entry: `POST /api/wallet/topup`
- Tạo transaction pending trong `WalletTopupService`
- Tạo PayOS link
- Khi PayOS callback/webhook về, flow này không tự xử lý
- Việc confirm hiện đi nhờ router `SNAKEAID-*` và side-effect được xử lý trong `SnakeCatchingPaymentService.HandleWalletTopupAsync`

Đây là coupling sai ownership và là điểm ưu tiên xử lý đầu tiên.

#### 2. Snake catching

- PayOS flow ở `SnakeCatchingPaymentService`
- Wallet flow lại nằm ở `WalletPaymentService`
- Một domain nhưng money logic đang split ownership

Đây là flow rối nhất về tổ chức code.

#### 3. Consultation

- Có service owner tương đối rõ
- PayOS và Wallet cùng hội tụ về `MoveMoneyToEscrowAsync`
- Flow confirm/webhook do chính consultation payment service xử lý

Đây là flow gần target pattern nhất.

#### 4. Snakebite incident

- Có service owner tương đối rõ
- PayOS và Wallet cùng hội tụ về `MoveMoneyToEscrowAsync`
- Có cả refund trong cùng domain flow

Đây cũng là flow gần target pattern.

## Mục tiêu

### Mục tiêu chính

Thống nhất toàn bộ money aspect theo một pattern rõ ràng:

- mỗi money flow có owner service riêng
- không flow nào dùng tạm callback/processor của flow khác
- PayOS routing phân biệt được flow một cách deterministic
- money movement sẽ được chuẩn hóa theo owner flow; shared primitive chỉ được trích ở phase cuối nếu pattern đủ mạnh và không làm mờ domain side-effect
- transaction/ledger rõ nghĩa, không dùng nhòe semantic giữa các flow

### Quang cảnh sau refactor

Sau refactor, money aspect phải có trạng thái cuối như sau:

- `wallet topup` là một flow độc lập, có prefix riêng, callback owner riêng, và chỉ chịu trách nhiệm nạp tiền vào ví user
- `snake catching` là một flow độc lập, toàn bộ money logic của nó nằm dưới owner service của chính catching, không còn split qua service ngoài flow
- `consultation` vẫn giữ ownership theo flow hiện tại và chỉ align semantic cần thiết trong lượt refactor này
- `snakebite incident` vẫn giữ ownership theo flow hiện tại và chỉ align semantic cần thiết trong lượt refactor này

Sau khi hoàn tất:

- 4 flow đều đi qua cùng một pattern lifecycle
- 4 flow không còn mượn callback hoặc processor của nhau
- phần dùng chung chỉ là money primitive
- phần domain state và business side-effect vẫn thuộc từng flow owner
- `WalletPaymentService` không còn tồn tại như một owner riêng; logic của nó được absorb vào `SnakeCatchingPaymentService`

### Mục tiêu cụ thể

- tách `wallet topup` khỏi `snake catching`
- gom money logic của `snake catching` về một owner thống nhất
- chuẩn hóa prefix routing cho các flow PayOS
- chuẩn hóa contract xử lý các pha:
  - create intent
  - confirm/webhook
  - move money
  - apply domain state
  - payout/refund nếu có
- giảm lặp code primitive tiền mà không làm domain bị trộn lẫn
- cleanup semantic của `TransactionType.WalletTopup` và `TransactionType.WalletWithdraw` ngay trong chính lượt refactor này
- nếu đổi front-facing route/DTO thì phải có doc migration cho frontend trong cùng lượt refactor

## Nguyên tắc Codebase

### 1. Ownership rõ theo flow

Mỗi flow phải có service owner riêng cho toàn bộ vòng đời payment của flow đó.

Không để:

- topup dùng callback handler của snake catching
- snake catching wallet payment nằm ở service ngoài domain owner

Decision đã chốt:

- `WalletTopup` giữ owner riêng
- logic của `WalletPaymentService` được absorb vào `SnakeCatchingPaymentService`
- `consultation` là baseline pattern chính
- `snakebite incident` được xem là đã follow baseline ở tầng ownership

### 2. Shared primitive, không shared domain side-effect

Được phép dùng chung các primitive kiểu:

- move money to escrow
- credit/debit system wallet
- create paired ledger entries
- transfer payout
- refund transfer

Không dùng chung:

- rule xác nhận business state
- route callback theo flow
- status transition đặc thù domain

### 3. Prefix routing phải unique

Mỗi flow PayOS phải có prefix riêng:

- `TOPUP-`
- `CATCHING-`
- `INCIDENT-`
- `CONSULTPAY-`

Decision đã chốt:

- `wallet topup` dùng `TOPUP-`
- `snake catching` dùng `CATCHING-`
- chấp nhận migration cost để loại bỏ `SNAKEAID-` và lấy lại naming trật tự theo flow
- không reuse prefix giữa các flow

### 4. Domain state update phải nằm sau money confirmation đúng owner

Sau khi payment được xác nhận:

- owner service của flow đó mới được cập nhật state domain
- không service khác thay mặt update domain state

### 5. Refactor theo bước nhỏ, luôn giữ behavior đang chạy

Không big bang rewrite.

Mỗi bước phải:

- có scope nhỏ
- có test/verification rõ
- có khả năng dừng và resume
- nếu có breaking change front-facing thì phải có migration doc đi kèm

### 6. Không tạo thêm service “tiện tay”

Nếu tạo service mới, phải trả lời rõ:

- service này owner flow nào
- primitive nào là shared
- dependency nào là inward và dependency nào là outward

Tránh tạo thêm abstraction mơ hồ làm codebase khó đọc hơn.

## Giải pháp xử lý

### Target pattern

Mỗi flow tiền tệ chuẩn hóa theo cấu trúc:

1. `CreatePaymentIntent`
- validate domain state
- validate amount
- tạo pending transaction hoặc payment intent
- trả checkout url nếu là PayOS

2. `ConfirmPayment`
- nhận input từ entrypoint hiện tại:
  - manual confirm đi bằng `transactionId`
  - return đi bằng `orderCode`
  - webhook đi bằng payload đã verify
- manual confirm và return phải hội tụ về cùng processing path sau bước lookup ban đầu
- resolve owner flow cuối cùng bằng `description prefix`
- verify trạng thái gateway
- idempotency guard

3. `ApplyMoneyMovement`
- thực hiện money movement trong owner flow
- chỉ trích shared primitive ở phase cuối nếu pattern đã đủ rõ
- không khóa trước `MoneyEscrowService` / `MoneyLedgerService` / `MoneyTransferService` trong target hiện tại

4. `ApplyDomainSideEffects`
- update status/domain record của chính flow đó
- gửi notification nếu flow cần

5. `PostPaymentActions`
- payout
- commission
- refund

### Shared module strategy

Decision đã chốt:

- cleanup semantic của `TransactionType.WalletTopup` và `TransactionType.WalletWithdraw` được làm ngay trong lượt refactor này
- quyết định cuối cùng về shared crosscutting chỉ được chốt ở phase cuối, sau khi 4 flow đã được chuẩn hóa ownership và semantics
- không khóa shared abstraction dựa trên hiện trạng code chưa chuẩn hóa
- lượt refactor hiện tại không tạo `MoneyTransferService`
- `MoneyEscrowService` và `MoneyLedgerService` mới chỉ là candidate để đánh giá lại ở phase cuối, không phải target-state đã chốt

Nếu phase cuối chứng minh là cần, layer shared chỉ được xử lý money movement mức thấp và không được biết domain state.

### Vị trí class/service nếu tạo mới

Nếu cần tạo mới trong repo, vị trí phải bám convention hiện tại:

- interface đặt tại `SnakeAid.Service/Interfaces`
- implementation đặt tại `SnakeAid.Service/Implements`
- controller giữ mỏng và tiếp tục đặt tại `SnakeAid.Api/Controllers`

Các class owner chắc chắn thuộc target-state hiện tại:

- `SnakeAid.Service/Interfaces/IWalletTopupService.cs`
- `SnakeAid.Service/Implements/WalletTopupService.cs`
- `SnakeAid.Service/Interfaces/ISnakeCatchingPaymentService.cs`
- `SnakeAid.Service/Implements/SnakeCatchingPaymentService.cs`
- `SnakeAid.Service/Interfaces/IConsultationPaymentService.cs`
- `SnakeAid.Service/Implements/ConsultationPaymentService.cs`
- `SnakeAid.Service/Interfaces/ISnakebiteIncidentPaymentService.cs`
- `SnakeAid.Service/Implements/SnakebiteIncidentPaymentService.cs`

Nếu phase cuối chứng minh shared primitive là cần, placement của `MoneyEscrowService` / `MoneyLedgerService` sẽ follow cùng convention repo nhưng chưa được coi là target-state bắt buộc ở thời điểm hiện tại.

Lưu ý:

- đây là placement tham chiếu ưu tiên, không phải danh sách bắt buộc cứng
- placement thực tế được phép linh hoạt miễn vẫn bám convention repo
- chỉ tạo class khi refactor thực tế chứng minh là cần
- chỉ chốt shared class sau phase chuẩn hóa flow

### Flow owner đề xuất

- `WalletTopupService`
- `SnakeCatchingPaymentService`
- `ConsultationPaymentService`
- `SnakebiteIncidentPaymentService`

Trong target state:

- `WalletPaymentService` được absorb vào `SnakeCatchingPaymentService`
- `PayOsController` chỉ làm routing theo prefix

### PayOS routing đề xuất

Giữ `PayOsController` là entrypoint callback chung.

Routing trong `PayOsController` phải explicit:

- parse prefix
- map prefix -> flow owner
- flow owner tự confirm/process webhook

Không tạo thêm router abstraction riêng.

Nguyên tắc là:

- `PayOsController` chỉ nhận request và dispatch theo prefix
- `PayOsController` không chứa domain side-effect
- flow owner service mới là nơi confirm/process webhook và apply business logic
- routing key cuối cùng là `description prefix`
- manual confirm, return, webhook được phép có input khác nhau nhưng phải hội tụ về cùng rule dispatch này
- `manual confirm` tiếp tục nhận `transactionId`
- `return` tiếp tục nhận `orderCode`

### Documentation structure cho code mới

Để tránh plan doc phình thành tech debt, documentation được tách 2 vai trò:

- `money-aspect.refactoring.md`: plan, scope, mục tiêu, roadmap, tracking
- `money-aspect.sourcemap.md`: sơ đồ structure mới, class ownership, function graph, sequence diagram

`sourcemap` là structural memory cho:

- developer đọc nhanh architecture mới
- reviewer đối chiếu ownership
- agent resume mà không phải re-parse lại toàn bộ code

## Roadmap

### Phase 0. Freeze hiện trạng

Trạng thái: `DONE`

Checklist:

- [x] chốt danh sách 4 flow tiền tệ cần quản lý
- [x] chốt prefix hiện tại của từng flow
- [x] chốt entrypoints controller/service hiện tại
- [x] chốt các integration test đang có
- [x] bổ sung regression notes cho topup đang đi nhờ snake catching

Resume output mong muốn:

- có bảng mapping flow -> controller -> service -> prefix -> callback handler

### Phase 0 output

| Flow | Primary controller entry | Current owner service | Prefix hiện trạng | Callback/confirm handler hiện trạng |
|---|---|---|---|---|
| Wallet Topup | `WalletController.POST /api/wallet/topup` | `WalletTopupService` | `SNAKEAID-` | đang đi nhờ `PayOsController` dispatch sang `SnakeCatchingPaymentService.HandleWalletTopupAsync` |
| Snake Catching | `SnakeCatchingPaymentsController`, `WalletController.POST /api/wallet/payment` | `SnakeCatchingPaymentService` + `WalletPaymentService` | `SNAKEAID-` | `PayOsController` dispatch về snake catching |
| Consultation | `ConsultationPaymentsController` | `ConsultationPaymentService` | `CONSULTPAY-` | `PayOsController` dispatch về consultation |
| Snakebite Incident | `SnakebiteIncidentController` | `SnakebiteIncidentPaymentService` | `INCIDENT-` | `PayOsController` dispatch về incident |

### Phase 1. Tách wallet topup khỏi snake catching

Trạng thái: `DONE`

Mục tiêu:

- topup có prefix riêng
- topup có confirm/webhook handler riêng
- topup không còn gọi gián tiếp vào `SnakeCatchingPaymentService`

Checklist:

- [x] tạo prefix riêng cho topup
- [x] hoàn thiện `WalletTopupService` để tự sở hữu callback/confirm
- [x] cập nhật PayOS router
- [x] thêm test routing cho topup
- [x] verify topup credit đúng wallet user

Done khi:

- topup không còn phụ thuộc vào `SnakeCatchingPaymentService.HandleWalletTopupAsync`

### Phase 2. Ownership repair and callback isolation

Mục tiêu của phase này:

- xử lý full ownership defect ở `wallet topup` và `snake catching`
- không mở rộng cùng lúc sang structural rewrite sâu cho `consultation` và `snakebite incident`

Checklist:

- [x] đổi prefix `wallet topup` sang `TOPUP-`
- [x] đổi prefix `snake catching` sang `CATCHING-`
- [x] để `PayOsController` dispatch theo `description prefix` mới mà không tạo router abstraction
- [x] thêm hoặc hoàn thiện callback/confirm handling đúng owner cho `wallet topup`
- [x] absorb logic của `WalletPaymentService` vào `SnakeCatchingPaymentService`
- [x] xóa route leak `POST /api/wallet/payment`
- [x] chuyển snake catching wallet payment sang route đúng domain
- [x] viết doc migration cho frontend nếu route/DTO thay đổi
- [x] giữ `consultation` và `snakebite incident` ổn định, chỉ align semantic tối thiểu cần thiết trong phase này

Migration note cho frontend/mobile:

- endpoint tạo snake catching wallet payment đổi từ `POST /api/wallet/payment` sang `POST /api/snakecatching/payment/wallet`
- request DTO giữ nguyên `CreateSnakeCatchingPaymentRequest`, chỉ đổi route owner
- PayOS description của snake catching đổi từ `SNAKEAID-{orderCode}` sang `CATCHING-{orderCode}`
- wallet topup đổi PayOS description từ `SNAKEAID-{orderCode}` sang `TOPUP-{orderCode}`
- nếu client đang parse prefix cũ hoặc hardcode route cũ thì phải migrate cùng release backend này

### Phase 3. Trích shared money primitives

Trạng thái: `DONE`

Mục tiêu:

- chưa chốt abstraction shared
- chỉ chuẩn bị dữ kiện để đánh giá shared primitive ở phase cuối

Checklist:

- [x] xác định phần lặp giữa các `MoveMoneyToEscrowAsync`
- [x] xác định phần nào là duplication thật, phần nào chỉ là coupling tạm thời do flow chưa chuẩn hóa
- [x] ghi lại blast radius nếu trích shared primitive quá sớm

Done khi:

- đủ dữ kiện để chốt abstraction shared ở phase cuối, không quyết định sớm

### Phase 3 output

Kết luận code-verified:

- duplication thật mạnh nhất nằm giữa `ConsultationPaymentService.MoveMoneyToEscrowAsync` và `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync`
- phần lặp gồm:
  - lấy hoặc tạo system wallet
  - nếu payment method là `Wallet` thì debit user wallet và validate số dư
  - nếu payment method là `PayOS` thì giữ nguyên user wallet balance hiện tại
  - credit system wallet như escrow
  - insert hoặc reuse payment transaction theo `skipExistingPaymentInsert`
  - insert paired system credit transaction
  - trả về `TransactionId`, user wallet balance sau xử lý, system wallet balance sau xử lý, timestamp, external transaction id
- khác biệt còn lại giữa consultation và incident chủ yếu là:
  - `TransactionType` domain payment
  - `ReferenceId` semantic (`booking/request/consultation` vs `incident`)
  - wording description
  - exception message
  - caller side-effect sau escrow
- đây là candidate shared primitive hợp lý về mặt kỹ thuật, nhưng chưa extract trong phase này vì phase 3 chỉ chốt dữ kiện, không chốt abstraction

Không phải duplication nên extract ngay:

- `WalletTopupService.ProcessConfirmedPaymentAsync` chỉ credit user wallet; semantic là topup inflow, không phải escrow primitive
- `SnakeCatchingPaymentService.CreateWalletPaymentAsync` debit user wallet, credit system wallet, tạo paired transactions, rồi update snake catching domain state trong cùng owner flow; hình dạng giống escrow nhưng còn trộn business transition `IsPrePaid`, `PrePaidAt`, `Status`
- `SnakeCatchingPaymentService.ProcessWebhookCoreAsync` credit system wallet và tạo system credit transaction, nhưng vẫn chứa PayOS idempotency, catching status update, commission branch, và legacy `WalletTopup` branch
- `SnakeCatchingPaymentService.TransferSnakeCatchingFundsToRescuerAsync` không chỉ là payout primitive; nó aggregate paid catching transactions, trừ commission, tạo `PlatformFee`, tạo `CatcherPayout`, và chuyển request sang `Completed`
- `SnakeCatchingPaymentService.RefundSnakeCatchingTransactionAsync`, `ConsultationPaymentService.RefundFromEscrowAsync`, và incident refund có movement tương tự nhau, nhưng transaction type, public API, owner side-effect, và refund semantics chưa đủ sạch để trích shared refund primitive trong lượt này

Blast radius nếu extract shared primitive quá sớm:

- shared primitive có thể vô tình encode `WalletTopup` như generic system escrow credit, trong khi phase cuối vẫn chưa cleanup semantic ledger
- có thể kéo domain state transition của snake catching vào shared layer, trái rule shared primitive không biết domain side-effect
- có thể làm mờ ranh giới topup inflow với domain payment escrow/outflow
- có thể thay đổi response balance fields của consultation/incident nếu return tuple hoặc commit timing không giữ nguyên
- có thể làm PayOS idempotency guard và `skipExistingPaymentInsert` bị áp dụng sai giữa manual confirm, return, webhook, và wallet payment
- có thể tăng rủi ro test regression vì coverage giữa 4 flow không đồng đều; consultation có baseline tốt hơn, snake catching còn nhiều logic domain-specific trong cùng service

Decision sau phase 3:

- không tạo `MoneyEscrowService`, `MoneyLedgerService`, `MoneyTransferService`, hoặc shared refund/payout primitive ở phase này
- giữ candidate shared mạnh nhất là escrow primitive giữa consultation và incident để đánh giá lại ở phase 5
- nếu phase 5 tạo shared primitive, boundary chỉ được xử lý wallet movement và paired ledger insert mức thấp; domain state update vẫn nằm ở owner service

### Phase 4. Chuẩn hóa PayOS flow router

Trạng thái: `DONE`

Mục tiêu:

- routing theo prefix rõ ràng
- confirm/webhook/manual confirm dùng cùng nguyên tắc dispatch

Checklist:

- [x] chuẩn hóa prefix constants
- [x] gom routing logic về một chỗ
- [x] thêm test uniqueness của prefix
- [x] thêm test router dispatch đúng owner

Done khi:

- không còn hardcode mơ hồ theo description prefix ở nhiều nơi

### Phase 4 output

Kết luận code-verified:

- PayOS prefix đã được gom về `PayOsPaymentFlowPrefixes` với 4 flow:
  - `Topup` -> `TOPUP-`
  - `SnakeCatching` -> `CATCHING-`
  - `SnakebiteIncident` -> `INCIDENT-`
  - `Consultation` -> `CONSULTPAY-`
- `PayOsController` không còn tự hardcode chuỗi prefix trong các nhánh dispatch chính; controller dùng `PayOsPaymentFlowPrefixes.TryResolve(...)` rồi dispatch theo `PayOsPaymentFlow`
- `ConfirmPayment`, `Webhook`, và `ConfirmByOrderCodeAsync` hiện dùng cùng rule resolve prefix trước khi gọi owner service
- `PayOsDescriptionLookup` vẫn giữ query dạng explicit `StartsWith(...)` để tránh rủi ro EF translation, nhưng các prefix trong query lấy từ cùng constants
- Các service owner đã dùng cùng constants khi build/parse PayOS description:
  - `WalletTopupService`
  - `SnakeCatchingPaymentService`
  - `SnakebiteIncidentPaymentService`
  - `ConsultationPaymentService`
- grep production code cho literal `TOPUP-`, `CATCHING-`, `INCIDENT-`, `CONSULTPAY-`, `SNAKEAID-` chỉ còn match ở file constants `PayOsPaymentFlowPrefixes.cs`

Test/verification:

- thêm `PayOsPaymentFlowPrefixesTests` để verify prefix uniqueness, mapping flow -> prefix, resolve description -> flow, và build prefix theo order code
- mở rộng `PayOsTopupRoutingTests` để verify `ConfirmPayment`, `Webhook`, và `ConfirmByOrderCodeAsync` dispatch đúng owner cho cả 4 flow
- `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "FullyQualifiedName~PayOs"` pass `92/92`
- full `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj` hiện fail `2/202` bởi lỗi ngoài scope PayOS:
  - `ShiftServiceTests.GetAssignmentsByDateAsync_ShouldIncludeOvernightShiftFromPreviousDay`: EF mapping `Point.UserData`
  - `ScheduledConsultationIntegrationTests.CreateScheduledBookingAsync_ShouldReserveSlot_AndCreateBookingAndConsultation`: slot test đã bắt đầu

### Phase 5. Ledger semantics cleanup and shared crosscutting decision

Trạng thái: `DONE`

Mục tiêu phase cuối:

- chốt semantic ledger sau khi flow ownership đã ổn định
- chỉ lúc này mới quyết định shared primitive boundary

Checklist:

- [x] cleanup hết chỗ còn dùng `WalletTopup` như generic system credit
- [x] cleanup hết chỗ còn dùng `WalletWithdraw` như generic payout/refund source
- [x] đối chiếu lại 4 flow sau khi chuẩn hóa ownership
- [x] đánh giá lại có cần `MoneyEscrowService` hay không
- [x] đánh giá lại có cần `MoneyLedgerService` hay không
- [x] không tạo `MoneyTransferService` nếu pattern shared chưa tự chứng minh đủ mạnh

### Phase 5 output

Kết luận code-verified:

- thêm `TransactionType.EscrowHold = 34` cho system wallet giữ tiền trong escrow
- thêm `TransactionType.EscrowRelease = 35` cho system wallet giải phóng tiền khỏi escrow
- `TransactionType.WalletTopup` hiện chỉ còn được dùng bởi `WalletTopupService` cho user-facing wallet topup, và trong `TransactionService` system filter
- `TransactionType.WalletWithdraw` hiện chỉ còn được dùng bởi `WalletWithdrawService` cho user-facing withdrawal, và trong `TransactionService` system filter
- các system wallet ledger entry đã đổi semantic:
  - consultation escrow source -> `EscrowHold`
  - consultation refund/settlement source -> `EscrowRelease`
  - snakebite incident escrow source -> `EscrowHold`
  - snakebite incident refund source -> `EscrowRelease`
  - snake catching system escrow source -> `EscrowHold`
  - snake catching payout/refund source -> `EscrowRelease`
- `SnakeCatchingPaymentService` không còn legacy branch `HandleWalletTopupAsync`; topup callback owner vẫn là `WalletTopupService`
- `TransactionService` system transaction group đã include `EscrowHold` và `EscrowRelease`

Shared primitive decision:

- không tạo `MoneyTransferService`
- không tạo `MoneyEscrowService` trong lượt này
- không tạo `MoneyLedgerService` trong lượt này
- lý do:
  - ledger semantic cleanup đã xử lý nguồn gây nhòe nghĩa chính của phase 5
  - candidate shared mạnh nhất vẫn là escrow primitive giữa consultation và incident, nhưng extraction sẽ đổi nhiều private flow contract cùng lúc sau khi enum semantic vừa thay đổi
  - snake catching vẫn có domain side-effect riêng trong money path, nên chưa nên kéo vào shared primitive
  - giữ duplication cục bộ ở consultation/incident hiện an toàn hơn tạo shared abstraction mới chưa bắt buộc

Test/verification:

- thêm assertion trong `ConsultationPaymentIntegrationTests` để verify escrow ledger dùng `EscrowHold` / `EscrowRelease`, đồng thời không tạo system `WalletTopup` / `WalletWithdraw` cho escrow source
- `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj --filter "ConsultationPaymentIntegrationTests|SnakebiteIncidentPaymentServiceTests|WalletTopupServiceTests|WalletWithdrawServiceTests|WalletWithdrawalFlowIntegrationTests|PayOs"` pass `115/115`
- grep production code xác nhận `TransactionType.WalletTopup` / `TransactionType.WalletWithdraw` chỉ còn ở đúng owner service và system transaction grouping
- full `dotnet test SnakeAid.Tests/SnakeAid.Tests.csproj` vẫn fail `2/202` bởi cùng 2 lỗi ngoài scope PayOS/money-aspect đã ghi ở Phase 4:
  - `ShiftServiceTests.GetAssignmentsByDateAsync_ShouldIncludeOvernightShiftFromPreviousDay`: EF mapping `Point.UserData`
  - `ScheduledConsultationIntegrationTests.CreateScheduledBookingAsync_ShouldReserveSlot_AndCreateBookingAndConsultation`: slot test đã bắt đầu

### Phase 6. Transaction-sourced escrow ledger

Trạng thái: `TODO`

Mục tiêu:

- bãi bỏ `System Wallet` như két sắt cố định của escrow
- chuyển escrow balance từ wallet side-effect sang ledger được suy ra từ `Transaction`
- giữ rule `Transaction` là NoSQL-style ledger entity: `TransactionType` xác định semantic, `ReferenceId` xác định entity đích
- không làm platform fee consultation trong phase này; chỉ chuẩn hóa nguồn sự thật của escrow trước

Decision đã chốt cho phase này:

- không còn tăng/giảm balance của account `system.wallet` khi hold/release escrow
- không dùng system wallet balance để validate refund/payout; thay bằng transaction-sourced escrow availability theo `ReferenceId`
- không tạo schema migration vì dữ liệu dev có thể drop; nếu enum/domain contract đổi thì code và tests là nguồn sự thật
- `SystemWalletBalance*` là field front-facing nhạy cảm; nếu đổi hoặc bỏ phải ghi vào `money-aspect.changelog.md`

Research finding trước implementation:

- `ConsultationPaymentService.MoveMoneyToEscrowAsync`, `RefundFromEscrowAsync`, `TransferEscrowToExpertAsync` đang trực tiếp tạo/lấy system wallet và update balance
- `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync` và `RefundSnakebiteIncidentTransactionAsync` cũng phụ thuộc system wallet balance
- `SnakeCatchingPaymentService` phụ thuộc system wallet ở cả wallet payment, PayOS webhook, refund, transfer-to-rescuer, và response balance
- front-facing response hiện đang expose system wallet fields:
  - `ConsultationPaymentResponse.SystemWalletBalanceAfter`
  - `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter`
  - `RefundTransactionResponse.SystemWalletBalanceBefore`
  - `RefundTransactionResponse.SystemWalletBalanceAfter`
  - `TransferToRescuerResponse.SystemWalletBalanceBefore`
  - `TransferToRescuerResponse.SystemWalletBalanceAfter`

Target ledger equation:

- hold source là payment transaction thật của flow:
  - consultation: `ConsultationPayment`
  - snake catching: `CatchingPayment` / `CatchingDeposit`
  - snakebite incident: `SnakebiteIncidentPayment`
- release/refund sinks là domain transaction thật của flow:
  - consultation: `ExpertPayout`, `ConsultationRefund`, và Phase 7 sẽ thêm `PlatformFee`
  - snake catching: `CatcherPayout`, `CatchingRefund`, `PlatformFee`
  - snakebite incident: `SnakebiteIncidentRefund`
- escrow available amount theo `ReferenceId` = paid/held amount - released/refunded/fee amount
- `EscrowHold` / `EscrowRelease` là transitional naming từ Phase 5; sau Phase 6 sẽ xóa khỏi enum khi production logic không còn sử dụng

Phase 6A. Regression test / freeze current escrow contract with characterization tests:

- [ ] thêm hoặc cập nhật tests để capture current consultation hold/refund/settlement behavior trước khi bỏ system wallet
- [ ] thêm tests cho transaction-sourced availability: hold amount, refunded amount, settled amount, remaining amount
- [ ] grep toàn repo production cho `SystemWalletUserId`, `systemWallet`, `SystemWalletBalance`, `EscrowHold`, `EscrowRelease`
- [ ] lập bảng affected public DTO fields và ghi watchlist vào `money-aspect.changelog.md`

Phase 6B. Consultation transaction-sourced escrow first:

- [ ] refactor `ConsultationPaymentService.MoveMoneyToEscrowAsync` để không tạo/update system wallet
- [ ] refactor `RefundFromEscrowAsync` để validate bằng transaction-sourced availability thay vì `systemWallet.Balance`
- [ ] refactor `TransferEscrowToExpertAsync` để validate bằng transaction-sourced availability thay vì `systemWallet.Balance`
- [ ] giữ expert/user wallet mutation vì expert và member vẫn có ví thật
- [ ] cập nhật `ConsultationPaymentIntegrationTests` để không seed/assert system wallet balance
- [ ] quyết định response `SystemWalletBalanceAfter`: giữ nullable và trả `null` trong phase chuyển tiếp, hoặc đổi contract có changelog rõ

Phase 6C. Snakebite incident transaction-sourced escrow:

- [ ] refactor `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync` để không tạo/update system wallet
- [ ] refactor `RefundSnakebiteIncidentTransactionAsync` để validate bằng transaction-sourced availability
- [ ] cập nhật `SnakebiteIncidentPaymentServiceTests` và property tests đang assert `SystemWalletBalance*`
- [ ] nếu `SnakebiteIncidentPaymentResponse.SystemWalletBalanceAfter` đổi semantic hoặc bị null hóa, ghi vào changelog vì đây là DTO public

Phase 6D. Snake catching transaction-sourced escrow:

- [ ] refactor wallet payment và PayOS webhook path để không credit system wallet
- [ ] refactor `TransferSnakeCatchingFundsToRescuerAsync` để tính available paid amount từ transactions thay vì system wallet balance
- [ ] refactor `RefundSnakeCatchingTransactionAsync` để validate bằng transaction-sourced availability
- [ ] giữ platform fee hiện hữu của snake catching trong owner service, chưa share abstraction
- [ ] cập nhật response `TransferToRescuerResponse` / `RefundTransactionResponse` nếu bỏ `SystemWalletBalance*`, kèm changelog

Phase 6E. Remove transitional system escrow artifacts:

- [ ] xóa hoặc ngừng dùng `SystemWalletUserId` trong payment services nếu không còn production usage
- [ ] xóa `EscrowHold` / `EscrowRelease` khỏi production path và khỏi enum sau khi không còn logic sử dụng
- [ ] cập nhật `TransactionService` grouping để không đưa transaction-sourced escrow vào nhóm `system` một cách mơ hồ
- [ ] cập nhật `money-aspect.sourcemap.md` target-state sau khi code khớp quyết định mới
- [ ] targeted tests tối thiểu: `ConsultationPaymentIntegrationTests|SnakebiteIncidentPaymentServiceTests|SnakebiteIncidentPaymentPropertyTests|SnakeCatching|PayOs`

### Phase 7. Consultation platform fee on escrow release

Trạng thái: `TODO`

Mục tiêu:

- khi consultation settlement release, expert không nhận 100% order amount nữa
- tạo doanh thu nền tảng bằng `PlatformFee`
- release của consultation sau Phase 6 phải tạo 2 transaction chính:
  - `PlatformFee` cho phí nền tảng
  - `ExpertPayout` cho lợi nhuận ròng của expert
- expert wallet vẫn được cộng tiền bình thường với net amount

Decision đề xuất trước implementation:

- không tạo `ConsultationEscrowRelease` nếu release đã được biểu diễn bằng cặp `PlatformFee` + `ExpertPayout`
- không dùng `EscrowRelease` generic cho consultation settlement sau Phase 7
- giữ `TransactionType.PlatformFee` thay vì tạo enum platform fee riêng cho consultation, vì enum hiện tại đã có semantic platform commission chung
- dùng `ReferenceId = consultationId` cho cả `PlatformFee` và `ExpertPayout` trong settlement consultation để cùng scope với payout hiện tại
- fee config nên đi qua `SystemSettingKeys`, không hardcode phần trăm trong service như `SnakeCatchingPaymentService.commissionFee`

Decision đã chốt trước implementation:

- default consultation platform fee percent là `20%` nếu system setting chưa tồn tại
- rounding rule ưu tiên expert: tính `expertNetAmount` rồi làm tròn lên theo đơn vị VND; `platformFeeAmount = grossAmount - expertNetAmount`
- client cần thấy fee breakdown khi settlement/response có contract liên quan consultation payout hoặc transaction detail
- `EscrowHold` / `EscrowRelease` nên bị xóa sau khi production logic không còn sử dụng

Phase 7A. Fee configuration:

- [ ] thêm key vào `SystemSettingKeys`, ví dụ `Consultation:PlatformFeePercent`
- [ ] inject `ISystemSettingService` vào `ConsultationPaymentService` nếu chọn dynamic setting
- [ ] thêm helper tính fee: `grossAmount`, `feeAmount`, `expertNetAmount`
- [ ] helper dùng default fee percent `20%` khi setting chưa tồn tại hoặc chưa load được
- [ ] làm tròn lên `expertNetAmount` theo đơn vị VND rồi tính `feeAmount` bằng phần còn lại để ưu tiên expert
- [ ] validate percent trong khoảng an toàn, ví dụ `0 <= percent < 1`

Phase 7B. Settlement behavior:

- [ ] cập nhật `TransferEscrowToExpertAsync` để tính fee và net amount
- [ ] tạo `PlatformFee` transaction với amount = fee
- [ ] tạo `ExpertPayout` transaction với amount = gross - fee
- [ ] cộng expert wallet bằng net amount, không phải gross amount
- [ ] idempotency guard vẫn dựa vào `ExpertPayout` hoặc một settlement marker rõ ràng để không double payout

Phase 7C. Tests and reporting:

- [ ] cập nhật `ConsultationPaymentIntegrationTests.SettleConsultationEscrowAsync_ShouldBeIdempotent` để assert expert wallet nhận net amount
- [ ] assert tạo đúng 1 `PlatformFee` transaction và 1 `ExpertPayout` transaction
- [ ] assert tổng `PlatformFee + ExpertPayout` bằng original `ConsultationPayment`
- [ ] cập nhật `TransactionService` grouping nếu cần để `PlatformFee` vẫn xuất hiện trong group phù hợp
- [ ] nếu response trả fee breakdown mới thì update `money-aspect.changelog.md` với `grossAmount`, `platformFeePercent`, `platformFeeAmount`, `expertNetAmount`

## Tracking Progress

### Current status

- Phase 0: `DONE`
- Phase 1: `DONE`
- Phase 2: `DONE`
- Phase 3: `DONE`
- Phase 4: `DONE`
- Phase 5: `DONE`
- Phase 6: `TODO`
- Phase 7: `TODO`

### Latest confirmed findings

- `wallet topup` đã dùng prefix `TOPUP-`
- target prefix mới của `wallet topup` là `TOPUP-`
- `wallet topup` đã tự owner callback/confirm path qua `WalletTopupService`
- target prefix mới của `snake catching` là `CATCHING-`
- `consultation` dùng prefix `CONSULTPAY-`
- `snakebite incident` dùng prefix `INCIDENT-`
- `snake catching` đã absorb `WalletPaymentService` vào `SnakeCatchingPaymentService`
- route leak `POST /api/wallet/payment` đã bị xóa; route wallet payment hiện thuộc `POST /api/snakecatching/payment/wallet`
- Phase 3 đã xác nhận duplication thật mạnh nhất là escrow primitive giữa `ConsultationPaymentService.MoveMoneyToEscrowAsync` và `SnakebiteIncidentPaymentService.MoveMoneyToEscrowAsync`
- Phase 3 đã xác nhận `WalletTopupService` không được dùng chung escrow primitive vì semantic là topup inflow vào user wallet
- Phase 3 đã xác nhận snake catching vẫn có nhiều money movement trộn domain side-effect, nên không extract shared primitive từ snake catching trong lượt này
- Phase 3 đã xác nhận không tạo shared money primitive mới trước phase 5
- Phase 4 đã gom PayOS prefix constants về `PayOsPaymentFlowPrefixes`
- Phase 4 đã chuẩn hóa dispatch ở `PayOsController` qua `PayOsPaymentFlowPrefixes.TryResolve(...)` cho manual confirm, return confirm helper, và webhook
- Phase 4 đã xác nhận production prefix literals chỉ còn nằm ở `PayOsPaymentFlowPrefixes.cs`
- Phase 4 targeted PayOS tests pass `92/92`; full test suite còn 2 failure ngoài scope PayOS như ghi ở `Phase 4 output`
- Phase 5 đã thêm `EscrowHold` và `EscrowRelease` để thay semantic generic `WalletTopup` / `WalletWithdraw` cho system escrow ledger
- Phase 5 đã xác nhận `WalletTopup` và `WalletWithdraw` chỉ còn dùng cho owner service thật và system transaction filter
- Phase 5 đã quyết định không tạo `MoneyEscrowService`, `MoneyLedgerService`, hoặc `MoneyTransferService` trong lượt này
- Phase 5 targeted payment/withdrawal/PayOS tests pass `115/115`
- Phase 6 research xác nhận system wallet vẫn là side-effect thật trong consultation, snakebite incident, và snake catching escrow paths
- Phase 6 research xác nhận các response public còn expose `SystemWalletBalance*`, nên mọi thay đổi field này phải đi qua `money-aspect.changelog.md`
- Phase 7 research xác nhận consultation settlement hiện payout 100% amount cho expert và chưa tạo `PlatformFee`
- Phase 7 decision đã chốt default consultation platform fee là `20%` nếu system setting chưa tồn tại
- Phase 7 decision đã chốt rounding ưu tiên expert: làm tròn lên `expertNetAmount`, phí nền tảng là phần còn lại
- Phase 7 decision đã chốt client cần fee breakdown khi có response/contract liên quan consultation payout hoặc transaction detail
- Phase 7 decision đã chốt xóa `EscrowHold` / `EscrowRelease` sau khi production logic không còn sử dụng

## Resume Guide

Khi quay lại refactor, luôn bắt đầu theo thứ tự:

1. đọc mục `Latest confirmed findings`
2. xem phase nào đang `IN PROGRESS`
3. xác nhận branch/code hiện tại đã hoàn thành checklist nào
4. chỉ làm tiếp phase kế tiếp hoặc checklist còn dở

Nếu bị gián đoạn giữa chừng:

- luôn kiểm tra có breaking change front-facing nào chưa được viết migration doc hay không

- update lại trạng thái phase
- tick checklist đã xong
- ghi thêm finding mới vào `Latest confirmed findings`

## Out of Scope

- redesign toàn bộ payment API contract cho mobile trong một lần
- đổi schema transaction hàng loạt mà không có migration plan riêng
- merge money refactor với các business refactor không liên quan




