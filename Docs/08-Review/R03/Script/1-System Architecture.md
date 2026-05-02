# Script thuyết trình - Slide 1: System Architecture

Phần đầu tiên, em xin giới thiệu tổng thể kiến trúc của SnakeAid.

Ở lớp ngoài cùng, hệ thống có hai kênh client chính.

Mobile Flutter phục vụ các vai trò sử dụng linh hoạt như Member, Rescuer và Expert khi cần hỗ trợ từ xa.

Web Next.js phù hợp hơn cho các tác vụ vận hành và quản trị như Operator và Admin.

Toàn bộ lưu lượng từ client sẽ đi qua Cloudflare Tunnel trước khi vào backend, vừa hỗ trợ routing vừa bảo vệ hạ tầng self-hosted phía sau.

Ở trung tâm là backend .NET kết hợp Entity Framework để xử lý các flow chính như emergency, rescue, consultation và thanh toán.

Song song với đó là khối AI Inference dùng FastAPI và ONNX để nhận diện loài rắn và hỗ trợ đánh giá ban đầu.

Hệ thống cũng tích hợp Supabase PostgreSQL, Cloudinary, PayOS, LiveKit, Resend, LocationIQ và Firebase Cloud Messaging.

Về triển khai, các service chạy trên Docker trong môi trường self-hosted Linux Server, có Jenkins cho CI/CD, Portainer cho quản lý container và lớp monitoring để hỗ trợ vận hành ổn định.
