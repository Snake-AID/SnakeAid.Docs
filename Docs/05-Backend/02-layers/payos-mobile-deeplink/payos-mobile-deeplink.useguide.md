# PayOS Mobile Deeplink Useguide

## Mục đích

Tài liệu này dành cho mobile dev để hiểu backend contract liên quan đến việc:

- tạo link thanh toán PayOS
- nhận lại `transactionId`, `orderCode`, `checkoutUrl`
- kiểm tra payment đã được backend xác nhận hay chưa

Quy tắc quan trọng:

- deeplink không phải source of truth cuối cùng
- transaction detail mới là nguồn xác nhận completion cho Flutter
- `ExternalTransactionId` khác rỗng là tín hiệu confirmed mạnh nhất hiện tại

## Prefix contract đã verify

| Flow | Prefix |
|---|---|
| Topup | `TOPUP-` |
| Snake catching | `CATCHING-` |
| Snakebite incident | `INCIDENT-` |
| Consultation | `CONSULTPAY-` |

Flutter nên verify:

- `description.startsWith(prefix + orderCode)`
- `externalTransactionId` khác rỗng

## Shared transaction lookup

### Endpoint

`GET /api/transactions/{id}`

### Mục đích

Đây là endpoint Flutter nên gọi sau khi nhận deeplink return để kiểm tra trạng thái transaction thật.

### Response fields quan trọng

- `id`
- `referenceId`
- `amount`
- `transactionType`
- `description`
- `paymentMethod`
- `externalTransactionId`
- `createdAt`

### Response example

```json
{
  "success": true,
  "message": "Transaction retrieved successfully.",
  "data": {
    "id": "0b5d970f-3f5e-4f0b-9f9e-2864d4d3f111",
    "userName": "member01",
    "fullName": "Nguyen Van A",
    "referenceId": "6d72c716-73d4-4ea7-8b21-c4d6a5321001",
    "amount": 150000,
    "currency": "VND",
    "transactionType": "SnakebiteIncidentPayment",
    "description": "INCIDENT-240413001",
    "paymentMethod": "PayOS",
    "externalTransactionId": "INCIDENT-MANUAL-9af5f3ab",
    "createdAt": "2026-04-13T07:22:10.000Z"
  }
}
```

## Wallet topup

### Create topup link

`POST /api/wallet/topup`

### Request

```json
{
  "amount": 100000,
  "description": "Nap tien vi"
}
```

### Constraints

- `amount` từ `1,000` đến `10,000,000`
- user phải đăng nhập

### Success response example

```json
{
  "success": true,
  "message": "Wallet top-up payment link created successfully",
  "data": {
    "transactionId": "d8a8cf86-0ce6-4b2d-9f97-5ce6ff110001",
    "userId": "f67d0d76-c4e5-4ff7-8fd5-50fe9d220001",
    "amount": 100000,
    "status": "Pending",
    "checkoutUrl": "https://pay.payos.vn/web/abcxyz",
    "orderCode": 240413101,
    "paymentLinkId": "plink_123456",
    "expiresAt": "2026-04-13T08:00:00.000Z",
    "provider": "PayOS"
  }
}
```

### Flutter note

Sau deeplink return:

- poll transaction detail theo `transactionId`
- verify `description.startsWith("TOPUP-" + orderCode)`
- verify `externalTransactionId` khác rỗng
- chỉ fallback `POST /api/v1/PayOs/confirm-payment` khi poll không đủ

## Snakebite incident payment

### Create PayOS payment link

`POST /api/SnakebiteIncident/{incidentId}/payment/payos`

### Request

```json
{
  "amount": 150000,
  "description": "Thanh toan phi cuu ho",
  "transactionType": "SnakebiteIncidentPayment"
}
```

### Success response example

```json
{
  "success": true,
  "message": "Payment link created successfully.",
  "data": {
    "transactionId": "fa1014ec-0a44-4a84-8d17-27a934410001",
    "snakebiteIncidentId": "6d72c716-73d4-4ea7-8b21-c4d6a5321001",
    "amount": 150000,
    "status": "Pending",
    "checkoutUrl": "https://pay.payos.vn/web/incident-abc",
    "orderCode": 240413201,
    "paymentLinkId": "plink_incident_01",
    "expiresAt": "2026-04-13T08:05:00.000Z",
    "provider": "PayOS"
  }
}
```

### Flutter note

`member_incident_finished_detail_screen.dart` nên được chuẩn hóa theo contract này:

- match `event.orderCode`
- query transaction detail bằng `transactionId`
- verify:
  - `transactionType == SnakebiteIncidentPayment`
  - `paymentMethod == PayOS`
  - `description.startsWith("INCIDENT-" + orderCode)`
  - `externalTransactionId` khác rỗng

## Snake catching payment

### Create PayOS payment link

`POST /api/SnakeCatchingPayments/create-link`

### Request

```json
{
  "snakeCatchingRequestId": "d7f0fd38-2df4-4d36-8c65-ef6c32c01001",
  "amount": 300000,
  "transactionType": "CatchingDeposit",
  "description": "Catching deposit"
}
```

### Success response example

```json
{
  "success": true,
  "message": "PayOS payment link created successfully",
  "data": {
    "transactionId": "2b4ec4af-4d7d-4ca8-a17f-479fbb100001",
    "snakeCatchingRequestId": "d7f0fd38-2df4-4d36-8c65-ef6c32c01001",
    "amount": 300000,
    "status": "Pending",
    "checkoutUrl": "https://pay.payos.vn/web/catching-abc",
    "orderCode": 240413301,
    "paymentLinkId": "plink_catching_01",
    "expiresAt": "2026-04-13T08:10:00.000Z",
    "provider": "PayOS"
  }
}
```

### Flutter note

`activity_detail_screen.dart` hiện cần nâng model transaction để đọc đủ:

- `description`
- `paymentMethod`
- `externalTransactionId`

Không nên chỉ dựa vào `status == paid`.

## Consultation PayOS payment

### Confirm consultation payment

`POST /api/consultations/payments/confirm`

### Request

```json
{
  "transactionId": "5f6ac4d6-4f41-4f4d-9ef4-c1b5d2100001"
}
```

### Response example

```json
{
  "success": true,
  "data": {
    "referenceId": "52cd7941-4fb8-4c13-a6db-5e9b2c880001",
    "referenceType": "Booking",
    "transactionId": "5f6ac4d6-4f41-4f4d-9ef4-c1b5d2100001",
    "amount": 200000,
    "currency": "VND",
    "paymentMethod": "PayOs",
    "status": "Paid",
    "provider": "PayOS",
    "checkoutUrl": null,
    "orderCode": 240413401,
    "paymentLinkId": "plink_consult_01",
    "externalTransactionId": "CONSULT-MANUAL-5f6ac4d6"
  }
}
```

### Flutter note

`payment_confirmation_screen.dart` hiện có thể giữ endpoint này như fallback domain-specific, nhưng happy path vẫn nên là:

- verify transaction/backend state sau deeplink
- chỉ gọi confirm endpoint khi transaction vẫn chưa reflected sau retry

## Generic fallback confirm

### Endpoint

`POST /api/v1/PayOs/confirm-payment`

### Mục đích

Manual confirm fallback khi webhook hoặc return processing chưa reflected kịp vào transaction state.

### Request

```json
{
  "transactionId": "d8a8cf86-0ce6-4b2d-9f97-5ce6ff110001"
}
```

### Khi nào được gọi

- đã có pending context hợp lệ
- deeplink success đã quay về app
- `GET /api/transactions/{id}` retry nhiều lần mà `externalTransactionId` vẫn rỗng

### Khi nào không nên gọi ngay

- vừa nhận deeplink success và chưa poll transaction
- flow topup/deposit dùng `confirm-payment` như bước đầu tiên

## Decision cho mobile dev

Nếu cần viết handler PayOS mới, luôn follow thứ tự này:

1. Lưu `transactionId`, `orderCode`, `flowType`.
2. Nhận deeplink event.
3. Match `orderCode`.
4. Poll `GET /api/transactions/{id}`.
5. Verify `prefix + externalTransactionId + transactionType`.
6. Chỉ fallback confirm API khi backend state chưa reflected.
