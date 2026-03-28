---
doc_role: roadmap
module: consultation
status: active
created_at: 2026-03-28
last_updated: 2026-03-28
---

# Consultation Implementation Roadmap

## Investigation Summary (2026-03-28)

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1 | PayOS method cho consultation | DONE | `ConsultationPaymentService` da implement day du: create intent, confirm, webhook, cho ca scheduled va emergency |
| 2 | Lock schedule time khi co emergency request | DONE | `AcceptEmergencyRequestAsync` reserve overlapping `ExpertTimeSlot` trong 30 phut (Slot Paradox) |
| 3 | "Cuoc hop hien tai" chua bao gom emergency | TODO | `GetMyBookingsAsync` chi query `ConsultationBooking` (scheduled). Khong co endpoint nao tra emergency consultations dang active |
| 4 | "Lich su cuoc hop" chua bao gom emergency | TODO | Cung van de — `GetMyBookingsAsync` chi tra scheduled bookings, khong include emergency |
| 5 | Chua co co che topup tien vao vi | TODO | `WalletTopupService` da implement nhung CHUA CO controller endpoint expose ra API |
| 6 | Chua co get consultation feedback | TODO | Chi co `POST .../reviews` (create). Khong co GET endpoint cho user/expert lay consultation reviews |

## Implementation Plan

### Task 3+4: Unified Consultation History (includes emergency)

**Problem**: `GET /api/users/me/consultations/scheduled` chi tra scheduled bookings. User khong thay emergency consultations trong "cuoc hop hien tai" va "lich su".

**Solution**: Tao endpoint moi tra unified consultation list (ca scheduled va emergency) hoac mo rong endpoint hien tai.

**Approach**: Tao endpoint moi `GET /api/users/me/consultations` tra tat ca consultations (scheduled + emergency) cua user, grouped by status (ongoing vs completed).

**Files to change**:
- `SnakeAid.Service/Interfaces/IConsultationService.cs` — add `GetMyConsultationsAsync`
- `SnakeAid.Service/Implements/ConsultationService.cs` — implement query across `Consultation` entity
- `SnakeAid.Api/Controllers/ConsultationsController.cs` — add GET endpoint
- `SnakeAid.Core/Responses/Consultation/` — add `MyConsultationResponse` DTO

**Validation**:
- Endpoint tra ca scheduled va emergency consultations
- Filter by status (ongoing/completed)
- Include expert name, room id, consultation type, timestamps

---

### Task 5: Wallet Top-up API Endpoint

**Problem**: `WalletTopupService.CreateWalletTopupAsync` da implement nhung khong co controller endpoint.

**Solution**: Add endpoint trong `WalletController` hoac tao `WalletTopupController`.

**Files to change**:
- `SnakeAid.Api/Controllers/WalletController.cs` — add `POST /api/wallet/topup` endpoint
- Wire `IWalletTopupService` vao controller

**Validation**:
- `POST /api/wallet/topup` voi `{ "amount": 100000 }` tra ve `CreateWalletTopupResponse` voi `checkoutUrl`
- PayOS webhook/return flow credit wallet balance sau khi thanh toan

---

### Task 6: Get Consultation Feedback

**Problem**: Chi co create review (`POST /api/consultations/{id}/reviews`). Khong co endpoint GET de user xem review da submit hoac expert xem reviews nhan duoc.

**Solution**: Add endpoint `GET /api/consultations/{consultationId}/reviews` tra review cua consultation do.

**Files to change**:
- `SnakeAid.Service/Interfaces/IConsultationService.cs` — add `GetConsultationReviewAsync`
- `SnakeAid.Service/Implements/ConsultationService.cs` — implement query
- `SnakeAid.Api/Controllers/ConsultationsController.cs` — add GET endpoint

**Validation**:
- Endpoint tra review cua consultation cu the
- Chi participant (caller/callee) moi xem duoc
- Tra `UserFeedbackResponse` shape

---

## Progress Tracking

- [x] Task 5: Wallet Top-up API Endpoint — `POST /api/wallet/topup` added to `WalletController` (2026-03-28)
- [x] Task 6: Get Consultation Feedback — `GET /api/consultations/{id}/reviews` added to `ConsultationsController` (2026-03-28)
- [x] Task 3+4: Unified Consultation History — `GET /api/users/me/consultations` added, returns both scheduled + emergency (2026-03-28)
