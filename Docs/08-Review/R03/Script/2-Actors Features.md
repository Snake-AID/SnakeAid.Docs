# Script thuyết trình - Slide 2: Actors & Features

Tiếp theo, em xin trình bày ba nhóm người dùng chính ở phía trước của hệ thống, gồm Member, Rescuer và Expert.

Với Member, thứ tự các ý trên slide cũng phản ánh khá đúng hành trình sử dụng thực tế. Khi gặp sự cố, nhu cầu đầu tiên luôn là được hướng dẫn sơ cứu ngay lập tức để tránh xử lý sai trong những phút đầu. Sau đó, người dùng cần biết nên đi đâu để điều trị, nên hệ thống hỗ trợ tìm cơ sở gần nhất có khả năng xử lý và có huyết thanh phù hợp. Trong lúc đó, bản đồ realtime cùng phần theo dõi vết cắn và triệu chứng giúp hệ thống giữ được trạng thái member một cách liên tục, thay vì chỉ tiếp nhận thông tin rời rạc.

Sau lớp nhu cầu khẩn cấp là lớp nhu cầu dịch vụ. Member có thể quản lý chi phí liên quan đến từng loại dịch vụ, sử dụng AI để nhận diện loài rắn từ ảnh, gửi báo cáo khi xảy ra sự cố rắn cắn hoặc khi phát hiện rắn trong khu dân cư, đồng thời tiếp cận nội dung phòng tránh và blog kiến thức để chủ động hơn về mặt an toàn. Cuối cùng, khi cần đánh giá chuyên sâu hơn, Member có thể đặt lịch tư vấn với Expert. Điều này cho thấy Member không chỉ là người dùng cuối, mà còn là điểm khởi đầu của gần như toàn bộ flow nghiệp vụ trên nền tảng.

Tiếp sang Rescuer, nhóm này đại diện cho lực lượng phản ứng hiện trường. Bắt đầu từ việc nhận cảnh báo hoặc yêu cầu cứu hộ, Rescuer sẽ đi vào phần quản lý nhiệm vụ được giao. Các tính năng như hướng dẫn an toàn khi bắt và di dời rắn, ghi nhận hoạt động rescue, báo cáo kết quả và theo dõi bản đồ không chỉ phục vụ thao tác tại hiện trường, mà còn giúp chuẩn hóa quy trình xử lý. Nhờ đó, hệ thống có thể theo dõi tiến độ, đánh giá hiệu quả thực hiện và lưu lại lịch sử vận hành để phục vụ điều phối về sau.

Với Expert, đây là lớp chuyên môn hỗ trợ từ xa. Expert có thể tham gia tư vấn khẩn cấp 1-1, quản lý lịch làm việc cá nhân, thực hiện tư vấn từ xa và xác minh thêm loài rắn từ hình ảnh khi cần bổ sung độ tin cậy cho đánh giá ban đầu. Các tính năng về báo cáo doanh thu và thiết lập phí tư vấn cũng cho thấy Expert không chỉ đóng vai trò chuyên môn, mà còn là một actor có mô hình dịch vụ riêng trong hệ sinh thái SnakeAid.

Nhìn tổng thể, ba nhóm này tạo thành lớp nghiệp vụ phía trước của nền tảng: Member tạo nhu cầu, Rescuer xử lý hiện trường, còn Expert bổ sung năng lực chuyên môn từ xa. Chính sự phối hợp đó mới giúp SnakeAid đi xa hơn một ứng dụng tra cứu thông tin, để trở thành một nền tảng hỗ trợ xử lý sự cố tương đối trọn vẹn.
