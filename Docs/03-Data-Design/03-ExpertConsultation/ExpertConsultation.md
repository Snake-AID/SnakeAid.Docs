# SNAKEAID - EXPERT CONSULTATION (NGHIỆP VỤ TƯ VẤN CHUYÊN GIA)

## 1. Mục tiêu & phạm vi
- Cung cấp dịch vụ tư vấn chuyên gia rắn cho Patient theo hình thức chat/video call.
- Bao phủ quy trình: tìm expert -> chọn loại tư vấn -> upload tài liệu -> thanh toán trước (escrow) -> tư vấn -> đánh giá.
- Áp dụng cho tư vấn thường (instant/scheduled). Không bao gồm tư vấn SOS khẩn cấp khi bị rắn cắn và hỗ trợ Rescuer (được mô tả ở luồng riêng).

## 2. Actors & hệ thống liên quan
- **Patient (Mobile App):** tìm expert, tạo yêu cầu, thanh toán, tham gia phiên tư vấn, đánh giá.
- **Snake Expert:** nhận/duyệt yêu cầu, tư vấn, kết thúc phiên, nhận thanh toán.
- **Backend System:** quản lý danh sách expert, lịch, escrow, trạng thái phiên, lưu lịch sử.
- **Payment Gateway:** xử lý thanh toán, giữ tiền escrow.
- **Chat/Video Service:** kênh trao đổi real-time.
- **Admin (gián tiếp):** giám sát, xử lý khiếu nại/hoàn tiền.

## 3. Điều kiện vào/ra
**Đầu vào bắt buộc**
- Patient đã đăng nhập.
- Chọn Expert + loại tư vấn (ngay/đặt lịch).
- Tài liệu: ảnh rắn/vết cắn + mô tả vấn đề + câu hỏi cụ thể.
- Thanh toán thành công phí tư vấn (FE-27).

**Đầu ra**
- Phiên tư vấn hoàn tất (chat/video call) hoặc lịch hẹn được xác nhận.
- Hướng dẫn/khuyến nghị tư vấn được ghi nhận.
- Hóa đơn điện tử + lịch sử giao dịch (FE-29, FE-30).
- Đánh giá Expert (rating + nhận xét).

## 4. Luồng nghiệp vụ chính (Patient -> Expert)

### 4.1. Tìm kiếm và chọn Expert
1. Patient vào mục "Tư vấn chuyên gia".
2. Hệ thống hiển thị danh sách Expert: tên, chuyên môn, rating, phí tư vấn.
3. Patient lọc theo chuyên ngành, sắp xếp theo rating/giá.
4. Patient chọn một Expert để xem chi tiết.

### 4.2. Chọn loại tư vấn
1. Patient chọn:
   - **Tư vấn ngay** (chỉ khi Expert online).
   - **Đặt lịch tư vấn** (chọn ngày/giờ và thời lượng tư vấn).
2. Nếu Expert offline và Patient chọn tư vấn ngay -> hệ thống gợi ý chuyển sang đặt lịch hoặc chọn Expert khác.

### 4.3. Upload tài liệu & thanh toán trước
1. Patient upload tài liệu: ảnh rắn/vết cắn, mô tả vấn đề, câu hỏi cụ thể.
2. Hệ thống tạo hóa đơn và yêu cầu thanh toán (FE-27).
3. Payment Gateway xử lý thanh toán -> tiền vào **ESCROW**.
4. Backend tạo Consultation Request và gửi đến Expert.

### 4.4. Xử lý yêu cầu từ Expert
- **Tư vấn ngay:** Expert nhận thông báo khẩn.
  - Nếu **chấp nhận** -> mở phiên chat/video call.
  - Nếu **từ chối** -> hoàn tiền 100% và gợi ý Expert khác.
- **Đặt lịch:** Expert xác nhận lịch hẹn -> hệ thống gửi lịch cho Patient và nhắc trước 30 phút.

### 4.5. Phiên tư vấn trực tuyến
1. Đến giờ hẹn, hệ thống gửi thông báo cho cả hai.
2. Patient và Expert tham gia phiên chat/video call (FE-10).
3. Expert xem tài liệu đã gửi và tư vấn:
   - Nhận diện loài rắn, đánh giá nguy cơ.
   - Hướng dẫn sơ cứu/biện pháp xử lý.
   - Khuyến nghị đến cơ sở y tế khi cần (FE-07/08/09).
4. Expert có thể yêu cầu bổ sung ảnh/thông tin nếu cần.

### 4.6. Hoàn tất, thanh toán & đánh giá
1. Expert đánh dấu "Hoàn thành".
2. Hệ thống giải ngân từ escrow -> **90% Expert / 10% Platform**.
3. Xuất hóa đơn điện tử (FE-14).
4. Patient đánh giá Expert (rating 1-5 sao + nhận xét).
5. Lưu lịch sử tư vấn và doanh thu (FE-30, FE-15, FE-16).

## 5. Thanh toán & giá dịch vụ (tóm tắt)

### 5.1. Gói tư vấn từ xa
- thanh toán theo thời lượng tư vấn nhân cho giá tiền mỗi phút của chuyên gia đó

ví dụ tư vấn viên A có giá tư vấn là 5000vnd/phút thì tư vấn 10 phút sẽ là 50 000 vnd  

### 5.2. Quy tắc thanh toán
- Thanh toán 100% trước khi Expert nhận yêu cầu.
- Tiền được giữ trong **ESCROW** cho đến khi Expert đánh dấu hoàn thành.
- Expert nhận tiền trong 5-10 phút sau khi hoàn tất.

### 5.3. Phương thức thanh toán
- Momo, VNPay, ZaloPay, thẻ (Visa/Master/JCB).

## 6. Chính sách hủy/hoàn tiền & ngoại lệ
- **Patient hủy trước khi Expert chấp nhận:** hoàn 100%.
- **Patient hủy sau khi Expert chấp nhận:** mất 100% (Expert đã giữ lịch).
- **Expert không tham gia đúng giờ (quá 5 phút):** hoàn 100% cho Patient.
- **Expert đến muộn > 15 phút:** Patient có quyền hủy và hoàn 100%.
- **Khiếu nại chất lượng:** mở trong 24h, Admin xem xét hoàn 50% tùy trường hợp.

## 7. SLA/Thời gian mục tiêu
- Tư vấn ngay: Expert phản hồi <= 2 phút (nếu online).
- Tư vấn đặt lịch: hệ thống nhắc trước 30 phút.
- Thời lượng buổi tư vấn: 15-60 phút (tùy gói).
- Xử lý thanh toán: < 10 giây; giải ngân: 5-10 phút.

## 8. Dữ liệu ghi nhận & giám sát
- Consultation request, trạng thái, lịch hẹn.
- Tài liệu upload (ảnh/video/mô tả).
- Metadata phiên chat/video (thời gian bắt đầu/kết thúc).
- Payment, invoice, escrow status.
- Feedback/rating, lịch sử tư vấn.

## 9. Features liên quan
| Feature ID | Mô tả | Actor |
|---|---|---|
| FE-10 | Tư vấn online qua chat/video call | Expert/Patient |
| FE-13 | Thiết lập mức phí tư vấn | Expert |
| FE-14 | Nhận thanh toán + xuất hóa đơn | Expert |
| FE-15 | Xem báo cáo doanh thu | Expert |
| FE-16 | Theo dõi lượt tư vấn & đánh giá | Expert |
| FE-27 | Thanh toán phí tư vấn | Patient |
| FE-29 | Theo dõi trạng thái thanh toán, hóa đơn | Patient |
| FE-30 | Xem lịch sử giao dịch/tư vấn | Patient |
| FE-30 (Admin) | Thiết lập phí tư vấn | Admin |
| FE-33 (Admin) | Báo cáo tài chính | Admin |

## 10. ERD
- Tham khảo: `Docs/03-BussinesFlow/03-ExpertConsultation/ExpertConsultation.dbml`
