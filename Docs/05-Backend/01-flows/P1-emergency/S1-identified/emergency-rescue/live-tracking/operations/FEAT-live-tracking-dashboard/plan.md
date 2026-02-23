---
doc_role: operation
operation_id: FEAT-live-tracking-dashboard
type: FEAT
status: done
created_at: 2026-02-23
affects:
  - SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml
  - SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs
  - SnakeAid.Api/Hubs/RescuerHub.cs
  - SnakeAid.Api/Services/SignalRRescueNotificationService.cs
  - SnakeAid.Service/Implements/RescuerLocationService.cs
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

1. **Header**: Avatar, FullName, và Badge hiển thị loại Rescuer (Type), Rating (⭐), TotalMissions (🚑). Đi kèm là **Status Badge** chỉ báo trạng thái luân chuyển (IDLE / PINGED / BUSY).
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

- **New:** `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml` — Giao diện 3 Tab (Connected / Ghost Scanner / All Rescuers) + toàn bộ SignalR JS + AJAX logic.
- **New:** `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs` — Cấp JWT Admin token qua `AdminToken` property; handlers `OnGetGhostsAsync` & `OnGetAllRescuersAsync`.
- **Modified:** `SnakeAid.Api/Hubs/RescuerHub.cs` — Thêm `JoinAsMonitor()`, `GetConnectedRescuers()`, và instrumentation phát `AdminLog` tới nhóm `"Monitors"` tại `JoinAsRescuer`, `AcceptRequest`, `OnDisconnectedAsync`. `UpdateLocation` phát `LocationUpdated` tới `Clients.Caller` và `Clients.Group("Monitors")`.
- **Modified:** `SnakeAid.Api/Services/SignalRRescueNotificationService.cs` — Thêm hỗ trợ phát `NewRescueRequest`, `RequestTaken`, `RequestCancelled`, `RequestExpired` tới nhóm `"Monitors"`.
- **Modified:** `SnakeAid.Service/Implements/RescuerLocationService.cs` — Throttle window sử dụng `_throttleInterval` từ cấu hình (`LocationUpdate:ThrottleIntervalSeconds`, mặc định 10s).

## 5. Risks & Constraints

- **Security Constraint**: The page generates a system-level token. Since it sits inside `Pages/Admin`, it should only be accessible locally during dev or behind admin auth in production.
- **Hub Modification Risk**: We must avoid polluting the actual mobile clients with "Monitor" events. Admin logs should only be sent to the "Monitor" group.

## 6. Validation Plan

1. **UC1**: Open Monitor, connect AVD. Monitor shows connected. Kill App on AVD. Monitor shows disconnected.
2. **UC2**: Trigger location update stream on AVD. Monitor console shows updates arriving and verifies logs aren't spamming faster than throttle policy.
3. **UC3 & UC4**: Trigger SOS. Monitor displays Session 1 and 60s countdown. Wait 60s. Monitor displays logic advancing to Session 2.
4. **UC5**: Connect 2 AVDs. Trigger SOS. Both receive it. AVD 1 accepts. AVD 2 accepts 1s later. Monitor explicitly logs AVD 1 as winner and AVD 2 receiving `RequestTaken`.

## 7. Post-Implementation Bug Fixes

Các lỗi sau được phát hiện và vá sau khi feature được deliver:

| File                                  | Bug                                                                                                                                                             | Fix                                                                                                                                                      |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `RescuerHub.cs` · `UpdateLocation`    | Sau khi GUID parse thất bại, code vẫn tiếp tục log success và broadcast tới Caller/Monitors                                                                     | Moved log + `SendAsync` calls vào bên trong nhánh GUID hợp lệ; nhánh `else` chỉ log warning rồi kết thúc                                                 |
| `Index.cshtml` · `startPingTimer`     | `clearInterval(connectedRescuers[id].timerInterval)` được gọi sau khi đã xác nhận `connectedRescuers[id]` là falsy → TypeError                                  | Dùng `const intervalId = setInterval(...)` local closure; guard bên trong tick bằng `if (!connectedRescuers[id]) { clearInterval(intervalId); return; }` |
| `Index.cshtml` · `createRescuerCard`  | Early-return khi card đã tồn tại khiến placeholder card (từ `LocationUpdated`) chặn dữ liệu đầy đủ hơn từ `AdminLog/RescuerJoined`                              | Thêm `updateRescuerCard(id, rescuer)` helper; `createRescuerCard` gọi helper thay vì return sớm                                                          |
| `Index.cshtml` · reconnect            | `withAutomaticReconnect()` được dùng nhưng không có `onreconnected` handler → mất membership nhóm Monitors sau mỗi reconnect                                    | Thêm `connection.onreconnected(async () => { await connection.invoke("JoinAsMonitor"); })`                                                               |
| `SignalRRescueNotificationService.cs` | `Monitors` broadcast của `RequestTaken`, `RequestCancelled`, `RequestExpired` nằm bên trong `TryGetValue` guard → dashboard bỏ lỡ events khi rescuer đã offline | Chuyển `Clients.Group("Monitors").SendAsync(...)` ra ngoài `TryGetValue` block                                                                           |
| `RescuerLocationService.cs`           | Cache expiry hardcode `TimeSpan.FromSeconds(5)` — ngắn hơn `_throttleInterval` (10s) → throttle bị vô hiệu hóa trong nửa sau cửa sổ                             | Thay bằng `_memoryCache.Set(cacheKey, DateTime.UtcNow, _throttleInterval)`                                                                               |
