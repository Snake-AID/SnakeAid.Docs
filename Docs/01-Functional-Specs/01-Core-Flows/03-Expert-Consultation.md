# 3. Luồng chính: Tư vấn chuyên gia

---

## 3.1 Giai đoạn yêu cầu tư vấn (Patient)

**Flow 3.1 — Đặt lịch tư vấn với chuyên gia**

1. Patient truy cập **"Tư vấn chuyên gia"**.
2. Xem danh sách Snake Expert:
   - Hiển thị: Tên, Chuyên môn, Rating, Phí tư vấn
   - Lọc theo: Chuyên ngành (Rắn độc Việt Nam / Rắn ngoại lai / Điều trị nọc độc)
   - Sắp xếp theo: Rating / Phí tư vấn
3. Chọn một Expert.
4. Chọn loại tư vấn:
   - **[A]** Tư vấn ngay (nếu Expert online).
   - **[B]** Đặt lịch tư vấn — chọn ngày giờ.
5. Upload tài liệu cần tư vấn:
   - Ảnh rắn hoặc vết cắn
   - Mô tả vấn đề
   - Câu hỏi cụ thể
6. Thanh toán phí tư vấn trước:
   - Số tiền tạm giữ (escrow)
   - Chỉ chuyển cho Expert sau khi tư vấn xong
7. Kịch bản:
   - **Nếu [A]** → Expert nhận thông báo → nếu chấp nhận → bắt đầu chat/video call → chuyển sang **Flow 3.3**.
   - **Nếu [B]** → Expert nhận yêu cầu → xác nhận lịch → gửi lịch hẹn & nhắc nhở 30 phút.

---

## 3.2 Giai đoạn Rescuer xin hỗ trợ từ Expert

**Flow 3.2 — Tư vấn khẩn cấp cho Rescuer**

1. Rescuer tại hiện trường gặp khó khăn nhận diện rắn.
2. Trong app Rescuer chọn **"Yêu cầu hỗ trợ chuyên gia"**.
3. Chụp ảnh/video rắn real-time.
4. Hệ thống tìm Expert đang online (ưu tiên chuyên về khu vực này) và gửi thông báo khẩn cấp đến top 3.
5. Expert phản hồi nhanh nhất sẽ được kết nối.
6. Bắt đầu tư vấn qua chat/video call: Expert xác định loài rắn, tư vấn cách bắt an toàn và cảnh báo rủi ro.
7. Sau tư vấn: Expert cập nhật kết quả vào hệ thống; Rescuer kết thúc phiên.
8. Thanh toán tự động theo chính sách (nền tảng trả phí cho Expert hoặc Rescuer chia sẻ % theo thỏa thuận).

---

## 3.3 Giai đoạn tư vấn trực tuyến

**Flow 3.3 — Buổi tư vấn giữa Patient và Expert**

1. Đến giờ hẹn, cả hai nhận thông báo.
2. Bắt đầu phiên tư vấn — Chat text hoặc Video call.
3. Expert xem thông tin Patient đã gửi: ảnh, mô tả, câu hỏi.
4. Expert tư vấn, đưa khuyến nghị, có thể yêu cầu thêm thông tin.
5. Expert có thể: cập nhật hướng dẫn sơ cứu, cung cấp thông tin liều lượng huyết thanh hoặc khuyến nghị đến bệnh viện.
6. Kết thúc buổi tư vấn: Expert đánh dấu "Hoàn thành" và thời gian tư vấn được ghi nhận.
7. Hệ thống xử lý thanh toán: chuyển tiền từ escrow sang Expert, trừ phí nền tảng 10%, xuất hóa đơn.
8. Patient đánh giá Expert (1–5 sao + nhận xét).
9. Lưu lịch sử tư vấn và báo cáo cho Patient / Expert / Admin.

---
