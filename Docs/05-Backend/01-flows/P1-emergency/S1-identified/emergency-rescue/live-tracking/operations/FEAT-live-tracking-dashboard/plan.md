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

**Khu vực 1 (Main - 75%): Bảng Điều Khiển Rescuers (Rescuers Grid)**
Hiển thị danh sách các Rescuer đang kết nối dưới dạng mạng lưới các thẻ (Cards).
Mỗi **Rescuer Card** bao gồm:

1. **Header**: Avatar, FullName, PhoneNumber, và Badge hiển thị loại Rescuer (Emergency/Catching/Both).
2. **Status Bar**: Chấm trạng thái Online/Offline, Rating (⭐), CompletedMissions/TotalMissions, và Reputation Points.
3. **Live Location (UC2)**: Hiển thị tọa độ (Lat, Lng) và `Last Update: X seconds ago`. Vùng này sẽ tự động **nhấp nháy highlight** khi có sự kiện `LocationUpdated` để test Throttling 10s.
4. **Session State (UC1, UC3, UC5)**: Màu sắc viền (Border) và Background của thẻ sẽ thay đổi realtime theo vòng đời Incident:
   - _Mặc định_: Đang rảnh (Màu xám/trắng).
   - _Khi có SOS (Nằm trong bán kính)_: Viền **nháy Vàng** (Pinged - Đang chờ Accept).
   - _Khi Win ca cấp cứu_: Thẻ đổi màu **Xanh Lá** đậm (Winner/Accepted).
   - _Khi bị nẫng tay trên_: Thẻ đổi màu **Xám Xỉn** (Taken/Lost).

**Khu vực 2 (Sidebar - 25%): Active Incident Tracker (UC3 & UC4)**
Theo dõi tiến độ chung của hệ thống khi có 1 ca cấp cứu (Incident) nổ ra.

- **Incident Info**: Incident ID, Session Number hiện tại, Radius hiện tại (10km, 20km).
- **Timeout Countdown**: Một thanh Progress Bar khổng lồ đếm ngược 60 giây. Cứ hết 60s mà không ai nhận thì reset lại vòng mới, Radius tự tăng cường lên.
- **Race Condition Log**: Ghi chú vắn tắt xem ai Vừa Win, ai bị Expired. Hệ thống tự động đẩy dữ liệu sang làm đổi màu các thẻ Rescuer bên nhóm Khu Vực 1.

**Technical Approach:**

- **Backend (`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs`)**: Page model that auto-generates a temporary Admin JWT to inject into the view.
- **Frontend (`SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml`)**:
  - Use raw HTML/CSS/JS.
  - Integrate `@microsoft/signalr` JS client to connect to `/rescuer-hub` as an "Admin/Monitor" role.
  - _Note:_ The Hub might need a minor adjustment to broadcast administrative logs to a specific "MonitorGroup" so the dashboard can listen without intercepting actual Rescuer messages, OR the dashboard simply connects and listens to standard events if applicable.
- **Authentication**: The backend `Index.cshtml.cs` will auto-generate a valid `access_token` (JWT) with Admin privileges. The Razor View will read this token and pass it to the SignalR connection automatically, meaning the user just needs to open the page.

## 4. Impacted Components

- **Affected Pages:**
- `SnakeAid.Api/Pages/Demo/RescueDemo.cshtml` (add link to monitor)
- **New:** `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml`
- **New:** `SnakeAid.Api/Pages/Admin/LiveTracking/Index.cshtml.cs`
- **Modified:** `SnakeAid.Api/Hubs/RescuerHub.cs` (Minor changes to emit connection state/admin logs to a "Monitor" SignalR group if necessary).

## 5. Risks & Constraints

- **Security Constraint**: The page generates a system-level token. Since it sits inside `Pages/Admin`, it should only be accessible locally during dev or behind admin auth in production.
- **Hub Modification Risk**: We must avoid polluting the actual mobile clients with "Monitor" events. Admin logs should only be sent to the "Monitor" group.

## 6. Validation Plan

1. **UC1**: Open Monitor, connect AVD. Monitor shows connected. Kill App on AVD. Monitor shows disconnected.
2. **UC2**: Trigger location update stream on AVD. Monitor console shows updates arriving and verifies logs aren't spamming faster than throttle policy.
3. **UC3 & UC4**: Trigger SOS. Monitor displays Session 1 and 60s countdown. Wait 60s. Monitor displays logic advancing to Session 2.
4. **UC5**: Connect 2 AVDs. Trigger SOS. Both receive it. AVD 1 accepts. AVD 2 accepts 1s later. Monitor explicitly logs AVD 1 as winner and AVD 2 receiving `RequestTaken`.
