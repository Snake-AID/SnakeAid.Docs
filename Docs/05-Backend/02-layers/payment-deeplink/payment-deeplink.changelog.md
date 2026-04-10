# Payment Deeplink API Breaking Change Note

## Purpose

File này là note migration cho mobile/frontend của PayOS return deeplink rollout.

Mục tiêu:

- chỉ ra behavioral break của `/payos/return` và `/payos/cancel`
- giúp mobile dev biết flow cũ sẽ sai ở đâu
- chỉ rõ app cần sửa gì để follow behavior backend mới

File này không phải roadmap nội bộ.

## Release Scope

Lượt thay đổi này đổi terminal step của PayOS browser return:

- backend không còn dùng HTML terminal page làm success path chính
- backend `/api/v1/payos/return` sẽ redirect sang app deeplink
- backend `/api/v1/payos/cancel` sẽ redirect sang app deeplink
- mobile phải nhận callback từ global deeplink flow thay vì chỉ trông vào screen-local listener

## What Mobile Must Stop Assuming

Sau rollout này, mobile/frontend không được tiếp tục assume:

- `GET /api/v1/payos/return` luôn kết thúc bằng một HTML page trong webview
- user phải ở lại webview để thấy kết quả thanh toán
- callback payment chỉ đến khi user còn đứng ở `deposit_money_screen` hoặc `activity_detail_screen`
- deeplink callback tự nó là source of truth cho payment success

## New Contract Snapshot

| Endpoint | New behavior |
|---|---|
| `GET /api/v1/payos/return` | backend auto-confirm theo `orderCode`, sau đó redirect sang `snakeaid://payment/return?...` |
| `GET /api/v1/payos/cancel` | backend redirect sang `snakeaid://payment/cancel?...` |

## Breaking Changes Summary

### 1. Return no longer ends in HTML

**What changed**

- `/api/v1/payos/return` không còn là terminal HTML success/failure page như hành vi chính
- browser/webview sẽ bị điều hướng sang deeplink app

**What breaks in mobile**

- code hoặc QA flow đang expect user ở lại trang HTML sau thanh toán
- flow UX phụ thuộc vào việc user tự đóng webview bằng nút `Close Window`

**What mobile must do**

- chuẩn bị app để nhận `snakeaid://payment/return?...`
- khi nhận deeplink, điều hướng/reload UI trong app thay vì chờ HTML terminal page

### 2. Cancel no longer ends in HTML

**What changed**

- `/api/v1/payos/cancel` không còn là terminal HTML cancel page như hành vi chính
- browser/webview sẽ bị điều hướng sang `snakeaid://payment/cancel?...`

**What breaks in mobile**

- code hoặc UX đang assume cancel xong user ở lại HTML page

**What mobile must do**

- bắt `snakeaid://payment/cancel?...`
- hiển thị trạng thái hủy trong app
- không gọi confirm path khi deeplink báo cancel

### 3. Payment callback handling must be global

**What changed**

- architecture target mới không còn coi screen-local listener là callback entrypoint chính

**What breaks in mobile**

- flow chỉ xử lý callback nếu user tình cờ còn ở đúng màn hình payment

**What mobile must do**

- thêm global payment deeplink router/coordinator
- global router phải xử lý được:
  - app foreground
  - app background
  - app cold start

## Target Deeplink Shape

### Success

`snakeaid://payment/return?success=true&orderCode=...&status=PAID&cancel=false&id=...`

### Cancel

`snakeaid://payment/cancel?success=false&orderCode=...&status=CANCELLED&cancel=true&id=...`

### Error fallback

`snakeaid://payment/return?success=false&orderCode=...&status=...&cancel=...&reason=...`

## What Mobile Must Do

- parse `success`, `orderCode`, `status`, `cancel`, `id`, `reason`
- topup flow:
  - nếu còn pending `transactionId`, gọi manual confirm nếu flow đó vẫn cần
  - refresh wallet sau success callback
- catching / incident / consultation:
  - dùng deeplink như trigger để reload payment state từ backend
- không suy business success chỉ từ việc app đã mở lại; phải đọc lại state hoặc dùng confirm result/backend result phù hợp

## User Journey After Rollout

1. user bấm thanh toán trong app
2. app mở PayOS checkout
3. user thanh toán hoặc hủy trên PayOS
4. PayOS quay về backend `/api/v1/payos/return` hoặc `/api/v1/payos/cancel`
5. backend auto-confirm nếu là success path
6. backend redirect browser/webview sang `snakeaid://payment/...`
7. app mở lại và xử lý callback trong Flutter

## Notes

- source of truth vẫn là backend confirm/webhook, không phải bản thân deeplink
- nếu redirect deeplink thất bại ở tầng platform/browser, đây là edge case rollout riêng; không được dùng nó làm lý do giữ HTML terminal page làm primary success path
