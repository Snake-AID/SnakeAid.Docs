Dưới đây là toàn bộ các điểm trong 2 doc hiện vẫn còn mang tính đề xuất, lựa chọn, hoặc chưa khóa cứng hoàn toàn. Tôi gom thành các quyết định cần chốt để biến 2 doc thành decision-only.

**1. Shared money module có tồn tại hay không**
Hiện doc đang ngầm chọn hướng “có shared money primitive”, nhưng chưa khóa cứng mức độ.

Các phương án:
1. Có shared module đầy đủ.
   Nghĩa là tạo layer dùng chung cho money primitive và 3 flow domain + topup đều gọi vào đó khi phù hợp.
2. Có shared module tối thiểu.
   Chỉ trích các phần thật sự lặp lớn, còn lại giữ trong flow owner service.
3. Không tạo shared module.
   Chỉ tách topup và gom ownership snake catching, chưa đụng shared primitive ở vòng refactor này.

Điểm cần chốt:
- Có hay không có shared money layer trong đợt refactor này.

**2. Bộ class shared nào là bắt buộc**
Hiện doc đang liệt kê các class như dự kiến, chưa phải quyết định cuối:
- `IMoneyEscrowService` / `MoneyEscrowService`
- `IMoneyTransferService` / `MoneyTransferService`
- `IMoneyLedgerService` / `MoneyLedgerService`

Các phương án:
1. Tạo đủ cả 3 cặp service.
2. Chỉ tạo `MoneyEscrowService`.
3. Chỉ tạo `MoneyLedgerService`.
4. Tạo `MoneyEscrowService` + `MoneyLedgerService`, không tạo `MoneyTransferService`.
5. Không tạo service shared nào mới.

Điểm cần chốt:
- Class shared nào sẽ thực sự được tạo trong lần refactor này.

**3. `MoneyTransferService` có tồn tại hay không**
Đây là điểm anh vừa bắt rất đúng. Hiện wording cũ gây hiểu nhầm và bản thân class này cũng chưa chắc cần.

Các phương án:
1. Giữ `MoneyTransferService`, nhưng đổi semantics rõ ràng thành kiểu:
   - `increase wallet balance`
   - `decrease wallet balance`
   - `transfer system-held money`
   - `refund settled money`
2. Bỏ hẳn `MoneyTransferService`, vì operations này quá rời rạc hoặc hiện chưa đủ lý do gom chung.
3. Dời các thao tác balance movement vào `MoneyLedgerService`.
4. Dời các thao tác balance movement vào từng flow owner service, không shared.

Điểm cần chốt:
- Có giữ `MoneyTransferService` hay bãi bỏ hoàn toàn.

**4. `WalletTopupPaymentService` có tạo mới hay không**
Hiện doc đang định hướng rõ về service owner riêng cho topup, nhưng vẫn chưa khóa là tạo service mới hay rename/absorb từ class cũ.

Các phương án:
1. Tạo mới `IWalletTopupPaymentService` + `WalletTopupPaymentService`.
2. Không tạo class mới, mở rộng `WalletTopupService` hiện tại để nó sở hữu cả create + confirm + webhook.
3. Giữ `WalletTopupService` cho create intent, và tạo thêm service riêng chỉ cho callback/confirm.
4. Rename `WalletTopupService` thành `WalletTopupPaymentService` rồi mở rộng trách nhiệm.

Điểm cần chốt:
- Owner service của topup sau refactor sẽ là class mới hay tận dụng class cũ.

**5. Số lượng class mới tối đa được phép**
Hiện doc ghi “nếu cần tạo mới”, tức là chưa chốt mức độ can thiệp.

Các phương án:
1. Chấp nhận tạo đầy đủ class mới theo target structure.
2. Chỉ cho phép tạo tối đa 1 class owner mới và 1 class shared mới.
3. Chỉ cho phép tạo class owner mới, không tạo shared class mới.
4. Không tạo class mới trừ khi thật sự bị block.

Điểm cần chốt:
- Trần số lượng abstraction mới được phép sinh ra trong đợt này.

**6. `WalletPaymentService` sẽ bị xử lý thế nào**
Hiện doc đang để mở:
- “loại bỏ hoặc absorb”
- “deprecate”

Các phương án:
1. Xóa hẳn `WalletPaymentService` trong đợt này.
2. Absorb logic vào `SnakeCatchingPaymentService`, rồi xóa file luôn.
3. Absorb logic nhưng giữ file tạm thời, mark deprecated, cleanup sau.
4. Giữ nguyên `WalletPaymentService` trong đợt này, chỉ cắt topup trước.

Điểm cần chốt:
- Fate cuối cùng của `WalletPaymentService` trong chính đợt refactor này.

**7. Mức refactor của snake catching trong đợt này**
Hiện roadmap nói sẽ “gom ownership”, nhưng chưa khóa mức độ.

Các phương án:
1. Gom toàn bộ money logic snake catching về một owner trong đợt này.
2. Chỉ gom wallet payment của snake catching, còn payout/commission giữ nguyên chỗ cũ.
3. Chỉ xử lý topup ở đợt này, snake catching ownership để phase sau.
4. Gom một phần, miễn callback/payment ownership đã thống nhất.

Điểm cần chốt:
- Snake catching có được refactor full ownership trong lượt này hay không.

**8. Mức refactor của consultation và snakebite incident**
Hiện doc nói 2 flow này “gần target pattern”, nhưng chưa chốt có đụng sâu trong lượt này không.

Các phương án:
1. Refactor cả 2 flow trong cùng đợt để dùng chung primitive mới.
2. Chỉ refactor topup + catching trước, consultation/incident chỉ align ở doc.
3. Refactor consultation + incident chỉ khi cần để support shared layer.
4. Không sửa behavior 2 flow này trong lượt này.

Điểm cần chốt:
- Consultation và incident có nằm trong implementation scope thực tế hay chỉ là baseline tham chiếu.

**9. `PayOsController` dispatch bằng gì**
`PaymentFlowRouter` đã bị loại bỏ. Nhưng cách dispatch trong `PayOsController` vẫn chưa khóa hoàn toàn ở mức implementation detail.

Các phương án:
1. `switch` / `if` theo prefix string constant.
2. Bảng map prefix -> service trong controller.
3. Private method `ResolveFlowByPrefix()` + `Dispatch...()` trong chính controller.
4. Dispatch theo transaction type sau khi resolve transaction, không lấy prefix làm primary key.
5. Prefix là primary key, transaction type chỉ để verify consistency.

Điểm cần chốt:
- Cơ chế dispatch chính xác trong `PayOsController`.

**10. Prefix cuối cùng của 4 flow**
Hiện doc đã nêu:
- `TOPUP-`
- `CATCHING-`
- `INCIDENT-`
- `CONSULTPAY-`

Nhưng đây mới là định hướng, chưa phải quyết định final.

Các phương án cần chốt:
- `TOPUP-` có chốt không
- `CATCHING-` có chốt không hay giữ `SNAKEAID-`
- `CONSULTPAY-` giữ nguyên hay đổi cho đồng dạng naming
- `INCIDENT-` giữ nguyên hay đổi

Điểm cần chốt:
- Bộ prefix final của 4 flow.

**11. `CATCHING-` hay giữ `SNAKEAID-`**
Đây là điểm riêng đáng tách ra vì có tradeoff thật.

Các phương án:
1. Đổi sang `CATCHING-` để semantic rõ.
2. Giữ `SNAKEAID-` để giảm impact, chỉ tách topup ra prefix mới.
3. Tạm hỗ trợ cả hai prefix trong giai đoạn chuyển tiếp.
4. Đổi mới hoàn toàn và migration một lượt.

Điểm cần chốt:
- Prefix chính thức của snake catching sau refactor.

**12. Transaction semantic cleanup có nằm trong scope hay không**
Hiện roadmap phase 5 đang để mở.

Các phương án:
1. Có làm trong cùng đợt refactor này.
2. Chỉ cleanup một phần liên quan trực tiếp topup.
3. Không làm semantic cleanup trong đợt này, chỉ ghi nhận follow-up.

Điểm cần chốt:
- Có chạm vào `TransactionType` / description semantics trong lượt này không.

**13. Mức đổi contract API/client-visible**
Hiện doc có câu “cập nhật doc và test nếu contract đọc transaction có đổi”, tức là vẫn để mở.

Các phương án:
1. Không đổi API contract nào cho mobile/client.
2. Chấp nhận đổi internal contract nhưng không đổi public response.
3. Cho phép đổi contract transaction APIs nếu thật sự cần.
4. Chưa chốt, tùy implementation.

Điểm cần chốt:
- Public API contract có được phép đổi hay phải giữ nguyên tuyệt đối.

**14. Notification có thuộc money flow lifecycle hay không**
Trong target pattern hiện có dòng:
- `gửi notification nếu flow cần`

Đây là một optional behavior, chưa phải quyết định.

Các phương án:
1. Notification nằm ngoài scope refactor money.
2. Notification vẫn là step chuẩn trong `PostPaymentActions`.
3. Chỉ flow nào hiện có notification thì giữ nguyên, không chuẩn hóa thêm.

Điểm cần chốt:
- Notification có được xem là một phần của pattern money chuẩn hay không.

**15. `payment intent` hay chỉ `pending transaction`**
Trong target pattern hiện có wording:
- “tạo pending transaction hoặc payment intent”

Đây là 2 abstraction khác nhau, đang bị để mở.

Các phương án:
1. Chốt dùng khái niệm `pending transaction` thống nhất.
2. Chốt dùng `payment intent` như abstraction chung.
3. Topup dùng `pending transaction`, các flow khác giữ model hiện tại.
4. Không introduce khái niệm mới, bám naming hiện có từng flow.

Điểm cần chốt:
- Thuật ngữ chuẩn cho record khởi tạo payment.

**16. `transactionId` hay `orderCode` là key resolve chính**
Trong target pattern hiện có:
- “resolve payment theo `transactionId` hoặc `orderCode`”

Đây là chưa khóa.

Các phương án:
1. `orderCode` là key chính cho PayOS callback/return.
2. `transactionId` là key chính nội bộ, `orderCode` chỉ để bridge gateway.
3. callback dùng `orderCode`, confirm manual có thể dùng `transactionId`.
4. Chốt một key duy nhất cho toàn hệ thống.

Điểm cần chốt:
- Key chuẩn để resolve payment trong flow confirm/webhook.

**17. `refund` và `payout` có được chuẩn hóa chung không**
Trong doc đang ghi:
- `payout/refund nếu có`
- `refund transfer`

Vẫn còn ở mức optional/phạm vi chưa rõ.

Các phương án:
1. Chuẩn hóa cả `refund` và `payout` trong shared money primitives.
2. Chỉ chuẩn hóa `refund`.
3. Chỉ chuẩn hóa `payout`.
4. Không chuẩn hóa, giữ trong từng flow owner.

Điểm cần chốt:
- `refund/payout` có vào shared layer trong đợt này không.

**18. `money-aspect.sourcemap.md` là target-state hay implementation-state**
Hiện nó là target structural memory, nhưng chưa ghi rõ chế độ cập nhật.

Các phương án:
1. `sourcemap` mô tả target architecture cuối cùng.
2. `sourcemap` phải luôn mô tả implementation-state hiện tại theo từng phase.
3. Tách 2 vùng trong cùng file:
   - current state
   - target state

Điểm cần chốt:
- Vai trò chính xác của `sourcemap`.

**19. File placement list là bắt buộc hay tham chiếu**
Hiện wording là “nếu phải tạo mới, vị trí mục tiêu là...”

Các phương án:
1. Chốt cứng: nếu tạo mới thì bắt buộc đúng các path đó.
2. Chốt mềm: bám `Interfaces` / `Implements`, còn tên file/class có thể điều chỉnh.
3. Không chốt trước placement, để implementation tự quyết.

Điểm cần chốt:
- Placement có phải decision cuối hay chưa.

**20. Mức độ “decision-only” của 2 doc**
Đây là meta-decision nhưng rất quan trọng.

Các phương án:
1. Cả 2 doc chỉ chứa quyết định đã chốt, không còn chữ kiểu `ví dụ`, `có thể`, `nếu cần`.
2. `refactoring.md` là decision-only, `sourcemap.md` cho phép target-state abstraction.
3. Cả 2 doc vẫn cho phép một ít target wording miễn không mơ hồ.

Điểm cần chốt:
- Mức độ cứng của doc sau vòng chỉnh này.

Nếu anh muốn làm nhanh và sạch, tôi đề xuất anh trả lời lại theo format ngắn như sau:

1. Shared money layer: `A/B/C`
2. Shared classes: `...`
3. MoneyTransferService: `giữ/bỏ`
4. Topup owner: `...`
5. New class limit: `...`
6. WalletPaymentService fate: `...`
7. Snake catching scope: `...`
8. Consultation/incident scope: `...`
9. PayOsController dispatch: `...`
10. Final prefixes: `...`
11. Catching prefix: `...`
12. Transaction semantic cleanup: `...`
13. Public API contract: `...`
14. Notification in pattern: `...`
15. Pending transaction vs payment intent: `...`
16. Resolve key: `...`
17. Refund/payout standardization: `...`
18. Sourcemap role: `...`
19. File placement rigidity: `...`
20. Decision-only strictness: `...`

Sau khi anh chốt, tôi sẽ sửa cả 2 file một lượt để không còn wording “may appear”, “ví dụ”, “nếu cần”, “đề xuất”.