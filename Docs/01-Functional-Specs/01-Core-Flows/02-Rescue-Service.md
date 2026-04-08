# 2. Luồng chính: Yêu cầu cứu hộ rắn

---

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Rescue requests are now handled by a center-operated flow: Operator receives the request first, verifies it with the Member, then assigns an online, in-shift Rescuer.
>
> [Legacy]
> This file previously described a direct rescuer self-pick model after earlier edits.
>
> [Migration Impact]
> Rescue dispatch is a breaking-change area for backend migration. Keep legacy behavior visible at the changed points below.

> [!IMPORTANT]
> Payment Correction - 2026-04-08
>
> Rescue/catching payments are one-way payments into the system/platform. Do not model this flow as escrow-to-rescuer, rescuer revenue share, or 85% / 10% / 5% split.
>
> Escrow, net payout, and platform fee split apply only to Expert Consultation. Rescuers are system staff, so payment settlement does not release customer money to a rescuer wallet.

## 2.1 Giai đoạn phát hiện và báo cáo rắn (Member)

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

## 2.2 Giai đoạn operator tiếp nhận và điều phối yêu cầu

**Flow 2.2 — Operator verify và assign Rescuer**

1. Hệ thống nhận yêu cầu cứu hộ từ Member.
2. Operator là người nhìn thấy request đầu tiên trên hệ thống.
3. Operator liên hệ lại Member để xác minh thông tin sự cố.
4. Incident chuyển sang trạng thái **VERIFY**.
5. Sau khi xác minh hợp lệ, Operator xác định:
   - Vị trí GPS của yêu cầu
   - Mức độ khẩn cấp
   - Loại rắn (từ AI)
6. Operator chọn Rescuer phù hợp và kiểm tra:
   - Rescuer đang online hay không
   - Rescuer có trong ca trực hay không
   - Rescuer có phù hợp với khu vực và mức độ khẩn cấp hay không
7. Hệ thống tạo **RESCUE PING REQUEST (PENDING)** cho Rescuer đã được assign.
8. Kịch bản:
   - **Nếu Rescuer chấp nhận** → Chuyển sang **Flow 2.3**.
   - **Nếu Rescuer từ chối hoặc không phản hồi** → Operator chọn Rescuer khác để assign lại.
   - **Nếu Member hủy yêu cầu** → Incident chuyển sang trạng thái hủy.

> [!NOTE]
> [Legacy]
> The previously documented flow let Rescuer browse requests by area and self-pick jobs, with first-come-first-served locking.
>
> [Migration Impact]
> Replace self-pick queue behavior with operator queue, verification state, assignment action, and rescue ping lifecycle.

---

## 2.3 Giai đoạn thực hiện cứu hộ (Rescuer)

**Flow 2.3 — Quá trình cứu hộ rắn**

1. Snake Rescuer nhận **RESCUE PING REQUEST** và chấp nhận nhiệm vụ.
2. Hệ thống tự động:
   - Thông báo cho Member: "Yêu cầu đã được xác minh và đã có Rescuer nhận nhiệm vụ"
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
5. Member theo dõi trên bản đồ.
6. Rescuer đến nơi → cập nhật "Đã đến".
7. Thực hiện bắt rắn → cập nhật "Đang xử lý".
8. Sau khi bắt xong:
   - Chụp ảnh rắn đã bắt
   - Xác nhận loài rắn
   - Cập nhật trạng thái "Hoàn thành"
9. Trường hợp Rescuer hủy nhiệm vụ:
   - Incident chuyển sang **MISSION ABORT**
   - Rescuer bắt buộc cung cấp lý do hủy
   - Operator tiếp nhận lại để điều phối bước tiếp theo
10. Lưu thông tin vào database và cập nhật database Admin.
11. Thanh toán & đánh giá → Chuyển sang **Flow 2.4**.

---

## 2.4 Giai đoạn thanh toán và đánh giá

**Flow 2.4 — Hoàn tất giao dịch**

1. Rescuer đánh dấu "Hoàn thành nhiệm vụ".
2. Hệ thống gửi thông báo đến Member: "Cứu hộ hoàn tất. Vui lòng thanh toán và đánh giá."
3. Member xác nhận và thanh toán:
   - Hiển thị hóa đơn: Phí cứu hộ
   - Phương thức: Momo / VNPay / ZaloPay / Thẻ
4. Sau khi thanh toán thành công → Member đánh giá Rescuer (1–5 sao + nhận xét).
5. Hệ thống ghi nhận thanh toán một chiều vào system/platform; không phân chia 85% / 10% / 5% và không release tiền sang ví Rescuer.
6. Rescuer nhận thông báo và cập nhật rating.
7. Lưu lịch sử giao dịch và báo cáo cho Member / Rescuer / Admin.

---
