# Báo cáo Triển khai: Live Tracking Ingestion (LT-1)

**Ngày**: 14/02/2026
**Giai đoạn**: Global Phase 2 (LT-1)
**Tính năng**: Tiếp nhận Vị trí Trực tiếp của Cứu hộ viên (Rescuer Live Location Ingestion)

## 1. Mục tiêu
Kích hoạt khả năng tiếp nhận và lưu trữ tin cậy dữ liệu vị trí của cứu hộ viên, đảm bảo hệ thống điều phối (`rescue-trigger`) luôn có dữ liệu vị trí mới nhất để ghép cặp cứu hộ viên chính xác.

## 2. Các thành phần đã triển khai

### 2.1. Tầng Service
- **`IRescuerLocationService`**: Định nghĩa contract cho việc cập nhật vị trí.
- **`RescuerLocationService`**:
  - **Logic cốt lõi**: Xác thực giới hạn vĩ độ/kinh độ (Latitude/Longitude).
  - **Điều tiết (Throttling)**: Áp dụng khoảng thời gian có thể cấu hình được (Configurable) cho mỗi cứu hộ viên bằng `IMemoryCache`.
    - **Cập nhật**: Đã chuyển từ hardcode 1s sang đọc từ cấu hình `LocationUpdate:ThrottleIntervalSeconds` (Mặc định: 10s).
  - **Lưu trữ (Persistence)**: Ghi trực tiếp vào `RescuerProfile.LastLocation` (PostGIS `geometry(Point, 4326)`) và cập nhật `LastLocationUpdate`.
  - **Xử lý lỗi**: Log lỗi nhưng không làm gián đoạn kết nối SignalR (Silent failure).

### 2.2. Tầng API (Hub)
- **`RescuerHub.UpdateLocation`**:
  - Giữ nguyên chữ ký hàm (signature) và hành vi phản hồi (echo) hiện tại để tương thích với client cũ.
  - Inject `IRescuerLocationService` để thực hiện lưu trữ dữ liệu bất đồng bộ.
  - Thêm logging để truy vết.
  - **Lưu ý về Mocking**: Hàm `UpdateLocation` trong `TestChatHub.cs` (hoặc `RescueDemoController` flow) đang được sử dụng cho mục đích demo/mocking.

### 2.3. Tầng Data & DI
- **Dependency Injection**:
  - Đã chuyển sang sử dụng **Scrutor** để tự động đăng ký `IRescuerLocationService` theo convention của dự án (trong `Program.cs`), thay vì đăng ký thủ công trong `DependencyInjection.cs`.
- **Database**:
  - `RescuerProfile`: Lưu vị trí mới nhất (`LastLocation`).
  - **Lịch sử vị trí**: Entity `LocationEvent` (trong `SnakeAid.Core.Domains`) là nơi lưu trữ lịch sử vị trí (SessionId, AccountId, Location, RecordedAt). Tuy nhiên, flow hiện tại (LT-1) chưa ghi vào bảng này.

## 3. Các quyết định kỹ thuật chính & Giải đáp

| Hạng mục | Chi tiết |
|---|---|
| **Session-Based Broadcasting** | Hiện tại việc broadcast vị trí là "Global" (ví dụ: Admin thấy tất cả). "Session-Based" nghĩa là chỉ broadcast vị trí của cứu hộ viên X cho những người đang trong cùng phiên cứu hộ (Session Y) với họ (ví dụ: nạn nhân, tổng đài viên phụ trách session đó). Việc này sử dụng `Clients.Group(sessionId)` thay vì `Clients.All`. |
| **Lịch sử vị trí** | Hiện tại ERD đã có entity `LocationEvent` để lưu lịch sử. Flow LT-1 hiện tại chỉ update `RescuerProfile` (vị trí cuối). LT-2 sẽ bổ sung việc ghi nhận vào `LocationEvent` để vẽ lại hành trình. |
| **Mocking Function** | Function `UpdateLocation` trong `TestChatHub` đang đóng vai trò mock/test. Cần phân biệt rõ với `RescuerHub.UpdateLocation` (production). |
| **Scrutor** | Đã chuyển sang dùng Scrutor. Class `RescuerLocationService` tự động được scan và đăng ký Scoped. |
| **Throttling & Pin** | Đã điều chỉnh logic throttling đọc từ config. Mặc định set là **10s**. <br> **Tại sao cần Throttling?** <br> 1. **Bảo vệ Cơ sở dữ liệu (Database Load)**: Nếu 1000 cứu hộ viên gửi vị trí mỗi giây (1000 WRITES/s), Database sẽ quá tải vì phải lock row liên tục để update `LastLocation`. Throttling giảm xuống còn 1 WRITE/10s cho mỗi người, giảm tải 10 lần. <br> 2. **Tiết kiệm pin thiết bị (Battery)**: Việc gửi request liên tục (radio/network usage) là nguyên nhân gây hao pin hàng đầu trên mobile. Giãn cách thời gian gửi giúp radio của điện thoại có thời gian nghỉ (idle). |
| **ST_DWithin** | Flow rescuer pairing sử dụng `Distance(location) <= radius` của Npgsql/Entity Framework. Khi dịch sang SQL, PostGIS sẽ tối ưu hóa điều kiện này tương đương với `ST_DWithin` (hoặc `ST_Distance` kết hợp index spatial) để đảm bảo hiệu năng. |

## 4. Quy trình hiện tại (Workflow)
1. **Rescuer App** gửi `UpdateLocation(lat, lng)` qua SignalR (sau mỗi 10s).
2. **Hub** nhận cuộc gọi, phản hồi lại `LocationUpdated`.
3. **Service** kiểm tra cấu hình `LocationUpdate:ThrottleIntervalSeconds` (10s):
   - Nếu < 10s kể từ lần cập nhật trước: Bỏ qua (để giảm tải DB).
   - Nếu >= 10s: Tiếp tục.
4. **Service** cập nhật `RescuerProfile` trong PostgreSQL.
5. **Hệ thống Điều phối** truy vấn `RescuerProfile` sử dụng `Distance() <= Radius` (PostGIS spatial query) để tìm cứu hộ viên.

## 5. Kết luận
Hệ thống đã được tinh chỉnh theo phản hồi: tốt cho pin hơn (10s throttling), code gọn gàng hơn (Scrutor), và làm rõ các khái niệm về session/history. Sẵn sàng cho Global Phase 3 (LT-2).
