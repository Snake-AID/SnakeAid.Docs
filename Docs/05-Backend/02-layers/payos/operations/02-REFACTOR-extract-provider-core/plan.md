---
doc_role: operation
operation_id: 02-REFACTOR-extract-provider-core
type: REFACTOR
status: done
created_at: 2026-03-09
affects:
  - SnakeAid.Service/Interfaces/IPayOsPaymentService.cs
  - SnakeAid.Service/Services/PayOs/PayOsPaymentService.cs
  - SnakeAid.Api/Controllers/PayOsController.cs
  - SnakeAid.Core/Requests/PayOS/*
  - SnakeAid.Core/Responses/PayOS/*
---

# Plan - 02-REFACTOR-extract-provider-core

## 1. As-Is

From `payos.sourcecode.md`:
- `IPayOsClient` is relatively provider-oriented
- `IPayOsPaymentService` and `PayOsPaymentService` are coupled to snake catching
- public contracts embed `SnakeCatchingRequestId`

## 2. Gap Analysis

Current module boundary is wrong:
- provider logic and business orchestration are mixed
- a second domain would likely duplicate PayOS orchestration

## 3. To-Be Design

Extract a reusable `PayOsProvider` layer around PayOS:
- create-link / cancel-link / get-link-info / verify-webhook capabilities
- PayOS-specific but domain-neutral request/response contracts for provider interaction
- keep snake-catching business orchestration outside `PayOsProvider`

## 4. Impacted Components

- PayOS service interfaces
- PayOS DTO layer
- PayOS controller surface
- any snake-catching services currently calling business-specific PayOS methods

## 5. Risks & Constraints

- do not break live gateway behavior during refactor
- do not lose existing webhook compatibility
- preserve transaction traceability

## 6. Validation Plan

- `PayOsProvider` methods are reusable across target business domains
- snake-catching terms no longer appear in `PayOsProvider` DTOs
- webhook verification remains functional
