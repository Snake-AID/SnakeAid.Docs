# Script thuyết trình - Slide 1: System Architecture

Phần đầu tiên, em xin giới thiệu bức tranh tổng thể của nền tảng SnakeAid dưới góc nhìn kiến trúc hệ thống.

Ở lớp ngoài cùng, hệ thống có hai kênh client chính. Thứ nhất là ứng dụng mobile được phát triển bằng Flutter, phục vụ cho các vai trò sử dụng nền tảng một cách linh hoạt như Member, Rescuer và cả Expert khi cần hỗ trợ từ xa. Thứ hai là frontend web sử dụng Next.js, phù hợp hơn cho các tác vụ vận hành và quản trị, ví dụ như Operator và Admin.

Toàn bộ lưu lượng từ client sẽ đi qua Cloudflare Tunnel trước khi vào backend. Cách tổ chức này giúp tăng tính an toàn khi public dịch vụ, đồng thời hỗ trợ routing và bảo vệ hạ tầng self-hosted ở phía sau.

Nằm ở trung tâm là backend .NET, kết hợp với Entity Framework để xử lý nghiệp vụ và làm việc với dữ liệu. Đây là nơi điều phối các flow chính của SnakeAid như tiếp nhận ca khẩn cấp, điều phối rescue, booking consultation, quản lý thanh toán, cũng như đồng bộ trạng thái theo thời gian thực.

Song song với backend là khối AI Inference chạy bằng FastAPI và ONNX. Khối này phụ trách các tác vụ AI như nhận diện loài rắn từ hình ảnh và hỗ trợ đánh giá mức độ nguy hiểm ban đầu. Điểm em muốn nhấn mạnh là AI trong hệ thống đóng vai trò hỗ trợ quyết định, còn kết luận nghiệp vụ cuối cùng vẫn được kiểm soát bởi con người và các rule an toàn.

Về dữ liệu và tích hợp ngoài, hệ thống sử dụng Supabase PostgreSQL làm database, Cloudinary làm CDN lưu trữ media, PayOS để xử lý thanh toán, LiveKit cho video call, Resend cho email và LocationIQ cho geocoding. Ngoài ra, Firebase Cloud Messaging được dùng để đẩy notification theo thời gian thực đến thiết bị người dùng.

Để xử lý bất đồng bộ, hệ thống có RabbitMQ phục vụ cho queue và event-driven processing. Cách tiếp cận này phù hợp với các tác vụ như gửi thông báo mà không làm chậm luồng chính.

Về mặt triển khai, toàn bộ các service được chạy trong Docker Network trên self-hosted Linux Server. Jenkins đảm nhiệm CI/CD, Portainer hỗ trợ quản lý container, còn lớp observability và monitoring giúp đội vận hành theo dõi sức khỏe hệ thống.

Riêng với pipeline AI, mô hình được fine-tune qua Roboflow và Google Colab, sau đó đẩy lên Hugging Face Model Registry. Mã nguồn được quản lý trên GitHub và image được lưu ở Docker Hub để phục vụ quá trình build và deploy.

Nhìn tổng thể, kiến trúc này cho thấy SnakeAid được định hướng như một nền tảng đa kênh, có backend nghiệp vụ làm trung tâm, có AI hỗ trợ, có tích hợp realtime và thanh toán, đồng thời đủ điều kiện để vận hành ổn định trong môi trường production.
