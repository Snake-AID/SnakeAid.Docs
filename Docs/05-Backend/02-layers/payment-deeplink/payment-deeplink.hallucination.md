# Payment Deeplink Research Buckets

## Purpose

File này không dùng để chốt thiết kế.

File này dùng để gom các ngộ nhận dễ xuất hiện quanh PayOS return deeplink, để tránh implement lệch semantic giữa backend và mobile.

Nguyên tắc:

- research trước
- chốt decision sau
- chỉ update `payment-deeplink.refactoring.md` và `payment-deeplink.sourcecode.md` khi đã có kết luận đủ chắc

## Bucket A. Return URL semantics

### Fact đã verify

- `PayOsController.Return()` hiện là browser redirect entrypoint do PayOS gọi về
- method này hiện:
  - đọc query `code`, `id`, `cancel`, `status`, `orderCode`
  - best-effort auto-confirm bằng `ConfirmByOrderCodeAsync(orderCode)`
  - render HTML success/failure page
- `PayOsController.Cancel()` hiện render HTML cancel page

### Decision đã chốt

- `/return` không bị xóa
- `/cancel` không bị xóa
- backend vẫn là cầu nối giữa PayOS browser redirect và app deeplink
- terminal HTML page không còn là success path chính

### Điều không được hallucinate

- không được hiểu rằng deeplink rollout nghĩa là PayOS sẽ gọi thẳng backend webhook
- không được hiểu rằng `/return` trở nên vô nghĩa sau khi app có deeplink
- không được hiểu rằng `returnUrl` và webhook là cùng một thứ

## Bucket B. Deeplink vs source of truth

### Fact đã verify

- mobile hiện đã có `app_links`
- mobile hiện đã có logic bắt callback ở một số screen
- backend hiện có confirm path riêng:
  - manual confirm theo `transactionId`
  - return confirm theo `orderCode`
  - webhook confirm theo verified payload

### Decision đã chốt

- deeplink không phải source of truth
- backend confirm/webhook vẫn là source of truth
- deeplink chỉ là UX/user-return channel

### Điều không được hallucinate

- không được assume nhận `snakeaid://payment/return?...success=true...` là đủ để sửa domain state ở mobile mà không cần đọc lại backend
- không được assume cancel deeplink đồng nghĩa PayOS transaction bị rollback ở provider
- không được assume app mở lại thành công trên mọi thiết bị là bằng chứng confirm backend đã thành công

## Bucket C. Platform support assumptions

### Fact đã verify

- Android manifest đã có:
  - custom scheme `snakeaid://`
  - app link `https://snakeaid-dev.duykhiem.id.vn/payos/*`
- iOS `Info.plist` hiện chỉ thấy custom scheme `snakeaid://`
- mobile code hiện đã recognize cả:
  - `snakeaid://payment/...`
  - `https://snakeaid-dev.duykhiem.id.vn/payos/...`

### Decision đã chốt

- rollout này dùng `snakeaid://payment/...` làm target deeplink chuẩn
- không mở thêm scope universal-link parity cho iOS trong lượt này
- backend redirect strategy được ưu tiên hơn direct PayOS-to-app custom scheme

### Điều không được hallucinate

- không được assume Android app link behavior hiện có là đủ để coi iOS đã support tương đương
- không được assume direct custom-scheme return từ PayOS là target đã chốt

## Bucket D. Mobile ownership boundary

### Fact đã verify

- `deposit_money_screen.dart` đang tự nghe PayOS callback
- `activity_detail_screen.dart` đang tự nghe PayOS callback
- `deep_link_handler.dart` hiện mới log host `payment`, chưa làm payment routing đầy đủ

### Decision đã chốt

- cần global payment deeplink router/coordinator
- screen-local listener không còn là target architecture

### Điều không được hallucinate

- không được tiếp tục implement thêm callback logic mới rải ở từng screen
- không được để business confirm chỉ chạy được khi user tình cờ đang đứng ở đúng screen payment

## Bucket E. Error-path behavior

### Decision đã chốt

- nếu `Return()` gặp lỗi confirm, backend vẫn phải cố redirect người dùng về app với trạng thái lỗi
- chỉ fallback HTML khi redirect response không thể phát ra an toàn

### Điều không được hallucinate

- không được nuốt lỗi confirm rồi redirect success giả
- không được giữ HTML làm default path chỉ vì “an toàn hơn” nếu deeplink contract đã được chốt
