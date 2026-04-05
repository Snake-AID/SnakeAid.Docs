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

Trạng thái: `TODO`

Mục tiêu:

- chưa chốt abstraction shared
- chỉ chuẩn bị dữ kiện để đánh giá shared primitive ở phase cuối

Checklist:

- [ ] xác định phần lặp giữa các `MoveMoneyToEscrowAsync`
- [ ] xác định phần nào là duplication thật, phần nào chỉ là coupling tạm thời do flow chưa chuẩn hóa
- [ ] ghi lại blast radius nếu trích shared primitive quá sớm

Done khi:

- đủ dữ kiện để chốt abstraction shared ở phase cuối, không quyết định sớm

### Phase 4. Chuẩn hóa PayOS flow router

Trạng thái: `TODO`

Mục tiêu:

- routing theo prefix rõ ràng
- confirm/webhook/manual confirm dùng cùng nguyên tắc dispatch

Checklist:

- [ ] chuẩn hóa prefix constants
- [ ] gom routing logic về một chỗ
- [ ] thêm test uniqueness của prefix
- [ ] thêm test router dispatch đúng owner

Done khi:

- không còn hardcode mơ hồ theo description prefix ở nhiều nơi

### Phase 5. Ledger semantics cleanup and shared crosscutting decision

Mục tiêu phase cuối:

- chốt semantic ledger sau khi flow ownership đã ổn định
- chỉ lúc này mới quyết định shared primitive boundary

Checklist:

- [ ] cleanup hết chỗ còn dùng `WalletTopup` như generic system credit
- [ ] cleanup hết chỗ còn dùng `WalletWithdraw` như generic payout/refund source
- [ ] đối chiếu lại 4 flow sau khi chuẩn hóa ownership
- [ ] đánh giá lại có cần `MoneyEscrowService` hay không
- [ ] đánh giá lại có cần `MoneyLedgerService` hay không
- [ ] không tạo `MoneyTransferService` nếu pattern shared chưa tự chứng minh đủ mạnh

## Tracking Progress

### Current status

- Phase 0: `DONE`
- Phase 1: `DONE`
- Phase 2: `DONE`
- Phase 3: `TODO`
- Phase 4: `TODO`
- Phase 5: `TODO`

### Latest confirmed findings

- `wallet topup` đã dùng prefix `TOPUP-`
- target prefix mới của `wallet topup` là `TOPUP-`
- `wallet topup` đã tự owner callback/confirm path qua `WalletTopupService`
- target prefix mới của `snake catching` là `CATCHING-`
- `consultation` dùng prefix `CONSULTPAY-`
- `snakebite incident` dùng prefix `INCIDENT-`
- `snake catching` đã absorb `WalletPaymentService` vào `SnakeCatchingPaymentService`
- route leak `POST /api/wallet/payment` đã bị xóa; route wallet payment hiện thuộc `POST /api/snakecatching/payment/wallet`

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




