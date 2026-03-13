# MAIN FLOW - HỆ THỐNG SNAKEAID

## Thông tin dự án
- **Tên dự án:** AI-Powered Platform for Snakebite First Aid and Rescue Support (SnakeAid)
- **Mục đích:** Xác định luồng chính của hệ thống cho các tình huống sử dụng quan trọng

---

> [!IMPORTANT]
> Changed Requirement Summary
>
> [Current]
> The new operating model is center-led: `Operator` sees requests first, verifies with the Member, assigns an in-shift / online Rescuer, and monitors execution.
>
> [Legacy]
> Older flows emphasized more direct system-to-rescuer or rescuer-self-pick handling.
>
> [Migration Impact]
> Use this file as a high-level map of changed business flow. Detailed legacy notes appear in the affected flow files.

## SƠ ĐỒ TỔNG QUAN CÁC LUỒNG CHÍNH

**Phiên bản văn bản (Headings & Bullet list):**

### MEMBER
- **[1] Bị rắn cắn**
  - Sơ cứu
  - Nhận diện AI
  - Gọi SOS
  - Gửi yêu cầu cứu hộ
  - Thanh toán

### OPERATOR
- **[2] Tiếp nhận và điều phối yêu cầu**
  - Nhìn thấy request đầu tiên
  - Xác minh với Member
  - Kiểm tra Rescuer online / trong ca trực
  - Assign Rescuer
  - Theo dõi trạng thái nhiệm vụ trên map

> [!NOTE]
> [Legacy]
> Previously documented rescue flows let the system or Rescuer take a more direct role in selecting missions.

### RESCUER
- **[3] Nhận nhiệm vụ đã được điều phối**
  - Nhận rescue ping request
  - Chấp nhận nhiệm vụ
  - Di chuyển & chia sẻ vị trí
  - Bắt rắn
  - Báo cáo & cập nhật hệ thống
  - Nhận thanh toán

### EXPERT
- **[4] Xác minh / tư vấn**
  - Tư vấn (chat / video)
  - Cập nhật database loài rắn
  - Nhận thanh toán

### ADMIN
- **Vai trò chính (giám sát & quản lý):**
  - Quản lý User
  - Quản lý Database Rắn
  - Quản trị cấu hình trung tâm
  - Giám sát Real-time
  - Gửi Cảnh báo Cộng đồng
  - Quản lý Tài chính & báo cáo

> **Ghi chú:** Các hoạt động của Member / Rescuer / Expert đều được ghi nhận và/hoặc chuyển tiếp về Admin để giám sát, cập nhật DB hoặc kích hoạt các hành động tiếp theo.

---

## 7. MA TRẬN TƯƠNG TÁC GIỮA CÁC MODULE

| Tình huống | Member | Operator | Rescuer | Expert | Admin / Manager | AI System |
|------------|--------|----------|---------|--------|-----------------|-----------|
| Rắn cắn khẩn cấp | Kích hoạt | Tiếp nhận / xác minh | Thực hiện nếu được điều phối | (Nếu cần) | Giám sát | Nhận diện + Đánh giá |
| Cứu hộ rắn | Tạo yêu cầu | Verify + assign | Nhận ping + thực hiện | (Hỗ trợ) | Giám sát | Nhận diện sơ bộ |
| Tư vấn từ xa | Yêu cầu | - | - | Tư vấn | Giám sát | - |
| Cập nhật database | - | - | Góp ảnh | Xác minh | Quản lý | Học từ dữ liệu mới |
| Cảnh báo cộng đồng | Nhận | Theo dõi / kích hoạt theo vận hành | Nhận | - | Gửi | Phân tích xu hướng |
| Thanh toán | Trả tiền | - | Nhận tiền | Nhận tiền | Quản lý | - |

---

## 8. THỜI GIAN XỬ LÝ DỰ KIẾN

| Luồng | Thời gian dự kiến | Ghi chú |
|-------|-------------------|---------|
| Nhận diện rắn bằng AI | < 5 giây | Tùy chất lượng ảnh |
| Đánh giá mức độ nghiêm trọng | < 3 giây | AI xử lý |
| Tìm cơ sở điều trị gần nhất | < 2 giây | Truy vấn database |
| Operator verify và assign Rescuer | 1-5 phút | Tùy tốc độ xác minh và trạng thái ca trực |
| Rescuer di chuyển đến hiện trường | 10-30 phút | Tùy khoảng cách |
| Bắt rắn | 5-20 phút | Tùy loài và tình huống |
| Tư vấn chuyên gia | 15-30 phút | Tùy độ phức tạp |
| Thanh toán qua cổng | < 10 giây | Nếu mạng ổn định |
