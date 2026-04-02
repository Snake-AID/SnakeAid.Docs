---
doc_role: backlog
module: wallet-withdrawal
kind: gap-analysis
doc_type: backlog
status: active
last_updated: 2026-04-02
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

### Medium Priority
- Xác định và chuẩn hóa rule trừ tiền ở `create` hay `complete`
- Chuẩn hóa contract masking giữa admin và user
- Thay mock bank directory bằng source dữ liệu thật

### Low Priority
- Bổ sung rate limiting nếu nghiệp vụ cần
- Bổ sung audit log nếu nghiệp vụ cần
- Bổ sung webhook/push notification nếu mobile cần realtime update

## Detailed Backlog

### 1. `bankBin` chưa đi hết flow

**Current state**
- Request `POST /api/withdrawals/create` bắt buộc có `bankBin`
- Entity `WalletWithdraw` không lưu `bankBin`
- Khi approve, QR payload đang dùng cứng bank BIN `970400`

**Impact**
- QR có thể không phản ánh đúng ngân hàng user chọn
- Frontend/mobile gửi `bankBin` nhưng backend không dùng đúng

**Backend actions**
- Thêm trường `BankBin` vào entity `WalletWithdraw`
- Lưu `bankBin` khi tạo withdrawal
- Dùng `withdrawal.BankBin` khi generate VietQR
- Backfill migration nếu cần cho dữ liệu cũ

**Suggested acceptance criteria**
- Tạo withdrawal với `bankBin = 970405`
- Approve xong, `vietQrPayload` phải chứa `970405`

### 2. Thiếu endpoint admin lấy chi tiết withdrawal

**Current state**
- Có `GET /api/admin/withdrawals`
- Có `GET /api/admin/withdrawals/pending`
- Không có `GET /api/admin/withdrawals/{id}`

**Impact**
- Admin UI khó load màn hình detail riêng
- Test script hiện đang kỳ vọng endpoint này

**Backend actions**
- Bổ sung `GET /api/admin/withdrawals/{id}`
- Trả về cùng shape với `WithdrawalResponse`

**Suggested acceptance criteria**
- Admin gọi được detail bằng `id`
- 404 khi không tồn tại
- 403 khi token không có role admin

### 3. Response contract chưa đồng nhất

**Current state**
- `GET /api/wallet/me` và `GET /api/wallet/banks` dùng `ApiResponse<T>`
- `/api/withdrawals/*` và `/api/admin/withdrawals/*` trả raw object / raw array

**Impact**
- Frontend/mobile phải viết nhiều kiểu parser
- Tăng nguy cơ lỗi khi xây generic API layer

**Backend actions**
- Chọn một convention chung cho toàn bộ wallet module:
  - hoặc tất cả dùng `ApiResponse<T>`
  - hoặc tất cả dùng raw payload
- Cập nhật swagger/docs theo convention thống nhất

**Suggested acceptance criteria**
- Toàn bộ wallet withdrawal endpoints dùng một response envelope thống nhất

### 4. Quy tắc trừ tiền khỏi ví cần chốt rõ

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

### 5. Masking contract giữa admin và user chưa được formalize

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

### 6. Bank directory đang là mock data

**Current state**
- `GET /api/wallet/banks` trả danh sách hardcoded từ `BankDirectoryService`

**Impact**
- Danh sách ngân hàng không được cập nhật động
- Dễ lệch với hệ thống VietQR thực tế

**Backend actions**
- Kết nối nguồn bank directory thật
- Cập nhật cache strategy
- Có fallback nếu source ngoài lỗi

### 7. Thiếu rate limiting rõ ràng cho withdrawal

**Current state**
- Chưa thấy rule rate limit riêng cho withdrawal endpoints

**Impact**
- Có thể bị spam create/cancel/approve

**Backend actions**
- Nếu nghiệp vụ cần:
  - thêm rate limit cho user actions
  - thêm rate limit cho admin actions

### 8. Thiếu audit trail rõ ràng cho admin actions

**Current state**
- Chưa có tài liệu hoặc model riêng thể hiện audit cho approve/reject/complete/fail

**Impact**
- Khó truy vết thao tác vận hành nếu có dispute

**Backend actions**
- Ghi nhận actor, action, timestamp, reason, old status, new status

### 9. Thiếu cơ chế realtime status update

**Current state**
- Client phải poll `GET /api/withdrawals/{id}` hoặc `GET /api/withdrawals/me`

**Impact**
- Mobile/frontend không có cơ chế nhận update push/webhook/realtime event

**Backend actions**
- Nếu sản phẩm cần realtime:
  - thêm event publish / webhook / push notification

## Suggested Delivery Order

1. Fix `bankBin`
2. Add admin detail endpoint
3. Decide and unify response envelope
4. Decide wallet balance deduction strategy
5. Replace mock bank directory
6. Add rate limit / audit / realtime mechanisms if required
