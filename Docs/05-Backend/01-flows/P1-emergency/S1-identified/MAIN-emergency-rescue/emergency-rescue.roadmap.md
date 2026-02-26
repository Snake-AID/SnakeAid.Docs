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

---

## Deferred Alignment Notes (Implement later)

Section nay la ghi chu can nhac ky thuat de doi den luc implement, KHONG lam code ngay trong turn nay.

### 1) UpdateLocation current state
- `RescuerHub.UpdateLocation(...)` hien tai chi log + echo event cho caller.
- Chua persist DB, chua validate payload, chua dung claim identity.

### 2) ASP.NET Identity feasibility
- Co the bo `userId` khoi payload.
- Lay account id tu `ClaimTypes.NameIdentifier` trong token la kha thi va nen dung.

### 3) LT-1 ingest policy (de xuat can chot)
- Client publish interval: ~2s (co the tinh chinh sau load test).
- Server min interval (throttle): ~1s/account.
- Stale threshold: 30-60s (qua nguong thi khong dung de pairing).
- Payload toi thieu LT-1: `lat`, `lng`, `timestamp?`, `accuracy?`.
- `speed`/`heading` de optional, uu tien LT-2.

### 4) Entity strategy for location fields
- Giu `RescuerProfile.LastLocation` + `LastLocationUpdate` cho dispatch matching.
- Dung `TrackingSession.MemberLocation` + `RescuerLocation` cho vi tri theo phien.
- Dung `LocationEvent.Location` cho history/audit.
- Khong them `MemberProfile.LastLocation` o LT-1 (tranh profile-scoped location khong can thiet).

### 5) Data model normalization note
- Can can nhac chuan hoa `LocationEvent.Location` ve `geometry(Point, 4326)` de dong bo voi cac field PostGIS khac.
- Viec nay de phase implementation (co migration plan).

### 6) Team conflict note (Global Phase 1 song song)
- Diem de va cham nhat la `RescuerHub`.
- LT-1 nen gioi han scope vao location ingestion path, tranh cham vao accept/reject orchestration neu khong can thiet.
