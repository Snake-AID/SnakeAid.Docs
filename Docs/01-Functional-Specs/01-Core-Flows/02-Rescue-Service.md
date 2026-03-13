# 2. Luồng chính: Yêu cầu cứu hộ rắn

---

## 2.1 Giai đoạn phát hiện và báo cáo rắn (Patient)

**Flow 2.1 — Báo cáo phát hiện rắn**

1. Người dùng phát hiện rắn (chưa bị cắn).
2. Mở ứng dụng → Chọn **"Báo cáo phát hiện rắn"**.
3. Chụp ảnh rắn (có thể nhiều góc độ).
4. Hệ thống tự động lấy vị trí GPS.
5. Người dùng bổ sung thông tin:
   - Vị trí cụ thể (trong nhà / ngoài trời / vườn...)
   - Kích thước ước tính
   - Mô tả hành vi rắn
   - Mức độ khẩn cấp (rắn trong nhà / khu vực đông người)
6. AI phân tích ảnh và đưa ra nhận định sơ bộ:
   - Loài rắn có thể
   - Độc tính
   - Mức độ nguy hiểm
7. Hiển thị 2 lựa chọn:
   - **[A]** Yêu cầu đội cứu hộ đến bắt rắn (có phí).
   - **[B]** Chỉ cảnh báo cộng đồng (miễn phí).
8. Hành động:
   - **Nếu [A]** → Chuyển sang **Flow 2.2**.
   - **Nếu [B]** → Gửi cảnh báo → Kết thúc.

---

## 2.2 Giai đoạn hiển thị yêu cầu cho đội cứu hộ

**Flow 2.2 — Rescuer tự nhận yêu cầu cứu hộ**

1. Hệ thống nhận yêu cầu cứu hộ từ Patient.
2. Hệ thống xác định:
   - Vị trí GPS của yêu cầu
   - Mức độ khẩn cấp
   - Loại rắn (từ AI)
3. Hệ thống lọc và hiển thị yêu cầu cho các Snake Rescuer phù hợp theo khu vực:
   - Chỉ hiển thị cho Rescuer đang online trong khu vực gần hiện trường
   - Hiển thị thông tin vị trí, ảnh rắn, kết quả AI, mức độ khẩn cấp và phí cứu hộ niêm yết
   - Sắp xếp danh sách theo khoảng cách và thời gian tạo yêu cầu
4. Snake Rescuer chủ động mở danh sách yêu cầu cứu hộ và chọn job muốn nhận.
5. Cơ chế giữ job:
   - Rescuer nào nhấn nhận trước sẽ giữ job
   - Ngay sau đó hệ thống khóa yêu cầu để các Rescuer khác không thể nhận cùng lúc
   - Patient được thông báo ngay khi đã có Rescuer nhận thành công
6. Kịch bản:
   - **Nếu có Rescuer nhận job** → Chuyển sang **Flow 2.3**.
   - **Nếu chưa có ai nhận** → Yêu cầu tiếp tục hiển thị trong danh sách theo khu vực.
   - **Nếu quá thời gian chờ mà vẫn không có ai nhận** → Hệ thống tự động chuyển yêu cầu thành cảnh báo cộng đồng và thông báo cho Patient.

---

## 2.3 Giai đoạn thực hiện cứu hộ (Rescuer)

**Flow 2.3 — Quá trình cứu hộ rắn**

1. Snake Rescuer mở danh sách yêu cầu và nhận thành công một job.
2. Hệ thống tự động:
   - Thông báo cho Patient: "Đã tìm thấy đội cứu hộ"
   - Hiển thị thông tin Rescuer (tên, ảnh, rating, SĐT)
   - Kích hoạt chia sẻ vị trí real-time
3. Rescuer chuẩn bị:
   - Xem lại ảnh rắn và kết quả AI
   - Đọc hướng dẫn an toàn
   - Chuẩn bị thiết bị
   - Nếu cần → Liên hệ Expert → Chuyển sang **Flow 3.2**
4. Rescuer di chuyển:
   - Cập nhật trạng thái "Đang trên đường"
   - Bật chia sẻ vị trí real-time
5. Patient theo dõi trên bản đồ.
6. Rescuer đến nơi → cập nhật "Đã đến".
7. Thực hiện bắt rắn → cập nhật "Đang xử lý".
8. Sau khi bắt xong:
   - Chụp ảnh rắn đã bắt
   - Xác nhận loài rắn
   - Cập nhật trạng thái "Hoàn thành"
9. Lưu thông tin vào database và cập nhật database Admin.
10. Thanh toán & đánh giá → Chuyển sang **Flow 2.4**.

---

## 2.4 Giai đoạn thanh toán và đánh giá

**Flow 2.4 — Hoàn tất giao dịch**

1. Rescuer đánh dấu "Hoàn thành nhiệm vụ".
2. Hệ thống gửi thông báo đến Patient: "Cứu hộ hoàn tất. Vui lòng thanh toán và đánh giá."
3. Patient xác nhận và thanh toán:
   - Hiển thị hóa đơn: Phí cứu hộ + Phí nền tảng (10%)
   - Phương thức: Momo / VNPay / ZaloPay / Thẻ
4. Sau khi thanh toán thành công → Patient đánh giá Rescuer (1–5 sao + nhận xét).
5. Hệ thống phân chia thanh toán:
   - 85% → Tài khoản Rescuer
   - 10% → Phí nền tảng
   - 5% → Quỹ bảo hiểm
6. Rescuer nhận thông báo và cập nhật rating.
7. Lưu lịch sử giao dịch và báo cáo cho Patient / Rescuer / Admin.

---
