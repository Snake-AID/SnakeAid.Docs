# SNAKEAID - SNAKECATCHING (NGHIỆP VỤ THỢ CỨU HỘ BẮT RẮN)

## 1. Mục tiêu & phạm vi
- Cung cấp dịch vụ **bắt/di dời rắn theo yêu cầu** khi khách hàng phát hiện rắn **chưa bị cắn**.
- Không áp dụng cho tình huống rắn cắn khẩn cấp (được chuyển sang luồng SOS y tế).
- Bao phủ toàn bộ hành trình: báo cáo → ghép thợ → xử lý hiện trường → thanh toán → đánh giá.

## 2. Actors & hệ thống liên quan
- **Khách hàng (Patient module):** gửi yêu cầu, theo dõi, thanh toán, đánh giá.
- **Snake Rescuer:** tiếp nhận, di chuyển, bắt rắn, cập nhật trạng thái.
- **Snake Expert (tùy chọn):** hỗ trợ nhận diện và tư vấn kỹ thuật bắt an toàn.
- **AI System:** nhận diện loài rắn, mức nguy hiểm từ ảnh.
- **Backend System:** matching, điều phối, thông báo, lưu dữ liệu.
- **Payment Gateway:** xử lý thanh toán/escrow.
- **Admin:** giám sát real-time, xử lý tranh chấp, báo cáo tài chính.

## 3. Điều kiện vào/ra
**Đầu vào bắt buộc**
- Ảnh rắn (nhiều góc độ nếu có).
- Vị trí GPS + mô tả vị trí cụ thể (trong nhà/ngoài trời/vườn...).
- Mô tả hành vi rắn + mức độ khẩn cấp.

**Đầu ra**
- Rắn được bắt/di dời hoặc xử lý theo kết quả thực tế.
- Hóa đơn điện tử + lịch sử giao dịch.
- Đánh giá và phản hồi khách hàng.
- Bản ghi dữ liệu (ảnh, vị trí, thời gian, loài rắn).

## 4. Luồng nghiệp vụ chính (Bắt rắn có phí)

### 4.1. Báo cáo phát hiện rắn
1. Khách hàng mở app, chọn **"Báo cáo phát hiện rắn"**.
2. Chụp ảnh rắn, hệ thống lấy **GPS tự động (FE-18)**.
3. Nhập thông tin bổ sung: vị trí cụ thể, kích thước ước tính, hành vi, mức khẩn.
4. AI phân tích ảnh → trả về **loài rắn (FE-12), độc tính (FE-13), mức nguy hiểm**.
5. Khách hàng chọn **"Yêu cầu đội cứu hộ đến bắt rắn (có phí)"**.

### 4.2. Matching đội cứu hộ
1. Backend xác định yêu cầu (GPS, mức khẩn, loại rắn).
2. Tìm Rescuer phù hợp:
   - Online trong bán kính 10km.
   - Có kinh nghiệm với loài rắn này.
   - Rating tốt.
3. Gửi thông báo đến **top 3 Rescuer**, thời gian chấp nhận **2 phút**.
4. Nếu không có ai nhận:
   - Mở rộng bán kính lên 20km.
   - Gửi top 5 Rescuer tiếp theo.
   - Tăng mức phí đề xuất **+20%**.
5. Nếu vẫn không có Rescuer sau 5 phút → thông báo thất bại và gợi ý gọi trung tâm kiểm soát động vật/115.

### 4.3. Thực hiện cứu hộ
1. Rescuer chấp nhận yêu cầu (FE-06).
2. Hệ thống hiển thị thông tin Rescuer cho khách hàng (tên, ảnh, rating, SĐT).
3. Rescuer chuẩn bị:
   - Xem ảnh và kết quả AI (FE-21).
   - Đọc hướng dẫn an toàn (FE-09, FE-10).
   - Chuẩn bị thiết bị (FE-23).
4. Nếu chưa chắc chắn loài rắn → kích hoạt **hỗ trợ Expert** (FE-12).
5. Rescuer di chuyển, cập nhật trạng thái:
   - **Đang trên đường** (FE-07)
   - Bật GPS real-time (FE-18)
6. Khách hàng theo dõi trên bản đồ (FE-24/25/26), nhận thông báo "cách 5 phút".
7. Rescuer đến nơi, cập nhật **Đã đến** (FE-20), sau đó **Đang xử lý**.
8. Bắt rắn, chụp ảnh xác nhận (FE-16), cập nhật **Hoàn thành**.
9. Backend lưu dữ liệu (FE-15): vị trí, thời gian, loài rắn, kết quả xử lý.

### 4.4. Thanh toán & đánh giá
1. Hệ thống tạo hóa đơn, gửi yêu cầu thanh toán.
2. Khách hàng thanh toán số dư qua Momo/VNPay/ZaloPay/Thẻ (FE-28).
3. Phân chia doanh thu:
   - **Rescuer 85%**
   - **Platform 10%**
   - **Quỹ bảo hiểm 5%**
4. Khách hàng đánh giá (1-5 sao + nhận xét).
5. Lưu lịch sử giao dịch (FE-30), doanh thu Rescuer (FE-25), báo cáo Admin (FE-33).

## 5. Nhánh miễn phí: Cảnh báo cộng đồng
- Nếu khách hàng chọn **"Chỉ cảnh báo cộng đồng"**:
  - Backend gửi cảnh báo đến người dùng lân cận (FE-20).
  - Không phát sinh thanh toán, kết thúc luồng.

## 6. Hỗ trợ chuyên gia (Rescuer ↔ Expert)
- Rescuer gửi ảnh/video real-time khi gặp rắn khó xác định.
- Backend ưu tiên Expert online theo khu vực và kinh nghiệm.
- Expert tư vấn qua chat/video: nhận diện loài, mức nguy hiểm, kỹ thuật bắt an toàn.
- Thanh toán hỗ trợ:
  - Platform trả phí Expert **hoặc** Rescuer chia sẻ **10%** phí cứu hộ cho Expert (luồng hỗ trợ khẩn cấp).

## 7. Chính sách giá & phụ phí (tóm tắt)

### 7.1. Giá cơ bản theo kích thước & độc tính
| Kích thước | Độ dài | Không độc | Độc nhẹ | Độc mạnh | Cực độc |
|---|---|---|---|---|---|
| Nhỏ | < 1m | 500,000 đ | 900,000 đ | 1,200,000 đ | 1,500,000 đ |
| Vừa | 1-2m | 1,000,000 đ | 1,500,000 đ | 2,000,000 đ | 2,500,000 đ |
| Lớn | 2-3m | 2,000,000 đ | 2,500,000 đ | 3,500,000 đ | 4,500,000 đ |
| Rất lớn | > 3m | 3,000,000 đ | 4,000,000 đ | 5,500,000 đ | 7,000,000 đ |

### 7.2. Phụ phí theo khoảng cách (1 chiều)
| Khoảng cách | Giá/km |
|---|---|
| 0-3 km | 8,000 đ |
| 3-10 km | 9,000 đ |
| 10-20 km | 10,000 đ |
| 20-50 km | 11,000 đ |
| > 50 km | 12,000 đ |

**Lưu ý:** Rescuer có quyền từ chối nếu > 30 km; đường khó đi +20%.

### 7.3. Phụ phí theo thời gian
| Khung giờ | Phụ thu |
|---|---|
| 06h-18h | 0 đ |
| 18h-22h | +50,000 đ |
| 22h-06h | +100,000 đ |
| Chủ nhật/Lễ | +80,000 đ |
| Tết Nguyên Đán | +200,000 đ |

### 7.4. Phụ phí theo môi trường
| Địa điểm | Phụ thu |
|---|---|
| Trong nhà | +150,000 đ |
| Sân vườn | 0 đ |
| Trên cây | +300,000 đ |
| Dưới nước | +500,000 đ |
| Trong tường/trần | +600,000 đ |
| Khu công nghiệp | +1,000,000 đ |
| Resort/Villa | +2,000,000 đ |
| Khách sạn/Nhà nghỉ | +800,000 đ |
| Quán cà phê/Karaoke | +1,200,000 đ |
| Bãi rác/cống rãnh | +500,000 đ |

### 7.5. Phụ phí đặc biệt
- Khẩn cấp trong 30 phút: +500,000 đ.
- Nhiều con rắn: +50% mỗi con từ con thứ 2.
- Cần 2 Rescuer: +1,500,000 đ (rắn > 3m/cực nguy hiểm).
- Diện tích lớn > 1000m2: +1,000,000 đ.
- Không xác định được loài: +300,000 đ.
- Cần bẫy (không thấy rắn): +800,000 đ.
- Dịch vụ phòng ngừa/kiểm tra: +500,000 đ.

## 8. Thanh toán & tiền cọc (Rescue Request)
- **Cọc cố định 150,000 đ** để Rescuer chấp nhận yêu cầu (tiền vào ESCROW).
- Sau khi hoàn thành, khách hàng thanh toán **số dư còn lại**.
- Cọc được trừ vào tổng chi phí (không cộng thêm).
- Phân chia mặc định: **Rescuer 85% / Platform 10% / Quỹ bảo hiểm 5%**.
- Nếu không thanh toán số dư trong 24h → nhắc nhở; 72h → khóa tài khoản.

## 9. Chính sách hủy & ngoại lệ
- **Hủy trước khi Rescuer chấp nhận:** hoàn 100% cọc.
- **Hủy sau khi Rescuer chấp nhận:** mất cọc; phân chia 85/10/5.
- **Rescuer không đến:** hoàn cọc 100% + bù 50K + voucher 100K; Rescuer bị cảnh cáo/giảm rating.
- **Địa chỉ sai/ảo:** mất cọc; tài khoản bị blacklist.
- **Rắn tự chạy mất:** chia đôi cọc (75K Rescuer / 75K hoàn khách).
- **Không tìm được Rescuer:** thông báo thất bại, gợi ý liên hệ trung tâm kiểm soát động vật.

## 10. Dữ liệu ghi nhận & giám sát
- Dữ liệu lưu trữ: ảnh rắn (trước/sau), GPS, mô tả, thời gian phản hồi, trạng thái, kết quả xử lý, hóa đơn, đánh giá.
- Admin theo dõi bản đồ real-time, thời gian phản hồi, tỷ lệ hoàn thành, doanh thu.

## 11. SLA/Thời gian mục tiêu
- Tìm Rescuer phù hợp: **< 30 giây** (tối đa 2 phút).
- Rescuer di chuyển: **10-30 phút** (tùy khoảng cách).
- Bắt rắn: **5-20 phút** (tùy loài/tình huống).
- Thanh toán: **< 10 giây** (nếu mạng ổn định).

## ERD

<iframe width="560" height="315" src='https://dbdiagram.io/e/6952e38939fa3db27bc32d65/6953993739fa3db27bccea45'></iframe>
