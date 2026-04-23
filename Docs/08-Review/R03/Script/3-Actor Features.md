# Script thuyết trình - Slide 3: Operator & Admin Features

Sau khi đi qua các actor trực tiếp tham gia vào flow nghiệp vụ, phần cuối em xin chuyển sang hai vai trò hậu trường nhưng có ý nghĩa quyết định đối với vận hành hệ thống, đó là Operator và Admin.

Đầu tiên là Operator. Có thể xem đây là bộ phận điều phối trung tâm của nền tảng. Operator theo dõi các yêu cầu rescue hoặc bắt rắn, kiểm tra ca trực và giám sát quá trình xử lý thực tế. Trong các flow quan trọng của SnakeAid, đặc biệt là emergency snakebite và snake catching service, tất cả request đều đi vào hàng chờ của Operator trước khi được xác minh, phân loại ưu tiên và gán cho Rescuer phù hợp.

Vai trò của Operator giúp hệ thống tránh việc dispatch tự động một cách thiếu kiểm soát. Thay vào đó, hệ thống có một lớp kiểm duyệt nghiệp vụ để bảo đảm ca việc được xử lý đúng người, đúng mức độ ưu tiên và đúng trạng thái vận hành tại thời điểm đó.

Tiếp theo là Admin. Nếu Operator thiên về điều phối hằng ngày, thì Admin chịu trách nhiệm quản trị toàn bộ nền tảng. Admin quản lý người dùng và phân quyền, quản lý cơ sở dữ liệu loài rắn, cấu hình ca trực, quản lý request, quản lý cơ sở điều trị, quản lý nội dung, theo dõi thống kê báo cáo, cũng như quản lý giao dịch và các tác vụ vận hành khác.

Nói cách khác, Admin là vai trò bảo đảm nền tảng có dữ liệu đúng, cấu hình đúng và chính sách đúng. Đây cũng là nhóm chịu trách nhiệm duy trì chất lượng dữ liệu đầu vào cho AI, kiểm soát nội dung chuyên môn, theo dõi tài chính và hỗ trợ các yêu cầu về audit hoặc compliance nếu cần.

Khi đặt phần này cạnh nhóm actor ở slide trước, có thể thấy SnakeAid được thiết kế theo một mô hình khá rõ. Phía trước là các actor tạo và xử lý nhu cầu thực tế, còn phía sau là các actor vận hành và quản trị để giữ cho toàn bộ hệ thống hoạt động ổn định, có kiểm soát và có khả năng mở rộng.

Đó cũng là điểm em muốn nhấn mạnh ở phần này: SnakeAid không chỉ là một ứng dụng cho người dùng cuối, mà là một nền tảng hoàn chỉnh, có đầy đủ lớp tác nghiệp, điều phối và quản trị.
