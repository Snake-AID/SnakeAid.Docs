# 1. Luồng chính: Xử lý sự cố rắn cắn khẩn cấp

---

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> The system only supports initial rescue handling. Medical treatment is not a core business scope, and antivenom management is no longer a strategic module.
>
> [Legacy]
> Earlier flow versions included stronger in-system treatment-facility and antivenom-oriented support.
>
> [Migration Impact]
> Preserve legacy treatment-related requirements for backend migration, but do not treat them as the target product scope.

## 1.1 Giai đoạn phát hiện và xử lý ban đầu (Member)

**Flow 1.1 — Khi người dùng bị rắn cắn**

1. Người dùng mở ứng dụng **SnakeAid**.
2. Chọn chức năng **"Tôi bị rắn cắn — Cần trợ giúp khẩn cấp"**.
3. Hệ thống hiển thị hướng dẫn sơ cứu ngay lập tức:
   - Hướng dẫn băng ép từng bước
   - Hình ảnh minh họa cách băng ép đúng
   - Cảnh báo hành động cấm kỵ **(KHÔNG rạch vết thương, KHÔNG hút độc)**
4. Người dùng thực hiện sơ cứu theo hướng dẫn.
5. Hệ thống yêu cầu chụp ảnh rắn (nếu có thể).
6. Nếu có ảnh rắn:
   - AI nhận diện loài rắn.
   - Hiển thị kết quả: tên, độc tính, mức độ nguy hiểm.
   - Đề xuất biện pháp sơ cứu phù hợp.
   Nếu không có ảnh rắn: chuyển sang bước 7.
7. Yêu cầu chụp ảnh vết cắn và nhập triệu chứng:
   - Nhập mô tả triệu chứng (đau, sưng, tê, buồn nôn...)
   - Chụp ảnh vết cắn
8. AI đánh giá mức độ nghiêm trọng:
   - Phân loại: **Nhẹ / Trung bình / Nặng / Nguy kịch**.
9. Hành động theo mức độ:
   - **Nếu Nặng hoặc Nguy kịch**:
     - Hệ thống tự động cảnh báo khẩn cấp.
     - Hiển thị nút **"GỌI CẤP CỨU NGAY"** nổi bật.
     - Chuyển sang **Flow 1.2**.
   - **Nếu Nhẹ hoặc Trung bình**:
     - Hiển thị hướng dẫn tiếp tục sơ cứu.
     - Đề xuất tìm cơ sở y tế gần nhất.
     - Chuyển sang **Flow 1.3**.

> **Ghi chú:** Trong mọi trường hợp nghi ngờ nặng, khuyến cáo người dùng liên hệ cấp cứu ngay lập tức và theo dõi các dấu hiệu sinh tồn.

---

## 1.2 Giai đoạn gọi cấp cứu và chia sẻ vị trí

**Flow 1.2 — Kích hoạt SOS và gọi cấp cứu**

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> A rescue request should be understood as entering the center-operated rescue flow, where Operator is expected to receive and process the request first.
>
> [Legacy]
> Older flow versions implied a more direct emergency-call pattern and less explicit center-side operational handling.
>
> [Migration Impact]
> Backend SOS handling should be reviewed to ensure it can create incidents for operator verification and assignment.

1. Người dùng nhấn nút **SOS**.
2. Hệ thống tự động:
   - Lấy vị trí GPS hiện tại.
3. Hệ thống chia sẻ vị trí real-time:
   - Kích hoạt chế độ theo dõi liên tục.
4. Hệ thống gửi thông tin bổ sung cho đội cấp cứu:
   - Kết quả nhận diện rắn (nếu có).
   - Ảnh vết cắn.
   - Triệu chứng đã ghi nhận.
   - Mức độ nghiêm trọng do AI đánh giá.
5. Hiển thị màn hình chờ cấp cứu:
   - Tiếp tục hiển thị hướng dẫn sơ cứu.

---

## 1.3 Giai đoạn tìm cơ sở điều trị gần nhất

**Flow 1.3 — Định vị bệnh viện có huyết thanh**

> [!WARNING]
> Changed Requirement
>
> [Current]
> Hospital routing and antivenom management are no longer part of the strategic core scope. If retained, this flow should be treated as legacy or external-reference-only support.
>
> [Legacy]
> The previous requirement set modeled treatment-facility discovery and antivenom availability as active system features.
>
> [Migration Impact]
> Do not delete backend capability blindly. Mark treatment-facility logic as legacy and assess deprecation or externalization path first.

1. Người dùng chọn **"Tìm bệnh viện gần nhất"**.
2. Hệ thống lấy vị trí GPS hiện tại.
3. Truy vấn cơ sở dữ liệu cơ sở điều trị:
   - Lọc các bệnh viện/trạm y tế có huyết thanh kháng nọc.
   - Ưu tiên cơ sở trong bán kính **20 km**.
   - Sắp xếp theo khoảng cách.
4. Hiển thị bản đồ kèm danh sách cơ sở y tế:
   - Tính khoảng cách và thời gian ước tính.
   - Hiển thị loại huyết thanh có sẵn tại mỗi cơ sở.
   - Đánh dấu cơ sở mở cửa **24/7**.
5. Người dùng chọn một cơ sở trên danh sách.
6. Hệ thống cung cấp: 
   - Nút **"Chỉ đường"** (mở Google Maps / Apple Maps).
   - Số điện thoại để gọi trước.
   - Thông tin chi tiết về cơ sở.
7. Lưu lịch sử vào hồ sơ sức khỏe của người dùng.

---
