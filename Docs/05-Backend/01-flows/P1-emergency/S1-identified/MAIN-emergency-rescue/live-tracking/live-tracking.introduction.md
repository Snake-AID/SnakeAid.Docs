---
doc_role: baseline
module: live-tracking
kind: flow
status: active
last_updated: 2026-02-15
owners: [backend-team]
---

# Live Tracking Module - Introduction

## Roadmap Position
This domain follows:
- `LT-1` (Global Phase 2): ingestion foundation.
- `LT-2` (Global Phase 3): full live tracking pipeline.

Roadmap source:
- `../emergency-rescue.roadmap.md`

## Context
Live tracking is a cross-layer capability for emergency rescue:
- Patient should see rescuer movement in near real time.
- Rescuer should receive dispatch offers quickly and react in-app.
- Admin should observe active incidents and spatial trends.

The architecture direction is documented in:
- `live-tracking.architecture.md`

This file describes what this layer means in SnakeAid and what is currently implemented in codebase.

## Business Goal

1. Dispatch the right rescuer quickly using location.
2. Keep rescue session visible in real time.
3. Provide fallback delivery for critical dispatch events.
4. Keep enough history for auditing and map analytics.

## Current Maturity (as of 2026-02-06)

Implemented:
- Session-based dispatch for rescue requests.
- Radius progression (`10 -> 20 -> 30`) with timeout-driven auto expansion.
- SignalR-based rescuer request delivery and acceptance race handling.

Partially implemented:
- Rescuer location message exists in hub (`UpdateLocation`) but is not persisted or fan-out to patient/admin.

Not implemented yet:
- Tracking snapshot API for patient/admin map.
- Tracking history API.
- Redis NOW-store/presence/geo-index.
- FCM fallback for critical events.
- Session-based map group broadcast for patient/admin clients.

## Domain Boundaries vs Rescue Trigger
- `rescue-trigger` owns dispatch state machine and assignment decisions.
- `live-tracking` owns location ingestion, session stream, snapshot/history, and realtime data pipeline.
- `rescue-trigger` RT-2 depends on `live-tracking` LT-2 contracts for Redis GEO readiness.

## Scope Boundary for This Layer

In-scope:
- Dispatch and live location pipeline contracts.
- Service boundaries: incident, session, mission, notification, timeout orchestration.
- Realtime transport patterns and data ownership guidance.

Out-of-scope:
- UI rendering specifics in Flutter.
- Payment, review, and non-tracking mission business logic.

## Why This Matters for Current Backend

Existing rescue-trigger flow already provides a strong dispatch base.
The live-tracking layer now needs to:
- convert dispatch-only realtime into full session tracking,
- separate transport concerns from state ownership,
- add durable/recoverable mechanisms (snapshot, history, fallback).

## Related Documents

- Roadmap: `../emergency-rescue.roadmap.md`
- Architecture direction: `live-tracking.architecture.md`
- Current code state: `live-tracking.sourcecode.md`
- Completion roadmap: `live-tracking.plan.md`
- Implementation prompt template: `live-tracking.prompt.md`
- Integration contract and current API/hub behavior: `live-tracking.usageguide.md`
# SnakeAid – Live Location Tracking (LLT)

Tài liệu này mô tả **thiết kế kỹ thuật mức “solution architecture + hành vi hệ thống”** cho nghiệp vụ **Live Location Tracking** trong SnakeAid (ASP.NET monolith + Flutter), nhằm giúp triển khai đúng hướng **mà không khóa chặt vào chi tiết code/SQL/command**.

> Mục tiêu của tài liệu: người implement có thể build đúng hành vi và ranh giới trách nhiệm, đồng thời vẫn có chỗ linh hoạt để chọn cách code phù hợp codebase.

---

## 1. Phạm vi nghiệp vụ

### 1.1 Những gì LLT phải làm

* **Dispatch theo vị trí**: khi Patient SOS, hệ thống tìm Rescuer gần nhất/phù hợp và gửi “offer” như Grab.
* **Realtime tracking trong phiên**: Patient/Admin thấy vị trí rescuer thay đổi theo thời gian.
* **Fallback thông báo quan trọng**: khi realtime không tới (mất socket, app bị kill, network chuyển…), vẫn đảm bảo offer/huỷ/hoàn tất tới người nhận.
* **Audit & analytics**: lưu được dấu vết vị trí theo thời gian để truy vết và phân tích (heatmap/vùng nguy cơ).

### 1.2 Những gì LLT không cố làm ở MVP

* Không tối ưu thuật toán định tuyến/ETA nâng cao ngay từ đầu.
* Không yêu cầu background tracking phức tạp nếu team chọn policy “Rescuer phải mở app khi nhận đơn”.

---

## 2. Nguyên tắc thiết kế (Design invariants)

### 2.1 Phân tách NOW vs PAST

* **Redis = NOW**: dữ liệu realtime, tươi, có hạn sử dụng (TTL), có thể mất mà không phá hệ thống.
* **PostgreSQL + PostGIS = PAST**: dữ liệu bền vững, phục vụ replay/audit/analytics.

### 2.2 Transport vs Decision

* **SignalR** là kênh truyền realtime (transport).
* **Offer/Dispatch/Tracking API** mới là nơi quyết định nghiệp vụ (decision): ai nhận offer, khi nào fallback push, khi nào kết thúc session.

### 2.3 Không có client gọi trực tiếp Redis/PostGIS

* Flutter **chỉ nói chuyện với Backend** (REST + SignalR).

---

## 3. Tech stack và vai trò (không đi sâu implement)

### 3.1 Flutter

* Location capture: GPS (tần suất/độ chính xác có throttle).
* Map render: marker/polyline/heatmap layer.
* Realtime: SignalR client.
* Push: FCM.

### 3.2 ASP.NET Core Monolith

* REST APIs: tạo SOS, dispatch, snapshot map, history.
* SignalR Hub: stream vị trí + broadcast theo session group.
* Notification module: gửi FCM.
* Background job runner (Hangfire/Quartz): timeout offer, retention/aggregation.

### 3.3 Data

* Redis: GEO query + last-known + presence TTL + các “locks” chống race.
* Postgres + PostGIS: incident/session/offer state + location history + spatial analytics.

---

## 4. Backend decomposition (trong monolith)

Các “service/module” dưới đây là **ranh giới trách nhiệm**, không bắt buộc là project/namespace riêng.

1. **SOS / Incident Service**

* Nhận SOS / report, tạo incident + rescue session.

2. **Dispatch / Matching Service**

* Tìm rescuer phù hợp dựa trên vị trí và điều kiện (availability/presence/rule ranking).

3. **Offer Service**

* Tạo offer, nhận accept/reject, xử lý timeout/redispatch.
* Quyết định fallback push cho các event quan trọng.

4. **Tracking API**

* Cung cấp snapshot vị trí hiện tại và history cho map.

5. **SignalR Hub**

* Nhận location publish và fan-out tới các client đã join session.
* Cập nhật Redis (NOW state) theo nhịp realtime.

6. **Notification Module**

* Gửi push qua FCM cho event quan trọng.

7. **Background Jobs**

* Offer timeout / redispatch.
* Persist/aggregate location history theo nhịp thưa.

---

## 5. Data ownership & retention

### 5.1 Redis (NOW)

Redis giữ các nhóm dữ liệu sau:

* **Presence**: rescuer còn online/active trong một khoảng thời gian ngắn.
* **Last-known location**: vị trí gần nhất của rescuer (và có thể patient nếu cần).
* **Geo index**: phục vụ tìm rescuer gần nhất theo bán kính.
* **Atomic guard/lock**: chống 2 rescuer nhận cùng một offer.

**Nguyên tắc retention:**

* Dữ liệu NOW phải có TTL. TTL chỉ cần đủ dài để “không rớt” trong trường hợp mạng chập chờn ngắn.

### 5.2 Postgres + PostGIS (PAST)

* **Incident / Session / Offer**: trạng thái nghiệp vụ bền vững.
* **Location history**: chuỗi điểm theo thời gian (append-only) cho audit/replay.
* **Spatial analytics**: heatmap, vùng polygon, thống kê theo khu vực.

**Nguyên tắc retention:**

* Lịch sử có thể rất lớn → cần chính sách sampling và/hoặc retention (theo ngày/tháng) cho đồ án.

---

## 6. Luồng nghiệp vụ (behavioral flows)

### 6.1 Flow A – Dispatch + kích hoạt tracking

Mục tiêu: từ SOS → gửi offer → chốt assigned rescuer → bắt đầu tracking.

Hành vi mong muốn:

1. Patient gửi SOS → Backend tạo incident + session.
2. Backend chạy matching dựa trên vị trí (Redis GEO) và điều kiện.
3. Backend tạo offer (pending) và gửi tới Rescuer qua SignalR.
4. Nếu Rescuer accept → Backend chốt session (assigned) và phát sự kiện cho Patient/Admin.
5. Ngay sau khi accept, Rescuer bắt đầu publish location → tracking realtime khởi động.
6. Nếu SignalR không tới/không ACK trong thời gian ngắn → Backend kích hoạt FCM để đảm bảo Rescuer vẫn nhận được offer.

**Điểm nhấn:** Flow A **không kết thúc ở “Offer Accepted”** mà kết thúc khi **điểm location đầu tiên** được publish và NOW state được cập nhật.

### 6.2 Flow B – Live tracking trong session

Mục tiêu: rescuer stream vị trí và client render liên tục.

Hành vi mong muốn:

1. Rescuer publish location (lat/lng/timestamp/accuracy…) qua SignalR.
2. Hub fan-out `LocationUpdated` tới Patient/Admin đã join session.
3. Redis được cập nhật last-known + presence theo nhịp realtime.
4. Tracking API cung cấp snapshot (Redis-first) và history (PostGIS) cho các tình huống:

   * người dùng mở map lần đầu
   * reconnect
   * Redis miss

---

## 7. Khi nào cần FCM (fallback matrix)

### 7.1 Sự kiện bắt buộc push

* NewOffer
* Offer expiring/timeout
* Offer cancelled / session cancelled
* Session completed (nếu cần)

### 7.2 Khi SignalR có thể không tới

* App bị OS kill/đóng hẳn.
* Mạng đổi (Wi‑Fi ↔ 4G), websocket drop.
* Thiết bị vào chế độ tiết kiệm pin, background throttling.
* Captive portal / proxy / NAT.

**Nguyên tắc:**

* SignalR là kênh realtime chính.
* FCM là kênh đảm bảo delivery cho event quan trọng.

---

## 8. Hợp đồng dữ liệu (contracts) giữa backend và Flutter

Tài liệu này không cố định endpoint/method name, nhưng yêu cầu các **shape** sau tồn tại:

### 8.1 Snapshot payload

* Session metadata: status, assigned rescuer, timestamps.
* Current rescuer position: lat/lng + timestamp + staleness.
* Optional: ETA/route summary (nếu có).

### 8.2 Realtime event payload

* SessionId
* RescuerId
* Location: lat/lng + timestamp + accuracy + (optional speed/heading)
* Sequence/ordering hint để client bỏ điểm cũ.

### 8.3 Offer payload

* OfferId, SessionId, expiry window
* Incident location (để rescuer quyết định)
* Minimal patient info cần thiết (ẩn danh nếu muốn)

---

## 9. NFR – Non-functional requirements (định hướng, không khóa cứng)

### 9.1 Performance

* Payload realtime tối giản.
* Throttle publish vị trí để tránh spam UI và server.
* Snapshot nên Redis-first để nhanh.

### 9.2 Reliability

* Có retry/fallback cho offer quan trọng.
* Không phụ thuộc vào Redis để giữ sự thật nghiệp vụ.

### 9.3 Security (đồ án có thể giản lược)

* Không cho client truy cập trực tiếp Redis.
* Session tracking phải có kiểm soát quyền (patient/rescuer/admin theo session).

---

## 10. Spatial analytics (Admin)

### 10.1 Heatmap

* Dựa trên incident locations và/hoặc sightings.
* Backend trả về dữ liệu “density/weight” phù hợp heatmap layer (không bắt buộc trả toàn bộ raw points).

### 10.2 Polygon zones

* Lưu vùng nguy cơ (polygon) trong PostGIS.
* Backend trả GeoJSON hoặc format tương đương để client render polygon.

### 10.3 Intersection/Buffer

* Backend có thể trả boolean/level (incident thuộc vùng nguy cơ hay không) để UI hiển thị cảnh báo.

---

## 11. Guardrails cho implementer

Những điều **không được phá** để tránh sai hành vi:

* Không để SignalR “ôm” business logic (chỉ transport + fan-out + update NOW state).
* Không dùng PostGIS làm realtime store cho mỗi điểm GPS.
* Không dùng Redis làm nguồn chân lý cho trạng thái session/offer.
* Không để client gọi Redis trực tiếp.
* Offer phải có cơ chế chống race (lock/atomic).
* Snapshot phải có đường fallback khi Redis miss.

---

## 12. Checklist triển khai MVP

1. SOS + session state machine (Postgres).
2. Matching theo vị trí (Redis GEO).
3. Offer lifecycle + timeout/redispatch.
4. SignalR realtime stream + group per session.
5. Redis NOW state: presence + last-known.
6. Snapshot API (Redis-first, PostGIS fallback).
7. FCM cho offer quan trọng.
8. Persist location history theo nhịp thưa cho audit.
9. Admin heatmap/polygon (tối thiểu 1 feature).
