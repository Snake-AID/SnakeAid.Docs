---
doc_role: operation
operation_id: 03-emergency-consultation
type: FEAT
status: done
created_at: 2026-03-05
merged_from: [04-FEAT-emergency-consultation]
affects:
  - Api/Hubs/ExpertHub.cs
  - Api/Controllers/ConsultationsController.cs
  - Api/Controllers/ExpertController.cs
  - Service/Implements/ExpertService.cs
  - Service/Implements/EmergencyConsultationService.cs
  - Core/Domains/ExpertProfile.cs
  - Core/Domains/ConsultationPingRequest.cs
  - Tests/Integration/ExpertControllerIntegrationTests.cs
  - Tests/Unit/ExpertServiceTests.cs
---

# Operation 03: Emergency Consultation Flow

## Mục tiêu

Implement luồng tư vấn ngay: user chọn expert online → thanh toán → gửi request → expert accept/reject → vào phòng. Bao gồm ExpertHub presence và directory filter/sort mở rộng.

## Scope đã implement

### ExpertHub (`/hubs/expert`)

- `JoinAsExpert` — expert đăng ký presence, set `IsOnline = true`, broadcast `ExpertPresenceChanged`
- `JoinAsMember` — user subscribe presence, nhận `OnlineExpertsSnapshot`
- `JoinEmergencyRequestRoom(requestId)` — user subscribe status updates cho request cụ thể (owner-only)
- Events: `OnlineExpertsSnapshot`, `ExpertPresenceChanged`, `EmergencyConsultationRequest`, `EmergencyRequestStatusChanged`
- `EnablePresenceSelfHealing` hardcoded `false`

### REST Endpoints

- `POST /api/consultations/emergency-requests` — tạo request, bắt đầu ở `PendingPayment`
- `POST /api/consultations/emergency-requests/{requestId}/payments` — thanh toán, chuyển `PendingExpertResponse`, push request sang expert
- `POST /api/consultations/emergency-requests/{requestId}/accept` — expert accept, tạo consultation, reserve overlapping slots (Slot Paradox)
- `POST /api/consultations/emergency-requests/{requestId}/reject` — expert reject, refund escrow

### Directory Filter/Sort (corrective scope từ Op 01)

- `GET /api/experts` mở rộng: `specialization`, `isOnline`, `sortBy` (isOnline/rating/consultationFee), `sortOrder`
- `ExpertDirectoryQueryRequest` kế thừa pagination
- Deterministic secondary ordering cho stable pagination

### Emergency State Machine

```
PendingPayment → (payment success) → PendingExpertResponse → AcceptedByExpert
                                                            → DeclinedByExpert → refund
                                                            → Expired → refund
```

### Domain Rules

- Request chỉ push sang expert sau khi payment success
- Expert accept: tạo consultation `Ongoing`, reserve overlapping `ExpertTimeSlot` trong 30 phút (Slot Paradox)
- Expert reject: refund escrow ngay lập tức
- Expiry: background service mark `Expired` + refund + push `EmergencyRequestStatusChanged`
- TTL hardcoded 2 phút
- Chỉ targeted expert mới accept/reject được
- Chỉ request owner mới join được request room
