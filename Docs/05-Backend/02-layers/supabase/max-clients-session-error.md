# Supabase MaxClientsInSessionMode Error

## Hiện tượng (Symptom)

Khi Backend API chịu tải lớn (nhiều request đồng thời) hoặc có các thao tác gọi db dồn dập (như trong endpoint `/api/auth/login`), hệ thống sẽ ném ra exception sau và làm sập request:

```json
{
  "ExceptionType": "PostgresException",
  "ExceptionMessage": "XX000: MaxClientsInSessionMode: max clients reached - in Session mode max clients are limited to pool_size"
}
```

## Giải thích cơ chế gây ra lỗi (Root Cause)

### 1. Cơ chế Connection Pooling của Supabase (Supavisor)

Supabase sử dụng Supavisor làm connection pooler đứng giữa ứng dụng (Backend) và database (PostgreSQL) thực tế. Supavisor cung cấp 2 chế độ pooling (hoạt động trên các port khác nhau):

- **Session Pooling (Port 5432 - Mặc định):** Mỗi khi ứng dụng mở một kết nối tới Supabase, pooler sẽ duy trì kết nối này xuyên suốt cho đến khi ứng dụng ngắt kết nối. Tại chế độ này, số lượng kết nối tối đa bị giới hạn rất nghiêm ngặt bởi cấu hình `pool_size` của Supabase (thường khá thấp so với số request web server có thể xử lý).
- **Transaction Pooling (Port 6543):** Supavisor chia sẻ một lượng nhỏ các kết nối vật lý tới database cho nhiều kết nối từ ứng dụng. Một kết nối vật lý chỉ bị giữ trong thời gian diễn ra một transaction (hoặc statement). Cho phép ứng dụng mở hàng nghìn kết nối logic mà không làm kiệt quệ tài nguyên của database.

### 2. Hành vi của Entity Framework Core (EF Core) và Npgsql

Mặc định, `Npgsql` (driver kết nối Postgres trong C#) tự động quản lý một connection pool nội bộ trên RAM của server Backend (Max Pool Size mặc định là 100). Khi có nhiều request API tới cùng lúc (ví dụ login), EF Core sẽ lấy các kết nối từ Npgsql pool. Nếu pool nội bộ chưa đủ, Npgsql sẽ khởi tạo thêm các TCP connection mới ra ngoài tới Supabase.

### 3. Sự xung đột dẫn đến lỗi

Khi ứng dụng được cấu hình kết nối tới Supabase qua **Port 5432 (Session Mode)**, mỗi connection mà Npgsql mở ra sẽ chiếm dụng 1 slot trong Session Pool của Supabase.
Do lượng request tăng cao, Npgsql tiến hành scale số connection lên để phục vụ, làm tổng số connection vượt quá giới hạn `pool_size` của Supavisor. Khi vượt ngưỡng này, Supavisor ngay lập tức từ chối các kết nối mới với lỗi `MaxClientsInSessionMode`.

---

## Cách giải quyết (Resolution)

Để giải quyết vấn đề này, chúng ta cần chuyển sang dùng **Transaction Pooling (Port 6543)**.
Tuy nhiên, khi sử dụng Transaction Pooling, do 1 kết nối logic từ C# có thể được phân phối luân phiên cho nhiều session vật lý khác nhau bên dưới database, tính năng **Prepared Statements** của Npgsql (lưu cache câu lệnh SQL ở mức session) sẽ bị lỗi (gây lỗi _prepared statement does not exist_).

Do đó, cần áp dụng 2 thay đổi khi khởi tạo kết nối Database:

1. Đổi Port sang `6543` (Transaction Pooling).
2. Tắt Prepared Statements thông qua `MaxAutoPrepare = 0` (hoặc `No Reset On Close=true`).

**Cách áp dụng trong code (đã có ở cấu hình `Program.cs`):**

```csharp
var connectionString = builder.Configuration.GetConnectionString("SupabaseConnection");

// Xử lý để tự động chuyển connection string dùng port 5432 sang chế độ thích hợp của Supabase
if (!string.IsNullOrEmpty(connectionString) && connectionString.Contains("pooler.supabase.com"))
{
    var npgsqlBuilder = new Npgsql.NpgsqlConnectionStringBuilder(connectionString);

    // 1. Chuyển sang Transaction pooling mode (Port 6543)
    if (npgsqlBuilder.Port == 5432)
    {
        npgsqlBuilder.Port = 6543;
    }

    // 2. Tắt prepared statements để hoạt động tốt với transaction pooling
    npgsqlBuilder.MaxAutoPrepare = 0;

    connectionString = npgsqlBuilder.ConnectionString;
}

// ... Đưa connectionString này vào EF Core
builder.Services.AddDbContext<SnakeAidDbContext>(options => { ... });
```

Hành động này giúp Backend có thể phục vụ số lượng request cực lớn mà không làm kiệt quệ Postgres connection pool trên Supabase.

---

## Phụ lục 1: Những lưu ý quan trọng khi chuyển sang Transaction Pooling

Khi cấu hình Backend sử dụng Transaction Pooling (Port `6543`), bạn **bắt buộc** phải lưu ý những hạn chế về mặt kiến trúc sau đây:

### 1. Tắt Prepared Statements ở phía Client (Cực kỳ quan trọng)

Trong Transaction mode, vì connection vật lý bị trả về pool ngay sau mỗi transaction, nếu client A tạo ra một Prepared Statement trên connection X, sau đó connection X được giao cho client B, client B sẽ không chạy được câu lệnh đó (hoặc bị rác dữ liệu/lỗi cache).

**Lưu ý "chết người" với `Npgsql` (C#):**
Mặc dù bạn KHÔNG tự viết code để chạy Prepared Statements, thư viện **`Npgsql` lại BẬT mặc định tính năng Auto-Prepare**. Nó sẽ âm thầm tự tạo prepared statements cho mọi truy vấn để tối ưu hóa.

**Giải pháp & Cách cấu hình:**
Bạn bắt buộc phải tắt tính năng auto-prepare này đi bằng cấu hình `MaxAutoPrepare=0` (hoặc `No Reset On Close=true`) trong Connection String.

- **Trường hợp 1 (Code đã update tự cấu hình - `Program.cs`):** File code đã có logic kiểm tra nếu kết nối tới `pooler.supabase.com` thì tự chèn `MaxAutoPrepare=0` lúc ứng dụng khởi chạy.
- **Trường hợp 2 (Cầu hình trên Environment Server khi chưa deploy code mới):** Bạn **BẮT BUỘC** phải nối thủ công chuỗi cấu hình biến môi trường trên server, nếu không api sẽ sập hàng loạt khi đổi port 6543:
  - _Ví dụ chuỗi đúng trên Deployment:_ `...;Server=aws-abc.pooler.supabase.com;Port=6543;Database=postgres;MaxAutoPrepare=0;`

### 2. Không thể lưu trạng thái cấp Session (Session State)

Mọi thiết lập ở mức độ Session sẽ bị mất ngay khi query gởi đi hoàn tất. Các tính năng sau của Postgres **KHÔNG THỂ SỬ DỤNG** giữa nhiều query rời rạc khi dùng Transaction pooling:

- Biến tạm thời gán bằng lệnh `SET` (ví dụ `SET TIME ZONE`, `SET statement_timeout`).
- Bảng tạm (Temporary Tables).
- Tính năng lắng nghe Notification qua `LISTEN / NOTIFY`.
- Khóa tư vấn ở mức session (Session-level Advisory Locks).
  Nếu bạn có dùng các tính năng này ở một luồng cụ thể nào đó (ví dụ background job), bạn cân nhắc dùng cổng `5432` Session Mode riêng cho luồng đó, còn API web thì dùng `6543`.

### 3. Connection Limits vẫn tồn tại

Mặc dù Transaction mode giúp dùng chung connection cực kì hiệu quả, nhưng Pooler của Supabase bản Free vẫn có trần giới hạn (ví dụ tối đa khoảng 200 pooler connections). Nếu số lượng tính toán concurrent quá lớn hoặc Database query quá chậm chặn mất slot trong pool quá lâu, pooler vẫn sẽ bị tràn hoặc timeout chờ cấp connection.

---

## Phụ lục 2: Điều tra thực tế (Wiki-Researcher) về Supabase Connection Pool

Dưới đây là kết quả điều tra và đối chiếu các thông tin theo chuẩn xác minh thực tế (Fact-based Evidence) từ tài liệu chính thức của Supabase:

### 1. Supabase có 3 chế độ connect?

**Đánh giá:** Chỉ đúng đắn một phần lịch sử (MEDIUM Confidence - dựa trên sự tiến hóa của kiến trúc).

- **Thực tế hiện tại:** Supabase (thông qua pooler thế hệ mới **Supavisor**) tập trung và hỗ trợ chính thức **2 chế độ pooling**:
  1.  **Transaction Mode (Port 6543):** Giải phóng connection ngay sau mỗi transaction/query. (Mặc định cho web serverlerless/API).
  2.  **Session Mode (Port 5432):** Giữ connection cho tới khi ngắt kết nối (Mặc định cho các kết nối dài hạn, GUI tools).
- **Chế độ thứ 3 (Statement Mode):** Đây thực chất là thuật ngữ của **PgBouncer** (công cụ pooler cũ trước đây của Supabase). Statement mode chặt chẽ hơn Transaction (trả connection sau mỗi câu lệnh, không được dùng transaction nhiều câu lệnh). Tuy nhiên, trên UI và documentation hiện tại của Supabase, họ chủ yếu chỉ định tuyến giữa Transaction và Session bằng Supavisor.
- Bên cạnh các chế độ pooler, luôn tồn tại kết nối **Direct Connection** (bỏ qua mọi pooler, đi thẳng vào Postgres instance).

### 2. Supabase Free chỉ cho phép Session pool?

**Đánh giá:** Sai (HIGH Confidence - verified by documentation).

- **Sự thật:** Gói Free của Supabase được thừa hưởng hạ tầng **Shared Pooler** (dùng Supavisor). Shared Pooler này **hỗ trợ ĐẦY ĐỦ** cả Transaction Mode lẫn Session Mode.
- **Giới hạn thực tế của gói Free (Nano Compute):**
  - **Direct connections:** Khoảng 60 kết nối.
  - **Pooler connections:** Cho phép tới **200 kết nối** (Sử dụng cổng 6543 qua IPv4).
- **Kết luận:** Bạn hoàn toàn CÓ THỂ dùng cấu hình Transaction Pooling ở port `6543` trên gói Supabase Free mà không gặp rào cản tính năng nào. Đây chính là cách thiết kế "chuẩn sách giáo khoa" để scale trên các gói Free/Serverless.
