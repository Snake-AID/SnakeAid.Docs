---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: hallucination
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-investigated
---
# Rủi Ro Hallucination: Lịch Sử Huỷ Instant Consultation

## H-001: Expert huỷ instant request thì có nên xuất hiện trong lịch sử consultation không?

- trạng thái: `Open`
- ngày phát hiện: `2026-05-04`
- phạm vi: member history và expert history
- endpoint liên quan:
  - `GET /api/users/me/consultations`
  - `GET /api/experts/me/consultations`

## 1. Sự thật hiện tại trong code

Instant/emergency request hiện được lưu bằng `ConsultationPingRequest`.

Khi expert accept:

- backend tạo một `Consultation`
- `ConsultationPingRequest.Status = AcceptedByExpert`
- `ConsultationPingRequest.ConsultationId` có giá trị
- request này có thể xuất hiện trong consultation history

Khi expert reject/cancel:

- backend không tạo `Consultation`
- `ConsultationPingRequest.Status = DeclinedByExpert`
- `ConsultationPingRequest.ConsultationId = null`
- request này hiện không xuất hiện trong consultation history

Vấn đề cần quyết định: request bị expert huỷ là một request-level event, nhưng frontend/mobile muốn nhìn thấy nó trong phần lịch sử. Có 3 hướng xử lý khả thi.

## 2. Phương án 1: Sửa api contract thành 2 loại riêng, bắt mobile code 2 màn hình lịch sử

### Mô tả

Sửa contract của history endpoint để response có thể trả về cả:

- history row từ `Consultation`
- history row từ `ConsultationPingRequest`

Backend vẫn có thể dùng endpoint hiện tại:

- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

Nhưng contract phải nói rõ một row có thể là consultation thật hoặc instant request.

### Hành vi hệ thống

- Accepted scheduled consultation vẫn là consultation row.
- Accepted emergency consultation vẫn là consultation row vì đã có `Consultation`.
- Expert-rejected emergency request được trả về như request row.
- Request row không có consultation thật.
- Request row không có room thật.
- Request row không được dùng cho các action chỉ dành cho consultation.

Ví dụ response row cho request bị expert huỷ:

```json
{
  "recordKind": "EmergencyRequest",
  "consultationId": null,
  "emergencyRequestId": "11111111-1111-1111-1111-111111111111",
  "type": "Emergency",
  "status": "Cancelled",
  "requestStatus": "DeclinedByExpert",
  "roomId": null,
  "startTime": "2026-05-04T03:10:00Z",
  "endTime": "2026-05-04T03:10:00Z"
}
```

### Tác động tới mobile

Mobile phải hiểu history có 2 loại dữ liệu.

Mobile phải code UI theo 2 màn hình/section riêng:

- consultation history
- instant request history

Backend contract vẫn nên có field phân biệt rõ loại row để tránh mobile phải tự đoán hoặc trộn nhầm action giữa 2 màn hình.

### Ưu điểm

- Đúng bản chất dữ liệu.
- Không tạo fake `Consultation`.
- Không cần fake `consultationId`.
- Frontend biết rõ row nào là session thật, row nào chỉ là request bị huỷ.
- Ít gây nhiễu cho room, chat, review, payment, settlement, admin report.

### Rủi ro

- Đây là breaking/change contract cho mobile.
- `consultationId` cần nullable hoặc cần model response mới.
- Mobile phải sửa UI/render logic.
- Cần document rõ action nào không được phép với request row.

## 3. Phương án 2: Giữ contract cũ, trộn `ConsultationPingRequest` vào consultation history response

### Mô tả

Giữ response contract gần như hiện tại của `/me/consultations`, nhưng backend query thêm các `ConsultationPingRequest` bị expert huỷ rồi trộn vào danh sách history cùng với `Consultation`.

Phương án này không tạo thêm `Consultation` trong database.

### Hành vi hệ thống

- Backend lấy các consultation thật như hiện tại.
- Backend lấy thêm các ping request có `Status = DeclinedByExpert`.
- Backend map ping request thành history row trong cùng response contract.
- Row bị huỷ vẫn không có `Consultation` thật phía database.

### Vấn đề chính

Contract hiện tại được thiết kế cho consultation row, không phải request row.

Các field dễ bị sai nghĩa:

- `consultationId`: request bị huỷ không có consultation id.
- `roomId`: request bị huỷ không có room.
- `startTime`: nếu dùng `RequestedAt` hoặc `RespondedAt` thì đây là thời điểm request, không phải thời điểm session bắt đầu.
- `endTime`: nếu dùng `RespondedAt` thì đây là thời điểm expert huỷ, không phải thời điểm session kết thúc.
- `status`: `DeclinedByExpert` là `ConsultationPingStatus`, không phải `ConsultationStatus`.

### Các cách lách contract và rủi ro

Nếu `consultationId` vẫn non-null, backend phải chọn một cách không sạch:

- dùng `Guid.Empty`
- nhét `emergencyRequestId` vào `consultationId`
- sinh random id không tồn tại trong bảng `Consultation`
- trả `null` dù contract cũ không nói nullable

Các cách này đều làm mobile dễ hiểu nhầm row đó là consultation thật.

### Rủi ro tiềm tàng

- Mobile có thể gọi nhầm consultation detail bằng id không hợp lệ.
- Mobile có thể gọi nhầm message history, review, join room, report absent.
- UI phải suy luận bằng giá trị đặc biệt thay vì contract rõ ràng.
- Sorting/filtering dễ mơ hồ vì trộn `ConsultationStatus` với `ConsultationPingStatus`.
- Sau này khi có cancelled consultation thật, frontend khó phân biệt với request bị expert reject.

### Khi nào có thể chấp nhận

Chỉ nên cân nhắc nếu muốn thay đổi backend rất nhanh và mobile chấp nhận xử lý technical debt rõ ràng.

Nếu chọn hướng này, vẫn nên thêm ít nhất một discriminator như `recordKind`; nhưng khi thêm discriminator/nullability thì thực chất đã chuyển gần sang Phương án 1.

## 4. Phương án 3: Giữ contract cũ bằng cách tạo Fake `Consultation`

### Mô tả

Khi expert huỷ instant request, backend tạo một `Consultation` dù không có session thật. Mục tiêu là để history response vẫn có `consultationId` thật và không phải sửa contract.

Sau đó:

- `ConsultationPingRequest.ConsultationId` trỏ tới consultation mới tạo
- history endpoint có thể fetch như accepted emergency consultation
- response nhìn giống consultation history row bình thường

### Hành vi hệ thống

- Expert reject request.
- Backend tạo một `Consultation` trạng thái `Cancelled`.
- Backend gắn consultation này vào `ConsultationPingRequest`.
- History trả về row có `consultationId` non-null.

### Vấn đề chính

Database sẽ có một Fake `Consultation` không đại diện cho consultation session thật.

Trong domain hiện tại, `Consultation` là session/call entity. Nó có các field required:

- `CallerId`
- `CalleeId`
- `RoomId`
- `StartTime`
- `Status`
- `Type`

Vì vậy "Consultation rỗng" không thật sự rỗng được. Backend vẫn phải tự điền các field bắt buộc.

### Rủi ro tiềm tàng

- `RoomId` phải được fake hoặc tạo dù không có call room thật.
- Chat/SignalR/room join có thể nhầm đây là consultation thật nếu chỉ check `consultationId`.
- Review/message history/detail API có thể nhìn thấy consultation này.
- Admin/reporting có thể đếm nhầm rejected request thành cancelled consultation.
- Payment/settlement/cleanup job phải thêm guard để không xử lý fake consultation như session thật.
- Các query theo `Consultation.Type = Emergency` bị nhiễu vì có cả emergency session thật và Fake `Consultation`.
- Business logic sau này phải nhớ phân biệt "cancelled consultation thật" với "request bị expert reject nhưng được tạo consultation để thoả contract".

### Ưu điểm

- Ít thay đổi API contract nhất ở bề mặt.
- Mobile có thể tiếp tục nhận `consultationId` non-null.
- Dễ làm cho history endpoint nếu chỉ nhìn từ response hiện tại.

### Nhược điểm

- Đẩy complexity xuống database và domain model.
- Tạo dữ liệu không phản ánh đúng nghiệp vụ.
- Rủi ro dài hạn cao hơn hai phương án còn lại.

## 5. So sánh nhanh

| Tiêu chí                                    | Phương án 1: Tách contract + mobile 2 screen | Phương án 2: Giữ contract, ép ping vào response | Phương án 3: Tạo Fake `Consultation`          |
| --------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------- | --------------------------------------------------- |
| Đúng bản chất domain                      | Cao                                              | Trung bình thấp                                     | Thấp                                               |
| Có fake `Consultation` trong DB            | Không                                           | Không                                                | Có                                                 |
| Có fake/mơ hồ `consultationId` trong API | Không                                           | Có nguy cơ cao                                      | Không, nhưng bản thân `Consultation` là fake |
| Tác động mobile                            | Cao, phải sửa rõ ràng                        | Trung bình, nhưng dễ suy luận mơ hồ             | Thấp ban đầu                                     |
| Rủi ro room/chat/review                      | Thấp                                            | Trung bình                                           | Cao                                                 |
| Rủi ro reporting/payment/cleanup             | Thấp                                            | Trung bình                                           | Cao                                                 |
| Dễ maintain lâu dài                        | Cao                                              | Thấp                                                 | Thấp                                               |

## 6. Nhận định hiện tại

Nếu mục tiêu là đúng domain và dễ maintain lâu dài, phương án 1 là sạch nhất.

Nếu mục tiêu là đổi ít ở database nhưng vẫn giữ endpoint cũ, phương án 2 có thể làm được nhưng contract sẽ mơ hồ nếu không thêm field phân biệt.

Nếu mục tiêu là giữ nguyên contract bằng mọi giá, phương án 3 giải quyết được bề mặt API nhưng tạo rủi ro lớn nhất cho domain, reporting, room/chat, payment và các flow consultation-scoped.

## 7. Quyết định cần chốt

Cần chọn một trong 3 hướng:

1. Tách bạch API contract để hỗ trợ cả `Consultation` và `ConsultationPingRequest`, đồng thời mobile code 2 màn hình/section lịch sử.
2. Giữ API contract, fetch thêm `ConsultationPingRequest` rồi ép/trộn vào history response.
3. Giữ API contract, khi expert huỷ thì tạo Fake `Consultation` để thoả contract.

Quyết định này ảnh hưởng trực tiếp tới:

- DTO response của member/expert history
- logic query trong consultation history service
- cách mobile render history
- cách backend bảo vệ các action chỉ hợp lệ với consultation thật
