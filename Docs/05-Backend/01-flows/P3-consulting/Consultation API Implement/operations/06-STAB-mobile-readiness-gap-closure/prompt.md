---
doc_role: operation
operation_id: 06-STAB-mobile-readiness-gap-closure
generated_from: plan.md
status: draft
created_at: 2026-03-07
---

# Prompt: Implement Mobile Readiness Gap Closure for Operations 01, 03, 04

## Requirements

Implement the backend gaps from Operations 01, 03, and 04 that still prevent mobile from treating the consultation wireframe as `Build Now`.

Specific tasks:

1. Complete remaining Operation 01 gaps by extending expert pricing/settings to support separate fees for:
   - scheduled consultation
   - emergency consultation
2. Complete remaining Operation 01 gaps by extending expert profile/list contracts with the wireframe statistics required for mobile:
   - total consultations
   - average response time
   - success rate
3. Keep `IsVerified` out of MVP scope in Operation 06. Do not spend this operation trying to finalize verification semantics.
4. Complete remaining Operation 03 gaps by adding scheduled consultation payment APIs:
   - create payment session / checkout
   - support payment method selection
   - support `SApay` balance payment
   - query payment status
   - move successful payment into `SApay` escrow
5. Complete remaining Operation 03 gaps by extending completion payloads so user/expert completion screens can render payment summaries consistently.
6. Complete remaining Operation 03 gaps by settling escrow to expert on consultation completion. Treat slot-end completion and explicit finish API as equally valid completion triggers and make settlement idempotent.
7. Complete remaining Operation 04 gaps by adding emergency consultation payment APIs before expert action:
   - create payment session / checkout
   - support payment method selection
   - support `SApay` balance payment
   - query payment status
   - move successful payment into `SApay` escrow
8. Complete remaining Operation 04 gaps by adding backend-driven emergency expiry handling and publishing `Expired` through `EmergencyRequestStatusChanged` to the request room.
9. Complete remaining Operation 04 gaps by refunding escrow immediately back to member `SApay` balance when emergency request is `Rejected` or `Expired`.
10. Complete remaining Operation 04 gaps by settling escrow to expert only after emergency consultation completes.
11. Add tests covering:
   - dual pricing behavior
   - expert stats mapping
   - scheduled payment to escrow lifecycle
   - emergency payment to escrow lifecycle
   - emergency expiry broadcast
   - refund on reject/expired
   - idempotent settlement on completion

## Constraints

- Keep Operation 06 focused on gaps from Operations 01, 03, and 04.
- Do not implement consultation chat, in-room UI signaling, or expert side-panel snake lookup here; those remain in Operation 05.
- Do not fake production semantics for payment success or profile metrics.
- `IsVerified` is explicitly deferred and is not an MVP completion target in this operation.
- Preserve `ApiResponse<T>` success contracts and typed exception flow per backend culture.
- Treat Operation 06 as a support pass only; do not redefine business ownership away from Operations 01, 03, and 04.
