# Rescue Trigger Plan (2 Domain Phases)

Liên kết roadmap tổng:
- `../emergency-rescue.roadmap.md`

## Function Guardrail (Bắt buộc đọc trước khi code)

Nguồn trạng thái chi tiết:
- `rescue-trigger.sourcecode.md` -> `Function Implementation Status (Agent Guardrail)`

Rule:
1. Ưu tiên sửa/mở rộng function hiện có.
2. Không tạo flow/function mới nếu đã có function tương đương đang chạy.
3. Nếu function đã tồn tại nhưng hành vi thiếu, chỉ patch phần thiếu.

Tóm tắt nhanh:
1. Đã implement và chạy: `CreateIncidentAsync`, `StartRescueAsync`, `CreateSessionAsync`, `BroadcastRequestsAsync`, `HandleSessionTimeoutAsync`, `TryExpandAndCreateNewSessionAsync`, `AcceptRequestAsync`, `RejectRequestAsync`.
2. Đã có code nhưng chưa active path chính: `TriggerRescueAsync`, `AcceptRescueAsync`, `RejectRescueAsync`.
3. Đã có nhưng incomplete: `RaiseSessionRangeAsync`, `RescuerHub.UpdateLocation`, wiring `NotifyRequestExpiredAsync`.
4. Chưa có: locator abstraction và Redis-first matching path.

## Domain Phase RT-1 (Global Phase 1)
Tên phase: Ổn định Dispatch Core

Mục tiêu:
1. Hoàn thiện orchestration và state transition của dispatch flow.
2. Loại bỏ sai khác giữa manual raise-range và timeout expansion.
3. Hardening auth/identity cho hub.

Phạm vi chính:
1. `SnakeAid.Service/Implements/SnakebiteIncidentService.cs`
2. `SnakeAid.Service/Implements/RescueRequestSessionService.cs`
3. `SnakeAid.Api/Hubs/RescuerHub.cs`

Work items:
1. Unify raise-range path:
- Manual raise-range phải tạo session + broadcast + schedule timeout tương đương timeout path.
 - Không tạo flow mới ngoài `RaiseSessionRangeAsync`; patch trực tiếp function hiện có.
2. Hardening hub:
- Bắt buộc auth.
- Identity lấy từ token claim, không trust `userId` client gửi lên.
 - Không tạo hub mới nếu chưa có yêu cầu tách transport.
3. Timeout semantics:
- Đảm bảo request/session state khi timeout/cancel/accept luôn nhất quán.
 - Tận dụng `HandleSessionTimeoutAsync` hiện có, không duplicate timeout processor.

Query strategy trong RT-1:
- Dùng PostGIS hiện tại (`LastLocation.Distance`) làm query chính.
- Không chốt cứng kiến trúc locator để chuẩn bị RT-2.

Done criteria:
1. Không còn chênh behavior giữa manual và auto expansion.
2. Race condition accept được khóa đúng.
3. Hub không cho impersonation.

---

## Domain Phase RT-2 (Global Phase 4)
Tên phase: Dynamic Locator Switch (Redis-first)

Mục tiêu:
1. Chuyển pairing sang realtime query.
2. Vẫn giữ an toàn khi fallback.

Phạm vi chính:
1. `RescueRequestSessionService` matching path.
2. Locator abstraction mới (gợi ý `IRescuerLocator`).
3. Feature flag + telemetry cho rollout.

Work items:
1. Tách locator abstraction:
- `GetCandidatesAsync(incidentLocation, radius, constraints)`.
 - Chỉ tách abstraction ở query layer, không rewrite toàn bộ session orchestration.
2. Implement Redis-first strategy:
- Query candidate từ Redis GEO.
3. Implement PostGIS fallback:
- Fallback khi Redis miss/degraded.
4. Rollout safety:
- Feature flag để chuyển dần traffic.
- Metrics: query latency, hit ratio, fallback ratio.

Done criteria:
1. Pairing sử dụng Redis-first trong production mode.
2. Fallback PostGIS chạy được và đã test.
3. Không phá API contract hiện tại của `rescue-trigger`.
