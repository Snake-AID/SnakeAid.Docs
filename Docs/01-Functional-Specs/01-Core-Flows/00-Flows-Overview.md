# MAIN FLOW - HỆ THỐNG SNAKEAID

## Thông tin dự án
- **Tên dự án:** AI-Powered Platform for Snakebite First Aid and Rescue Support (SnakeAid)
- **Mục đích:** Xác định luồng chính của hệ thống cho các tình huống sử dụng quan trọng

---

## SƠ ĐỒ TỔNG QUAN CÁC LUỒNG CHÍNH

**Phiên bản văn bản (Headings & Bullet list):**

### PATIENT
- **[1] Bị rắn cắn**
  - Sơ cứu
  - Nhận diện AI
  - Gọi SOS
  - Tìm bệnh viện
  - Yêu cầu cứu hộ
  - Thanh toán

### RESCUER
- **[2] Nhận cảnh báo**
  - Chấp nhận yêu cầu
  - Di chuyển & chia sẻ vị trí
  - Bắt rắn
  - Báo cáo & cập nhật hệ thống
  - Nhận thanh toán

### EXPERT
- **[3] Xác minh**
  - Tư vấn (chat / video)
  - Cập nhật database loài rắn
  - Nhận thanh toán

### ADMIN
- **Vai trò chính (giám sát & quản lý):**
  - Quản lý User
  - Quản lý Database Rắn
  - Quản lý Bệnh viện
  - Giám sát Real-time
  - Gửi Cảnh báo Cộng đồng
  - Quản lý Tài chính & báo cáo

> **Ghi chú:** Các hoạt động của Patient / Rescuer / Expert đều được ghi nhận và/hoặc chuyển tiếp về Admin để giám sát, cập nhật DB hoặc kích hoạt các hành động tiếp theo.

---

## 7. MA TRẬN TƯƠNG TÁC GIỮA CÁC MODULE

| Tình huống | Patient | Rescuer | Expert | Admin | AI System |
|------------|---------|---------|--------|-------|-----------|
| Rắn cắn khẩn cấp | Kích hoạt | - | (Nếu cần) | Giám sát | Nhận diện + Đánh giá |
| Cứu hộ rắn | Yêu cầu | Thực hiện | (Hỗ trợ) | Giám sát | Nhận diện sơ bộ |
| Tư vấn từ xa | Yêu cầu | - | Tư vấn | Giám sát | - |
| Cập nhật database | - | Góp ảnh | Xác minh | Quản lý | Học từ dữ liệu mới |
| Cảnh báo cộng đồng | Nhận | Nhận | - | Gửi | Phân tích xu hướng |
| Thanh toán | Trả tiền | Nhận tiền | Nhận tiền | Quản lý | - |

---

## 8. THỜI GIAN XỬ LÝ DỰ KIẾN

| Luồng | Thời gian dự kiến | Ghi chú |
|-------|-------------------|---------|
| Nhận diện rắn bằng AI | < 5 giây | Tùy chất lượng ảnh |
| Đánh giá mức độ nghiêm trọng | < 3 giây | AI xử lý |
| Tìm cơ sở điều trị gần nhất | < 2 giây | Truy vấn database |
| Tìm Rescuer phù hợp | < 30 giây | Tối đa 2 phút |
| Rescuer di chuyển đến hiện trường | 10-30 phút | Tùy khoảng cách |
| Bắt rắn | 5-20 phút | Tùy loài và tình huống |
| Tư vấn chuyên gia | 15-30 phút | Tùy độ phức tạp |
| Thanh toán qua cổng | < 10 giây | Nếu mạng ổn định |
