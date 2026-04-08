# 4. Luồng chính: Quản trị và điều phối hệ thống (Admin / Manager / Operator)

---

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Operational handling is now split: `Operator` handles frontline rescue verification and assignment, while `Admin / Manager` focuses on governance, configuration, reporting, and oversight.
>
> [Legacy]
> Older requirements grouped more operational behavior under Admin and also kept treatment-facility / antivenom management in active scope.
>
> [Migration Impact]
> This file is a major backend-migration reference. Separate dispatch responsibilities, managerial controls, and deprecated medical-support modules carefully.

> [!IMPORTANT]
> Payment Correction - 2026-04-08
>
> Admin configuration must not assume rescue/catching revenue share fields such as `% chia cho Rescuer` or `% quỹ bảo hiểm` as current target-state money flow.
>
> Target hiện tại: Incident và Catching thu một chiều vào system/platform. Escrow + net payout + platform fee split chỉ áp dụng cho Expert Consultation.

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
   - Cập nhật cho ứng dụng Member / Rescuer / Expert
9. Admin kiểm tra kết quả: test nhận diện bằng ảnh mẫu và xem độ chính xác.



## 4.2 Quản lý cơ sở điều trị

**Flow 4.2 — Cập nhật thông tin bệnh viện**

> [!WARNING]
> Changed Requirement
>
> [Current]
> Treatment-facility and antivenom management are no longer core strategic scope.
>
> [Legacy]
> The previous requirement set treated hospital and antivenom data as a maintained module.
>
> [Migration Impact]
> Preserve existing backend structures for migration analysis, but mark this domain as legacy unless a reduced reference-only use case is retained.

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
8. Hệ thống tự động đồng bộ với ứng dụng Member (bản đồ & thông tin cập nhật ngay lập tức).



## 4.3 Giám sát hoạt động real-time

**Flow 4.3 — Theo dõi hệ thống trên bản đồ**

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Operator must be able to verify requests, monitor online rescuers, check shift coverage, assign missions, and track mission execution on the map.
>
> [Legacy]
> Older flow versions focused more on generic map monitoring and less on explicit operator-led dispatch control.
>
> [Migration Impact]
> Backend needs online-presence, shift, operator assignment, rescue ping, and mission-monitoring support.

1. Operator hoặc Manager mở Dashboard **"Giám sát Real-time"**.
2. Bản đồ hiển thị trạng thái:
   - 🔴 Ca rắn cắn / cứu hộ đang xử lý
   - 🟡 Yêu cầu đang ở trạng thái VERIFY hoặc chờ assign
   - 🟠 RESCUE PING REQUEST đang chờ Rescuer phản hồi
   - 🟢 Ca đã hoàn thành trong ngày
   - 🔵 Vị trí Rescuer đang online
   - ⚫ Vị trí Expert đang online
3. Operator có thể click vào điểm để xem chi tiết, theo dõi vị trí Rescuer, trạng thái nhiệm vụ, và tình trạng ca trực.
4. Xem biểu đồ nhiệt (Heat Map) — phân bố sự cố theo thời gian và loài.
5. Nếu phát hiện bất thường hoặc yêu cầu chờ quá lâu: gửi cảnh báo cộng đồng hoặc can thiệp vận hành (ưu tiên ca khẩn cấp).
6. Xuất báo cáo cuối ngày: tổng số ca, thời gian phản hồi trung bình, tỷ lệ hoàn thành, doanh thu.



## 4.4 Quản lý tài chính

**Flow 4.4 — Báo cáo và phân chia doanh thu**

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> The system currently serves a single rescue center, and incident cost is calculated from the rescue center to the incident location.
>
> [Legacy]
> Previous pricing assumptions were broader and less explicitly anchored to a single-center operating model.
>
> [Migration Impact]
> Review pricing rules, dispatch distance calculation, and financial reporting assumptions in backend services.

1. Admin truy cập **"Quản lý Tài chính"**.
2. Thiết lập phí dịch vụ: phí cứu hộ, phí tư vấn, và cấu hình platform fee cho Expert Consultation; không cấu hình % chia cho Rescuer hoặc % quỹ bảo hiểm cho rescue/catching target-state.
3. Xem báo cáo doanh thu theo ngày/tuần/tháng/năm, theo Rescuer/Expert.
4. Quản lý thanh toán: danh sách giao dịch chờ, kiểm tra và xử lý giao dịch lỗi.
5. Xử lý tranh chấp: kiểm tra lịch sử GPS / trạng thái thanh toán và đưa ra quyết định hoàn tiền hoặc từ chối.
6. Quản lý hoàn tiền: duyệt yêu cầu, hoàn tiền trong 3–5 ngày làm việc và cập nhật trạng thái.
7. Xuất báo cáo tài chính định kỳ cho Ban Giám đốc.



---
