---
doc_role: operation
operation_id: FEAT-live-tracking-dashboard
type: FEAT
status: done
created_at: 2026-02-23
affects:
  - SnakeAid.Api/Pages/Demo
  - SnakeAid.Api/Hubs/RescuerHub.cs
---

# Plan: Live Tracking Monitoring Dashboard

## 1. As-Is State

Currently, the `live-tracking` (LT-1) and `rescue-trigger` (RT-1) flows rely on SignalR WebSocket connections (`/rescuer-hub`) and background timeout services to manage the dispatch lifecycle of a snakebite emergency.
When developers or QA engineers test the mobile Flutter application (via AVD or physically), they are blind to the backend's real-time state. To verify if a location was ingested, throttled, if a request expired, or if a race condition was handled correctly (User A accepts before User B), they must manually query the database (PostGIS) or sift through thousands of lines of Visual Studio console logs. There is no centralized, real-time "Admin/God View" specialized for observing the dispatch and tracking lifecycle.

## 2. Gap Analysis

**Missing Capabilities:**

- No visual interface to see currently connected Rescuers in real-time.
- No real-time log of SignalR events (`LocationUpdated`, `NewRescueRequest`, `RequestExpired`, `RequestTaken`) emitted by the Hub to the clients.
- Difficulty in reproducing and observing race conditions and session timeouts without digging into DB state after the fact.

## 3. To-Be Design

Build a read-only **Live Tracking Monitoring Dashboard** using ASP.NET Core Razor Pages, designed specifically to address the pain points of testing Flutter clients.

**Location**: `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml`

**Layout & UX Structure:**

Trang Monitor sẽ được bố cục lại theo hướng Component-based, loại bỏ Terminal cũ, chia màn hình thành 2 khu vực chính (Layout 75% - 25%):

**Khu vực 1 (Main - 75%): Bảng Điều Khiển Grid đa Tab (Tabs Layout)**
Được chia thành 3 Tab chính để quản lý trạng thái kết nối và user:

- **Tab 1 - Connected Rescuers:** Hiển thị mạng lưới các thẻ Rescuer đang kết nối thành công.
- **Tab 2 - Ghost Scanner:** Hiển thị mạng lưới các tài khoản bị "Ghost" (IsOnline = true trong DB nhưng mất kết nối SignalR). Cho phép Admin ấn nút _Rescan_ để quét chủ động các ca Lệch Pha (State Drift).
- **Tab 3 - All Rescuers:** Liệt kê toàn bộ Rescuers có trong Database bất kể trạng thái nào. Dùng để đối chiếu và theo dõi tổng dung lượng nhân sự. Các user offline sẽ được làm mờ (dimmed) đi.

Mỗi **Rescuer Card (Tab 1)** bao gồm:

1. **Header**: Avatar, FullName, PhoneNumber, và Badge hiển thị loại Rescuer. Đi kèm là **Status Badge** chỉ báo trạng thái luân chuyển (IDLE / PINGED / BUSY).
2. **Ping Timer**: Một vòng tròn đếm ngược 10 giây. Nó sẽ lặp lại mỗi khi nhận được sự kiện `LocationUpdated`. Nếu quá 10s không có ping (do logic tối ưu pin <10 mét của Flutter), vòng tròn sẽ chuyển sang màu xỉn (stale) để báo hiệu người dùng đang đứng yên thay vì mất mạng.
3. **Metrics Area**: Khu vực dải thẻ (Pills) nằm dưới cùng, gộp chung các thông số để tiết kiệm diện tích và dễ nhìn:
   - 📡 **Websocket**: `SignalR` (Màu xanh nếu connected)
   - 🗄️ **Database**: `IsOnline` (Màu xanh nếu true, đỏ nếu false)
   - 📍 **Tọa độ**: `Lat` và `Lng` (Sẽ chớp sáng khi có data mới).
4. **Session State (UC1, UC3, UC5)**: Màu sắc thẻ và Status Badge thay đổi realtime theo vòng đời Incident:
   - _Mặc định_: Status `IDLE`, thẻ màu xám/trắng.
   - _Khi có SOS_: Status `PINGED`, viền nháy Vàng.
   - _Khi Win ca cấp cứu_: Status `BUSY`, thẻ đổi màu Xanh Lá đậm.
   - _Khi bị nẫng tay trên/Hết hạn_: Thẻ xám lại, Status văng về `IDLE`.

**Khu vực 2 (Sidebar - 25%): Active Incident Tracker (UC3 & UC4) & Log Console**
Theo dõi tiến độ chung của hệ thống khi có 1 ca cấp cứu (Incident) nổ ra.

- **Incident Info**: Incident ID, Session Number hiện tại, Radius hiện tại (10km, 20km).
- **Timeout Countdown**: Một thanh Progress Bar khổng lồ đếm ngược 60 giây. Cứ hết 60s mà không ai nhận thì reset lại vòng mới, Radius tự tăng cường lên.
- **Race Condition Log**: Ghi chú vắn tắt xem ai Vừa Win, ai bị Expired. Hệ thống tự động đẩy dữ liệu sang làm đổi màu các thẻ Rescuer bên nhóm Khu Vực 1.

**Technical Approach:**

- **Backend (`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs`)**:
  - Page model that auto-generates a temporary Admin JWT to inject into the view.
  - Cung cấp các RESTful API endpoints (Razor Page Handlers) như `?handler=Ghosts` và `?handler=AllRescuers` để gọi riêng rẽ các tác vụ I/O nặng (quét DB) thay vì nhồi nhét vào SignalR Hub.
- **Frontend (`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml`)**:
  - Dùng HTML/CSS/JS thuần với giao diện 3 Tabs.
  - Tích hợp `@microsoft/signalr` JS client để kết nối nhóm "Monitors".
  - Sử dụng `fetch()` AJAX để xử lý các luồng dữ liệu nặng và scan lệch pha (State Drift).
- **Authentication**: Backend sinh `access_token` JWT quyền Admin, frontend tự nạp token vào Socket và fetch queries.

## 4. Impacted Components

- **Affected Pages:**
- `SnakeAid.Api/Pages/Demo/RescueDemo.cshtml` (add link to monitor)
- **New:** `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml` (Giao diện 3 khu vực + AJAX logic)
- **New:** `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs` (Cấp Token + Cung cấp Handlers `OnGetGhostsAsync` & `OnGetAllRescuersAsync`)
- **Modified:** `SnakeAid.Api/Hubs/RescuerHub.cs` (Sạch sẽ, chỉ thuần túy phát sóng RT event như `LocationUpdated`, `RescuerJoined`, không chứa logic truy xuất DB nặng ngoài trừ lúc Join/Disconnect).

## 5. Risks & Constraints

- **Security Constraint**: The page generates a system-level token. Since it sits inside `Pages/Admin`, it should only be accessible locally during dev or behind admin auth in production.
- **Hub Modification Risk**: We must avoid polluting the actual mobile clients with "Monitor" events. Admin logs should only be sent to the "Monitor" group.

## 6. Validation Plan

1. **UC1**: Open Monitor, connect AVD. Monitor shows connected. Kill App on AVD. Monitor shows disconnected.
2. **UC2**: Trigger location update stream on AVD. Monitor console shows updates arriving and verifies logs aren't spamming faster than throttle policy.
3. **UC3 & UC4**: Trigger SOS. Monitor displays Session 1 and 60s countdown. Wait 60s. Monitor displays logic advancing to Session 2.
4. **UC5**: Connect 2 AVDs. Trigger SOS. Both receive it. AVD 1 accepts. AVD 2 accepts 1s later. Monitor explicitly logs AVD 1 as winner and AVD 2 receiving `RequestTaken`.
