# 1. Luồng chính: Xử lý sự cố rắn cắn khẩn cấp

---

## 1.1 Giai đoạn phát hiện và xử lý ban đầu (Patient)

**Flow 1.1 — Khi người dùng bị rắn cắn**

1. Người dùng mở ứng dụng **SnakeAid**.
2. Chọn chức năng **"Tôi bị rắn cắn — Cần trợ giúp khẩn cấp"**.
3. Hệ thống hiển thị hướng dẫn sơ cứu ngay lập tức:
   - `FE-01` — Hướng dẫn băng ép từng bước
   - `FE-02` — Hình ảnh minh họa cách băng ép đúng
   - `FE-03` — Cảnh báo hành động cấm kỵ **(KHÔNG rạch vết thương, KHÔNG hút độc)**
4. Người dùng thực hiện sơ cứu theo hướng dẫn.
5. Hệ thống yêu cầu chụp ảnh rắn (nếu có thể).
6. Nếu có ảnh rắn:
   - AI nhận diện loài rắn (`FE-12`).
   - Hiển thị kết quả: tên, độc tính, mức độ nguy hiểm (`FE-13`).
   - Đề xuất biện pháp sơ cứu phù hợp (`FE-14`).
   Nếu không có ảnh rắn: chuyển sang bước 7.
7. Yêu cầu chụp ảnh vết cắn và nhập triệu chứng:
   - `FE-09` — Nhập mô tả triệu chứng (đau, sưng, tê, buồn nôn...)
   - `FE-10` — Chụp ảnh vết cắn
8. AI đánh giá mức độ nghiêm trọng (`FE-15`):
   - Phân loại: **Nhẹ / Trung bình / Nặng / Nguy kịch** (`FE-17`).
9. Hành động theo mức độ:
   - **Nếu Nặng hoặc Nguy kịch**:
     - Hệ thống tự động cảnh báo khẩn cấp (`FE-16`).
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

1. Người dùng nhấn nút **SOS** (`FE-04`).
2. Hệ thống tự động:
   - Lấy vị trí GPS hiện tại.
   - Gọi trực tiếp đến đường dây nóng cấp cứu **115**.
   - Gửi tin nhắn SMS chứa tọa độ GPS đến **115**.
3. Hệ thống chia sẻ vị trí real-time (`FE-05`):
   - Gửi link theo dõi vị trí cho đội cấp cứu.
   - Kích hoạt chế độ theo dõi liên tục.
4. Hệ thống gửi thông tin bổ sung cho đội cấp cứu:
   - Kết quả nhận diện rắn (nếu có).
   - Ảnh vết cắn.
   - Triệu chứng đã ghi nhận.
   - Mức độ nghiêm trọng do AI đánh giá.
5. Hiển thị màn hình chờ cấp cứu:
   - Timer đếm thời gian.
   - Nút **"Hủy cuộc gọi"** (nếu tình hình thay đổi).
   - Tiếp tục hiển thị hướng dẫn sơ cứu.
6. Đồng thời gửi thông báo cho người thân (nếu đã cấu hình trong hồ sơ).

---

## 1.3 Giai đoạn tìm cơ sở điều trị gần nhất

**Flow 1.3 — Định vị bệnh viện có huyết thanh**

1. Người dùng chọn **"Tìm bệnh viện gần nhất"**.
2. Hệ thống lấy vị trí GPS hiện tại.
3. Truy vấn cơ sở dữ liệu cơ sở điều trị (`FE-06`):
   - Lọc các bệnh viện/trạm y tế có huyết thanh kháng nọc.
   - Ưu tiên cơ sở trong bán kính **20 km**.
   - Sắp xếp theo khoảng cách.
4. Hiển thị bản đồ kèm danh sách cơ sở y tế:
   - `FE-07` — Tính khoảng cách và thời gian ước tính.
   - `FE-08` — Hiển thị loại huyết thanh có sẵn tại mỗi cơ sở.
   - Đánh dấu cơ sở mở cửa **24/7**.
5. Người dùng chọn một cơ sở trên danh sách.
6. Hệ thống cung cấp: 
   - Nút **"Chỉ đường"** (mở Google Maps / Apple Maps).
   - Số điện thoại để gọi trước.
   - Thông tin chi tiết về cơ sở.
7. Lưu lịch sử vào hồ sơ sức khỏe của người dùng (`FE-11`).

---
