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
- logic escrow, transfer, payout, refund dùng chung primitive nhưng không chia sẻ domain side-effect
- transaction/ledger rõ nghĩa, không dùng nhòe semantic giữa các flow

### Quang cảnh sau refactor

Sau refactor, money aspect phải có trạng thái cuối như sau:

- `wallet topup` là một flow độc lập, có prefix riêng, callback owner riêng, và chỉ chịu trách nhiệm nạp tiền vào ví user
- `snake catching` là một flow độc lập, toàn bộ money logic của nó nằm dưới owner service của chính catching, không còn split qua service ngoài flow
- `consultation` vẫn giữ ownership theo flow hiện tại, nhưng phần money primitive dùng chung sẽ được chuẩn hóa
- `snakebite incident` vẫn giữ ownership theo flow hiện tại, nhưng phần money primitive dùng chung sẽ được chuẩn hóa

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
- gọi shared primitive:
  - escrow
  - wallet debit/credit
  - system wallet movement
  - ledger pair

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

Candidate sẽ được đánh giá lại ở phase cuối:

- `MoneyEscrowService`
- `MoneyLedgerService`

Nếu được tạo ở phase cuối, layer shared chỉ xử lý:

- wallet movement
- system wallet movement
- transaction record creation
- idempotent money operation

Layer shared không được biết:

- snake catching status
- consultation booking status
- snakebite incident status
- wallet topup UI flow

### Vị trí class/service nếu tạo mới

Nếu cần tạo mới trong repo, vị trí phải bám convention hiện tại:

- interface đặt tại `SnakeAid.Service/Interfaces`
- implementation đặt tại `SnakeAid.Service/Implements`
- controller giữ mỏng và tiếp tục đặt tại `SnakeAid.Api/Controllers`

Các class dự kiến có thể xuất hiện:

- `SnakeAid.Service/Interfaces/IWalletTopupPaymentService.cs`
- `SnakeAid.Service/Implements/WalletTopupPaymentService.cs`
- `SnakeAid.Service/Interfaces/IMoneyEscrowService.cs`
- `SnakeAid.Service/Implements/MoneyEscrowService.cs`
- `SnakeAid.Service/Interfaces/IMoneyLedgerService.cs`
- `SnakeAid.Service/Implements/MoneyLedgerService.cs`

Lưu ý:

- đây là target placement, không phải cam kết sẽ tạo đủ tất cả class trên
- chỉ tạo class khi refactor thực tế chứng minh là cần
- chỉ chốt shared class sau phase chuẩn hóa flow

### Flow owner đề xuất

- `WalletTopupPaymentService`
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

Trạng thái: `TODO`

Checklist:

- [ ] chốt danh sách 4 flow tiền tệ cần quản lý
- [x] chốt prefix hiện tại của từng flow
- [ ] chốt entrypoints controller/service hiện tại
- [ ] chốt các integration test đang có
- [ ] bổ sung regression notes cho topup đang đi nhờ snake catching

Resume output mong muốn:

- có bảng mapping flow -> controller -> service -> prefix -> callback handler

### Phase 1. Tách wallet topup khỏi snake catching

Trạng thái: `TODO`

Mục tiêu:

- topup có prefix riêng
- topup có confirm/webhook handler riêng
- topup không còn gọi gián tiếp vào `SnakeCatchingPaymentService`

Checklist:

- [x] tạo prefix riêng cho topup
- [ ] tạo owner service cho topup callback/confirm
- [ ] cập nhật PayOS router
- [ ] thêm test routing cho topup
- [ ] verify topup credit đúng wallet user

Done khi:

- topup không còn phụ thuộc vào `SnakeCatchingPaymentService.HandleWalletTopupAsync`

### Phase 2. Hợp nhất ownership của snake catching money flow

Trạng thái: `TODO`

Mục tiêu:

- wallet payment của snake catching không nằm riêng ở `WalletPaymentService`

Checklist:

- [ ] xác định API contract nào của snake catching cần giữ nguyên
- [ ] chuyển wallet payment logic vào snake catching owner service
- [ ] giữ nguyên behavior payout/commission
- [ ] loại bỏ `WalletPaymentService`
- [ ] update tests

Done khi:

- snake catching payment logic nằm trong một owner thống nhất

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

### Phase 5. Ledger semantics cleanup và shared crosscutting decision

Trạng thái: `TODO`

Mục tiêu:

- làm rõ nghĩa transaction records
- chỉ chốt shared crosscutting sau khi flow đã được chuẩn hóa

Checklist:

- [ ] rà soát chỗ đang dùng `WalletTopup` như generic credit event
- [ ] quyết định semantic chuẩn cho system credit / escrow credit / user topup
- [ ] cập nhật naming và description
- [ ] đánh giá lại candidate `MoneyEscrowService`
- [ ] đánh giá lại candidate `MoneyLedgerService`
- [ ] không tạo `MoneyTransferService` nếu sau chuẩn hóa vẫn không có pattern đủ chặt
- [ ] cập nhật doc và test nếu contract đọc transaction có đổi

Done khi:

- nhìn transaction type có thể hiểu đúng ngữ nghĩa business
- decision về shared crosscutting được chốt từ target pattern đã chuẩn hóa, không từ hiện trạng méo

## Tracking Progress

### Current status

- Phase 0: `IN PROGRESS`
- Phase 1: `TODO`
- Phase 2: `TODO`
- Phase 3: `TODO`
- Phase 4: `TODO`
- Phase 5: `TODO`

### Latest confirmed findings

- `wallet topup` đang dùng prefix `SNAKEAID-`
- target prefix mới của `wallet topup` là `TOPUP-`
- `wallet topup` đang được PayOS route sang snake catching processor
- target prefix mới của `snake catching` là `CATCHING-`
- `consultation` dùng prefix `CONSULTPAY-`
- `snakebite incident` dùng prefix `INCIDENT-`
- `snake catching` đang split money ownership giữa `SnakeCatchingPaymentService` và `WalletPaymentService`

## Resume Guide

Khi quay lại refactor, luôn bắt đầu theo thứ tự:

1. đọc mục `Latest confirmed findings`
2. xem phase nào đang `IN PROGRESS`
3. xác nhận branch/code hiện tại đã hoàn thành checklist nào
4. chỉ làm tiếp phase kế tiếp hoặc checklist còn dở

Nếu bị gián đoạn giữa chừng:

- update lại trạng thái phase
- tick checklist đã xong
- ghi thêm finding mới vào `Latest confirmed findings`

## Out of Scope

- redesign toàn bộ payment API contract cho mobile trong một lần
- đổi schema transaction hàng loạt mà không có migration plan riêng
- merge money refactor với các business refactor không liên quan
