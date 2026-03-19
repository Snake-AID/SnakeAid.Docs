---
doc_role: operation
operation_id: 03-REFACTOR-extract-payment-context-contract
type: REFACTOR
status: done
created_at: 2026-03-09
updated_at: 2026-03-19
affects:
  - SnakeAid.Core/Requests/PayOS/*
  - SnakeAid.Core/Responses/PayOS/*
  - SnakeAid.Service/Interfaces/*
  - SnakeAid.Service/Services/PayOs/*
  - SnakeAid.Api/Controllers/PayOsController.cs
  - SnakeAid.Core/Mappings/PaymentContextMapper.cs
  - SnakeAid.Service/Interfaces/ISnakeCatchingPaymentOrchestrator.cs
  - SnakeAid.Service/Implements/SnakeCatchingPaymentOrchestrator.cs
---

# Plan - 03-REFACTOR-extract-payment-context-contract

## 1. As-Is

Current PayOS contracts are snake-catching-specific and cannot be safely reused by consultation or wallet top-up.

## 2. Gap Analysis

The system lacks a shared domain-neutral orchestration contract such as:
- payment intent
- payment reference type
- settlement/refund context

## 3. To-Be Design

Introduce a payment-context contract that lets any domain provide:
- reference id
- reference type
- amount
- actor ids
- item description
- success/failure callback semantics

This contract should be consumed by domain orchestrators while `PayOsProvider` remains PayOS-specific but domain-neutral.

## 4. Impacted Components

- PayOS DTO layer
- service contracts between domain layer and provider layer

## 5. Risks & Constraints

- must not leak domain-specific fields into generic contracts
- must keep provider API expressive enough for all three target domains:
  - snake catching
  - wallet top-up
  - consultation

## 6. Validation Plan

- one shared payment context can represent all three target domains
- no contract requires `SnakeCatchingRequestId`
- no contract assumes rescuer-specific payout semantics

## 7. Implementation Update (2026-03-19)

- Wired `PaymentContext` contract into runtime create-payment flow:
  - `PayOsController.CreatePaymentLink` now maps request via `PaymentContextMapper.ToPaymentContext(...)`
  - controller now calls `IPaymentOrchestrator.CreatePaymentLinkAsync(...)` instead of directly calling `IPayOsPaymentService` for create-link path
- Returned API response remains backward-compatible (`SnakeCatchingPaymentResponse`) by mapping from orchestrator `PaymentResult`.

## 8. Verification Evidence

- Runtime wiring evidence:
  - `SnakeAid.Api/Controllers/PayOsController.cs` (constructor injects `ISnakeCatchingPaymentOrchestrator`, create-link endpoint uses snake-specific orchestration method)
  - `SnakeAid.Core/Mappings/PaymentContextMapper.cs` (request -> payment context mapping used by controller)
- Build verification:
  - `dotnet build SnakeAid.Backend.sln` succeeded on 2026-03-19 after integration.

## 9. Naming Alignment Update (2026-03-19)

- Added `ISnakeCatchingPaymentOrchestrator` to expose snake-domain method names instead of neutral names at controller boundary.
- `PayOsController` now calls `CreateSnakeCatchingPaymentLinkAsync(...)` for create-link flow.
- Generic `IPaymentOrchestrator` remains for shared contract compatibility, while snake-specific naming is used in runtime entrypoints.

## 10. Current Runtime Boundary (2026-03-19)

- Snake catching API boundary now uses domain-specific orchestration contract:
  - `CreateSnakeCatchingPaymentLinkAsync(...)`
  - `CancelSnakeCatchingPaymentLinkAsync(...)`
  - `ProcessSnakeCatchingWebhookAsync(...)`
  - `ConfirmSnakeCatchingPaymentAsync(...)`
  - `ConfirmSnakeCatchingPaymentByOrderCodeAsync(...)`
  - `TransferSnakeCatchingFundsToRescuerAsync(...)`
  - `RefundSnakeCatchingTransactionAsync(...)`
- This operation is considered complete for contract extraction + runtime wiring of snake-catching entrypoints.
- Full provider/domain decoupling for all domains (consultation, wallet top-up) continues in later operations.
