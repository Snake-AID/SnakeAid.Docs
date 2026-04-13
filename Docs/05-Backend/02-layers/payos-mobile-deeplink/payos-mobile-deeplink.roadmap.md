# PayOS Mobile Deeplink Roadmap

## Mục tiêu sprint

Đưa toàn bộ flow PayOS trên Flutter về một chuẩn:

- deeplink là tín hiệu quay lại app
- transaction detail là nguồn xác nhận hoàn tất
- verify qua `prefix + orderCode + externalTransactionId`

## Phase 0. Baseline và khóa decision

### Mục tiêu

Khóa lại design trước khi sửa code để tránh tiếp tục tạo thêm screen-local flow.

### Checklist

- [x] verify backend `PayOsController.Return()` đang auto-confirm theo `orderCode`
- [x] verify các prefix backend: `TOPUP-`, `CATCHING-`, `INCIDENT-`, `CONSULTPAY-`
- [x] verify `member_incident_finished_detail_screen.dart` là baseline gần đúng nhất
- [x] xác nhận `deposit_money_screen.dart` đang lệch vì gọi `confirm-payment` làm happy path
- [x] xác nhận `activity_detail_screen.dart` thiếu data contract để check `externalTransactionId`
- [x] xác nhận `payment_confirmation_screen.dart` đang trộn 2 chiến lược confirm khác nhau

## Phase 1. Shared verification layer trong Flutter

### Mục tiêu

Tách logic verify PayOS khỏi từng screen.

### Deliverables

- `enum PayOsFlowType`
- `class PayOsPendingContext`
- `class PayOsVerificationResult`
- `class PayOsPaymentVerifier`
- `class PayOsPendingStore` nếu cần persist

### Checklist

- [ ] map `PayOsFlowType` -> backend prefix
- [ ] map `PayOsFlowType` -> expected transaction type
- [ ] tạo hàm verify dùng chung bằng `transactionId + orderCode + flowType`
- [ ] retry `GET /api/transactions/{id}` với backoff ngắn
- [ ] trả về các trạng thái typed:
  - `confirmed`
  - `cancelled`
  - `pending_backend_confirmation`
  - `mismatch`
  - `not_found`

## Phase 2. Chuẩn hóa transaction repository/model

### Mục tiêu

Mọi flow cần đọc đủ field để verify.

### Checklist

- [ ] thống nhất một transaction model chung có:
  - `id`
  - `referenceId`
  - `amount`
  - `transactionType`
  - `description`
  - `paymentMethod`
  - `externalTransactionId`
  - `createdAt`
- [ ] bỏ phụ thuộc vào model snake catching cũ chỉ có `status`
- [ ] giữ helper verify:
  - `hasExternalTransactionId`
  - `matchesPrefix(prefix, orderCode)`
  - `matchesTransactionType(type)`
  - `isPayOs`

## Phase 3. Refactor `member_incident_finished_detail_screen.dart`

### Mục tiêu

Giữ màn này làm reference implementation chính thức.

### Checklist

- [ ] thay hardcode `INCIDENT-` bằng shared prefix resolver
- [ ] thay `contains` bằng `startsWith`
- [ ] thay polling loop cục bộ bằng shared verifier
- [ ] chuẩn hóa snackbar/dialog message theo `PayOsVerificationResult`
- [ ] giữ fallback refresh incident status sau khi confirm

### Expected outcome

Màn incident trở thành sample tốt nhất để copy pattern.

## Phase 4. Refactor `deposit_money_screen.dart`

### Mục tiêu

Topup không còn dùng `confirm-payment` làm happy path.

### Checklist

- [ ] lưu `orderCode` cùng `transactionId`
- [ ] tạo pending context với `flowType = topup`
- [ ] khi deeplink success, gọi shared verifier trước
- [ ] chỉ dùng `POST /api/v1/PayOs/confirm-payment` như fallback cuối
- [ ] clear pending topup đúng khi:
  - confirm thành công
  - cancel thật
  - mismatch rõ ràng
- [ ] refresh wallet sau verify confirmed

### Risk

Topup là flow dễ bị resume sau app restart nên cần persist pending context cẩn thận.

## Phase 5. Refactor `payment_confirmation_screen.dart`

### Mục tiêu

Tách bạch:

- consultation payment chính
- wallet topup phụ trợ

### Checklist

- [ ] consultation PayOS dùng shared verifier với `CONSULTPAY-`
- [ ] topup phụ trợ cũng dùng chung verifier topup
- [ ] bỏ duplicated parse logic `status/cancel/path`
- [ ] chỉ để domain confirm endpoint consultation làm fallback phase 2 nếu cần
- [ ] chuẩn hóa navigation sau confirmed:
  - emergency request modal
  - booking success navigation

## Phase 6. Refactor `activity_detail_screen.dart`

### Mục tiêu

Snake catching flow verify được cả deposit round và final payment round bằng contract đầy đủ.

### Checklist

- [ ] mở rộng transaction repository của module snake catching hoặc dùng repository chung
- [ ] verify deposit bằng:
  - prefix `CATCHING-`
  - transaction type phù hợp round hiện tại
  - external transaction id khác rỗng
- [ ] verify final payment bằng cùng một cơ chế
- [ ] tránh việc deeplink success kích hoạt đồng thời cả deposit/final check nếu không có pending context tương ứng

## Phase 7. Resume và observability

### Mục tiêu

Không mất trạng thái khi app resume, background hoặc user quay lại muộn.

### Checklist

- [ ] persist pending context tối thiểu cho topup
- [ ] cân nhắc persist cho consultation/incident/catching nếu flow cần resume chắc chắn
- [ ] thêm debug logs có cấu trúc:
  - flow type
  - transaction id
  - order code
  - expected prefix
  - verification result

## Acceptance criteria

- Deeplink success không tự động đồng nghĩa với payment success.
- Mọi flow PayOS đều có thể verify bằng `externalTransactionId`.
- Mọi flow PayOS đều match được prefix đúng theo backend.
- Không còn màn nào coi `confirm-payment` là bước bắt buộc đầu tiên sau deeplink.
- Khi backend đã auto-confirm xong, Flutter chỉ cần poll transaction là đủ để chốt UI state.

## Fallback policy

Chỉ dùng fallback confirm API khi:

- đã nhận deeplink success
- đã poll transaction detail nhiều lần
- vẫn chưa có `externalTransactionId`
- nhưng `orderCode` và pending context vẫn match flow hiện tại

## Resume notes

Khi resume implementation, đọc theo thứ tự:

1. `payos-mobile-deeplink.introduction.md`
2. `payos-mobile-deeplink.sourcecode.md`
3. file screen đang refactor
4. `payos-mobile-deeplink.useguide.md`

## Current status

- Phase 0: done
- Phase 1: todo
- Phase 2: todo
- Phase 3: todo
- Phase 4: todo
- Phase 5: todo
- Phase 6: todo
- Phase 7: todo
