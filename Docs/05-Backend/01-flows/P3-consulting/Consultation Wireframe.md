---
Đây là wireframe của flow tư vấn
---

# Series Người dùng:

## Screen: Consulting Homepage

- Section "Buổi đặt lịch của tôi"
  - Card: Đang trong quá trình đặt: <Button: Thanh toán ngay>
  - Card: Đã đặt, đến giờ <Button: Vào phòng ngay>
  - Thông tin buổi đặt

## Screen: Danh sách chuyên gia

- Card: Chuyên gia
  - Họ tên
  - Chuyên ngành
  - Trạng thái xác minh
  - Đánh giá chuyên gia và số lượng đánh giá `Đánh giá (số lượng)`
  - Đơn giá tư vấn
    > Cần check các chuyên gia có online (Có thể cần signalR)

## Screen: Thông tin profile chuyên gia

- Thông tin cá nhân
  - Avatar
  - Họ tên
  - Online/Offline
  - Đánh giá và số lượng
  - Trạng thái xác minh
  - Chuyên ngành
  - Giới thiệu
  - Kinh nghiệm
  - Thống kê: Ca tư vấn - Thời gian phản hồi - tỉ lệ thành công
  - Phí tư vấn: Tư vấn ngay và Tư vấn đặt lịch
- Lịch trống mà chuyên gia đã set trước đó
  - Hiện các ngày khả dụng
- Các đánh gia từ người dùng
  - Card list:
    - Avatar khách
    - Họ tên khách
    - Số sao
    - Nội dung đánh giá

## Screen: Chọn loại tư vấn

- Section: Thông tin chuyên gia đang chọn
  - Avatar
  - Họ tên
  - Chuyên ngành
- Section: Tư vấn ngay
  - Trạng thái online/offline
  - Giá tư vấn ngay
  - Nút gửi yêu cầu tư vấn
- Section: Đặt lịch tư vấn
  - Đơn giá tư vấn đặt lịch
  - Nút chọn đặt lịch

## Usecase: Tư vấn ngay

> - Edge Case (Nghiệp vu: thời gian tư vấn được phân theo slot 30 phút cố định)
>   - Nếu slot n bị cấn thì slot n-1 sẽ không được cho phép đặt
>   - Rất dễ xảy ra trường hợp thời điểm người dùng yêu cầu tư vấn ngay thì đã ở giữa slot.
>     - Nếu Expert chấp nhận những yêu cầu có thời gian bắt đầu nằm ở giữa bước nhảy slot thì 2 slot bị lượt chấp nhận chiếm sẽ được disable ngầm dưới db
>   - Có thể xem hình ảnh minh họa:![slot paradox](<media/Slot Paradox.png>)

- Nhảy sang Screen: Thanh toán
- Thanh toán thành công: Mở phòng ngay

## Usecase: Đặt lịch tư vấn

- Chọn thời gian
  - Hiện các ngày khả dụng
  - Hiện các slot khả dụng (slot trống và slot được chuyên gia set trước đó nhưng chưa có ai đặt)

## Screen: Tài liệu tư vấn

- Mô tả vấn đề
  - Text input
- Câu hỏi cụ thể
  - Text input

## Screen: Xác nhận thanh toán

- Thông tin buổi tư vấn
  - Avatar chuyên gia
  - Họ tên chuyên gia
  - Loại tư vấn
  - Thời gian tư vấn
- Chi tiết thanh toán
  - Phí tư vấn
  - Phí nền tảng
  - Tổng tiền
- Phương thức thanh toán
  - SankeAidPay
  - PayOS

## Screen: Sảnh phòng chờ

- Tắt mở mic/cam
- Join phòng

## Screen: Trong phòng tư vấn

- Tắt mở mic/cam
- Chat
- Lật camera
- Kết thúc

## Screen: Chat

- Tin nhắn (Text)
- Tin nhắn soạn sẵn
- Hình ảnh
- Nhắn

## Screen: Hoàn thành

- Section: Thông tin chuyên gia
  - Avatar
  - Họ tên
  - Ngày tư vấn
  - Thời gian tư vấn
  - Trạng thái tư vấn: Đã hoàn thành

- Section: Thanh toán đã xử lý
  - Số tiền của buổi tư vấn
  - Phương thức thanh toán
- Section: Đánh giá / nhận xét
  - Star
  - Text input
  - Nút gửi đánh giá

# Series: Chuyên gia

## Screen: Thiết đặt

- Set giá cho buổi tư vấn đặt lịch
- Set giá cho buổi tư vấn ngay
- Set thời gian khả dụng tư vấn (giờ làm việc)
  > Có thể xem hình ảnh minh họa:![working hours](<media/Working Hours.png>)

## Screen: Các tư vấn khẩn cấp

- Option
  - Chấp nhận
  - Từ chối

## Screen: Sảnh phòng chờ

- Tắt mở mic/cam
- Join phòng

## Screen: Trong phòng tư vấn

- Tắt mở mic/cam
- Chat
- Lật camera
- Kết thúc

## Screen: Chat

- Core Function
  - Tin nhắn (Text)
  - Tin nhắn soạn sẵn
  - Hình ảnh
  - Nhắn
- Pop-ups
  - Ghi chú
  - Xem hồ sơ
  - Tra cứu rắn

## Side Screen: Tra cứu rắn

- Search
- Filter: Rắn độc / Miền Nam / Thường gặp
- Section: Thông tin cá thể rắn
  - Box:
    - Avatar
    - Tên thông dụng
    - Tên khoa học
    - Mức độ độc
    - Khu vực sinh sống
  - Đặc điểm nhận dạng
  - Loại độc
  - Sơ cứu
  - Huyết thanh kháng nọc

## Screen: Hoàn tất tư vấn

- Section: Thông tin chuyên gia
  - Avatar
  - Họ tên
  - Ngày tư vấn
  - Thời gian tư vấn
  - Trạng thái tư vấn: Đã hoàn thành

- Section: Trạng thái thanh toán
  - Trạng thái thanh toán: Thanh toán hoàn tất
  - Số tiền của buổi tư vấn
    - Phí tư vấn
    - Phí nền tảng
    - Thực nhận
