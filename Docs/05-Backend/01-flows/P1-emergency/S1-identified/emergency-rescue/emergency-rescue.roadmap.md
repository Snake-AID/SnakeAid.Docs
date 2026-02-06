# Emergency Rescue Roadmap (4 Phases)

Tài liệu này là định hướng chung cho 2 domain:
- `rescue-trigger`
- `live-tracking`

Mục tiêu: triển khai theo thứ tự để vừa đảm bảo dispatch hoạt động đúng nghiệp vụ, vừa tránh làm lại khi chuyển sang Redis GEO.

## Global Phase 1 - Ổn định Rescue Trigger Core (Owner: rescue-trigger)

Trọng tâm:
1. Chốt state machine dispatch (incident/session/request/mission).
2. Fix đường manual `raise-range` để đồng bộ với timeout expansion.
3. Harden hub auth/identity.
4. Hoàn thiện timeout semantics (`Taken`, `Cancelled`, `Expired`).

Nguồn query gần nhất ở phase này:
- Tạm dùng PostGIS hiện tại (`RescuerProfile.LastLocation`) làm nguồn query chính.

Output bắt buộc:
- Dispatch flow ổn định về trạng thái và race condition.

---

## Global Phase 2 - Live Tracking Ingestion Foundation (Owner: live-tracking)

Trọng tâm:
1. Có write-path vị trí thật từ rescuer app.
2. Persist vị trí vào DB (`LastLocation`, `LastLocationUpdate`).
3. Validate + throttle tần suất update.

Output bắt buộc:
- `rescue-trigger` query PostGIS có dữ liệu vị trí sống, không còn phụ thuộc dữ liệu cũ/mock.

---

## Global Phase 3 - Live Tracking Full Pipeline (Owner: live-tracking)

Trọng tâm:
1. Realtime stream cho viewer theo session.
2. Redis NOW-state (`last-known`, `presence`, `geo`) + PostGIS PAST-state (history/audit).
3. Snapshot/History API cho reconnect và replay.
4. Fallback notification cho event quan trọng.

Output bắt buộc:
- Live tracking end-to-end cho rescuer/patient/admin.

---

## Global Phase 4 - Rescue Trigger Dynamic Locator (Owner: rescue-trigger)

Trọng tâm:
1. Chuyển matching sang `Redis-first`.
2. `PostGIS-fallback` khi Redis miss hoặc degraded mode.
3. Tách abstraction locator để đổi strategy không đập flow (`IRescuerLocator`).
4. Feature flag + metrics để rollout an toàn.

Output bắt buộc:
- Pairing nhanh theo realtime location, vẫn đảm bảo an toàn khi fallback.

---

## Mapping nhanh theo domain

1. `rescue-trigger`:
- Domain Phase RT-1 = Global Phase 1
- Domain Phase RT-2 = Global Phase 4

2. `live-tracking`:
- Domain Phase LT-1 = Global Phase 2
- Domain Phase LT-2 = Global Phase 3

