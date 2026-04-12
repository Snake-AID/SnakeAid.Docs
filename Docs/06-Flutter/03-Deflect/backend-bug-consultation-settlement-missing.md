# Bug Report: ExpertPayout Transaction Not Created After Consultation Ends

**Severity:** High — financial data missing  
**Reported by:** Mobile Team  
**Date:** 2026-04-12  
**Affected endpoint:** `POST /api/consultations/{consultationId}/end`

---

## Summary

When a member taps **"Hoàn Thành Tư Vấn"** to end a scheduled consultation, the mobile app calls `POST /api/consultations/{consultationId}/end`. The backend returns `200 OK` with `is_success: true`, but no `ExpertPayout` (or `PlatformFee`) transaction is created.

As a result, the expert's income tab shows **0đ** even after completing paid consultations.

---

## Backend Code Findings (Verified)

`POST /api/consultations/{consultationId}/end` **does trigger settlement**.

Call chain:
1. `ConsultationsController.EndConsultation()`  
2. `ConsultationService.EndConsultationAsync()`  
3. `ConsultationPaymentService.SettleConsultationEscrowAsync()`  
4. `ConsultationPaymentService.TransferEscrowToExpertAsync()` inserts:
   - `TransactionType.PlatformFee` (if fee > 0)
   - `TransactionType.ExpertPayout`

So the endpoint itself is wired to settlement. If payouts are missing while `/end` still returns `200`, then **settlement is short‑circuiting before insert**.

Key short‑circuit conditions in `SettleConsultationEscrowAsync`:
1. If an existing `ExpertPayout` transaction already exists → returns `false` (no new insert)
2. If **no** `ConsultationBooking` **and** no `ConsultationPingRequest` is found for the consultation → returns `false`

If `ConsultationPayment` exists but is invalid or missing, the method throws a `409 Conflict` — **that would not return 200**.  
Therefore, a `200 OK` with missing payouts strongly suggests **(2)** above: the consultation has no booking/ping reference, so settlement exits early without error.

---

## Reproduction Steps

1. Member books a scheduled consultation with a valid `feeCost > 0`.
2. Member joins waiting room → enters video call → calls expert.
3. After the call, member navigates back to waiting room.
4. Member taps **"Hoàn Thành Tư Vấn"** → confirms in dialog → taps **"Kết Thúc"**.
5. Mobile calls `POST /api/consultations/{consultationId}/end`.
6. Backend responds `200 { is_success: true }`.
7. Mobile navigates to completion/review screen.
8. **Expected:** `ExpertPayout` + `PlatformFee` transactions are inserted.
9. **Actual:** No settlement transactions are inserted. Expert income tab shows 0đ.

---

## Evidence from Mobile Logs

```
📥 RESPONSE [200] => /api/experts/me/consultations
data: {items: [
  {consultationId: 04dd8cad-..., type: Scheduled, status: Completed, price: 2000.0, ...},
  ...
]}
```

Consultation reaches status `Completed` (verified via `/api/experts/me/consultations`), but querying `/api/transactions` with `transType=consultation` shows **no ExpertPayout rows** for that consultationId.

---

## Mobile-Side Behavior (Confirmed Correct)

Mobile calls the endpoint at the right time. Relevant code path:

```
File: lib/features/consultation/screens/members/consultation_waiting_room_screen.dart
Method: _endConsultation()
```

Sequence:
1. User confirms dialog
2. `await repo.endConsultation(widget.consultationId)` — calls `POST /api/consultations/{id}/end`
3. If `is_success == true` → navigate to `/consultation-complete`
4. If `false` → show error snackbar (no navigation)

The mobile-side fix made in this session (RoomExpiry path) is a **separate** issue. The manual "Hoàn Thành Tư Vấn" button correctly calls `/end` before navigating.

---

## Updated Hypothesis (Based on Backend Code)

Settlement **is invoked**, but exits early before insert. Most likely causes:

1. The consultation has **no `ConsultationBooking`** and **no `ConsultationPingRequest`** row.
   - In this case, `SettleConsultationEscrowAsync` returns `false` and `/end` still returns `200`.
2. The booking/ping exists but is **linked to a different consultationId** (data mismatch).
3. Payment transaction exists but is not linked to the booking/ping reference (wrong `ReferenceId`) — settlement throws a `409`, but this would surface as a non-200 unless swallowed by middleware.

---

## What Backend Should Verify (Concrete Checks)

1. **Does this consultation have a booking or ping row?**
   - `ConsultationBooking.ConsultationId == consultationId` OR
   - `ConsultationPingRequest.ConsultationId == consultationId`
   - If both missing → settlement will skip with no error

2. **Is there a `ConsultationPayment` transaction linked to the booking/ping reference?**
   - `Transaction.ReferenceId == bookingId` (scheduled)
   - `Transaction.ReferenceId == pingId` (emergency)
   - `TransactionType == ConsultationPayment`
   - `ExternalTransactionId != null`

3. **Does `/end` return 200 even when settlement returns false?**
   - Current behavior: yes, because the result is ignored in `EndConsultationAsync`.
   - This makes missing booking/ping look like a backend bug, even if it is a data creation issue upstream.

4. **Check transaction existence:**
   - `ExpertPayout` or `PlatformFee` with `ReferenceId = consultationId`
   - If neither exists and settlement ran, then it exited early.

---

## Sample Consultation IDs to Investigate

| consultationId | status | price | Expected transaction |
|---|---|---|---|
| `04dd8cad-5f9b-45a1-878b-1012b636647f` | Completed | 2000đ | ExpertPayout + PlatformFee |

---

## Expected Behavior After Fix

After calling `POST /api/consultations/{id}/end` with `is_success: true`, the following should be observable:

- `GET /api/transactions?transType=consultation` returns rows with `transactionType = ExpertPayout` linked to the consultation
- Expert's income tab shows the correct earnings
- Platform fee row is also created

---

## Contact

For API contract questions, refer to:  
- `lib/features/consultation/repository/consultation_repository.dart` → `endConsultation()`  
- `lib/features/expert/screens/expert_home_screen.dart` → `_IncomeTabState._fetchPage()` (queries `/api/transactions` with `transType=consultation`)
