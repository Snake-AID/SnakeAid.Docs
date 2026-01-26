# 2. Luồng chính: Yêu cầu cứu hộ rắn

---

## 2.1 Giai đoạn phát hiện và báo cáo rắn (Patient)

**Flow 2.1 — Báo cáo phát hiện rắn**

1. Người dùng phát hiện rắn (chưa bị cắn).
2. Mở ứng dụng → Chọn **"Báo cáo phát hiện rắn"**.
3. Chụp ảnh rắn (có thể nhiều góc độ).
4. Hệ thống tự động lấy vị trí GPS (`FE-18`).
5. Người dùng bổ sung thông tin:
   - Vị trí cụ thể (trong nhà / ngoài trời / vườn...)
   - Kích thước ước tính
   - Mô tả hành vi rắn
   - Mức độ khẩn cấp (rắn trong nhà / khu vực đông người)
6. AI phân tích ảnh và đưa ra nhận định sơ bộ:
   - Loài rắn có thể (`FE-12`)
   - Độc tính (`FE-13`)
   - Mức độ nguy hiểm
7. Hiển thị 2 lựa chọn:
   - **[A]** Yêu cầu đội cứu hộ đến bắt rắn (có phí).
   - **[B]** Chỉ cảnh báo cộng đồng (miễn phí).
8. Hành động:
   - **Nếu [A]** → Chuyển sang **Flow 2.2**.
   - **Nếu [B]** → Gửi cảnh báo (`FE-20`) → Kết thúc.

---

## 2.2 Giai đoạn kết nối với đội cứu hộ (Matching)

**Flow 2.2 — Phân công đội cứu hộ**

1. Hệ thống nhận yêu cầu cứu hộ từ Patient.
2. Hệ thống xác định:
   - Vị trí GPS của yêu cầu
   - Mức độ khẩn cấp
   - Loại rắn (từ AI)
3. Tìm kiếm Snake Rescuer phù hợp:
   - Đang online trong bán kính **5 km**
   - Có kinh nghiệm với loài rắn này
   - Đánh giá tốt từ khách hàng trước
   - Sắp xếp theo: Khoảng cách → Rating → Thời gian phản hồi
4. Gửi thông báo đến top 3 Snake Rescuer (`FE-01`):
   - Thông tin vị trí
   - Ảnh rắn và kết quả AI
   - Mức phí đề xuất
   - Thời gian chấp nhận: **2 phút**
5. Kịch bản:
   - **Nếu có Rescuer chấp nhận trong 2 phút** → Chuyển sang **Flow 2.3**.
   - **Nếu không có** → Mở rộng bán kính lên **10 km**, gửi đến top 5 và tăng phí đề xuất 20%.
   - **Nếu vẫn không có sau 5 phút** → Mở rộng bán kính lên **15 km**. Nếu vẫn không tìm thấy → Thông báo cho Patient: "Không tìm thấy đội cứu hộ" và đề xuất gọi trung tâm kiểm soát động vật hoặc 115.

---

## 2.3 Giai đoạn thực hiện cứu hộ (Rescuer)

**Flow 2.3 — Quá trình cứu hộ rắn**

1. Snake Rescuer chấp nhận yêu cầu (`FE-06`).
2. Hệ thống tự động:
   - Thông báo cho Patient: "Đã tìm thấy đội cứu hộ"
   - Hiển thị thông tin Rescuer (tên, ảnh, rating, SĐT)
   - Kích hoạt chia sẻ vị trí real-time
3. Rescuer chuẩn bị:
   - Xem lại ảnh rắn & kết quả AI (`FE-21`)
   - Đọc hướng dẫn an toàn (FE-09 / FE-10)
   - Chuẩn bị thiết bị (FE-23)
   - Nếu cần → Liên hệ Expert (`FE-12`) → Chuyển sang **Flow 3.2**
4. Rescuer di chuyển:
   - Cập nhật trạng thái "Đang trên đường" (`FE-07`)
   - Bật chia sẻ vị trí real-time (`FE-18`)
5. Patient theo dõi trên bản đồ (`FE-24` / `FE-25` / `FE-26`).
6. Rescuer đến nơi → cập nhật "Đã đến" (`FE-20`).
7. Thực hiện bắt rắn → cập nhật "Đang xử lý" (`FE-07`).
8. Sau khi bắt xong:
   - Chụp ảnh rắn đã bắt (`FE-16`)
   - Xác nhận loài rắn
   - Cập nhật trạng thái "Hoàn thành" (`FE-07`)
9. Lưu thông tin vào database (`FE-15`) và cập nhật database Admin.
10. Thanh toán & đánh giá → Chuyển sang **Flow 2.4**.

---

## 2.4 Giai đoạn thanh toán và đánh giá

**Flow 2.4 — Hoàn tất giao dịch**

1. Rescuer đánh dấu "Hoàn thành nhiệm vụ".
2. Hệ thống gửi thông báo đến Patient: "Cứu hộ hoàn tất. Vui lòng thanh toán và đánh giá."
3. Patient xác nhận & thanh toán (`FE-28`):
   - Hiển thị hóa đơn: Phí cứu hộ + Phí nền tảng (10%)
   - Phương thức: Momo / VNPay / ZaloPay / Thẻ
4. Sau khi thanh toán thành công → Patient đánh giá Rescuer (1–5 sao + nhận xét).
5. Hệ thống phân chia thanh toán:
   - 85% → Tài khoản Rescuer (`FE-26`)
   - 10% → Phí nền tảng
   - 5% → Quỹ bảo hiểm
6. Rescuer nhận thông báo & cập nhật rating (`FE-27`).
7. Lưu lịch sử giao dịch và báo cáo cho Patient / Rescuer / Admin (`FE-30` / `FE-25` / `FE-33`).

---
