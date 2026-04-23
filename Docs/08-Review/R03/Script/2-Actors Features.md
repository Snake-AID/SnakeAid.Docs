# Script thuyết trình - Slide 2: Actors & Features

Tiếp theo, em đi vào các nhóm người dùng chính của hệ thống và những tính năng cốt lõi mà mỗi nhóm sẽ sử dụng.

Đầu tiên là Member, tức người dùng cuối của nền tảng. Đây là nhóm có phạm vi chức năng rộng nhất vì họ là người khởi tạo phần lớn các nhu cầu thực tế. Với Member, hệ thống hỗ trợ sơ cứu khẩn cấp khi bị rắn cắn, xác định loài rắn từ ảnh bằng AI, theo dõi triệu chứng, tìm cơ sở điều trị gần nhất có thể xử lý ca bệnh, đồng thời cho phép kích hoạt SOS và theo dõi rescue trên bản đồ theo thời gian thực.

Bên cạnh tình huống khẩn cấp, Member còn có thể gửi yêu cầu bắt rắn, quản lý chi phí dịch vụ, đặt lịch hoặc gọi tư vấn với chuyên gia, cũng như tiếp cận kho nội dung về phòng tránh rắn và kiến thức an toàn. Nhìn từ góc độ sản phẩm, Member là trung tâm của cả ba flow lớn: emergency response, rescue service và expert consultation.

Nhóm thứ hai là Rescuer. Đây là lực lượng phản ứng tại hiện trường. Rescuer sẽ nhận cảnh báo hoặc yêu cầu điều phối, quản lý nhiệm vụ được giao, xem hướng dẫn an toàn khi tiếp cận và di dời rắn, cập nhật kết quả xử lý, và chia sẻ vị trí để hệ thống cũng như Member có thể theo dõi tiến độ thực hiện.

Điểm đáng chú ý là Rescuer không chỉ là người trực tiếp thực thi tác vụ, mà còn là một mắt xích quan trọng trong việc chuẩn hóa quy trình hiện trường. Việc ghi nhận hoạt động rescue giúp hệ thống tạo ra lịch sử vận hành, đo hiệu suất và nâng cao chất lượng dịch vụ về sau.

Nhóm thứ ba là Expert. Đây là các chuyên gia tham gia hỗ trợ từ xa. Expert có thể cung cấp tư vấn 1-1 trong tình huống khẩn cấp hoặc theo lịch hẹn, quản lý lịch làm việc cá nhân, thực hiện remote consultation, xác minh thêm về loài rắn từ hình ảnh và theo dõi doanh thu theo tháng hoặc theo quý. Ngoài ra, Expert cũng có thể thiết lập mức phí tư vấn trực tuyến của mình.

Nhìn tổng thể, ba actor này tạo thành lớp vận hành nghiệp vụ phía trước của SnakeAid. Member là người tạo nhu cầu, Rescuer là người xử lý ngoài hiện trường, còn Expert bổ sung năng lực chuyên môn từ xa. Chính sự kết hợp này giúp nền tảng không chỉ dừng ở mức cung cấp thông tin, mà thực sự hỗ trợ xử lý một ca sự cố từ đầu đến cuối.
