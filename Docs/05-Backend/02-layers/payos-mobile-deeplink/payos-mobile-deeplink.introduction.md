# PayOS Mobile Deeplink Introduction

## Mục tiêu

Lập kế hoạch refactor Flutter client để bắt deeplink sau khi thanh toán PayOS quay về app, sau đó kiểm tra giao dịch đã hoàn tất hay chưa bằng dữ liệu transaction thật từ backend thay vì dựa vào callback URL như nguồn sự thật cuối cùng.

Mục tiêu cụ thể:

- thống nhất một flow PayOS return cho mobile
- dùng `prefix` trong `Transaction.Description` và `ExternalTransactionId` để verify trạng thái hoàn tất
- chuẩn hóa một handler dùng lại được cho:
  - `member_incident_finished_detail_screen.dart`
  - `payment_confirmation_screen.dart`
  - `activity_detail_screen.dart`
  - `deposit_money_screen.dart`

## Hiện trạng backend đã verify

### 1. Backend đã auto-confirm tại PayOS return

`PayOsController.Return()` đang:

- nhận `code`, `status`, `cancel`, `id`, `orderCode`
- nếu `code == "00"` và `status == "PAID"` và `cancel == false` thì gọi `ConfirmByOrderCodeAsync(orderCode)`
- sau đó render HTML kết quả

Điều này có nghĩa:

- Flutter không nên coi `POST /api/v1/PayOs/confirm-payment` là happy path chính
- deeplink chỉ là tín hiệu "user đã quay lại app"
- source of truth vẫn là transaction/backend state sau khi backend đã xử lý return hoặc webhook

### 2. Backend route flow bằng prefix trong `Transaction.Description`

`PayOsPaymentFlowPrefixes` đang dùng các prefix sau:

- `TOPUP-`
- `CATCHING-`
- `INCIDENT-`
- `CONSULTPAY-`

`ConfirmByOrderCodeAsync(orderCode)` lookup transaction theo `orderCode`, lấy `Description`, resolve prefix, rồi dispatch đến service tương ứng:

- topup
- consultation
- snakebite incident
- snake catching

### 3. Dấu hiệu xác nhận transaction hoàn tất ở backend

Ở các service confirm hiện tại, dấu hiệu idempotent/confirmed rõ nhất là:

- `ExternalTransactionId` đã có giá trị
- `Description` chứa prefix đúng flow và `orderCode` tương ứng

Riêng consultation, response confirm còn expose thêm:

- `isEscrowed`
- `externalTransactionId`
- `orderCode`

## Hiện trạng Flutter đã verify

## 1. `member_incident_finished_detail_screen.dart`

Đây là màn đang gần đúng nhất với thiết kế backend.

Điểm đúng:

- subscribe `PaymentDeepLinkCoordinator`
- match `event.orderCode` với `_pendingPayOsOrderCode`
- không coi deeplink là payment success cuối cùng
- poll `GET /api/transactions/{id}`
- chỉ coi là confirmed khi:
  - `paymentMethod == PAYOS`
  - `transactionType == SnakebiteIncidentPayment`
  - `externalTransactionId` khác rỗng
  - `description` chứa `INCIDENT-{orderCode}`

Điểm cần chỉnh:

- check `description.contains(...)` nên chuẩn hóa thành `startsWith(prefix + orderCode)` để bám đúng backend design
- logic verify đang nằm cứng trong screen, chưa tái sử dụng được
- fallback hiện vẫn phụ thuộc nhiều vào state cục bộ của screen

Kết luận:

`member_incident_finished_detail_screen.dart` đúng hướng về mặt kiến trúc, nhưng chưa đủ sạch để nhân bản cho các flow khác.

## 2. `deposit_money_screen.dart`

Hiện đang:

- lưu `pendingTransactionId` và `checkoutUrl`
- bắt deeplink event
- nếu success thì gọi `walletRepository.confirmPayment(transactionId)`

Vấn đề:

- đang lấy `confirm-payment` làm happy path
- chưa verify `orderCode`
- chưa verify `prefix`
- chưa verify `externalTransactionId`
- không tận dụng việc backend return đã auto-confirm theo `orderCode`

Kết luận:

Đây là flow lệch với thiết kế backend hiện tại và cần refactor.

## 3. `payment_confirmation_screen.dart`

Hiện đang có 2 flow:

- consultation PayOS
- wallet topup phụ trợ khi ví không đủ

Điểm tốt:

- consultation flow có giữ `_pendingPayOsOrderCode` và `_pendingPayOsTransactionId`
- sau deeplink có retry confirm qua API domain và check `externalTransactionId`

Vấn đề:

- topup phụ trợ chỉ reload ví, chưa verify transaction theo `prefix/orderCode/externalTransactionId`
- consultation flow vẫn dùng endpoint confirm domain làm bước chính thay vì coi transaction/backend state là nguồn sự thật
- state pending đang phân tán theo nhiều biến riêng

Kết luận:

Flow consultation gần usable nhưng vẫn cần gom về cùng một contract verify như incident.

## 4. `activity_detail_screen.dart`

Hiện đang:

- bắt deeplink global event
- nếu success thì gọi `_checkPaymentStatus()` và `_checkFinalPaymentStatus()`
- poll transaction theo id

Điểm đúng:

- không coi deeplink là success cuối cùng
- có polling transaction

Vấn đề:

- model transaction của module snake catching chỉ đọc `status`, không đọc `externalTransactionId`, `paymentMethod`, `description`
- chưa kiểm tra prefix `CATCHING-`
- khó phân biệt deposit round và final payment round chỉ bằng deep link success chung chung

Kết luận:

Flow này đúng tinh thần nhưng data contract ở repository/model chưa đủ giàu để verify đúng thiết kế backend.

## Phương hướng refactor chốt

## 1. Chuẩn hóa một payment return coordinator dùng chung

Tạo một tầng dùng chung, ví dụ:

- `PayOsPendingContext`
- `PayOsFlowType`
- `PayOsReturnVerificationService`
- `PayOsPaymentReturnHandler`

Nhiệm vụ:

- nhận deeplink event từ `PaymentDeepLinkCoordinator`
- so khớp pending context theo `orderCode`
- query transaction detail theo internal `transactionId`
- verify theo contract chung
- trả về kết quả typed cho từng screen

## 2. Chuẩn hóa contract verify transaction

Happy path chung cho Flutter:

1. Nhận deeplink event.
2. Xác định pending payment context tương ứng.
3. Poll `GET /api/transactions/{transactionId}` với retry ngắn.
4. Chỉ coi là completed khi:
   - `externalTransactionId` khác rỗng
   - `description` bắt đầu bằng `expectedPrefix + orderCode`
   - `paymentMethod` là `PayOS` nếu field này có trong model
   - `transactionType` match flow đang xử lý

`POST /api/v1/PayOs/confirm-payment` chỉ giữ vai trò fallback chủ động khi:

- backend return/webhook chưa kịp hoàn tất
- transaction vẫn chưa có `externalTransactionId` sau một khoảng retry hữu hạn

## 3. Chuẩn hóa pending context thay vì để mỗi screen tự giữ state khác nhau

Pending context tối thiểu cần lưu:

- `flowType`
- `transactionId`
- `orderCode`
- `expectedPrefix`
- `expectedTransactionType`
- `referenceId`
- `startedAt`

Topup cần có khả năng persist tạm qua app restart.

Consultation và incident có thể persist hoặc rebuild từ route state nếu cần resume chắc chắn hơn.

## 4. Ưu tiên tận dụng pattern của incident screen

Refactor theo thứ tự:

1. Trích verification logic từ `member_incident_finished_detail_screen.dart` thành shared layer
2. Dùng shared layer cho `deposit_money_screen.dart`
3. Áp dụng cho `payment_confirmation_screen.dart`
4. Nâng `activity_detail_screen.dart` lên contract mới với transaction model đầy đủ hơn

## Phạm vi ngoài scope của đợt này

- đổi backend HTML return sang auto-redirect deeplink
- thay đổi PayOS returnUrl strategy ở backend
- thay đổi business rule confirm của backend

Các hạng mục trên có thể làm ở phase khác, nhưng không phải prerequisite để chuẩn hóa verify logic phía Flutter.
