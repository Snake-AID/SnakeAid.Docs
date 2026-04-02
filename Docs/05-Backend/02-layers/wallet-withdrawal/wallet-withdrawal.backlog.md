---
doc_role: backlog
module: wallet-withdrawal
kind: gap-analysis
doc_type: backlog
status: active
last_updated: 2026-04-03
owners: [backend-team]
---

# Wallet Withdrawal Backend Backlog

## Purpose

Tài liệu này liệt kê các điểm tồn đọng giữa flow withdrawal mong muốn và backend hiện tại, để backend team xử lý dần.

File này dành cho backend/internal discussion.
File `wallet-withdrawal.usageguide.md` giữ sạch để frontend/mobile dùng tích hợp API.

## Priority Summary

### High Priority
- Lưu và sử dụng đúng `bankBin` trong toàn bộ withdrawal flow
- Bổ sung endpoint admin lấy chi tiết một withdrawal
- Chuẩn hóa response contract giữa wallet endpoints và withdrawals endpoints
- Chốt data model cuối cùng cho withdrawal request
- Chốt rule amount limits và method contract cho cancel endpoint

### Medium Priority
- Xác định và chuẩn hóa rule trừ tiền ở `create` hay `complete`
- Chuẩn hóa contract masking giữa admin và user
- Thay mock bank directory bằng source dữ liệu thật
- Bổ sung admin metadata và audit trail
- Bổ sung notification flow
- Chuẩn hóa naming giữa docs và code

### Low Priority
- Bổ sung rate limiting nếu nghiệp vụ cần
- Bổ sung audit log nếu nghiệp vụ cần
- Bổ sung webhook/push notification nếu mobile cần realtime update

## Progress Snapshot

### Completed in current implementation slice

Slice backend vừa hoàn tất tương ứng với một phase contract cleanup, tập trung vào việc đưa contract hiện tại về trạng thái có thể tích hợp ổn định hơn mà không thay đổi business flow sâu hơn.

Đã hoàn thành trong slice này:
- Hoàn tất item `1. bankBin chưa đi hết flow`
- Hoàn tất item `3. Thiếu endpoint admin lấy chi tiết withdrawal`
- Hoàn tất item `4. Response contract chưa đồng nhất`
- Hoàn tất item `5. Method contract của endpoint cancel chưa thống nhất`

### Not completed in current implementation slice

Slice này chưa giải quyết:
- Item `2. Data model của withdrawal chưa khớp với yêu cầu nghiệp vụ`
- Phần `amount limits` trong item `6`
- Toàn bộ các quyết định business rule và hạ tầng ở item `7` trở đi

### Phase mapping

- Phase 1: Contract Cleanup and API Alignment
  - Scope:
    - persist/use `bankBin`
    - add admin detail endpoint
    - unify withdrawal response envelope
    - lock cancel contract to `POST`
  - Status: Completed on backend 2026-04-03

- Phase 2: Withdrawal Domain Finalization
  - Scope:
    - finalize data model
    - finalize amount limits
    - finalize balance deduction strategy
    - finalize masking and QR contract details
  - Status: Not started

- Phase 3: Operational Hardening
  - Scope:
    - bank directory source
    - notifications
    - admin metadata / audit trail
    - stable error codes
    - test hardening
  - Status: Not started

- Phase 4: Production Guardrails
  - Scope:
    - rate limiting
    - realtime updates
    - fraud checks
    - advanced business rules
  - Status: Not started

## Detailed Backlog

### 1. `bankBin` chưa đi hết flow

Status: Implemented on backend 2026-04-03

**Current state**
- Request `POST /api/withdrawals/create` bắt buộc có `bankBin`
- Entity/service/controller hiện đã lưu và trả `bankBin`
- Approve hiện dùng `withdrawal.BankBin` để generate VietQR

**Impact**
- QR có thể không phản ánh đúng ngân hàng user chọn
- Frontend/mobile gửi `bankBin` nhưng backend không dùng đúng

**Residual caveat**
- Dữ liệu cũ chưa có `BankBin` sẽ không được auto-backfill mặc định
- Approve sẽ fail rõ ràng nếu record legacy không có `BankBin`

**Suggested acceptance criteria**
- Tạo withdrawal với `bankBin = 970405`
- Approve xong, `vietQrPayload` phải chứa `970405`

### 2. Data model của withdrawal chưa khớp với yêu cầu nghiệp vụ

Status: Not started

**Current state**
- `WalletWithdraw` hiện chỉ có:
  - `BankAccount`
  - `BankName`
  - `Status`
  - `ProcessedAt`
  - `RejectionReason`
  - `VietQrPayload`
  - `VietQrImageBase64`
- Chưa có các field backend plan từng kỳ vọng:
  - `BankBin`
  - `AccountHolderName`
  - `ProcessedByAdminId`
  - `AdminNotes`
  - `RenderQrEnabled`

**Impact**
- Thiếu dữ liệu để hỗ trợ admin workflow đầy đủ
- Khó truy vết ai đã xử lý request
- Không lưu được thông tin người thụ hưởng đúng theo plan nghiệp vụ

**Backend actions**
- Chốt schema chính thức cho `WalletWithdraw`
- Thêm các field cần thiết vào entity và migration
- Update DTO mapping và swagger

**Suggested acceptance criteria**
- Entity và migration phản ánh đầy đủ schema đã chốt
- API request/response dùng đúng field names cuối cùng

### 3. Thiếu endpoint admin lấy chi tiết withdrawal

Status: Implemented on backend 2026-04-03

**Current state**
- Đã có `GET /api/admin/withdrawals/{id}`
- Endpoint trả cùng shape `WithdrawalResponse` trong `ApiResponse<T>`

**Impact**
- Admin UI khó load màn hình detail riêng
- Test script hiện đang kỳ vọng endpoint này

**Residual caveat**
- Chưa thêm admin-specific metadata ngoài `WithdrawalResponse` hiện tại

**Suggested acceptance criteria**
- Admin gọi được detail bằng `id`
- 404 khi không tồn tại
- 403 khi token không có role admin

### 4. Response contract chưa đồng nhất

Status: Implemented on backend 2026-04-03

**Current state**
- `GET /api/wallet/me`, `GET /api/wallet/banks`, `/api/withdrawals/*`, và `/api/admin/withdrawals/*` hiện dùng `ApiResponse<T>` success envelope

**Impact**
- Frontend/mobile phải viết nhiều kiểu parser
- Tăng nguy cơ lỗi khi xây generic API layer

**Residual caveat**
- Bash scripts và client integrations phải parse payload dưới `data`

**Suggested acceptance criteria**
- Toàn bộ wallet withdrawal endpoints dùng một response envelope thống nhất

### 5. Method contract của endpoint cancel chưa thống nhất

Status: Implemented on backend 2026-04-03

**Current state**
- Contract chính thức giữ nguyên `POST /api/withdrawals/{id}/cancel`
- Docs/test scripts đã được align theo `POST`

**Impact**
- Mobile/frontend dễ implement sai method
- Tạo lệch giữa docs, test case, và swagger kỳ vọng

**Residual caveat**
- Không cung cấp alias `DELETE`; client cũ phải dùng `POST`

### 6. Amount limits chưa khớp với kế hoạch nghiệp vụ trước đó

Status: Not started

**Current state**
- Code hiện validate:
  - min `10,000`
  - max `50,000,000`
- Kế hoạch triển khai cũ từng kỳ vọng:
  - min `50,000`
  - max `5,000,000`
  - daily limit `10,000,000`

**Impact**
- Không rõ production business rule cuối cùng là gì
- Frontend/mobile và admin ops có thể dùng sai threshold

**Backend actions**
- Chốt min/max amount chính thức
- Chốt có hay không daily limit
- Update validation attributes, tests, docs

### 7. Quy tắc trừ tiền khỏi ví cần chốt rõ

Status: Not started

**Current state**
- `create`: chưa trừ tiền
- `approve`: chưa trừ tiền
- `complete`: mới trừ tiền và tạo transaction

**Impact**
- Nếu user tạo nhiều request pending liên tiếp, có thể phát sinh cạnh tranh số dư
- Rule này cần được xác nhận là cố ý hay chỉ là tạm thời

**Questions for backend**
- Có cần reserve balance ngay khi `create` không?
- Có cần block tạo request mới khi tổng pending > balance không?

**Backend actions**
- Chốt business rule
- Nếu cần reserve balance:
  - trừ hoặc khóa balance tại `create`
  - hoàn trả tại `reject/fail/cancel`
- Nếu giữ logic hiện tại:
  - thêm validation chống oversubscription tại `approve/complete`

### 8. Masking contract giữa admin và user chưa được formalize

Status: Not started

**Current state**
- User endpoints mask `bankAccount`
- Admin endpoints trả đầy đủ `bankAccount`

**Impact**
- Cần xác nhận đây có phải behavior mong muốn không
- Có thể ảnh hưởng tới policy hiển thị dữ liệu nhạy cảm trên admin UI

**Backend actions**
- Chốt policy hiển thị:
  - admin thấy full
  - hay admin cũng chỉ thấy masked + quyền xem full riêng
- Ghi rõ vào docs và swagger

### 9. Bank directory đang là mock data

Status: Not started

**Current state**
- `GET /api/wallet/banks` trả danh sách hardcoded từ `BankDirectoryService`

**Impact**
- Danh sách ngân hàng không được cập nhật động
- Dễ lệch với hệ thống VietQR thực tế

**Backend actions**
- Kết nối nguồn bank directory thật
- Cập nhật cache strategy
- Có fallback nếu source ngoài lỗi

### 10. Thiếu notification flow cho withdrawal

Status: Not started

**Current state**
- Chưa thấy service notification riêng cho withdrawal
- Không có push/email flow cho create/approve/reject/complete/fail

**Impact**
- User không được báo trạng thái kịp thời
- Admin ops khó có cơ chế nhắc việc

**Backend actions**
- Chốt event matrix cần gửi notification
- Tạo notification service hoặc reuse hạ tầng hiện có
- Publish event tại các bước create/approve/reject/complete/fail

### 11. Thiếu audit trail rõ ràng cho admin actions

Status: Not started

**Current state**
- Chưa có entity / table audit log riêng
- Chưa có field lưu admin xử lý trong `WalletWithdraw`

**Impact**
- Khó truy vết ai approve/reject/complete/fail
- Khó điều tra dispute hoặc sai sót vận hành

**Backend actions**
- Thêm `ProcessedByAdminId` hoặc tương đương
- Tạo audit log riêng nếu cần
- Ghi nhận old status / new status / actor / reason / timestamp

### 12. Thiếu rate limiting rõ ràng cho withdrawal

Status: Not started

**Current state**
- Chưa thấy rule rate limit riêng cho withdrawal endpoints

**Impact**
- Có thể bị spam create/cancel/approve

**Backend actions**
- Nếu nghiệp vụ cần:
  - thêm rate limit cho user actions
  - thêm rate limit cho admin actions

### 13. Thiếu cơ chế realtime status update

Status: Not started

**Current state**
- Client phải poll `GET /api/withdrawals/{id}` hoặc `GET /api/withdrawals/me`

**Impact**
- Mobile/frontend không có cơ chế nhận update push/webhook/realtime event

**Backend actions**
- Nếu sản phẩm cần realtime:
  - thêm event publish / webhook / push notification

**Current state**
- Nếu không cần realtime:
  - chuẩn hóa polling contract và recommended polling interval

### 14. Thiếu các rule nghiệp vụ bổ sung từng được kỳ vọng trong kế hoạch

Status: Not started

**Current state**
- Chưa có daily limit
- Chưa có duplicate request prevention rõ ràng
- Chưa có fraud checks / suspicious pattern rules
- Chưa có account holder validation

**Impact**
- Flow hiện tại mới ở mức cơ bản, chưa cover các business guardrails nâng cao

**Backend actions**
- Chốt rule nào thực sự cần cho production
- Implement theo mức độ ưu tiên nghiệp vụ

### 15. Chưa có mã lỗi nghiệp vụ ổn định cho FE/mobile

Status: Not started

**Current state**
- Service/controller đang ném `InvalidOperationException` với message text
- Chưa thấy error code riêng cho các case:
  - insufficient balance
  - invalid status transition
  - not found
  - access denied

**Impact**
- Frontend/mobile khó map UI message ổn định
- Dễ vỡ contract khi text message thay đổi

**Backend actions**
- Chuẩn hóa exception types / error codes public
- Map về error response thống nhất

### 16. Chưa có test coverage đúng với implementation hiện tại

Status: In progress

**Current state**
- Có bash tests cho flow withdrawal
- Bash tests đã được chỉnh cho response envelope `.data` và admin detail endpoint hiện có
- Chưa thấy unit/integration test coverage đủ sâu cho service rules

**Impact**
- CI khó phản ánh đúng trạng thái chất lượng
- Một số regression có thể không bị bắt

**Backend actions**
- Sửa test expectations cho khớp contract đã chốt
- Bổ sung unit tests cho:
  - create
  - approve
  - reject
  - complete
  - fail
  - cancel
- Bổ sung integration tests cho status transitions và balance updates

### 17. `WalletWithdrawService` đang phụ thuộc `IWalletService` trả về DTO thay vì domain entity

Status: Not started

**Current state**
- `IWalletService.GetWalletByUserIdAsync` trả `WalletResponse`
- `WalletWithdrawService` dùng response này để lấy `Id` và `Balance`

**Impact**
- Layering không sạch
- Service domain đang phụ thuộc DTO response
- Khó mở rộng khi cần transaction-safe wallet operations

**Backend actions**
- Tách method domain-level trả `Wallet` entity hoặc wallet aggregate model
- Giữ `WalletResponse` cho API layer

### 18. Field naming giữa docs cũ và code hiện tại chưa thống nhất

Status: In progress

**Current state**
- Kế hoạch cũ dùng `BankAccountNumber`
- Code hiện dùng `BankAccount`
- Kế hoạch cũ có `AccountHolderName`
- Code hiện chưa có field này

**Impact**
- Nhầm lẫn khi đọc docs / viết FE / viết migration

**Backend actions**
- Chốt naming convention cuối cùng
- Đồng bộ entity / request / response / docs / tests

**Progress note**
- `bankBin`, `BankAccount`, admin detail endpoint, và `POST /api/withdrawals/{id}/cancel` đã được đồng bộ ở code và docs active
- Một số tên field từng nằm trong kế hoạch cũ như `AccountHolderName` vẫn chưa được implement vào backend

### 19. Admin approve flow chưa lưu người xử lý

Status: Not started

**Current state**
- `ApproveWithdrawalAsync`, `RejectWithdrawalAsync`, `CompleteWithdrawalAsync`, `FailWithdrawalAsync` nhận `adminUserId`
- Nhưng entity không lưu lại thông tin admin đã xử lý

**Impact**
- Không truy ra admin nào xử lý từng bước

**Backend actions**
- Thêm field `ProcessedByAdminId` hoặc action log riêng
- Persist giá trị admin vào DB khi approve/reject/complete/fail

### 20. QR generation contract cần được formalize

Status: In progress

**Current state**
- `VietQrAdapter` tự build payload string đơn giản
- Có fallback trả payload mà không có image nếu generate QR lỗi

**Progress note**
- QR generation không còn hardcode bank BIN; approve hiện dùng `withdrawal.BankBin`
- Contract fallback image/payload và chuẩn payload production vẫn chưa được formalize đầy đủ

**Impact**
- Cần xác nhận payload format đã đúng chuẩn production chưa
- FE/mobile cần biết khi nào `vietQrImageBase64` có thể null

**Backend actions**
- Chốt contract QR:
  - image có bắt buộc không
  - payload format chuẩn nào
  - behavior fallback ra sao
- Viết test cho payload/image generation

**Backend actions**
- Nếu sản phẩm cần realtime:
  - thêm event publish / webhook / push notification

## Suggested Delivery Order

1. Phase 1 completed:
   `bankBin` + admin detail endpoint + unified response envelope + cancel method contract
2. Next: chốt data model cuối cùng cho withdrawal
3. Next: chốt amount limits
4. Next: decide wallet balance deduction strategy
5. Next: add admin metadata + audit trail
6. Next: replace mock bank directory
7. Next: add notification flow
8. Next: harden test coverage và stable public error codes
9. Next: add rate limit / realtime / advanced fraud checks if required
