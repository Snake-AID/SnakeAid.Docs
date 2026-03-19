---
doc_role: operation
operation_id: 04-REFACTOR-migrate-snake-catching-to-provider
type: REFACTOR
status: done
created_at: 2026-03-09
affects:
  - SnakeAid.Service/Services/PayOs/PayOsPaymentService.cs
  - SnakeAid.Service/Implements/WalletPaymentService.cs
  - SnakeAid.Api/Controllers/PayOsController.cs
  - SnakeAid.Api/Controllers/WalletController.cs
  - SnakeAid.Service/Implements/SnakeCatchingRequestService.cs
  - SnakeAid.Service/Implements/SnakeCatchingMissionService.cs
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
