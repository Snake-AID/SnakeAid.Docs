---
doc_role: operation
operation_id: 04-REFACTOR-migrate-snake-catching-to-provider
type: REFACTOR
status: done
created_at: 2026-03-09
updated_at: 2026-03-19
affects:
  - SnakeAid.Service/Services/PayOs/PayOsPaymentService.cs
  - SnakeAid.Service/Implements/WalletPaymentService.cs
  - SnakeAid.Api/Controllers/PayOsController.cs
  - SnakeAid.Api/Controllers/WalletController.cs
  - SnakeAid.Service/Implements/SnakeCatchingRequestService.cs
  - SnakeAid.Service/Implements/SnakeCatchingMissionService.cs
  - SnakeAid.Service/Interfaces/ISnakeCatchingPaymentOrchestrator.cs
  - SnakeAid.Service/Implements/SnakeCatchingPaymentOrchestrator.cs
  - SnakeAid.Api/Program.cs
---

# Plan - 04-REFACTOR-migrate-snake-catching-to-provider

## 1. As-Is

Snake catching is the current domain embedded into the PayOS layer.

## 2. Gap Analysis

Even if provider-core is extracted, the existing snake-catching code must be migrated to the new abstraction. Otherwise the old coupled path will continue to exist beside the new one.

## 3. To-Be Design

Move snake-catching logic into a domain orchestrator that consumes the shared `PayOsProvider`.

## 4. Impacted Components

- snake-catching payment entrypoints
- snake-catching settlement and refund orchestration
- current PayOS controller contracts if they expose coupled naming

## 5. Risks & Constraints

- preserve existing snake-catching behavior
- do not break webhook confirmation path

## 6. Validation Plan

- snake-catching flow works without direct dependency on snake-catching-specific PayOS DTOs at provider boundary

## 7. Implementation Update (2026-03-19)

- Added webhook success idempotency guard in `PayOsPaymentService.ProcessWebhookCoreAsync(...)`:
  - if transaction already has `ExternalTransactionId`, service returns success and skips wallet credit + duplicate transaction insertion.
- Added payout idempotency guard in `TransferToRescuerAsync(...)`:
  - if existing `CatcherPayout` (internal) exists for same `SnakeCatchingRequestId`, service returns already-transferred response.
- Tightened payout domain boundary:
  - payout source set now only includes snake-catching payment types (`CatchingDeposit`, `CatchingPayment`).
  - removed cross-domain transaction types from payout aggregation path.
- Added payout sanity check:
  - throw business error when `netAmountToRescuer <= 0`.
- State consistency fix:
  - `WalletPaymentService` now sets `RequestStatus.Paid` (instead of `Completed`) for `CatchingPayment`, aligning with PayOS payment path.
  - `RequestStatus.Completed` in transfer flow is applied after successful transfer persistence.

## 8. Verification Evidence

- Business consistency evidence:
  - `SnakeAid.Service/Services/PayOs/PayOsPaymentService.cs`
    - webhook idempotency branch (`Payment already processed`)
    - payout idempotency (`existingPayout` check)
    - payout boundary and net amount check
    - completed state update after transfer writes
  - `SnakeAid.Service/Implements/WalletPaymentService.cs`
    - status transition changed to `RequestStatus.Paid` for wallet catching payment path
- Build verification:
  - `dotnet build SnakeAid.Backend.sln` succeeded on 2026-03-19 after these changes.

## 9. Orchestration Migration Update (2026-03-19)

- Migrated controller-facing snake-catching payment flow to snake-specific orchestration interface:
  - `ISnakeCatchingPaymentOrchestrator` introduced and registered in DI.
  - `PayOsController` now routes these endpoints through snake-specific methods:
    - `CreateSnakeCatchingPaymentLinkAsync(...)`
    - `CancelSnakeCatchingPaymentLinkAsync(...)`
    - `ProcessSnakeCatchingWebhookAsync(...)`
    - `ConfirmSnakeCatchingPaymentAsync(...)`
    - `ConfirmSnakeCatchingPaymentByOrderCodeAsync(...)`
    - `TransferSnakeCatchingFundsToRescuerAsync(...)`
- Refactor intent: keep snake-catching naming explicit at entrypoint while preserving provider-level abstraction (`IPayOsProvider`).

## 10. Verification Snapshot (2026-03-19)

- Controller callsites confirmed on snake-specific methods:
  - `SnakeAid.Api/Controllers/PayOsController.cs` lines: 50, 103, 161, 227, 526, 576
- DI registration confirmed:
  - `SnakeAid.Api/Program.cs` line: 135
- Business-consistency fixes still active in PayOS payment service:
  - Webhook idempotency (`Payment already processed`)
  - Payout idempotency (`existingPayout` guard)
  - Positive net payout check (`netAmountToRescuer > 0`)
  - Status consistency (`RequestStatus.Paid` before transfer, `RequestStatus.Completed` after transfer)

## 11. Scope Clarification

- Operation 04 migration target is met at API entrypoint/orchestration boundary for snake-catching flow.
- Some implementation logic remains duplicated between orchestrator and legacy PayOS service and should be consolidated in subsequent refactor steps.
