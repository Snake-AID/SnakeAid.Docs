---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: hallucination
status: current
last_updated: 2026-05-05
owners: [backend-team]
verification_status: code-investigated
---
# Rủi Ro Hallucination: Lịch Sử Huỷ Instant Consultation

## H-001: Terminal instant request không có `Consultation` thì có nên xuất hiện trong lịch sử consultation không?

- trạng thái: `Closed`
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

Khi request hết hạn:

- backend không tạo `Consultation`
- `ConsultationPingRequest.Status = Expired`
- `ConsultationPingRequest.ConsultationId = null`
- request này hiện không xuất hiện trong consultation history

Vấn đề đã quyết định: terminal instant request không có `Consultation` là request-level event, nhưng frontend/mobile vẫn cần nhìn thấy nó trong lịch sử. Ba hướng xử lý đã được phân tích trước khi chốt union contract.

## 2. Phương án 1: Union contract với DTO riêng cho request-level row

### Mô tả

Sửa contract của history endpoint để response có thể trả về union row:

- `kind = consultation`: history row từ `Consultation`
- `kind = instant`: request-level history row từ `ConsultationPingRequest`

Backend vẫn có thể dùng endpoint hiện tại:

- `GET /api/users/me/consultations`
- `GET /api/experts/me/consultations`

Nhưng contract phải nói rõ một item có thể là consultation thật hoặc instant request-level row. `kind = instant` là DTO riêng, không phải consultation DTO được ép nullable field.

### Hành vi hệ thống

- Accepted scheduled consultation vẫn là consultation row.
- Accepted emergency consultation vẫn là consultation row vì đã có `Consultation`.
- Expert-rejected emergency request được trả về như request-level row.
- Expired emergency request được trả về như request-level row.
- Request row không có consultation thật.
- Request row không có room thật.
- Request row không được dùng cho các action chỉ dành cho consultation.

Ví dụ member history row cho request bị expert huỷ:

```json
{
  "kind": "instant",
  "instantRequestId": "11111111-1111-1111-1111-111111111111",
  "type": "Emergency",
  "requestStatus": "DeclinedByExpert",
  "requestedAt": "2026-05-04T03:09:30Z",
  "respondedAt": "2026-05-04T03:10:00Z",
  "expertId": "22222222-2222-2222-2222-222222222222",
  "expertName": "Khiêm Expert",
  "expertAvatarUrl": null
}
```

### Tác động tới mobile

Mobile phải hiểu history có 2 loại dữ liệu và branch UI theo `kind`.

Backend contract phải có field phân biệt rõ loại row để tránh mobile phải tự đoán hoặc trộn nhầm action giữa consultation row và instant request row.

### Ưu điểm

- Đúng bản chất dữ liệu.
- Không tạo fake `Consultation`.
- Không cần fake `consultationId`.
- Frontend biết rõ row nào là session thật, row nào chỉ là request bị huỷ.
- Ít gây nhiễu cho room, chat, review, payment, settlement, admin report.

### Rủi ro

- Đây là breaking/change contract cho mobile.
- Mobile phải sửa UI/render logic theo union DTO.
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

| Tiêu chí                                    | Phương án 1: Union contract + DTO riêng | Phương án 2: Giữ contract, ép ping vào response | Phương án 3: Tạo Fake `Consultation`          |
| --------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------- | --------------------------------------------------- |
| Đúng bản chất domain                      | Cao                                              | Trung bình thấp                                     | Thấp                                               |
| Có fake `Consultation` trong DB            | Không                                           | Không                                                | Có                                                 |
| Có fake/mơ hồ `consultationId` trong API | Không                                           | Có nguy cơ cao                                      | Không, nhưng bản thân `Consultation` là fake |
| Tác động mobile                            | Cao, phải sửa rõ ràng                        | Trung bình, nhưng dễ suy luận mơ hồ             | Thấp ban đầu                                     |
| Rủi ro room/chat/review                      | Thấp                                            | Trung bình                                           | Cao                                                 |
| Rủi ro reporting/payment/cleanup             | Thấp                                            | Trung bình                                           | Cao                                                 |
| Dễ maintain lâu dài                        | Cao                                              | Thấp                                                 | Thấp                                               |

## 6. Nhận định đã chốt

Nếu mục tiêu là đúng domain và dễ maintain lâu dài, phương án 1 là sạch nhất và đã được chọn.

Nếu mục tiêu là đổi ít ở database nhưng vẫn giữ endpoint cũ, phương án 2 có thể làm được nhưng contract sẽ mơ hồ nếu không thêm field phân biệt.

Nếu mục tiêu là giữ nguyên contract bằng mọi giá, phương án 3 giải quyết được bề mặt API nhưng tạo rủi ro lớn nhất cho domain, reporting, room/chat, payment và các flow consultation-scoped.

## 7. Quyết định đã chốt

Đã chọn hướng 1:

1. Tách bạch API contract để hỗ trợ cả `Consultation` row và `ConsultationPingRequest` request-level row trong cùng history response.
2. Không tạo Fake `Consultation`.
3. Không ép request-level row vào consultation DTO.
4. Dùng union contract với discriminator `kind`.

Quyết định này ảnh hưởng trực tiếp tới:

- DTO response của member/expert history
- logic query trong consultation history service
- cách mobile render history
- cách backend bảo vệ các action chỉ hợp lệ với consultation thật

Decision còn mở duy nhất trong file này là status filter mapping cho `kind = instant`.

## 8. Decision Record

- trạng thái: `Closed`
- ngày chốt: `2026-05-04`
- người chốt: user
- lựa chọn: Phương án 1 - tách bạch API contract để history có thể biểu diễn cả real consultation row và instant request row.

Quyết định:

- Expert-rejected và expired instant/emergency request phải được biểu diễn như request-level history row, không tạo Fake `Consultation`.
- Backend không được dùng `Guid.Empty`, random id, hoặc `emergencyRequestId` để giả lập `consultationId`.
- `consultationId` của request-level row phải nullable hoặc không được coi là id consultation thật.
- Contract cần có discriminator để mobile render đúng loại row.
- Discriminator kind dùng các value:
  - `consultation`: row có record `Consultation` thật trong `Consultation.cs`.
  - `instant`: row request-level cho instant/emergency request bị expert cancel/reject trước khi có record `Consultation`.
- `kind = instant` là một DTO riêng trong cùng API contract/list response, không phải consultation DTO được ép nullable field.
- History response nên được hiểu như union contract: mỗi item là `kind = consultation` hoặc `kind = instant`.
- `kind = instant` chỉ được tạo cho trường hợp không có record `Consultation`.
- Scheduled consultation và các instant/emergency request đã có pair với `Consultation` vẫn là `kind = consultation`.
- Accepted instant/emergency request không thuộc phạm vi contract row mới; vì đã có linked `Consultation`, nó tiếp tục được xử lý như consultation history row.

Ghi chú contract:

- Field contract chính cho DTO `kind = instant` đã được chốt trong decision bổ sung ngày `2026-05-05`.
- Status filter mapping cho `kind = instant` vẫn để mở. Decision này có thể cần được mở lại khi phân tích admin endpoint/history contract.

Decision bổ sung ngày `2026-05-05` cho DTO `kind = instant`:

- DTO `kind = instant` giữ field phẳng, không dùng nested `counterparty`.
- Id field dùng `instantRequestId`, map từ `ConsultationPingRequest.Id`.
- Status field dùng `requestStatus`, map từ `ConsultationPingRequest.Status`, không dùng `status` để tránh nhầm với `ConsultationStatus`.
- Timestamp field dùng `requestedAt` và `respondedAt`.
- `respondedAt` là thời điểm phù hợp để hiển thị/sort row terminal như expert cancel/reject hoặc expired.
- DTO `kind = instant` không expose `rescueMissionId` trong scope history này.
- DTO `kind = instant` không expose `expiresAt` trong scope history này.
- DTO `kind = instant` không expose `price`, `grossPrice`, hoặc `netPrice` trong scope history này vì row không đại diện cho consultation/payment settlement đã hoàn thành.
- Member history dùng field phẳng phía expert:
  - `expertId`
  - `expertName`
  - `expertAvatarUrl`
- Expert history dùng field phẳng phía member/user:
  - `userId`
  - `userName`
  - `userAvatarUrl`
- `kind = instant` hiện cover các `ConsultationPingStatus` terminal không có `Consultation`:
  - `DeclinedByExpert`
  - `Expired`
- Verification ngày `2026-05-05`: đã kiểm tra production code liên quan `ConsultationPingRequest` trong `EmergencyConsultationService`, `ConsultationPaymentService`, và `ConsultationService`.
- `AcceptedByExpert` là flow duy nhất tạo record `Consultation` và set `ConsultationPingRequest.ConsultationId`.
- `DeclinedByExpert` là terminal request-level flow không tạo `Consultation`.
- `Expired` là terminal request-level flow không tạo `Consultation`; flow expire chỉ update `ConsultationPingRequest.Status = Expired`, set `RespondedAt`, và refund escrow nếu cần.
- `PendingPayment` và `PendingExpertResponse` cũng chưa có `Consultation`, nhưng là active request states, không phải terminal history row trong scope này.
- `RescuerCancelled` hiện chỉ tồn tại trong enum `ConsultationPingStatus`; chưa thấy production endpoint/service flow nào set trạng thái này. Không đưa vào `kind = instant` history cho tới khi có flow active/current trong code.
- Status filter behavior cho `kind = instant` chưa chốt trong lượt này. Tạm giữ là open decision vì có thể liên quan đến admin history endpoint và filter contract tổng thể.

Tác động implementation dự kiến:

- Member history và expert history cần query thêm `ConsultationPingRequest` terminal thuộc scope `kind = instant`, trước mắt gồm `DeclinedByExpert` và `Expired`.
- Accepted instant/emergency request vẫn map từ linked `Consultation` và trả về `kind = consultation`.
- Rejected/expired instant/emergency request phải map từ `ConsultationPingRequest`, không map từ fake consultation.
- Mobile phải branch UI theo kind thay vì suy luận từ `consultationId`, `roomId`, hoặc `status`.

## H-002: Status filter mapping cho `kind = instant` nên hoạt động như thế nào?

- trạng thái: `Open`
- ngày phát hiện: `2026-05-05`
- phạm vi: member history, expert history, có thể mở rộng sang admin history
- endpoint liên quan:
  - `GET /api/users/me/consultations`
  - `GET /api/experts/me/consultations`
  - admin history endpoint nếu áp dụng cùng union/filter model sau này

## 1. Lý do còn mở

Query history hiện có `status` filter theo `ConsultationStatus`.

DTO `kind = instant` dùng `requestStatus` từ `ConsultationPingStatus`, trước mắt gồm:

- `DeclinedByExpert`
- `Expired`

Hai enum này không cùng nghĩa:

- `ConsultationStatus.Cancelled` là trạng thái của consultation/session thật.
- `ConsultationPingStatus.DeclinedByExpert` là request bị expert từ chối trước khi có `Consultation`.
- `ConsultationPingStatus.Expired` là request hết hạn trước khi có `Consultation`.

## 2. Impact nếu đoán sai

- Mobile có thể filter lịch sử nhưng không thấy request-level row dù row đang tồn tại.
- Backend có thể map `Cancelled` sang `DeclinedByExpert` quá sớm và làm admin/reporting hiểu nhầm request-level cancellation là consultation cancellation.
- Nếu admin history dùng cùng filter sau này, filter contract có thể cần phân biệt `status` và `requestStatus`.

## 3. Candidate options

### Option A: `status` chỉ filter `kind = consultation`

- `kind = instant` không bị ảnh hưởng bởi `status`.
- Đơn giản cho backend hiện tại.
- Có thể gây bất ngờ nếu mobile kỳ vọng `status=Cancelled` bao gồm request bị expert reject.

### Option B: `status=Cancelled` bao gồm `requestStatus=DeclinedByExpert`

- Gần với UI label "Đã huỷ".
- Có nguy cơ trộn `ConsultationStatus` với `ConsultationPingStatus`.
- Chưa đủ rõ cho `Expired`.

### Option C: thêm/duy trì filter riêng `requestStatus`

- Rõ ràng nhất về contract.
- Mobile/admin có thể filter đúng request-level status.
- Là thay đổi contract rộng hơn và cần cân nhắc chung với admin endpoint.

## 4. Required user decision

Chưa chốt trong lượt này. Giữ open cho tới khi phân tích filter contract của member/expert history và admin history đủ rõ.
