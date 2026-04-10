# Payment Deeplink Refactoring

## Background

### Document mode

File này là `decision-only`.

Nó chỉ chứa:

- decision đã chốt cho deeplink payment return
- roadmap thực hiện
- tracking tiến độ bám theo decision đã chốt

Nó không dùng để:

- brainstorm phương án redirect khác nhau giữa chừng
- ghi tạm các ý tưởng chưa verify
- trộn implementation note vào roadmap

Mọi ambiguity mới phát sinh trong quá trình soi code hoặc triển khai phải được đưa sang `payment-deeplink.hallucination.md` trước khi được nâng cấp thành decision trong file này.

## Current Direction Summary

| Area | Current state | Target state |
|---|---|---|
| PayOS return | backend `/api/v1/payos/return` trả HTML terminal page | backend auto-confirm rồi redirect sang deeplink `snakeaid://payment/...` |
| PayOS cancel | backend `/api/v1/payos/cancel` trả HTML terminal page | backend redirect sang deeplink cancel để app nhận trạng thái hủy |
| Mobile callback handling | listener nằm rải ở từng screen payment | có global payment deeplink router xử lý được cả foreground, background, cold start |
| PayOS return URL strategy | mixed giữa HTTPS app link và custom scheme ở mobile-side assumptions | PayOS tiếp tục quay về backend HTTPS; backend là cầu nối cuối sang deeplink app |

## Problem Statement

Hiện tại PayOS user journey kết thúc ở một HTML page do backend render ra:

- user trả tiền xong nhưng vẫn ở trong browser/webview
- app không phải lúc nào cũng được mở lại
- callback handling trên mobile đang phụ thuộc vào việc user còn đứng ở đúng payment screen
- logic confirm/reload sau payment chưa có một entrypoint thống nhất ở mobile

Mục tiêu của lượt refactor này là chuyển terminal step từ HTML sang deeplink để Flutter nhận callback trực tiếp, trong khi vẫn giữ backend là điểm auto-confirm trung tâm.

## Main Decisions

### 1. PayOS vẫn quay về backend trước

Không dùng direct custom-scheme làm `returnUrl` của PayOS trong lượt này.

Decision đã chốt:

- PayOS `returnUrl` tiếp tục là backend HTTPS
- PayOS `cancelUrl` tiếp tục là backend HTTPS
- backend `/api/v1/payos/return` và `/api/v1/payos/cancel` là public callback entrypoint cho browser redirect

Lý do:

- backend cần cơ hội auto-confirm trước khi trả user về app
- iOS hiện mới support custom scheme `snakeaid://`, chưa có universal link/app link parity như Android
- giữ backend làm cầu nối giúp flow return/cancel nhất quán cho cả 4 payment flows

### 2. Backend return/cancel phải redirect sang deeplink app

Decision đã chốt:

- bỏ terminal HTML page làm success/cancel experience chính
- `Return()` sẽ redirect sang `snakeaid://payment/return?...`
- `Cancel()` sẽ redirect sang `snakeaid://payment/cancel?...`
- chỉ fallback về HTML khi bản thân backend không thể dựng redirect response an toàn

### 3. Mobile phải có global payment deeplink router

Decision đã chốt:

- không tiếp tục dựa vào screen-local listener làm entrypoint chính
- thêm global payment deeplink router/coordinator ở mobile
- router này phải xử lý được:
  - app đang mở
  - app background
  - app cold start từ deeplink

### 4. Source of truth không đổi

Decision đã chốt:

- deeplink chỉ là user-return channel
- source of truth vẫn là backend confirm/webhook
- mobile không được suy rằng chỉ cần nhận deeplink là business state đã đúng nếu backend confirm chưa thành công

## Target Contract

### Backend deep link shape

Target URI shape:

- success:
  - `snakeaid://payment/return?success=true&orderCode=...&status=PAID&cancel=false&id=...`
- cancel:
  - `snakeaid://payment/cancel?success=false&orderCode=...&status=CANCELLED&cancel=true&id=...`
- confirm/error fallback:
  - `snakeaid://payment/return?success=false&orderCode=...&status=...&cancel=...&reason=...`

Quy ước:

- `orderCode` luôn được mang theo nếu có
- `id` là PayOS transaction/link identifier nếu callback có trả
- `success` là flag chuẩn hóa do backend kết luận để mobile đọc dễ hơn
- `status` và `cancel` vẫn được giữ để tương thích với logic mobile hiện có

## Implementation Phases

## Phase 1. Backend redirect bridge

### Mục tiêu

Đổi terminal step của `return`/`cancel` từ HTML sang deeplink redirect.

### Checklist

- [ ] thay `BuildReturnHtml(...)` bằng helper dựng deeplink success/failure
- [ ] thay `BuildCancelHtml(...)` bằng helper dựng deeplink cancel
- [ ] giữ `ConfirmByOrderCodeAsync(orderCode)` ở `Return()`
- [ ] nếu auto-confirm fail, vẫn redirect về app với trạng thái lỗi thay vì nuốt lỗi rồi render HTML
- [ ] cập nhật Swagger description cho `return`/`cancel`

## Phase 2. Mobile global payment deeplink router

### Mục tiêu

Tạo một entrypoint deeplink payment chung cho app, không phụ thuộc user còn đứng ở đúng screen payment.

### Checklist

- [ ] thêm global deeplink handler/coordinator cho host `payment`
- [ ] parse được cả `/return` và `/cancel`
- [ ] phát event/state dùng chung cho UI
- [ ] xử lý được foreground/background/cold start
- [ ] không còn để `deposit_money_screen` và `activity_detail_screen` là điểm duy nhất bắt callback

## Phase 3. Screen integration and reload

### Mục tiêu

Nối kết quả payment deeplink vào topup/catching/incident/consultation UI.

### Checklist

- [ ] topup success vẫn confirm một lần rồi refresh wallet
- [ ] cancel path không confirm và hiển thị trạng thái hủy
- [ ] snake catching detail/payment flow reload đúng state sau callback
- [ ] incident/consultation flows nếu dùng PayOS return path phải đọc được payment result từ global router

## Phase 4. Docs and migration note

### Mục tiêu

Cảnh báo mobile dev rằng `/return` và `/cancel` không còn là HTML terminal page.

### Checklist

- [ ] cập nhật changelog cho mobile
- [ ] ghi rõ deeplink shape mới
- [ ] ghi rõ listener screen-local cũ không còn là architecture target

## Success Criteria

- Khi user thanh toán thành công trên PayOS, browser/webview được forward vào app thay vì dừng ở HTML page
- Backend vẫn auto-confirm theo `orderCode` trước khi trả user về app
- Mobile nhận được deep link callback ổn định dù app đang foreground, background hay cold start
- Topup/catching/incident/consultation không cần phụ thuộc vào HTML terminal page để kết thúc flow

## Current Status

- Phase 1: TODO
- Phase 2: TODO
- Phase 3: TODO
- Phase 4: TODO

## Latest Confirmed Findings

- `PayOsController.Return()` hiện auto-confirm rồi render HTML success/failure page
- `PayOsController.Cancel()` hiện render HTML cancel page
- backend `PayOsGateway` đang lấy `ReturnUrl`/`CancelUrl` từ `PayOsOptions`, tức là gateway hiện đang được cấu hình theo backend HTTPS callback
- mobile đã cài `app_links`
- Android đã support cả custom scheme `snakeaid://` và app link `https://snakeaid-dev.duykhiem.id.vn/payos/*`
- iOS hiện chỉ thấy custom scheme `snakeaid://`
- mobile hiện có listener PayOS callback rải ở `deposit_money_screen.dart` và `activity_detail_screen.dart`
- kiến trúc target của lượt này là backend redirect sang deeplink + mobile global router
