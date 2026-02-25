# Hướng dẫn xử lý lỗi MaxClientsInSessionMode trên Supabase

Tài liệu này giải thích chi tiết nguyên nhân phát sinh lỗi giới hạn Connection Pool trên Supabase khi ứng dụng chịu tải cao, cách thức hệ thống Connection Pool hoạt động, và giải pháp cấu hình Backend (C# Npgsql) để xử lý triệt để lỗi này.

---

## 1. Hiện tượng (Symptom)

Khi Backend API có lưu lượng truy cập lớn (như đăng nhập đồng loạt tại endpoint `/api/auth/login`), hệ thống có thể bị quá tải và trả về exception sau đây, làm gián đoạn request:

```json
{
  "ExceptionType": "PostgresException",
  "ExceptionMessage": "XX000: MaxClientsInSessionMode: max clients reached - in Session mode max clients are limited to pool_size"
}
```

## 2. Giải thích cơ chế gây ra lỗi (Root Cause)

### 2.1. Hai chế độ Connection Pooling của Supabase (Supavisor)

Supabase sử dụng **Supavisor** làm connection pooler chịu trách nhiệm điều phối lượng lớn client tới một số lượng hữu hạn các kết nối vật lý của database (PostgreSQL). Supavisor hỗ trợ 2 chế độ chính:

1. **Session Mode (Port 5432 - Cổng mặc định):**
   - **Cách hoạt động:** Khi ứng dụng (Npgsql) mở một kết nối mới tới Supabase, nó sẽ "chiếm" luôn 1 slot (kết nối vật lý) trong DB. Kết nối này bị giữ cho đến khi ứng dụng ngắt kết nối đi.
   - **Giới hạn:** Số lượng kết nối bị giới hạn nghiêm ngặt theo thông số `pool_size` (Ví dụ gói Free chỉ giới hạn vài chục kết nối). Thích hợp với các service background hoặc các GUI tool chạy dài hạn.
2. **Transaction Mode (Port 6543):**
   - **Cách hoạt động:** Kết nối DB chỉ được giao cho backend ở cấp độ **Từng Transaction/Query**. Ngay khi query thực thi xong, kết nối lập tức được trả về pool để người khác dùng, bất kể connection logic từ C# Backend vẫn đang mở.
   - **Ưu điểm:** Cho phép hàng nghìn connection từ các App/Web server dùng chung một số lượng kết nối vật lý rất nhỏ mà vẫn mượt mà. Phù hợp nhất cho REST API.

Dù sử dụng gói Supabase Free, hạ tầng **Shared Pooler (Supavisor)** vẫn hỗ trợ đầy đủ cả 2 chế độ trên, cho phép scale lên khoảng 200 pooler connections nếu dùng qua IPv4.

### 2.2. Sự xung đột với Npgsql và EF Core

Mặc định, `Npgsql` (driver kết nối Postgres trong C#) tự động duy trì một connection pool trên RAM nội bộ (Max connection = 100). Khi có lượng request lớn (ví dụ login), Entity Framework Core (EF Core) gặp áp lực và sẽ yêu cầu Npgsql mở thêm TCP connection trực tiếp ra ngoài tới Supabase.

**Vấn đề:** Nếu ứng dụng đang cấu hình dùng port `5432` (Session Mode), việc Npgsql scale-up mở 100 connections đồng nghĩa với 100 slot trong Supavisor Session Pool bị đòi hỏi. Khi con số này vượt qua cấu hình `pool_size` cho phép, Supavisor từ chối kết nối và ném ra lỗi `MaxClientsInSessionMode`.

---

## 3. Giải pháp (Resolution)

Để giải quyết vấn đề chịu tải cao, kiến trúc Backend cần được chuyển sang **Transaction Pooling (Port 6543)**. Tuy nhiên, việc chuyển đổi đòi hỏi phải xử lý triệt để hành vi cấp phát ngầm của thư viện `Npgsql`.

### 3.1. Hạn chế "chết người" của Npgsql Auto-Prepare

Trong Transaction mode, một kết nối liên tục bị tráo đổi giữa nhiều truy vấn HTTP khác nhau.

Theo mặc định, thư viện `Npgsql` (nằm dưới EF Core) có tính năng **Auto-Prepare**. Dù lập trình viên không chủ động viết lệnh gởi Prepared Statements, Npgsql vẫn sẽ ngầm tạo ra các lệnh `PREPARE` đẩy lên server để tối ưu hóa.
Nếu C# prepapre statement ở Connection X, sau đó connection này bị Supavisor cấp cho request của Client khác thì truy vấn mới sẽ gặp lỗi _“prepared statement does not exist”_ hoặc sai lệch cache.

**Yêu cầu bắt buộc:** Phải tắt hoàn toàn tính năng Auto-Prepare của Npgsql khi dùng Transaction Pooling.

### 3.2. Cấu hình Backend đúng chuẩn

SnakeAid Backend đã tự động hóa việc cấu hình này tại `Program.cs`. App có khả năng tự phát hiện domain của pooler và tự ép tắt Auto-prepare:

```csharp
var connectionString = builder.Configuration.GetConnectionString("SupabaseConnection");

// Xóa triệt để các rủi ro Auto-Prepare đồng thời chuyển đổi Port
if (!string.IsNullOrEmpty(connectionString) && connectionString.Contains("pooler.supabase.com"))
{
    var npgsqlBuilder = new Npgsql.NpgsqlConnectionStringBuilder(connectionString);

    // 1. Chuyển sang Transaction pooling mode (Port 6543)
    if (npgsqlBuilder.Port == 5432)
    {
        npgsqlBuilder.Port = 6543;
    }

    // 2. Tắt Prepared Statements để sống hòa thuận với Transaction Pooling
    npgsqlBuilder.MaxAutoPrepare = 0;

    connectionString = npgsqlBuilder.ConnectionString;
}

// Map chuỗi đã được tự cấu hình vào EF Core
builder.Services.AddDbContext<SnakeAidDbContext>(...);
```

#### Lưu ý cho quá trình Deployment (Môi trường DevOps)

Nếu bạn triển khai biến môi trường (Environment Variables) gán trực tiếp chuỗi Connection String lên server ở **những phiên bản code chưa có cơ chế tự định tuyến trong `Program.cs`**, bạn **BẮT BUỘC** phải chèn thủ công cờ cấu hình này vào chuỗi:

> `...;Server=aws-abc.pooler.supabase.com;Port=6543;Database=postgres;MaxAutoPrepare=0;`

Nếu chỉ đổi `Port=6543` mà thiếu `MaxAutoPrepare=0;`, API sẽ dính lỗi sụp đổ toàn mạng.

---

## 4. Đặc tả giới hạn của thiết kế (Trade-offs)

Khi kiến trúc đã nằm gọn trên Transaction Pooling, môi trường sẽ là _phi trạng thái (Stateless)_ hoàn toàn đối với database. Các tính năng sau đây của PostgreSQL sẽ **không hoạt động** giữa 2 câu query độc lập gửi liên tiếp từ C#:

1. **Session Variables:** Bạn không thể gán biến nhớ thông qua lệnh `SET` (ví dụ `SET statement_timeout = 1000`). Lệnh SET sẽ có tác dụng trong connection vật lý hiện tại nhưng có thể trôi theo connection đi qua request HTTP khác.
2. **Bảng Tạm (Temporary Tables):** Dữ liệu bảng tạm bị mất/đè chéo do tái sử dụng connection.
3. **Pusub Listeners:** Tính năng `LISTEN / NOTIFY` của Postgres không sử dụng được. (Dự án SnakeAid đã dùng SignalR thay thế).
4. **Session-level Advisory Locks:** Khóa đồng bộ mức session sẽ không theo được request HTTP.

_(Nếu cần dùng các session-specific feature này ở một BackgroundJob đặc thù, hãy cân nhắc cấu hình Npgsql kết nối Database bằng Port 5432 riêng cho module đó.)_
