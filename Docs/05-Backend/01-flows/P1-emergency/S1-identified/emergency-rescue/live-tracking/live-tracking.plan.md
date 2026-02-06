# Live Tracking Plan (2 Domain Phases)

Liên kết roadmap tổng:
- `../emergency-rescue.roadmap.md`

## Function Guardrail (Để tránh trùng với rescue-trigger)

Nguồn trạng thái dispatch functions:
- `../rescue-trigger/rescue-trigger.sourcecode.md` -> `Function Implementation Status (Agent Guardrail)`

Rule:
1. `live-tracking` không re-implement các function dispatch core đã thuộc `rescue-trigger`.
2. Nếu cần thay đổi dispatch dependency, patch interface/service hiện có thay vì tạo flow song song.
3. Mọi thay đổi ảnh hưởng dispatch phải update lại docs ở cả 2 domain.

## Domain Phase LT-1 (Global Phase 2)
Tên phase: Ingestion Foundation

Mục tiêu:
1. Thiết lập cơ chế cập nhật vị trí thật từ rescuer app.
2. Làm cho nguồn query PostGIS của rescue-trigger có dữ liệu sống.

Phạm vi chính:
1. `SnakeAid.Api/Hubs/RescuerHub.cs` (hoặc endpoint location tương đương).
2. Service layer để persist location.
3. `RescuerProfile.LastLocation`, `LastLocationUpdate`.

Work items:
1. Persist location:
- Ghi `LastLocation` và `LastLocationUpdate` khi rescuer publish.
 - Không tạo duplicate dispatch matching logic trong live-tracking service.
2. Data quality guard:
- Validate lat/lng.
- Throttle write để tránh spam.
- Chính sách stale location.
3. Security:
- Chỉ cho rescuer hợp lệ cập nhật vị trí của chính họ.

Done criteria:
1. Có dữ liệu vị trí realtime trong DB.
2. Dispatch query PostGIS match được theo vị trí cập nhật mới.
3. Không có lỗi ghi quá tải/identity mismatch trong luồng cập nhật vị trí.

---

## Domain Phase LT-2 (Global Phase 3)
Tên phase: Full Live Tracking Pipeline

Mục tiêu:
1. Hoàn thiện tracking end-to-end cho rescuer/patient/admin.
2. Chuẩn bị nền cho rescue-trigger chuyển sang Redis-first ở RT-2.

Phạm vi chính:
1. Realtime streaming theo session.
2. Redis NOW-state + PostGIS PAST-state.
3. Snapshot/History API.
4. Fallback delivery cho event quan trọng.

Work items:
1. Session viewer stream:
- Join/leave session group.
- Broadcast `LocationUpdated` đến patient/admin viewer.
 - Không thay thế luồng `NewRescueRequest` hiện có của rescue-trigger.
2. Data pipeline:
- Redis: presence, geo index, last-known.
- PostGIS: location history/audit (persist theo nhịp thưa).
3. Read APIs:
- Snapshot endpoint.
- History endpoint.
4. Reliability:
- Fallback notification cho event critical.
- Reconnect flow: snapshot trước, stream sau.

Done criteria:
1. Viewer map theo dõi rescuer realtime ổn định.
2. Reconnect không mất trạng thái chính.
3. Có đủ dữ liệu history để audit/replay.
4. Redis/PostGIS contracts sẵn sàng cho `rescue-trigger` RT-2.
