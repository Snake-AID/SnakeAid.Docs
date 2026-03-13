# 4. Luồng chính: Quản trị hệ thống (Admin)

---

## 4.1 Quản lý database loài rắn

**Flow 4.1 — Cập nhật thông tin loài rắn**

1. Admin đăng nhập vào Admin Portal.
2. Truy cập **"Quản lý Database Rắn"**.
3. Chọn chức năng:
   - **[A]** Thêm loài rắn mới
   - **[B]** Chỉnh sửa thông tin loài rắn hiện có
   - **[C]** Xóa loài rắn (nếu nhập nhầm)
4. Thêm mới — thông tin cần nhập:
   - Tên khoa học (Latin)
   - Tên tiếng Việt
   - Tên địa phương
   - Độc tính: Có độc / Không độc / Độc nhẹ
   - Mức độ nguy hiểm: Thấp / Trung bình / Cao / Rất cao
   - Đặc điểm nhận dạng (màu sắc, hoa văn, kích thước)
   - Phân bố địa lý (Miền Bắc/Trung/Nam, tỉnh)
   - Môi trường sống & hành vi
5. Upload hình ảnh rắn:
   - Tối thiểu 5 ảnh, nhiều góc độ, chất lượng cao
   - Gắn tag: Đầu, thân, đuôi, hoa văn
6. Phân loại theo khu vực: chọn tỉnh và mức độ phổ biến.
7. Lưu vào database.
8. Hệ thống tự động:
   - Đồng bộ dữ liệu mới với AI Model
   - Retrain mô hình nhận diện nếu cần
   - Cập nhật cho ứng dụng Patient / Rescuer / Expert
9. Admin kiểm tra kết quả: test nhận diện bằng ảnh mẫu và xem độ chính xác.



## 4.2 Quản lý cơ sở điều trị

**Flow 4.2 — Cập nhật thông tin bệnh viện**

1. Admin truy cập **"Quản lý Cơ sở Điều trị"**.
2. Chọn **[Thêm mới]** hoặc **[Chỉnh sửa]**.
3. Nhập thông tin bệnh viện:
   - Tên, địa chỉ chi tiết
   - Tọa độ GPS (hoặc chọn trên bản đồ)
   - Số điện thoại khẩn cấp, email
4. Cập nhật thông tin huyết thanh:
   - Danh sách loại huyết thanh (polyvalent, kháng nọc rắn hổ mang / lục / hổ...)
   - Số lượng tồn kho (nếu cơ sở chia sẻ)
   - Ngày cập nhật cuối cùng
5. Cấu hình thời gian hoạt động: giờ mở/đóng, 24/7, lịch nghỉ lễ.
6. Phân loại cơ sở: tuyến trung ương / tỉnh / trạm y tế / phòng khám tư.
7. Lưu vào database.
8. Hệ thống tự động đồng bộ với ứng dụng Patient (bản đồ & thông tin cập nhật ngay lập tức).



## 4.3 Giám sát hoạt động real-time

**Flow 4.3 — Theo dõi hệ thống trên bản đồ**

1. Admin mở Dashboard **"Giám sát Real-time"**.
2. Bản đồ hiển thị trạng thái:
   - 🔴 Ca rắn cắn đang xử lý
   - 🟡 Yêu cầu cứu hộ đang chờ
   - 🟢 Ca đã hoàn thành trong ngày
   - 🔵 Vị trí Rescuer đang online
   - ⚫ Vị trí Expert đang online
3. Admin có thể click vào điểm để xem chi tiết, theo dõi vị trí Rescuer và trạng thái nhiệm vụ.
4. Xem biểu đồ nhiệt (Heat Map) — phân bố sự cố theo thời gian và loài.
5. Nếu phát hiện bất thường: gửi cảnh báo cộng đồng hoặc can thiệp vận hành (ưu tiên ca khẩn cấp).
6. Xuất báo cáo cuối ngày: tổng số ca, thời gian phản hồi trung bình, tỷ lệ hoàn thành, doanh thu.



## 4.4 Quản lý tài chính

**Flow 4.4 — Báo cáo và phân chia doanh thu**

1. Admin truy cập **"Quản lý Tài chính"**.
2. Thiết lập phí dịch vụ: phí cứu hộ, phí tư vấn, % hoa hồng nền tảng, % chia cho Rescuer, % quỹ bảo hiểm.
3. Xem báo cáo doanh thu theo ngày/tuần/tháng/năm, theo Rescuer/Expert.
4. Quản lý thanh toán: danh sách giao dịch chờ, kiểm tra và xử lý giao dịch lỗi.
5. Xử lý tranh chấp: kiểm tra lịch sử GPS / trạng thái thanh toán và đưa ra quyết định hoàn tiền hoặc từ chối.
6. Quản lý hoàn tiền: duyệt yêu cầu, hoàn tiền trong 3–5 ngày làm việc và cập nhật trạng thái.
7. Xuất báo cáo tài chính định kỳ cho Ban Giám đốc.



---
